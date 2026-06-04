---
title: "Tipos genéricos, traits e lifetimes"
chapter_code: 10-00
slug: tipos-genericos-traits-e-lifetimes
challenge_day: 12
reading_minutes: 6
---

# Tipos Genéricos, Traits e Lifetimes

Toda linguagem de programação tem ferramentas para lidar efetivamente com a duplicação de conceitos. Em Rust, uma dessas ferramentas são os _generics_: substitutos abstratos de tipos concretos ou outras propriedades. Podemos expressar o comportamento de generics ou como eles se relacionam com outros generics sem saber o que estará no lugar deles ao compilar e executar o código.

Funções podem receber parâmetros de algum tipo genérico, em vez de um tipo concreto como `i32` ou `String`, da mesma forma que recebem parâmetros com valores desconhecidos para executar o mesmo código em vários valores concretos. Na verdade, já usamos generics no [Capítulo 6](/livro/cap06-00-enums-e-pattern-matching) com `Option<T>`, no [Capítulo 8](/livro/cap08-00-colecoes-comuns) com `Vec<T>` e `HashMap<K, V>`, e no [Capítulo 9](/livro/cap09-00-tratamento-de-erros) com `Result<T, E>`. Neste capítulo, você explorará como definir seus próprios tipos, funções e métodos com generics!

Primeiro, revisaremos como extrair uma função para reduzir duplicação de código. Em seguida, usaremos a mesma técnica para criar uma função genérica a partir de duas funções que diferem apenas nos tipos de seus parâmetros. Também explicaremos como usar tipos genéricos em definições de struct e enum.

Depois, você aprenderá a usar traits para definir comportamento de forma genérica. Você pode combinar traits com tipos genéricos para restringir um tipo genérico a aceitar apenas tipos que tenham um comportamento particular, em oposição a qualquer tipo.

Por fim, discutiremos _lifetimes_: uma variedade de generics que fornece ao compilador informações sobre como referências se relacionam entre si. Lifetimes nos permitem dar ao compilador informação suficiente sobre valores emprestados para que ele possa garantir que referências serão válidas em mais situações do que poderia sem nossa ajuda.

## Removendo Duplicação Extraindo uma Função

Generics nos permitem substituir tipos específicos por um placeholder que representa vários tipos para remover duplicação de código. Antes de mergulhar na sintaxe de generics, vejamos como remover duplicação de uma forma que não envolve tipos genéricos, extraindo uma função que substitui valores específicos por um placeholder que representa vários valores. Depois, aplicaremos a mesma técnica para extrair uma função genérica! Ao ver como reconhecer código duplicado que pode ser extraído para uma função, você começará a reconhecer código duplicado que pode usar generics.

Começaremos com o programa curto da Listagem 10-1, que encontra o maior número em uma lista.

**Arquivo: src/main.rs**

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("O maior número é {largest}");
}
```

<a id="listagem-10-1"></a>

[Listagem 10-1](#listagem-10-1): Encontrando o maior número em uma lista de números

Armazenamos uma lista de inteiros na variável `number_list` e colocamos uma referência ao primeiro número da lista em uma variável chamada `largest`. Em seguida, iteramos por todos os números da lista e, se o número atual for maior que o número armazenado em `largest`, substituímos a referência nessa variável. Porém, se o número atual for menor ou igual ao maior número visto até então, a variável não muda e o código passa para o próximo número da lista. Depois de considerar todos os números da lista, `largest` deve referenciar o maior número, que neste caso é 100.

Agora recebemos a tarefa de encontrar o maior número em duas listas diferentes de números. Para isso, podemos escolher duplicar o código da Listagem 10-1 e usar a mesma lógica em dois lugares diferentes do programa, como mostrado na Listagem 10-2.

**Arquivo: src/main.rs**

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("O maior número é {largest}");

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("O maior número é {largest}");
}
```

<a id="listagem-10-2"></a>

[Listagem 10-2](#listagem-10-2): Código para encontrar o maior número em _duas_ listas de números

Embora este código funcione, duplicar código é tedioso e propenso a erros. Também precisamos lembrar de atualizar o código em vários lugares quando quisermos alterá-lo.

Para eliminar essa duplicação, criaremos uma abstração definindo uma função que opera em qualquer lista de inteiros passada como parâmetro. Essa solução torna nosso código mais claro e nos permite expressar o conceito de encontrar o maior número em uma lista de forma abstrata.

Na Listagem 10-3, extraímos o código que encontra o maior número para uma função chamada `largest`. Em seguida, chamamos a função para encontrar o maior número nas duas listas da Listagem 10-2. Também poderíamos usar a função em qualquer outra lista de valores `i32` que tivéssemos no futuro.

**Arquivo: src/main.rs**

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("O maior número é {result}");

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("O maior número é {result}");
}
```

<a id="listagem-10-3"></a>

[Listagem 10-3](#listagem-10-3): Código abstraído para encontrar o maior número em duas listas

A função `largest` tem um parâmetro chamado `list`, que representa qualquer slice concreto de valores `i32` que possamos passar para a função. Como resultado, quando chamamos a função, o código executa nos valores específicos que passamos.

Em resumo, estes são os passos que seguimos para mudar o código da Listagem 10-2 para a Listagem 10-3:

1. Identificar código duplicado.
2. Extrair o código duplicado para o corpo da função e especificar as entradas e os valores de retorno desse código na assinatura da função.
3. Atualizar as duas instâncias de código duplicado para chamar a função em vez disso.

Em seguida, usaremos esses mesmos passos com generics para reduzir duplicação de código. Da mesma forma que o corpo da função pode operar em uma `list` abstrata em vez de valores específicos, generics permitem que o código opere em tipos abstratos.

Por exemplo, digamos que tivéssemos duas funções: uma que encontra o maior item em um slice de valores `i32` e outra que encontra o maior item em um slice de valores `char`. Como eliminaríamos essa duplicação? Vamos descobrir!
