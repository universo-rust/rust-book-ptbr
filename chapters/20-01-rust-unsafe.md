---
title: "Rust unsafe"
chapter_code: 20-01
slug: rust-unsafe
---

# Rust Unsafe

Todo o código que discutimos até aqui teve as garantias de segurança de memória de Rust aplicadas em tempo de compilação. No entanto, Rust tem uma segunda linguagem escondida dentro dela que não aplica essas garantias de segurança de memória: ela se chama _unsafe Rust_ e funciona como Rust comum, mas nos dá superpoderes extras.

Unsafe Rust existe porque, por natureza, a análise estática é conservadora. Quando o compilador tenta determinar se o código mantém as garantias, é melhor rejeitar alguns programas válidos do que aceitar alguns inválidos. Embora o código _possa_ estar correto, se o compilador Rust não tiver informação suficiente para ter confiança, ele rejeitará o código. Nesses casos, você pode usar código _unsafe_ para dizer ao compilador: “Confie em mim, eu sei o que estou fazendo.” Cuidado, porém: você usa _unsafe Rust_ por sua conta e risco — se usar código _unsafe_ incorretamente, podem ocorrer problemas por falta de segurança de memória, como desreferência de ponteiro nulo.

Outro motivo para Rust ter um alter ego _unsafe_ é que o hardware subjacente é inerentemente _unsafe_. Se Rust não permitisse operações _unsafe_, você não conseguiria fazer certas tarefas. Rust precisa permitir programação de sistemas de baixo nível, como interagir diretamente com o sistema operacional ou até escrever seu próprio sistema operacional. Trabalhar com programação de sistemas de baixo nível é um dos objetivos da linguagem. Vamos explorar o que podemos fazer com _unsafe Rust_ e como fazer.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

## Executando Superpoderes _unsafe_

Para mudar para _unsafe Rust_, use a palavra-chave `unsafe` e então inicie um novo bloco que contém o código _unsafe_. Você pode realizar cinco ações em _unsafe Rust_ que não pode em Rust seguro, que chamamos de _superpoderes unsafe_. Esses superpoderes incluem a capacidade de:

1. Desreferenciar um ponteiro bruto.
1. Chamar uma função ou método _unsafe_.
1. Acessar ou modificar uma variável estática mutável.
1. Implementar uma _trait unsafe_.
1. Acessar campos de `union`s.

É importante entender que `unsafe` não desliga o borrow checker nem desabilita nenhuma das outras verificações de segurança de Rust: se você usar uma referência em código _unsafe_, ela ainda será verificada. A palavra-chave `unsafe` só dá acesso a esses cinco recursos que então não são verificados pelo compilador quanto à segurança de memória. Você ainda terá algum grau de segurança dentro de um bloco _unsafe_.

Além disso, `unsafe` não significa que o código dentro do bloco é necessariamente perigoso ou que terá problemas de segurança de memória: a intenção é que, como programador, você garanta que o código dentro de um bloco `unsafe` acessará a memória de forma válida.

As pessoas cometem erros e isso acontecerá, mas ao exigir que essas cinco operações _unsafe_ fiquem dentro de blocos anotados com `unsafe`, você saberá que qualquer erro relacionado à segurança de memória deve estar dentro de um bloco _unsafe_. Mantenha blocos _unsafe_ pequenos; você agradecerá depois ao investigar bugs de memória.

Para isolar código _unsafe_ o máximo possível, o ideal é encapsular esse código em uma abstração segura e fornecer uma API segura, o que discutiremos mais adiante neste capítulo ao examinar funções e métodos _unsafe_. Partes da biblioteca padrão são implementadas como abstrações seguras sobre código _unsafe_ que foi auditado. Envolver código _unsafe_ em uma abstração segura impede que usos de `unsafe` vazem para todos os lugares em que você ou seus usuários queiram usar a funcionalidade implementada com código _unsafe_, porque usar uma abstração segura é seguro.

Vamos ver cada um dos cinco superpoderes _unsafe_ em sequência. Também veremos algumas abstrações que fornecem uma interface segura para código _unsafe_.

## Desreferenciando um Ponteiro Bruto

No Capítulo 4, na seção “Referências dangling”, mencionamos que o compilador garante que referências sejam sempre válidas. _Unsafe Rust_ tem dois novos tipos chamados _ponteiros brutos_ que são semelhantes a referências. Como referências, ponteiros brutos podem ser imutáveis ou mutáveis e são escritos como `*const T` e `*mut T`, respectivamente. O asterisco não é o operador de desreferência; faz parte do nome do tipo. No contexto de ponteiros brutos, _imutável_ significa que o ponteiro não pode ser atribuído diretamente depois de ser desreferenciado.

Diferente de referências e smart pointers, ponteiros brutos:

- Podem ignorar as regras de borrowing tendo ponteiros imutáveis e mutáveis ou vários ponteiros mutáveis para o mesmo local
- Não têm garantia de apontar para memória válida
- Podem ser nulos
- Não implementam limpeza automática

Ao optar por não fazer Rust aplicar essas garantias, você pode abrir mão de segurança garantida em troca de maior desempenho ou da capacidade de interagir com outra linguagem ou hardware onde as garantias de Rust não se aplicam.

A Listagem 20-1 mostra como criar um ponteiro bruto imutável e um mutável.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut num = 5;

    let r1 = &raw const num;
    let r2 = &raw mut num;
}
```

<a id="listagem-20-1"></a>

[Listagem 20-1](#listagem-20-1): Criando ponteiros brutos com os operadores de borrow bruto

Note que não incluímos a palavra-chave `unsafe` neste código. Podemos criar ponteiros brutos em código seguro; só não podemos desreferenciar ponteiros brutos fora de um bloco _unsafe_, como veremos em seguida.

Criamos ponteiros brutos usando os operadores de borrow bruto: `&raw const num` cria um ponteiro bruto imutável `*const i32`, e `&raw mut num` cria um ponteiro bruto mutável `*mut i32`. Como os criamos diretamente a partir de uma variável local, sabemos que esses ponteiros brutos em particular são válidos, mas não podemos assumir isso sobre qualquer ponteiro bruto.

Para demonstrar isso, em seguida criaremos um ponteiro bruto cuja validade não podemos ter tanta certeza, usando a palavra-chave `as` para fazer cast de um valor em vez de usar o operador de borrow bruto. A Listagem 20-2 mostra como criar um ponteiro bruto para um local arbitrário na memória. Tentar usar memória arbitrária é indefinido: pode haver dados naquele endereço ou não, o compilador pode otimizar o código para que não haja acesso à memória, ou o programa pode terminar com segmentation fault. Em geral, não há boa razão para escrever código assim, especialmente quando você pode usar um operador de borrow bruto, mas é possível.

**Arquivo: src/main.rs**

```rust
fn main() {
    let address = 0x012345usize;
    let r = address as *const i32;
}
```

<a id="listagem-20-2"></a>

[Listagem 20-2](#listagem-20-2): Criando um ponteiro bruto para um endereço de memória arbitrário

Lembre-se de que podemos criar ponteiros brutos em código seguro, mas não podemos desreferenciar ponteiros brutos e ler os dados apontados. Na Listagem 20-3, usamos o operador de desreferência `*` em um ponteiro bruto, o que exige um bloco `unsafe`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut num = 5;

    let r1 = &raw const num;
    let r2 = &raw mut num;

    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
```

<a id="listagem-20-3"></a>

[Listagem 20-3](#listagem-20-3): Desreferenciando ponteiros brutos dentro de um bloco `unsafe`

Criar um ponteiro não faz mal; só quando tentamos acessar o valor apontado é que podemos acabar lidando com um valor inválido.

Note também que nas Listagens 20-1 e 20-3 criamos ponteiros brutos `*const i32` e `*mut i32` que apontavam para o mesmo local na memória, onde `num` está armazenado. Se tentássemos criar uma referência imutável e uma mutável para `num`, o código não compilaria porque as regras de ownership de Rust não permitem uma referência mutável ao mesmo tempo que referências imutáveis. Com ponteiros brutos, podemos criar um ponteiro mutável e um imutável para o mesmo local e alterar dados pelo ponteiro mutável, potencialmente criando uma condição de corrida. Cuidado!

Com todos esses perigos, por que usar ponteiros brutos? Um caso de uso importante é ao interagir com código C, como veremos na próxima seção. Outro caso é ao construir abstrações seguras que o borrow checker não entende. Apresentaremos funções _unsafe_ e depois veremos um exemplo de abstração segura que usa código _unsafe_.

## Chamando uma Função ou Método _unsafe_

O segundo tipo de operação que você pode realizar em um bloco _unsafe_ é chamar funções _unsafe_. Funções e métodos _unsafe_ parecem exatamente com funções e métodos comuns, mas têm um `unsafe` extra antes do restante da definição. A palavra-chave `unsafe` neste contexto indica que a função tem requisitos que precisamos cumprir ao chamá-la, porque Rust não pode garantir que os atendemos. Ao chamar uma função _unsafe_ dentro de um bloco `unsafe`, estamos dizendo que lemos a documentação da função e assumimos a responsabilidade de cumprir os contratos da função.

Aqui está uma função _unsafe_ chamada `dangerous` que não faz nada no corpo:

**Arquivo: src/main.rs**

```rust
fn main() {
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
}
```

Precisamos chamar a função `dangerous` dentro de um bloco `unsafe` separado. Se tentarmos chamar `dangerous` sem o bloco `unsafe`, receberemos um erro:

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function `dangerous` is unsafe and requires unsafe block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function's documentation for information on how to avoid undefined behavior

For more information about this error, try `rustc --explain E0133`.
error: could not compile `unsafe-example` (bin "unsafe-example") due to 1 previous error
```

Com o bloco `unsafe`, estamos afirmando a Rust que lemos a documentação da função, entendemos como usá-la corretamente e verificamos que estamos cumprindo o contrato da função.

Para realizar operações _unsafe_ no corpo de uma função `unsafe`, você ainda precisa usar um bloco `unsafe`, assim como dentro de uma função comum, e o compilador avisará se você esquecer. Isso nos ajuda a manter blocos _unsafe_ o menor possível, já que operações _unsafe_ podem não ser necessárias em todo o corpo da função.

### Criando uma Abstração Segura sobre Código _unsafe_

Só porque uma função contém código _unsafe_ não significa que precisamos marcar a função inteira como _unsafe_. Na verdade, envolver código _unsafe_ em uma função segura é uma abstração comum. Como exemplo, vamos estudar a função `split_at_mut` da biblioteca padrão, que exige código _unsafe_. Exploraremos como poderíamos implementá-la. Este método seguro é definido em slices mutáveis: recebe um slice e o divide em dois, separando no índice passado como argumento. A Listagem 20-4 mostra como usar `split_at_mut`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];

    let (a, b) = r.split_at_mut(3);

    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
}
```

<a id="listagem-20-4"></a>

[Listagem 20-4](#listagem-20-4): Usando a função segura `split_at_mut`

Não podemos implementar esta função usando apenas Rust seguro. Uma tentativa poderia ser como na Listagem 20-5, que não compilará. Por simplicidade, implementaremos `split_at_mut` como função em vez de método e apenas para slices de valores `i32`, em vez de um tipo genérico `T`.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..])
}
```

<a id="listagem-20-5"></a>

[Listagem 20-5](#listagem-20-5): Uma tentativa de implementação de `split_at_mut` usando apenas Rust seguro

Esta função primeiro obtém o comprimento total do slice. Depois, afirma que o índice passado como parâmetro está dentro do slice, verificando se é menor ou igual ao comprimento. A asserção significa que, se passarmos um índice maior que o comprimento para dividir o slice, a função entrará em pânico antes de tentar usar esse índice.

Em seguida, retornamos dois slices mutáveis em uma tupla: um do início do slice original até o índice `mid` e outro de `mid` até o fim do slice.

Quando tentarmos compilar o código da Listagem 20-5, receberemos um erro:

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`
  |
  = help: use `.split_at_mut(position)` to obtain two mutable non-overlapping sub-slices

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` (bin "unsafe-example") due to 1 previous error
```

O borrow checker de Rust não consegue entender que estamos emprestando partes diferentes do slice; ele só sabe que estamos emprestando o mesmo slice duas vezes. Emprestar partes diferentes de um slice é fundamentalmente aceitável porque os dois slices não se sobrepõem, mas Rust não é inteligente o suficiente para saber disso. Quando sabemos que o código está correto, mas Rust não, é hora de recorrer a código _unsafe_.

A Listagem 20-6 mostra como usar um bloco `unsafe`, um ponteiro bruto e algumas chamadas a funções _unsafe_ para fazer a implementação de `split_at_mut` funcionar.

**Arquivo: src/main.rs**

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

<a id="listagem-20-6"></a>

[Listagem 20-6](#listagem-20-6): Usando código _unsafe_ na implementação da função `split_at_mut`

Lembre-se, na seção “O tipo slice” do Capítulo 4, que um slice é um ponteiro para alguns dados e o comprimento do slice. Usamos o método `len` para obter o comprimento de um slice e o método `as_mut_ptr` para acessar o ponteiro bruto de um slice. Neste caso, como temos um slice mutável de valores `i32`, `as_mut_ptr` retorna um ponteiro bruto do tipo `*mut i32`, que armazenamos na variável `ptr`.

Mantemos a asserção de que o índice `mid` está dentro do slice. Então chegamos ao código _unsafe_: a função `slice::from_raw_parts_mut` recebe um ponteiro bruto e um comprimento, e cria um slice. Usamos esta função para criar um slice que começa em `ptr` e tem `mid` itens. Depois, chamamos o método `add` em `ptr` com `mid` como argumento para obter um ponteiro bruto que começa em `mid`, e criamos um slice usando esse ponteiro e o número restante de itens após `mid` como comprimento.

A função `slice::from_raw_parts_mut` é _unsafe_ porque recebe um ponteiro bruto e precisa confiar que esse ponteiro é válido. O método `add` em ponteiros brutos também é _unsafe_ porque precisa confiar que o local deslocado também é um ponteiro válido. Portanto, precisamos colocar um bloco `unsafe` em torno das nossas chamadas a `slice::from_raw_parts_mut` e `add` para poder chamá-las. Olhando o código e adicionando a asserção de que `mid` deve ser menor ou igual a `len`, podemos dizer que todos os ponteiros brutos usados dentro do bloco `unsafe` serão ponteiros válidos para dados dentro do slice. Este é um uso aceitável e apropriado de `unsafe`.

Note que não precisamos marcar a função resultante `split_at_mut` como `unsafe`, e podemos chamar esta função a partir de Rust seguro. Criamos uma abstração segura para o código _unsafe_ com uma implementação da função que usa código _unsafe_ de forma segura, porque cria apenas ponteiros válidos a partir dos dados aos quais esta função tem acesso.

Em contraste, o uso de `slice::from_raw_parts_mut` na Listagem 20-7 provavelmente travaria quando o slice fosse usado. Este código pega um local de memória arbitrário e cria um slice com 10.000 itens.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;

    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
}
```

<a id="listagem-20-7"></a>

[Listagem 20-7](#listagem-20-7): Criando um slice a partir de um local de memória arbitrário

Não possuímos a memória neste local arbitrário, e não há garantia de que o slice que este código cria contenha valores `i32` válidos. Tentar usar `values` como se fosse um slice válido resulta em comportamento indefinido.

### Usando Funções `extern` para Chamar Código Externo

Às vezes seu código Rust pode precisar interagir com código escrito em outra linguagem. Para isso, Rust tem a palavra-chave `extern`, que facilita a criação e o uso de uma _Foreign Function Interface (FFI)_, que é uma forma de uma linguagem de programação definir funções e permitir que outra linguagem (estrangeira) chame essas funções.

A Listagem 20-8 demonstra como configurar uma integração com a função `abs` da biblioteca padrão C. Funções declaradas dentro de blocos `extern` são em geral _unsafe_ de chamar a partir de código Rust, então blocos `extern` também devem ser marcados como `unsafe`. O motivo é que outras linguagens não aplicam as regras e garantias de Rust, e Rust não pode verificá-las, então a responsabilidade cai sobre o programador garantir a segurança.

**Arquivo: src/main.rs**

```rust
unsafe extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

<a id="listagem-20-8"></a>

[Listagem 20-8](#listagem-20-8): Declarando e chamando uma função `extern` definida em outra linguagem

Dentro do bloco `unsafe extern "C"`, listamos os nomes e assinaturas das funções externas de outra linguagem que queremos chamar. A parte `"C"` define qual _application binary interface (ABI)_ a função externa usa: a ABI define como chamar a função no nível de assembly. A ABI `"C"` é a mais comum e segue a ABI da linguagem de programação C. Informações sobre todas as ABIs que Rust suporta estão disponíveis [na Referência do Rust](https://doc.rust-lang.org/reference/items/external-blocks.html#abi).

Cada item declarado dentro de um bloco `unsafe extern` é implicitamente _unsafe_. No entanto, algumas funções FFI _são_ seguras de chamar. Por exemplo, a função `abs` da biblioteca padrão C não tem considerações de segurança de memória, e sabemos que pode ser chamada com qualquer `i32`. Em casos assim, podemos usar a palavra-chave `safe` para dizer que esta função específica é segura de chamar mesmo estando em um bloco `unsafe extern`. Depois dessa mudança, chamá-la não exige mais um bloco `unsafe`, como mostra a Listagem 20-9.

**Arquivo: src/main.rs**

```rust
unsafe extern "C" {
    safe fn abs(input: i32) -> i32;
}

fn main() {
    println!("Absolute value of -3 according to C: {}", abs(-3));
}
```

<a id="listagem-20-9"></a>

[Listagem 20-9](#listagem-20-9): Marcando explicitamente uma função como `safe` dentro de um bloco `unsafe extern` e chamando-a com segurança

Marcar uma função como `safe` não a torna inerentemente segura! Em vez disso, é como uma promessa que você faz a Rust de que ela é segura. Ainda é sua responsabilidade garantir que essa promessa seja cumprida!

### Chamando Funções Rust a Partir de Outras Linguagens

Também podemos usar `extern` para criar uma interface que permite que outras linguagens chamem funções Rust. Em vez de criar um bloco `extern` inteiro, adicionamos a palavra-chave `extern` e especificamos a ABI a usar logo antes da palavra-chave `fn` da função relevante. Também precisamos adicionar uma anotação `#[unsafe(no_mangle)]` para dizer ao compilador Rust para não fazer _mangling_ no nome desta função. _Mangling_ é quando um compilador altera o nome que demos a uma função para um nome diferente que contém mais informação para outras partes do processo de compilação consumirem, mas é menos legível para humanos. Cada compilador de linguagem de programação faz _mangling_ de nomes de forma ligeiramente diferente, então para uma função Rust ser nomeável por outras linguagens, precisamos desabilitar o _mangling_ de nomes do compilador Rust. Isso é _unsafe_ porque pode haver colisões de nomes entre bibliotecas sem o _mangling_ embutido, então é nossa responsabilidade garantir que o nome que escolhemos seja seguro de exportar sem _mangling_.

No exemplo a seguir, tornamos a função `call_from_c` acessível a partir de código C, depois que for compilada para uma biblioteca compartilhada e linkada a partir de C:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

Este uso de `extern` exige `unsafe` apenas no atributo, não no bloco `extern`.

## Acessando ou Modificando uma Variável Estática Mutável

Neste livro, ainda não falamos sobre variáveis globais, que Rust suporta, mas que podem ser problemáticas com as regras de ownership de Rust. Se duas threads acessam a mesma variável global mutável, pode ocorrer uma condição de corrida.

Em Rust, variáveis globais são chamadas de variáveis _estáticas_. A Listagem 20-10 mostra um exemplo de declaração e uso de uma variável estática com um string slice como valor.

**Arquivo: src/main.rs**

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("value is: {HELLO_WORLD}");
}
```

<a id="listagem-20-10"></a>

[Listagem 20-10](#listagem-20-10): Definindo e usando uma variável estática imutável

Variáveis estáticas são semelhantes a constantes, que discutimos na seção “Declarando constantes” do Capítulo 3. Os nomes de variáveis estáticas usam `SCREAMING_SNAKE_CASE` por convenção. Variáveis estáticas só podem armazenar referências com lifetime `'static`, o que significa que o compilador Rust consegue deduzir o lifetime e não precisamos anotá-lo explicitamente. Acessar uma variável estática imutável é seguro.

Uma diferença sutil entre constantes e variáveis estáticas imutáveis é que valores em uma variável estática têm um endereço fixo na memória. Usar o valor sempre acessará os mesmos dados. Constantes, por outro lado, podem duplicar seus dados sempre que forem usadas. Outra diferença é que variáveis estáticas podem ser mutáveis. Acessar e modificar variáveis estáticas mutáveis é _unsafe_. A Listagem 20-11 mostra como declarar, acessar e modificar uma variável estática mutável chamada `COUNTER`.

**Arquivo: src/main.rs**

```rust
static mut COUNTER: u32 = 0;

/// SAFETY: Calling this from more than a single thread at a time is undefined
/// behavior, so you *must* guarantee you only call it from a single thread at
/// a time.
unsafe fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    unsafe {
        // SAFETY: This is only called from a single thread in `main`.
        add_to_count(3);
        println!("COUNTER: {}", *(&raw const COUNTER));
    }
}
```

<a id="listagem-20-11"></a>

[Listagem 20-11](#listagem-20-11): Ler ou escrever em uma variável estática mutável é _unsafe_

Como variáveis comuns, especificamos mutabilidade usando a palavra-chave `mut`. Qualquer código que leia ou escreva em `COUNTER` deve estar dentro de um bloco `unsafe`. O código da Listagem 20-11 compila e imprime `COUNTER: 3` como esperado porque é single-threaded. Ter várias threads acessando `COUNTER` provavelmente resultaria em condições de corrida, então é comportamento indefinido. Portanto, precisamos marcar a função inteira como `unsafe` para que quem chamar a função saiba o que pode e não pode fazer com segurança.

Sempre que escrevemos uma função _unsafe_, é idiomático escrever um comentário começando com `SAFETY` e explicando o que quem chama precisa fazer para chamar a função com segurança. Da mesma forma, sempre que realizamos uma operação _unsafe_, é idiomático escrever um comentário começando com `SAFETY` para explicar como as regras de segurança são mantidas.

Além disso, o compilador nega por padrão qualquer tentativa de criar referências a uma variável estática mutável por meio de um lint do compilador. Você deve optar explicitamente por sair dessa proteção adicionando uma anotação `#[allow(static_mut_refs)]` ou acessar a variável estática mutável via um ponteiro bruto criado com um dos operadores de borrow bruto. Isso inclui casos em que a referência é criada de forma invisível, como quando é usada no `println!` neste listing. Exigir que referências a variáveis estáticas mutáveis sejam criadas via ponteiros brutos ajuda a tornar os requisitos de segurança para usá-las mais evidentes.

Com dados mutáveis acessíveis globalmente, é difícil garantir que não haja condições de corrida, por isso Rust considera variáveis estáticas mutáveis _unsafe_. Quando possível, é preferível usar as técnicas de concorrência e smart pointers thread-safe que discutimos no Capítulo 16 para que o compilador verifique se o acesso a dados de threads diferentes é feito com segurança.

## Implementando uma _Trait unsafe_

Podemos usar `unsafe` para implementar uma _trait unsafe_. Uma _trait_ é _unsafe_ quando pelo menos um de seus métodos tem algum invariante que o compilador não pode verificar. Declaramos que uma _trait_ é `unsafe` adicionando a palavra-chave `unsafe` antes de `trait` e marcando a implementação da _trait_ como `unsafe` também, como mostra a Listagem 20-12.

**Arquivo: src/main.rs**

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

<a id="listagem-20-12"></a>

[Listagem 20-12](#listagem-20-12): Definindo e implementando uma _trait unsafe_

Ao usar `unsafe impl`, prometemos que manteremos os invariantes que o compilador não pode verificar.

Como exemplo, lembre-se das _traits_ marcadoras `Send` e `Sync` que discutimos na seção “Concorrência extensível com `Send` e `Sync`” do Capítulo 16: o compilador implementa essas _traits_ automaticamente se nossos tipos são compostos inteiramente de outros tipos que implementam `Send` e `Sync`. Se implementarmos um tipo que contém um tipo que não implementa `Send` ou `Sync`, como ponteiros brutos, e quisermos marcar esse tipo como `Send` ou `Sync`, precisamos usar `unsafe`. Rust não pode verificar que nosso tipo mantém as garantias de que pode ser enviado com segurança entre threads ou acessado de várias threads; portanto, precisamos fazer essas verificações manualmente e indicar isso com `unsafe`.

## Acessando Campos de uma `union`

A ação final que funciona apenas com `unsafe` é acessar campos de uma _union_. Uma _union_ é semelhante a uma `struct`, mas apenas um campo declarado é usado em uma instância particular de cada vez. _Unions_ são usadas principalmente para interagir com _unions_ em código C. Acessar campos de _union_ é _unsafe_ porque Rust não pode garantir o tipo dos dados atualmente armazenados na instância da _union_. Você pode aprender mais sobre _unions_ [na Referência do Rust](https://doc.rust-lang.org/reference/items/unions.html).

## Usando Miri para Verificar Código _unsafe_

Ao escrever código _unsafe_, você pode querer verificar se o que escreveu realmente é seguro e correto. Uma das melhores formas de fazer isso é usar Miri, uma ferramenta oficial do Rust para detectar comportamento indefinido. Enquanto o borrow checker é uma ferramenta _estática_ que funciona em tempo de compilação, Miri é uma ferramenta _dinâmica_ que funciona em tempo de execução. Ela verifica seu código executando seu programa, ou sua suíte de testes, e detectando quando você viola as regras que ela entende sobre como Rust deve funcionar.

Usar Miri exige uma build nightly do Rust (sobre a qual falamos mais no [Apêndice G: Como o Rust é feito e “Nightly Rust”](/livro/cap22-07-como-o-rust-e-feito-e-nightly-rust)). Você pode instalar tanto uma versão nightly do Rust quanto a ferramenta Miri digitando `rustup +nightly component add miri`. Isso não altera qual versão do Rust seu projeto usa; só adiciona a ferramenta ao seu sistema para você usar quando quiser. Você pode executar Miri em um projeto digitando `cargo +nightly miri run` ou `cargo +nightly miri test`.

Para um exemplo de quão útil isso pode ser, considere o que acontece quando a executamos contra a Listagem 20-7.

```bash
$ cargo +nightly miri run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.01s
     Running `file:///home/.rustup/toolchains/nightly/bin/cargo-miri runner target/miri/debug/unsafe-example`
warning: integer-to-pointer cast
 --> src/main.rs:5:13
  |
5 |     let r = address as *mut i32;
  |             ^^^^^^^^^^^^^^^^^^^ integer-to-pointer cast
  |
  = help: this program is using integer-to-pointer casts or (equivalently) `ptr::with_exposed_provenance`, which means that Miri might miss pointer bugs in this program
  = help: see https://doc.rust-lang.org/nightly/std/ptr/fn.with_exposed_provenance.html for more details on that operation
  = help: to ensure that Miri does not miss bugs in your program, use Strict Provenance APIs (https://doc.rust-lang.org/nightly/std/ptr/index.html#strict-provenance, https://crates.io/crates/sptr) instead
  = help: you can then set `MIRIFLAGS=-Zmiri-strict-provenance` to ensure you are not relying on `with_exposed_provenance` semantics
  = help: alternatively, `MIRIFLAGS=-Zmiri-permissive-provenance` disables this warning
  = note: BACKTRACE:
  = note: inside `main` at src/main.rs:5:13: 5:32

error: Undefined Behavior: pointer not dereferenceable: pointer must be dereferenceable for 40000 bytes, but got 0x1234[noalloc] which is a dangling pointer (it has no provenance)
 --> src/main.rs:7:35
  |
7 |     let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
  |                                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Undefined Behavior occurred here
  |
  = help: this indicates a bug in the program: it performed an invalid operation, and caused Undefined Behavior
  = help: see https://doc.rust-lang.org/nightly/reference/behavior-considered-undefined.html for further information
  = note: BACKTRACE:
  = note: inside `main` at src/main.rs:7:35: 7:70

note: some details are omitted, run with `MIRIFLAGS=-Zmiri-backtrace=full` for a verbose backtrace

error: aborting due to 1 previous error; 1 warning emitted
```

Miri nos avisa corretamente que estamos fazendo cast de um inteiro para um ponteiro, o que pode ser um problema, mas Miri não consegue determinar se há um problema porque não sabe como o ponteiro se originou. Depois, Miri retorna um erro onde a Listagem 20-7 tem comportamento indefinido porque temos um ponteiro dangling. Graças ao Miri, agora sabemos que há risco de comportamento indefinido e podemos pensar em como tornar o código seguro. Em alguns casos, Miri pode até fazer recomendações sobre como corrigir erros.

Miri não detecta tudo que você pode errar ao escrever código _unsafe_. Miri é uma ferramenta de análise dinâmica, então só detecta problemas com código que realmente é executado. Isso significa que você precisará usá-la junto com boas técnicas de teste para aumentar sua confiança sobre o código _unsafe_ que escreveu. Miri também não cobre todas as formas possíveis em que seu código pode ser unsound.

Dito de outro modo: se Miri _detectar_ um problema, você sabe que há um bug, mas só porque Miri _não_ detecta um bug não significa que não há um problema. Ela pode detectar muita coisa, porém. Tente executá-la nos outros exemplos de código _unsafe_ deste capítulo e veja o que ela diz!

Você pode aprender mais sobre Miri [em seu repositório no GitHub](https://github.com/rust-lang/miri).

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

## Usando Código _unsafe_ Corretamente

Usar `unsafe` para usar um dos cinco superpoderes que acabamos de discutir não está errado nem é desaprovado, mas é mais difícil acertar código _unsafe_ porque o compilador não pode ajudar a manter a segurança de memória. Quando você tiver motivo para usar código _unsafe_, pode fazê-lo, e ter a anotação explícita `unsafe` facilita rastrear a origem dos problemas quando eles ocorrem. Sempre que escrever código _unsafe_, você pode usar Miri para ajudar a ter mais confiança de que o código que escreveu mantém as regras de Rust.

Para uma exploração muito mais profunda de como trabalhar efetivamente com _unsafe Rust_, leia o guia oficial do Rust para `unsafe`, [The Rustonomicon](https://doc.rust-lang.org/nomicon/).
