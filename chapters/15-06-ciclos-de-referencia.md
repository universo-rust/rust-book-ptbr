---
title: "Ciclos de referência podem vazar memória"
chapter_code: 15-06
slug: ciclos-de-referencia-podem-vazar-memoria
---

# Ciclos de Referência Podem Vazar Memória

As garantias de segurança de memória do Rust tornam difícil, mas não impossível, criar acidentalmente memória que nunca é limpa (conhecida como _memory leak_). Impedir vazamentos de memória por completo não é uma das garantias do Rust, o que significa que vazamentos de memória são memory safe em Rust. Podemos ver que o Rust permite vazamentos de memória usando `Rc<T>` e `RefCell<T>`: é possível criar referências em que itens se referem uns aos outros em um ciclo. Isso cria vazamentos de memória porque a contagem de referências de cada item no ciclo nunca chegará a 0, e os valores nunca serão descartados.

## Criando um Ciclo de Referência

Vamos ver como um ciclo de referência pode acontecer e como evitá-lo, começando com a definição do enum `List` e um método `tail` na Listagem 15-25.

**Arquivo: src/main.rs**

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

<a id="listagem-15-25"></a>

[Listagem 15-25](#listagem-15-25): Uma definição de cons list que guarda um `RefCell<T>` para podermos modificar a que uma variante `Cons` se refere

Estamos usando outra variação da definição de `List` da Listagem 15-5. O segundo elemento na variante `Cons` agora é `RefCell<Rc<List>>`, o que significa que, em vez de ter a capacidade de modificar o valor `i32` como fizemos na Listagem 15-24, queremos modificar o valor `List` para o qual uma variante `Cons` aponta. Também estamos adicionando um método `tail` para nos facilitar acessar o segundo item se tivermos uma variante `Cons`.

Na Listagem 15-26, estamos adicionando uma função `main` que usa as definições da Listagem 15-25. Este código cria uma lista em `a` e uma lista em `b` que aponta para a lista em `a`. Depois, modifica a lista em `a` para apontar para `b`, criando um ciclo de referência. Há instruções `println!` ao longo do caminho para mostrar quais são as contagens de referência em vários pontos deste processo.

**Arquivo: src/main.rs**

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack.
    // println!("a next item = {:?}", a.tail());
}
```

<a id="listagem-15-26"></a>

[Listagem 15-26](#listagem-15-26): Criando um ciclo de referência de dois valores `List` apontando um para o outro

Criamos uma instância `Rc<List>` guardando um valor `List` na variável `a` com uma lista inicial de `5, Nil`. Depois, criamos uma instância `Rc<List>` guardando outro valor `List` na variável `b` que contém o valor `10` e aponta para a lista em `a`.

Modificamos `a` para que aponte para `b` em vez de `Nil`, criando um ciclo. Fazemos isso usando o método `tail` para obter uma referência ao `RefCell<Rc<List>>` em `a`, que colocamos na variável `link`. Depois, usamos o método `borrow_mut` no `RefCell<Rc<List>>` para mudar o valor interno de um `Rc<List>` que guarda um valor `Nil` para o `Rc<List>` em `b`.

Quando executamos este código, mantendo o último `println!` comentado por enquanto, obteremos esta saída:

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/cons-list`
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

A contagem de referências das instâncias `Rc<List>` em `a` e `b` é 2 depois que mudamos a lista em `a` para apontar para `b`. No final de `main`, o Rust descarta a variável `b`, o que diminui a contagem de referências da instância `Rc<List>` de `b` de 2 para 1. A memória que `Rc<List>` tem no heap não será descartada neste ponto porque sua contagem de referências é 1, não 0. Depois, o Rust descarta `a`, o que diminui a contagem de referências da instância `Rc<List>` de `a` de 2 para 1 também. A memória desta instância também não pode ser descartada, porque a outra instância `Rc<List>` ainda se refere a ela. A memória alocada para a lista permanecerá não coletada para sempre. Para visualizar este ciclo de referência, criamos o diagrama na Figura 15-4.

![Um retângulo 'a' apontando para um retângulo com 5. Um retângulo 'b' apontando para um retângulo com 10. O retângulo com 5 aponta para o com 10, e o com 10 aponta de volta para o com 5, criando um ciclo.](https://doc.rust-lang.org/book/img/trpl15-04.svg)

*Figura 15-4: Um ciclo de referência das listas `a` e `b` apontando uma para a outra*

Se você descomentar o último `println!` e executar o programa, o Rust tentará imprimir este ciclo com `a` apontando para `b` apontando para `a` e assim por diante até estourar a stack.

Comparado a um programa do mundo real, as consequências de criar um ciclo de referência neste exemplo não são muito graves: logo após criar o ciclo de referência, o programa termina. No entanto, se um programa mais complexo alocasse muita memória em um ciclo e a retivesse por muito tempo, o programa usaria mais memória do que precisava e poderia sobrecarregar o sistema, fazendo-o ficar sem memória disponível.

Criar ciclos de referência não é feito facilmente, mas também não é impossível. Se você tiver valores `RefCell<T>` que contêm valores `Rc<T>` ou combinações aninhadas semelhantes de tipos com interior mutability e contagem de referências, deve garantir que não cria ciclos; não pode confiar no Rust para detectá-los. Criar um ciclo de referência seria um bug lógico no seu programa que você deve minimizar usando testes automatizados, revisões de código e outras práticas de desenvolvimento de software.

Outra solução para evitar ciclos de referência é reorganizar suas estruturas de dados para que algumas referências expressem ownership e outras não. Como resultado, você pode ter ciclos feitos de alguns relacionamentos de ownership e alguns relacionamentos sem ownership, e apenas os relacionamentos de ownership afetam se um valor pode ser descartado. Na Listagem 15-25, sempre queremos que variantes `Cons` possuam sua lista, então reorganizar a estrutura de dados não é possível. Vamos olhar um exemplo usando grafos feitos de nós pais e nós filhos para ver quando relacionamentos sem ownership são uma forma apropriada de prevenir ciclos de referência.

## Prevenindo Ciclos de Referência Usando `Weak<T>`

Até agora, demonstramos que chamar `Rc::clone` aumenta o `strong_count` de uma instância `Rc<T>`, e uma instância `Rc<T>` só é limpa se seu `strong_count` for 0. Você também pode criar uma referência fraca ao valor dentro de uma instância `Rc<T>` chamando `Rc::downgrade` e passando uma referência ao `Rc<T>`. _Referências fortes_ são como você compartilha ownership de uma instância `Rc<T>`. _Referências fracas_ não expressam um relacionamento de ownership, e sua contagem não afeta quando uma instância `Rc<T>` é limpa. Elas não causarão um ciclo de referência, porque qualquer ciclo envolvendo algumas referências fracas será quebrado quando a contagem de referências fortes dos valores envolvidos for 0.

Quando você chama `Rc::downgrade`, obtém um smart pointer do tipo `Weak<T>`. Em vez de aumentar o `strong_count` na instância `Rc<T>` em 1, chamar `Rc::downgrade` aumenta o `weak_count` em 1. O tipo `Rc<T>` usa `weak_count` para rastrear quantas referências `Weak<T>` existem, de forma semelhante a `strong_count`. A diferença é que o `weak_count` não precisa ser 0 para a instância `Rc<T>` ser limpa.

Como o valor que `Weak<T>` referencia pode ter sido descartado, para fazer qualquer coisa com o valor para o qual um `Weak<T>` aponta você deve garantir que o valor ainda existe. Faça isso chamando o método `upgrade` em uma instância `Weak<T>`, que retornará um `Option<Rc<T>>`. Você obterá um resultado `Some` se o valor `Rc<T>` ainda não tiver sido descartado e um resultado `None` se o valor `Rc<T>` tiver sido descartado. Como `upgrade` retorna um `Option<Rc<T>>`, o Rust garantirá que os casos `Some` e `None` sejam tratados, e não haverá um ponteiro inválido.

Como exemplo, em vez de usar uma lista cujos itens conhecem apenas o próximo item, criaremos uma árvore cujos itens conhecem seus itens filhos _e_ seus itens pais.

### Criando uma Estrutura de Dados em Árvore

Para começar, construiremos uma árvore com nós que conhecem seus nós filhos. Criaremos uma struct chamada `Node` que guarda seu próprio valor `i32` bem como referências aos seus valores `Node` filhos:

**Arquivo: src/main.rs**

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

Queremos que um `Node` possua seus filhos, e queremos compartilhar essa ownership com variáveis para que possamos acessar cada `Node` na árvore diretamente. Para isso, definimos os itens `Vec<T>` como valores do tipo `Rc<Node>`. Também queremos modificar quais nós são filhos de outro nó, então temos um `RefCell<T>` em `children` em torno do `Vec<Rc<Node>>`.

Em seguida, usaremos nossa definição de struct e criaremos uma instância `Node` chamada `leaf` com o valor `3` e sem filhos, e outra instância chamada `branch` com o valor `5` e `leaf` como um de seus filhos, como mostrado na Listagem 15-27.

**Arquivo: src/main.rs**

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

<a id="listagem-15-27"></a>

[Listagem 15-27](#listagem-15-27): Criando um nó `leaf` sem filhos e um nó `branch` com `leaf` como um de seus filhos

Clonamos o `Rc<Node>` em `leaf` e armazenamos isso em `branch`, o que significa que o `Node` em `leaf` agora tem dois donos: `leaf` e `branch`. Podemos ir de `branch` para `leaf` através de `branch.children`, mas não há como ir de `leaf` para `branch`. A razão é que `leaf` não tem referência a `branch` e não sabe que estão relacionados. Queremos que `leaf` saiba que `branch` é seu pai. Faremos isso a seguir.

### Adicionando uma Referência de um Filho ao Seu Pai

Para fazer o nó filho estar ciente de seu pai, precisamos adicionar um campo `parent` à nossa definição de struct `Node`. O problema está em decidir qual deve ser o tipo de `parent`. Sabemos que não pode conter um `Rc<T>`, porque isso criaria um ciclo de referência com `leaf.parent` apontando para `branch` e `branch.children` apontando para `leaf`, o que faria seus valores `strong_count` nunca serem 0.

Pensando nos relacionamentos de outra forma, um nó pai deve possuir seus filhos: se um nó pai for descartado, seus nós filhos devem ser descartados também. No entanto, um filho não deve possuir seu pai: se descartarmos um nó filho, o pai ainda deve existir. Este é um caso para referências fracas!

Então, em vez de `Rc<T>`, faremos o tipo de `parent` usar `Weak<T>`, especificamente um `RefCell<Weak<Node>>`. Agora nossa definição de struct `Node` se parece com isto:

**Arquivo: src/main.rs**

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

Um nó poderá se referir ao seu nó pai, mas não possui o pai. Na Listagem 15-28, atualizamos `main` para usar esta nova definição para que o nó `leaf` tenha uma forma de se referir ao seu pai, `branch`.

**Arquivo: src/main.rs**

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

<a id="listagem-15-28"></a>

[Listagem 15-28](#listagem-15-28): Um nó `leaf` com uma referência fraca ao seu nó pai, `branch`

Criar o nó `leaf` se parece com a Listagem 15-27, com a exceção do campo `parent`: `leaf` começa sem pai, então criamos uma nova instância vazia de referência `Weak<Node>`.

Neste ponto, quando tentamos obter uma referência ao pai de `leaf` usando o método `upgrade`, obtemos um valor `None`. Vemos isso na saída da primeira instrução `println!`:

```text
leaf parent = None
```

Quando criamos o nó `branch`, ele também terá uma nova referência `Weak<Node>` no campo `parent` porque `branch` não tem nó pai. Ainda temos `leaf` como um dos filhos de `branch`. Depois que temos a instância `Node` em `branch`, podemos modificar `leaf` para dar a ele uma referência `Weak<Node>` ao seu pai. Usamos o método `borrow_mut` no `RefCell<Weak<Node>>` no campo `parent` de `leaf`, e então usamos a função `Rc::downgrade` para criar uma referência `Weak<Node>` a `branch` a partir do `Rc<Node>` em `branch`.

Quando imprimimos o pai de `leaf` novamente, desta vez obteremos uma variante `Some` contendo `branch`: agora `leaf` pode acessar seu pai! Quando imprimimos `leaf`, também evitamos o ciclo que eventualmente terminou em stack overflow como na Listagem 15-26; as referências `Weak<Node>` são impressas como `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

A falta de saída infinita indica que este código não criou um ciclo de referência. Também podemos dizer isso olhando os valores que obtemos de chamar `Rc::strong_count` e `Rc::weak_count`.

### Visualizando Mudanças em `strong_count` e `weak_count`

Vamos olhar como os valores `strong_count` e `weak_count` das instâncias `Rc<Node>` mudam criando um novo escopo interno e movendo a criação de `branch` para esse escopo. Ao fazer isso, podemos ver o que acontece quando `branch` é criado e depois descartado quando sai de escopo. As modificações são mostradas na Listagem 15-29.

**Arquivo: src/main.rs**

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

<a id="listagem-15-29"></a>

[Listagem 15-29](#listagem-15-29): Criando `branch` em um escopo interno e examinando contagens de referências fortes e fracas

Depois que `leaf` é criado, seu `Rc<Node>` tem uma contagem forte de 1 e uma contagem fraca de 0. No escopo interno, criamos `branch` e o associamos a `leaf`, momento em que, ao imprimir as contagens, o `Rc<Node>` em `branch` terá uma contagem forte de 1 e uma contagem fraca de 1 (para `leaf.parent` apontando para `branch` com um `Weak<Node>`). Quando imprimimos as contagens em `leaf`, veremos que terá uma contagem forte de 2 porque `branch` agora tem um clone do `Rc<Node>` de `leaf` armazenado em `branch.children`, mas ainda terá uma contagem fraca de 0.

Quando o escopo interno termina, `branch` sai de escopo e a contagem forte do `Rc<Node>` diminui para 0, então seu `Node` é descartado. A contagem fraca de 1 de `leaf.parent` não influencia se o `Node` é descartado ou não, então não obtemos vazamentos de memória!

Se tentarmos acessar o pai de `leaf` depois do fim do escopo, obteremos `None` novamente. No final do programa, o `Rc<Node>` em `leaf` tem uma contagem forte de 1 e uma contagem fraca de 0 porque a variável `leaf` é novamente a única referência ao `Rc<Node>`.

Toda a lógica que gerencia as contagens e o descarte de valores está embutida em `Rc<T>` e `Weak<T>` e suas implementações da trait `Drop`. Ao especificar que o relacionamento de um filho ao seu pai deve ser uma referência `Weak<T>` na definição de `Node`, você pode ter nós pais apontando para nós filhos e vice-versa sem criar um ciclo de referência e vazamentos de memória.

## Resumo

Este capítulo cobriu como usar smart pointers para fazer garantias e trocas diferentes daquelas que o Rust faz por padrão com referências comuns. O tipo `Box<T>` tem um tamanho conhecido e aponta para dados alocados no heap. O tipo `Rc<T>` rastreia o número de referências a dados no heap para que os dados possam ter vários donos. O tipo `RefCell<T>` com sua interior mutability nos dá um tipo que podemos usar quando precisamos de um tipo imutável, mas precisamos mudar um valor interior desse tipo; ele também impõe as regras de borrowing em tempo de execução em vez de tempo de compilação.

Também discutimos as traits `Deref` e `Drop`, que habilitam grande parte da funcionalidade de smart pointers. Exploramos ciclos de referência que podem causar vazamentos de memória e como evitá-los usando `Weak<T>`.

Se este capítulo despertou seu interesse e você quer implementar seus próprios smart pointers, consulte [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/) para mais informações úteis.

Em seguida, falaremos sobre concorrência em Rust. Você até aprenderá sobre alguns novos smart pointers!
