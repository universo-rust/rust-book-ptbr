---
title: "Vetores"
chapter_code: 08-01
slug: vetores
---

# Armazenando Listas de Valores com Vetores

O primeiro tipo de coleção que veremos é `Vec<T>`, também conhecido como vetor. Vetores permitem armazenar mais de um valor em uma única estrutura de dados que coloca todos os valores uns ao lado dos outros na memória. Vetores só podem armazenar valores do mesmo tipo. São úteis quando você tem uma lista de itens, como as linhas de texto em um arquivo ou os preços de itens em um carrinho de compras.

### Criando um novo vetor

Para criar um novo vetor vazio, chamamos a função `Vec::new`, como mostrado na Listagem 8-1.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v: Vec<i32> = Vec::new();
}
```

[Listagem 8-1](#listagem-8-1): Criando um novo vetor vazio para armazenar valores do tipo `i32`

Observe que adicionamos uma anotação de tipo aqui. Como não estamos inserindo valores neste vetor, o Rust não sabe que tipo de elementos pretendemos armazenar. Este é um ponto importante. Vetores são implementados usando generics; cobriremos como usar generics com seus próprios tipos no Capítulo 10. Por enquanto, saiba que o tipo `Vec<T>` fornecido pela biblioteca padrão pode conter qualquer tipo. Quando criamos um vetor para armazenar um tipo específico, podemos especificar o tipo entre colchetes angulares. Na Listagem 8-1, dissemos ao Rust que o `Vec<T>` em `v` armazenará elementos do tipo `i32`.

Mais frequentemente, você criará um `Vec<T>` com valores iniciais, e o Rust inferirá o tipo de valor que quer armazenar, então raramente precisará fazer esta anotação de tipo. O Rust fornece convenientemente a macro `vec!`, que criará um novo vetor que contém os valores que você der. A Listagem 8-2 cria um novo `Vec<i32>` que contém os valores `1`, `2` e `3`. O tipo inteiro é `i32` porque esse é o tipo inteiro padrão, como discutimos na seção Tipos de dados do Capítulo 3.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v = vec![1, 2, 3];
}
```

[Listagem 8-2](#listagem-8-2): Criando um novo vetor contendo valores

Como demos valores `i32` iniciais, o Rust pode inferir que o tipo de `v` é `Vec<i32>`, e a anotação de tipo não é necessária. Em seguida, veremos como modificar um vetor.

### Atualizando um vetor

Para criar um vetor e então adicionar elementos a ele, podemos usar o método `push`, como mostrado na Listagem 8-3.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
}
```

[Listagem 8-3](#listagem-8-3): Usando o método `push` para adicionar valores a um vetor

Como em qualquer variável, se quisermos poder alterar seu valor, precisamos torná-la mutável usando a palavra-chave `mut`, como discutido no Capítulo 3. Os números que colocamos são todos do tipo `i32`, e o Rust infere isso a partir dos dados, então não precisamos da anotação `Vec<i32>`.

### Lendo elementos de vetores

Há duas formas de referenciar um valor armazenado em um vetor: via indexação ou usando o método `get`. Nos exemplos a seguir, anotamos os tipos dos valores retornados por essas funções para maior clareza.

A Listagem 8-4 mostra ambos os métodos de acessar um valor em um vetor, com sintaxe de indexação e o método `get`.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
}
```

[Listagem 8-4](#listagem-8-4): Usando sintaxe de indexação e o método `get` para acessar um item em um vetor

Observe alguns detalhes aqui. Usamos o valor de índice `2` para obter o terceiro elemento porque vetores são indexados por número, começando em zero. Usar `&` e `[]` nos dá uma referência ao elemento no valor do índice. Quando usamos o método `get` com o índice passado como argumento, obtemos um `Option<&T>` que podemos usar com `match`.

O Rust fornece essas duas formas de referenciar um elemento para que você possa escolher como o programa se comporta quando tenta usar um valor de índice fora do intervalo dos elementos existentes. Como exemplo, vamos ver o que acontece quando temos um vetor de cinco elementos e então tentamos acessar um elemento no índice 100 com cada técnica, como mostrado na Listagem 8-5.

**Arquivo: src/main.rs (Este código entra em pânico!)**

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
}
```

[Listagem 8-5](#listagem-8-5): Tentando acessar o elemento no índice 100 em um vetor contendo cinco elementos

Quando executamos este código, o primeiro método `[]` fará o programa entrar em pânico porque referencia um elemento inexistente. Este método é melhor usado quando você quer que seu programa entre em pânico se houver tentativa de acessar um elemento além do fim do vetor.

Quando o método `get` recebe um índice fora do vetor, retorna `None` sem entrar em pânico. Você usaria este método se acessar um elemento além do intervalo do vetor pudesse acontecer ocasionalmente em circunstâncias normais. Seu código então terá lógica para lidar com ter `Some(&element)` ou `None`, como discutido no Capítulo 6. Por exemplo, o índice poderia vir de uma pessoa digitando um número. Se ela acidentalmente digitar um número muito grande e o programa obtiver um valor `None`, você poderia informar quantos itens há no vetor atual e dar outra chance de digitar um valor válido. Isso seria mais amigável ao usuário do que fazer o programa entrar em pânico por causa de um erro de digitação!

Quando o programa tem uma referência válida, o borrow checker impõe as regras de ownership e borrowing (cobertas no Capítulo 4) para garantir que esta referência e quaisquer outras referências ao conteúdo do vetor permaneçam válidas. Lembre-se da regra que diz que você não pode ter referências mutáveis e imutáveis no mesmo escopo. Essa regra se aplica na Listagem 8-6, onde mantemos uma referência imutável ao primeiro elemento em um vetor e tentamos adicionar um elemento ao final. Este programa não funcionará se também tentarmos referenciar esse elemento mais tarde na função.

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
}
```

[Listagem 8-6](#listagem-8-6): Tentando adicionar um elemento a um vetor enquanto mantém uma referência a um item

Compilar este código resultará neste erro:

```bash
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

O código da Listagem 8-6 pode parecer que deveria funcionar: por que uma referência ao primeiro elemento se importaria com mudanças no final do vetor? Este erro ocorre devido à forma como vetores funcionam: como vetores colocam os valores uns ao lado dos outros na memória, adicionar um novo elemento ao final do vetor pode exigir alocar nova memória e copiar os elementos antigos para o novo espaço, se não houver espaço suficiente para colocar todos os elementos uns ao lado dos outros onde o vetor está armazenado atualmente. Nesse caso, a referência ao primeiro elemento estaria apontando para memória desalocada. As regras de borrowing impedem que programas acabem nessa situação.

> **Nota:** Para mais detalhes sobre a implementação do tipo `Vec<T>`, consulte o Rustonomicon.

### Iterando sobre os valores em um vetor

Para acessar cada elemento em um vetor por vez, iteraríamos por todos os elementos em vez de usar índices para acessar um de cada vez. A Listagem 8-7 mostra como usar um loop `for` para obter referências imutáveis a cada elemento em um vetor de valores `i32` e imprimi-los.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
}
```

[Listagem 8-7](#listagem-8-7): Imprimindo cada elemento em um vetor iterando sobre os elementos usando um loop `for`

Também podemos iterar sobre referências mutáveis a cada elemento em um vetor mutável para fazer alterações em todos os elementos. O loop `for` na Listagem 8-8 adicionará `50` a cada elemento.

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}
```

[Listagem 8-8](#listagem-8-8): Iterando sobre referências mutáveis a elementos em um vetor

Para alterar o valor ao qual a referência mutável se refere, precisamos usar o operador de dereferência `*` para chegar ao valor em `i` antes de podermos usar o operador `+=`. Falaremos mais sobre o operador de dereferência na seção Seguindo a referência ao valor do Capítulo 15.

Iterar sobre um vetor, seja imutável ou mutavelmente, é seguro por causa das regras do borrow checker. Se tentássemos inserir ou remover itens nos corpos dos loops `for` nas Listagens 8-7 e 8-8, obteríamos um erro do compilador semelhante ao que obtivemos com o código da Listagem 8-6. A referência ao vetor que o loop `for` mantém impede modificação simultânea de todo o vetor.

### Usando um enum para armazenar vários tipos

Vetores só podem armazenar valores do mesmo tipo. Isso pode ser inconveniente; definitivamente há casos de uso para precisar armazenar uma lista de itens de tipos diferentes. Felizmente, as variantes de um enum são definidas sob o mesmo tipo enum, então quando precisamos de um tipo para representar elementos de tipos diferentes, podemos definir e usar um enum!

Por exemplo, digamos que queremos obter valores de uma linha em uma planilha em que algumas das colunas na linha contêm inteiros, alguns números de ponto flutuante e algumas strings. Podemos definir um enum cujas variantes conterão os diferentes tipos de valor, e todas as variantes do enum serão consideradas do mesmo tipo: o do enum. Então, podemos criar um vetor para conter esse enum e assim, em última instância, armazenar tipos diferentes. Demonstramos isso na Listagem 8-9.

**Arquivo: src/main.rs**

```rust
fn main() {
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
}
```

[Listagem 8-9](#listagem-8-9): Definindo um enum para armazenar valores de tipos diferentes em um vetor

O Rust precisa saber quais tipos estarão no vetor em tempo de compilação para saber exatamente quanta memória na heap será necessária para armazenar cada elemento. Também precisamos ser explícitos sobre quais tipos são permitidos neste vetor. Se o Rust permitisse que um vetor contivesse qualquer tipo, haveria chance de um ou mais dos tipos causarem erros com as operações realizadas nos elementos do vetor. Usar um enum mais uma expressão `match` significa que o Rust garantirá em tempo de compilação que todo caso possível é tratado, como discutido no Capítulo 6.

Se você não sabe o conjunto exaustivo de tipos que um programa receberá em tempo de execução para armazenar em um vetor, a técnica do enum não funcionará. Em vez disso, você pode usar um trait object, que cobriremos no Capítulo 18.

Agora que discutimos algumas das formas mais comuns de usar vetores, certifique-se de revisar a documentação da API para todos os muitos métodos úteis definidos em `Vec<T>` pela biblioteca padrão. Por exemplo, além de `push`, um método `pop` remove e retorna o último elemento.

### Descartar um vetor descarta seus elementos

Como qualquer outra `struct`, um vetor é liberado quando sai de escopo, como anotado na Listagem 8-10.

**Arquivo: src/main.rs**

```rust
fn main() {
    {
        let v = vec![1, 2, 3, 4];

        // faz algo com v
    } // <- v sai de escopo e é liberado aqui
}
```

[Listagem 8-10](#listagem-8-10): Mostrando onde o vetor e seus elementos são descartados

Quando o vetor é descartado, todo o seu conteúdo também é descartado, o que significa que os inteiros que contém serão limpos. O borrow checker garante que quaisquer referências ao conteúdo de um vetor só sejam usadas enquanto o próprio vetor for válido.

Vamos passar para o próximo tipo de coleção: `String`!
