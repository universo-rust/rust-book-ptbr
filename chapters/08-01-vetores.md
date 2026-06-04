---
title: "Vetores"
chapter_code: 08-01
slug: vetores
challenge_day: 10
reading_minutes: 17
---

# Armazenando Listas de Valores com Vetores

O primeiro tipo de coleção que veremos é `Vec<T>`, também conhecido como vetor. Com vetores, você armazena vários valores numa única estrutura de dados, com todos eles dispostos lado a lado na memória. Eles só aceitam valores do mesmo tipo. São úteis quando você tem uma lista de itens — as linhas de um arquivo de texto, os preços no carrinho de compras, e assim por diante.

### Criando um novo vetor

Para criar um vetor vazio, usamos a função `Vec::new`, como na Listagem 8-1.

**Arquivo: src/main.rs**

```rust
let v: Vec<i32> = Vec::new();
```

<a id="listagem-8-1"></a>

[Listagem 8-1](#listagem-8-1): Criando um novo vetor vazio para armazenar valores do tipo `i32`

Repare que adicionamos uma anotação de tipo. Como ainda não inserimos nenhum valor nesse vetor, o Rust não sabe que tipo de elementos pretendemos armazenar — e isso importa. Vetores são implementados com generics; no Capítulo 10, veremos como usá-los com tipos próprios. Por ora, basta saber que o `Vec<T>` da biblioteca padrão pode conter qualquer tipo. Ao criar um vetor para um tipo específico, indicamos esse tipo entre colchetes angulares. Na Listagem 8-1, dissemos ao Rust que o `Vec<T>` em `v` guardará elementos `i32`.

Na prática, você vai criar um `Vec<T>` já com valores iniciais, e o Rust inferirá o tipo — raramente precisará anotar. O Rust também oferece a macro `vec!`, que monta um vetor com os valores que você passar. A Listagem 8-2 cria um `Vec<i32>` com os valores `1`, `2` e `3`. O tipo inteiro é `i32` porque esse é o padrão, como vimos na seção Tipos de dados do Capítulo 3.

**Arquivo: src/main.rs**

```rust
let v = vec![1, 2, 3];
```

<a id="listagem-8-2"></a>

[Listagem 8-2](#listagem-8-2): Criando um vetor com valores iniciais

Como os valores iniciais são `i32`, o Rust infere que `v` é um `Vec<i32>` e a anotação de tipo não é necessária. Em seguida, veremos como modificar um vetor.

### Atualizando um vetor

Para criar um vetor e ir adicionando elementos, usamos o método `push`, como na Listagem 8-3.

**Arquivo: src/main.rs**

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

<a id="listagem-8-3"></a>

[Listagem 8-3](#listagem-8-3): Usando o método `push` para adicionar valores a um vetor

Como em qualquer variável, se quisermos alterar o valor, precisamos declará-la mutável com `mut`, como vimos no Capítulo 3. Os números que inserimos são todos `i32`, e o Rust infere isso pelos dados — não precisamos da anotação `Vec<i32>`.

### Lendo elementos de um vetor

Há duas formas de referenciar um valor armazenado em um vetor: pela indexação ou pelo método `get`. Nos exemplos a seguir, anotamos os tipos retornados para deixar o código mais claro.

A Listagem 8-4 mostra as duas formas de acessar um valor: indexação e `get`.

**Arquivo: src/main.rs**

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```

<a id="listagem-8-4"></a>

[Listagem 8-4](#listagem-8-4): Acessando um item em um vetor com indexação e com o método `get`

Alguns detalhes valem atenção. Usamos o índice `2` para obter o terceiro elemento porque vetores são indexados a partir de zero. Com `&` e `[]`, obtemos uma referência ao elemento naquele índice. Com o método `get`, passando o índice como argumento, obtemos um `Option<&T>` que podemos usar com `match`.

O Rust oferece essas duas formas para que você escolha como o programa se comporta ao acessar um índice fora do intervalo dos elementos existentes. Por exemplo, veja o que acontece quando temos um vetor com cinco elementos e tentamos acessar o índice 100 com cada abordagem, como na Listagem 8-5.

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

<a id="listagem-8-5"></a>

[Listagem 8-5](#listagem-8-5): Tentando acessar o elemento no índice 100 em um vetor com cinco elementos

Ao executar esse código, o primeiro método, `[]`, faz o programa entrar em pânico porque referencia um elemento inexistente. Use-o quando quiser que o programa falhe ao tentar acessar um elemento além do fim do vetor.

Quando o método `get` recebe um índice fora do vetor, retorna `None` sem entrar em pânico. Use-o quando, em circunstâncias normais, o acesso fora do intervalo puder acontecer de vez em quando. Seu código terá lógica para tratar tanto `Some(&element)` quanto `None`, como vimos no Capítulo 6. Por exemplo, o índice pode vir de um número digitado pelo usuário. Se ele errar e digitar um valor grande demais, o programa recebe `None` — e você pode informar quantos itens há no vetor e dar outra chance de informar um valor válido. Isso é mais amigável do que derrubar o programa por causa de um erro de digitação.

Quando o programa tem uma referência válida, o borrow checker aplica as regras de ownership e borrowing (vistas no Capítulo 4) para garantir que essa referência — e quaisquer outras ao conteúdo do vetor — permaneçam válidas. Lembre a regra que impede referências mutáveis e imutáveis no mesmo escopo: ela se aplica na Listagem 8-6, onde mantemos uma referência imutável ao primeiro elemento e tentamos adicionar outro ao final. O programa não compila se também precisarmos usar essa referência mais adiante na função.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {first}");
```

<a id="listagem-8-6"></a>

[Listagem 8-6](#listagem-8-6): Tentando adicionar um elemento a um vetor enquanto mantém uma referência a um item

Compilar esse código produz o seguinte erro:

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {first}");
  |                                      ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` (bin "collections") due to 1 previous error
```

O código da Listagem 8-6 parece que deveria funcionar: por que uma referência ao primeiro elemento se importaria com mudanças no final do vetor? O erro vem de como vetores funcionam internamente: como eles guardam os valores lado a lado na memória, adicionar um elemento ao final pode exigir alocar nova memória e copiar os elementos antigos para o novo espaço, se não couber tudo onde o vetor está armazenado hoje. Nesse caso, a referência ao primeiro elemento apontaria para memória já desalocada. As regras de borrowing impedem que o programa chegue a essa situação.

> **Nota:** Para saber mais sobre os detalhes de implementação do tipo `Vec<T>`, consulte o [_Rustonomicon_](https://doc.rust-lang.org/nomicon/vec/vec.html).

### Iterando sobre os valores em um vetor

Para acessar cada elemento de um vetor em sequência, percorremos todos eles em um loop, em vez de ir índice por índice. A Listagem 8-7 mostra como usar um loop `for` para obter referências imutáveis a cada elemento de um vetor de `i32` e imprimi-los.

**Arquivo: src/main.rs**

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{i}");
}
```

<a id="listagem-8-7"></a>

[Listagem 8-7](#listagem-8-7): Imprimindo cada elemento de um vetor com um loop `for`

Também podemos iterar sobre referências mutáveis a cada elemento de um vetor mutável para alterar todos de uma vez. O loop `for` na Listagem 8-8 soma `50` a cada elemento.

**Arquivo: src/main.rs**

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

<a id="listagem-8-8"></a>

[Listagem 8-8](#listagem-8-8): Iterando sobre referências mutáveis a elementos de um vetor

Para alterar o valor referenciado, usamos o operador de dereferência `*` para chegar ao valor em `i` antes de aplicar `+=`. Falaremos mais sobre dereferenciação na seção [Seguindo a Referência até o Valor](/livro/cap15-02-tratando-smart-pointers-como-referencias-com-a-trait-deref#seguindo-a-referência-até-o-valor), do Capítulo 15.

Iterar sobre um vetor — de forma imutável ou mutável — é seguro graças às regras do borrow checker. Se tentássemos inserir ou remover itens dentro dos loops `for` das Listagens 8-7 e 8-8, o compilador emitiria um erro parecido com o da Listagem 8-6. A referência ao vetor mantida pelo loop impede modificar o vetor inteiro ao mesmo tempo.

### Usando um enum para armazenar vários tipos

Vetores só aceitam valores do mesmo tipo — o que pode ser limitante. Há casos em que você precisa guardar itens de tipos diferentes numa mesma lista. Felizmente, as variantes de um enum pertencem ao mesmo tipo enum; quando precisamos de um tipo único para representar elementos distintos, podemos definir e usar um enum.

Por exemplo, imagine obter valores de uma linha de planilha em que algumas colunas têm inteiros, outras números de ponto flutuante e outras strings. Podemos definir um enum cujas variantes carregam cada tipo de valor; todas as variantes contam como o mesmo tipo — o do enum. Depois, criamos um vetor desse enum e, assim, armazenamos tipos diferentes. A Listagem 8-9 mostra isso.

**Arquivo: src/main.rs**

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<a id="listagem-8-9"></a>

[Listagem 8-9](#listagem-8-9): Definindo um enum para armazenar valores de tipos diferentes em um vetor

O Rust precisa saber, em tempo de compilação, quais tipos estarão no vetor para calcular exatamente quanta memória na heap cada elemento exige. Também precisamos deixar explícito quais tipos são permitidos. Se o Rust deixasse um vetor conter qualquer tipo, algum deles poderia entrar em conflito com as operações feitas nos elementos. Combinar um enum com uma expressão `match` permite que o Rust garanta em tempo de compilação que todo caso possível é tratado, como vimos no Capítulo 6.

Se você não conhece de antemão o conjunto completo de tipos que o programa receberá em tempo de execução, a técnica do enum não serve. Nesse caso, use um trait object — veremos isso no Capítulo 18.

Agora que vimos algumas das formas mais comuns de usar vetores, vale revisar a [documentação da API](https://doc.rust-lang.org/std/vec/struct.Vec.html): a biblioteca padrão define muitos métodos úteis em `Vec<T>`. Além de `push`, por exemplo, o método `pop` remove e devolve o último elemento.

### Descartar um vetor descarta seus elementos

Como qualquer outra `struct`, um vetor é descartado quando sai de escopo, como na Listagem 8-10.

**Arquivo: src/main.rs**

```rust
{
    let v = vec![1, 2, 3, 4];
}
```

<a id="listagem-8-10"></a>

[Listagem 8-10](#listagem-8-10): Onde o vetor e seus elementos são descartados

Quando o vetor é descartado, todo o conteúdo vai junto — os inteiros que ele guardava são liberados. O borrow checker garante que referências ao conteúdo de um vetor só sejam usadas enquanto o vetor em si for válido.

Vamos ao próximo tipo de coleção: `String`!
