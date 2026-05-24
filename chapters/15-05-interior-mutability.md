---
title: "`RefCell<T>` e o padrão de interior mutability"
chapter_code: 15-05
slug: refcell-e-o-padrao-de-interior-mutability
---

# `RefCell<T>` e o Padrão de Interior Mutability

_Interior mutability_ é um padrão de design em Rust que permite mutar dados mesmo quando há referências imutáveis a esses dados; normalmente, esta ação é proibida pelas regras de borrowing. Para mutar dados, o padrão usa código `unsafe` dentro de uma estrutura de dados para flexibilizar as regras usuais do Rust que governam mutação e borrowing. Código unsafe indica ao compilador que estamos verificando as regras manualmente em vez de confiar no compilador para verificá-las por nós; discutiremos código unsafe com mais detalhes no Capítulo 20.

Podemos usar tipos que usam o padrão de interior mutability apenas quando podemos garantir que as regras de borrowing serão seguidas em tempo de execução, embora o compilador não possa garantir isso. O código unsafe envolvido é então embrulhado em uma API segura, e o tipo externo permanece imutável.

Vamos explorar este conceito olhando o tipo `RefCell<T>` que segue o padrão de interior mutability.

## Impedindo as Regras de Borrowing em Tempo de Execução

Diferente de `Rc<T>`, o tipo `RefCell<T>` representa posse única sobre os dados que guarda. Então, o que torna `RefCell<T>` diferente de um tipo como `Box<T>`? Lembre-se das regras de borrowing que você aprendeu no Capítulo 4:

- Em qualquer momento, você pode ter _ou_ uma referência mutável _ou_ qualquer número de referências imutáveis (mas não ambas).
- Referências devem sempre ser válidas.

Com referências e `Box<T>`, os invariantes das regras de borrowing são impostos em tempo de compilação. Com `RefCell<T>`, esses invariantes são impostos _em tempo de execução_. Com referências, se você quebrar essas regras, obterá um erro do compilador. Com `RefCell<T>`, se você quebrar essas regras, seu programa entrará em pânico e encerrará.

As vantagens de verificar as regras de borrowing em tempo de compilação são que erros serão detectados mais cedo no processo de desenvolvimento, e não há impacto no desempenho em tempo de execução porque toda a análise é concluída antecipadamente. Por essas razões, verificar as regras de borrowing em tempo de compilação é a melhor escolha na maioria dos casos, que é o padrão do Rust.

A vantagem de verificar as regras de borrowing em tempo de execução é que certos cenários seguros em memória passam a ser permitidos, onde teriam sido proibidos pelas verificações em tempo de compilação. Análise estática, como o compilador Rust, é inerentemente conservadora. Algumas propriedades do código são impossíveis de detectar analisando o código: o exemplo mais famoso é o Problema da Parada, que está além do escopo deste livro, mas é um tópico interessante para pesquisar.

Como alguma análise é impossível, se o compilador Rust não puder ter certeza de que o código cumpre as regras de ownership, ele pode rejeitar um programa correto; assim, é conservador. Se o Rust aceitasse um programa incorreto, os usuários não poderiam confiar nas garantias que o Rust faz. No entanto, se o Rust rejeitar um programa correto, o programador será inconveniente, mas nada catastrófico pode ocorrer. O tipo `RefCell<T>` é útil quando você tem certeza de que seu código segue as regras de borrowing, mas o compilador não consegue entender e garantir isso.

Semelhante a `Rc<T>`, `RefCell<T>` é apenas para uso em cenários de thread única e dará um erro em tempo de compilação se você tentar usá-lo em um contexto multithread. Falaremos sobre como obter a funcionalidade de `RefCell<T>` em um programa multithread no Capítulo 16.

Aqui está um resumo das razões para escolher `Box<T>`, `Rc<T>` ou `RefCell<T>`:

- `Rc<T>` permite vários donos dos mesmos dados; `Box<T>` e `RefCell<T>` têm um único dono.
- `Box<T>` permite empréstimos imutáveis ou mutáveis verificados em tempo de compilação; `Rc<T>` permite apenas empréstimos imutáveis verificados em tempo de compilação; `RefCell<T>` permite empréstimos imutáveis ou mutáveis verificados em tempo de execução.
- Como `RefCell<T>` permite empréstimos mutáveis verificados em tempo de execução, você pode mutar o valor dentro do `RefCell<T>` mesmo quando o `RefCell<T>` é imutável.

Mutar o valor dentro de um valor imutável é o padrão de interior mutability. Vamos olhar uma situação em que interior mutability é útil e examinar como é possível.

## Usando Interior Mutability

Uma consequência das regras de borrowing é que, quando você tem um valor imutável, não pode emprestá-lo mutavelmente. Por exemplo, este código não compilará:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

Se você tentasse compilar este código, obteria o seguinte erro:

```bash
$ cargo run
   Compiling borrowing v0.1.0 (file:///projects/borrowing)
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
 --> src/main.rs:3:13
  |
3 |     let y = &mut x;
  |             ^^^^^^ cannot borrow as mutable
  |
help: consider changing this to be mutable
  |
2 |     let mut x = 5;
  |         +++

For more information about this error, try `rustc --explain E0596`.
error: could not compile `borrowing` (bin "borrowing") due to 1 previous error
```

No entanto, há situações em que seria útil que um valor se mutasse em seus métodos, mas parecesse imutável para outro código. Código fora dos métodos do valor não poderia mutar o valor. Usar `RefCell<T>` é uma forma de obter a capacidade de ter interior mutability, mas `RefCell<T>` não contorna completamente as regras de borrowing: o borrow checker no compilador permite esta interior mutability, e as regras de borrowing são verificadas em tempo de execução em vez de tempo de compilação. Se você violar as regras, obterá um `panic!` em vez de um erro do compilador.

Vamos trabalhar com um exemplo prático onde podemos usar `RefCell<T>` para mutar um valor imutável e ver por que isso é útil.

### Testando com Mock Objects

Às vezes, durante testes, um programador usará um tipo no lugar de outro tipo para observar comportamento particular e afirmar que está implementado corretamente. Este tipo substituto é chamado de _test double_. Pense nisso no sentido de um dublê de ação em cinema, onde uma pessoa entra e substitui um ator para fazer uma cena particularmente complicada. Test doubles substituem outros tipos quando executamos testes. _Mock objects_ são tipos específicos de test doubles que registram o que acontece durante um teste para que você possa afirmar que as ações corretas ocorreram.

O Rust não tem objetos no mesmo sentido que outras linguagens têm objetos, e o Rust não tem funcionalidade de mock object embutida na biblioteca padrão como algumas outras linguagens. No entanto, você pode definitivamente criar uma struct que sirva aos mesmos propósitos que um mock object.

Aqui está o cenário que testaremos: criaremos uma biblioteca que rastreia um valor contra um valor máximo e envia mensagens com base em quão próximo do máximo o valor atual está. Esta biblioteca poderia ser usada para rastrear a cota de um usuário para o número de chamadas de API que têm permissão de fazer, por exemplo.

Nossa biblioteca fornecerá apenas a funcionalidade de rastrear quão próximo do máximo um valor está e quais mensagens devem ser em quais momentos. Aplicações que usam nossa biblioteca deverão fornecer o mecanismo para enviar as mensagens: a aplicação poderia mostrar a mensagem ao usuário diretamente, enviar um e-mail, enviar SMS ou fazer outra coisa. A biblioteca não precisa saber esse detalhe. Tudo o que precisa é algo que implemente uma trait que forneceremos, chamada `Messenger`. A Listagem 15-20 mostra o código da biblioteca.

**Arquivo: src/lib.rs**

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

[Listagem 15-20](#listagem-15-20): Uma biblioteca para rastrear quão próximo um valor está de um valor máximo e avisar quando o valor atinge certos níveis

Uma parte importante deste código é que a trait `Messenger` tem um método chamado `send` que recebe uma referência imutável a `self` e o texto da mensagem. Esta trait é a interface que nosso mock object precisa implementar para que o mock possa ser usado da mesma forma que um objeto real. A outra parte importante é que queremos testar o comportamento do método `set_value` em `LimitTracker`. Podemos mudar o que passamos para o parâmetro `value`, mas `set_value` não retorna nada para fazermos asserções. Queremos poder dizer que, se criarmos um `LimitTracker` com algo que implemente a trait `Messenger` e um valor particular para `max`, o messenger é informado para enviar as mensagens apropriadas quando passamos números diferentes para `value`.

Precisamos de um mock object que, em vez de enviar e-mail ou SMS quando chamamos `send`, apenas mantenha registro das mensagens que é informado para enviar. Podemos criar uma nova instância do mock object, criar um `LimitTracker` que usa o mock object, chamar o método `set_value` em `LimitTracker` e então verificar que o mock object tem as mensagens que esperamos. A Listagem 15-21 mostra uma tentativa de implementar um mock object para fazer exatamente isso, mas o borrow checker não permitirá.

**Arquivo: src/lib.rs (Este código não compila!)**

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

[Listagem 15-21](#listagem-15-21): Uma tentativa de implementar um `MockMessenger` que o borrow checker não permite

Este código de teste define uma struct `MockMessenger` que tem um campo `sent_messages` com um `Vec` de valores `String` para rastrear as mensagens que é informado para enviar. Também definimos uma função associada `new` para facilitar criar novos valores `MockMessenger` que começam com uma lista vazia de mensagens. Em seguida, implementamos a trait `Messenger` para `MockMessenger` para que possamos dar um `MockMessenger` a um `LimitTracker`. Na definição do método `send`, pegamos a mensagem passada como parâmetro e a armazenamos na lista `sent_messages` do `MockMessenger`.

No teste, estamos testando o que acontece quando o `LimitTracker` é informado para definir `value` para algo que é mais de 75 por cento do valor `max`. Primeiro, criamos um novo `MockMessenger`, que começará com uma lista vazia de mensagens. Depois, criamos um novo `LimitTracker` e damos a ele uma referência ao novo `MockMessenger` e um valor `max` de `100`. Chamamos o método `set_value` no `LimitTracker` com um valor de `80`, que é mais de 75 por cento de 100. Então, afirmamos que a lista de mensagens que o `MockMessenger` está rastreando deve agora ter uma mensagem.

No entanto, há um problema com este teste, como mostrado aqui:

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference
  --> src/lib.rs:58:13
   |
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^ `self` is a `&` reference, so the data it refers to cannot be borrowed as mutable
   |
help: consider changing this to be a mutable reference in the `impl` method and the `trait` definition
   |
 2 ~     fn send(&mut self, msg: &str);
 3 | }
...
56 |     impl Messenger for MockMessenger {
57 ~         fn send(&mut self, message: &str) {
   |

For more information about this error, try `rustc --explain E0596`.
error: could not compile `limit-tracker` (lib test) due to 1 previous error
```

Não podemos modificar o `MockMessenger` para rastrear as mensagens, porque o método `send` recebe uma referência imutável a `self`. Também não podemos seguir a sugestão do texto de erro para usar `&mut self` tanto na definição do método `impl` quanto na definição da trait. Não queremos mudar a trait `Messenger` apenas por causa dos testes. Em vez disso, precisamos encontrar uma forma de fazer nosso código de teste funcionar corretamente com nosso design existente.

Esta é uma situação em que interior mutability pode ajudar! Armazenaremos `sent_messages` dentro de um `RefCell<T>`, e então o método `send` poderá modificar `sent_messages` para armazenar as mensagens que vimos. A Listagem 15-22 mostra como isso fica.

**Arquivo: src/lib.rs**

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

[Listagem 15-22](#listagem-15-22): Usando `RefCell<T>` para mutar um valor interior enquanto o valor externo é considerado imutável

O campo `sent_messages` agora é do tipo `RefCell<Vec<String>>` em vez de `Vec<String>`. Na função `new`, criamos uma nova instância `RefCell<Vec<String>>` em torno do vetor vazio.

Para a implementação do método `send`, o primeiro parâmetro ainda é um empréstimo imutável de `self`, o que corresponde à definição da trait. Chamamos `borrow_mut` no `RefCell<Vec<String>>` em `self.sent_messages` para obter uma referência mutável ao valor dentro do `RefCell<Vec<String>>`, que é o vetor. Então, podemos chamar `push` na referência mutável ao vetor para rastrear as mensagens enviadas durante o teste.

A última mudança que temos que fazer está na asserção: para ver quantos itens há no vetor interno, chamamos `borrow` no `RefCell<Vec<String>>` para obter uma referência imutável ao vetor.

Agora que você viu como usar `RefCell<T>`, vamos aprofundar como funciona!

### Rastreando Empréstimos em Tempo de Execução

Ao criar referências imutáveis e mutáveis, usamos a sintaxe `&` e `&mut`, respectivamente. Com `RefCell<T>`, usamos os métodos `borrow` e `borrow_mut`, que fazem parte da API segura que pertence a `RefCell<T>`. O método `borrow` retorna o tipo smart pointer `Ref<T>`, e `borrow_mut` retorna o tipo smart pointer `RefMut<T>`. Ambos os tipos implementam `Deref`, então podemos tratá-los como referências comuns.

O `RefCell<T>` rastreia quantos smart pointers `Ref<T>` e `RefMut<T>` estão ativos no momento. Cada vez que chamamos `borrow`, o `RefCell<T>` aumenta sua contagem de quantos empréstimos imutáveis estão ativos. Quando um valor `Ref<T>` sai de escopo, a contagem de empréstimos imutáveis diminui em 1. Assim como as regras de borrowing em tempo de compilação, `RefCell<T>` nos permite ter muitos empréstimos imutáveis ou um empréstimo mutável em qualquer ponto.

Se tentarmos violar essas regras, em vez de obter um erro do compilador como faríamos com referências, a implementação de `RefCell<T>` entrará em pânico em tempo de execução. A Listagem 15-23 mostra uma modificação da implementação de `send` na Listagem 15-22. Estamos deliberadamente tentando criar dois empréstimos mutáveis ativos para o mesmo escopo para ilustrar que `RefCell<T>` nos impede de fazer isso em tempo de execução.

**Arquivo: src/lib.rs (Este código entra em pânico!)**

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            let mut one_borrow = self.sent_messages.borrow_mut();
            let mut two_borrow = self.sent_messages.borrow_mut();

            one_borrow.push(String::from(message));
            two_borrow.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

[Listagem 15-23](#listagem-15-23): Criando duas referências mutáveis no mesmo escopo para ver que `RefCell<T>` entrará em pânico

Criamos uma variável `one_borrow` para o smart pointer `RefMut<T>` retornado por `borrow_mut`. Depois, criamos outro empréstimo mutável da mesma forma na variável `two_borrow`. Isso faz duas referências mutáveis no mesmo escopo, o que não é permitido. Quando executamos os testes de nossa biblioteca, o código na Listagem 15-23 compilará sem erros, mas o teste falhará:

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/limit_tracker-e599811fa246dbde)

running 1 test
test tests::it_sends_an_over_75_percent_warning_message ... FAILED

failures:

---- tests::it_sends_an_over_75_percent_warning_message stdout ----

thread 'tests::it_sends_an_over_75_percent_warning_message' panicked at src/lib.rs:60:53:
RefCell already borrowed
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_sends_an_over_75_percent_warning_message

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Observe que o código entrou em pânico com a mensagem `already borrowed: BorrowMutError`. É assim que `RefCell<T>` trata violações das regras de borrowing em tempo de execução.

Escolher detectar erros de borrowing em tempo de execução em vez de tempo de compilação, como fizemos aqui, significa que você potencialmente encontraria erros no seu código mais tarde no processo de desenvolvimento: possivelmente só quando seu código fosse implantado em produção. Além disso, seu código incorreria em uma pequena penalidade de desempenho em tempo de execução por rastrear os empréstimos em tempo de execução em vez de tempo de compilação. No entanto, usar `RefCell<T>` torna possível escrever um mock object que pode se modificar para rastrear as mensagens que viu enquanto você o usa em um contexto onde apenas valores imutáveis são permitidos. Você pode usar `RefCell<T>` apesar de suas trocas para obter mais funcionalidade do que referências comuns fornecem.

## Permitindo Múltiplos Donos de Dados Mutáveis

Uma forma comum de usar `RefCell<T>` é em combinação com `Rc<T>`. Lembre-se de que `Rc<T>` permite ter vários donos de alguns dados, mas só dá acesso imutável a esses dados. Se você tem um `Rc<T>` que guarda um `RefCell<T>`, pode obter um valor que pode ter vários donos _e_ que você pode mutar!

Por exemplo, lembre-se do exemplo de cons list na Listagem 15-18, onde usamos `Rc<T>` para permitir que várias listas compartilhassem ownership de outra lista. Como `Rc<T>` guarda apenas valores imutáveis, não podemos mudar nenhum dos valores na lista depois de criadas. Vamos adicionar `RefCell<T>` por sua capacidade de mudar os valores nas listas. A Listagem 15-24 mostra que, usando um `RefCell<T>` na definição de `Cons`, podemos modificar o valor armazenado em todas as listas.

**Arquivo: src/main.rs**

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {a:?}");
    println!("b after = {b:?}");
    println!("c after = {c:?}");
}
```

[Listagem 15-24](#listagem-15-24): Usando `Rc<RefCell<i32>>` para criar uma `List` que podemos mutar

Criamos um valor que é uma instância de `Rc<RefCell<i32>>` e o armazenamos em uma variável chamada `value` para podermos acessá-lo diretamente depois. Depois, criamos uma `List` em `a` com uma variante `Cons` que guarda `value`. Precisamos clonar `value` para que tanto `a` quanto `value` tenham ownership do valor interno `5` em vez de transferir ownership de `value` para `a` ou fazer `a` emprestar de `value`.

Envolvemos a lista `a` em um `Rc<T>` para que, quando criarmos as listas `b` e `c`, ambas possam se referir a `a`, o que fizemos na Listagem 15-18.

Depois de criarmos as listas em `a`, `b` e `c`, queremos adicionar 10 ao valor em `value`. Fazemos isso chamando `borrow_mut` em `value`, que usa o recurso de dereferenciação automática que discutimos em Onde está o operador `->`? no Capítulo 5 para dereferenciar o `Rc<T>` ao valor interno `RefCell<T>`. O método `borrow_mut` retorna um smart pointer `RefMut<T>`, e usamos o operador de dereferência nele e mudamos o valor interno.

Quando imprimimos `a`, `b` e `c`, podemos ver que todos têm o valor modificado de `15` em vez de `5`:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.63s
     Running `target/debug/cons-list`
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
```

Esta técnica é bem interessante! Usando `RefCell<T>`, temos um valor `List` externamente imutável. Mas podemos usar os métodos em `RefCell<T>` que fornecem acesso à sua interior mutability para modificar nossos dados quando precisamos. As verificações em tempo de execução das regras de borrowing nos protegem de data races, e às vezes vale a pena trocar um pouco de velocidade por essa flexibilidade em nossas estruturas de dados. Note que `RefCell<T>` não funciona para código multithread! `Mutex<T>` é a versão thread-safe de `RefCell<T>`, e discutiremos `Mutex<T>` no Capítulo 16.
