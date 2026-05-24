---
title: "Processando uma série de itens com iterators"
chapter_code: 13-02
slug: processando-uma-serie-de-itens-com-iterators
---

# Processando uma Série de Itens com Iterators

O padrão iterator permite executar alguma tarefa em uma sequência de itens, um por vez. Um iterator é responsável pela lógica de iterar sobre cada item e determinar quando a sequência terminou. Quando você usa iterators, não precisa reimplementar essa lógica você mesmo.

Em Rust, iterators são _lazy_ (preguiçosos), o que significa que não têm efeito até você chamar métodos que consomem o iterator para usá-lo por completo. Por exemplo, o código na Listagem 13-10 cria um iterator sobre os itens do vetor `v1` chamando o método `iter` definido em `Vec<T>`. Esse código por si só não faz nada útil.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();
}
```

[Listagem 13-10](#listagem-13-10): Criando um iterator

O iterator é armazenado na variável `v1_iter`. Depois de criar um iterator, podemos usá-lo de várias formas. Na Listagem 3-5, iteramos sobre um array usando um loop `for` para executar algum código em cada item. Por baixo dos panos, isso criou e consumiu implicitamente um iterator, mas até agora não detalhamos exatamente como isso funciona.

No exemplo da Listagem 13-11, separamos a criação do iterator do uso do iterator no loop `for`. Quando o loop `for` é chamado usando o iterator em `v1_iter`, cada elemento do iterator é usado em uma iteração do loop, que imprime cada valor.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {val}");
    }
}
```

[Listagem 13-11](#listagem-13-11): Usando um iterator em um loop `for`

Em linguagens que não têm iterators fornecidos pelas bibliotecas padrão, você provavelmente escreveria a mesma funcionalidade começando uma variável no índice 0, usando essa variável para indexar o vetor e obter um valor, e incrementando a variável em um loop até atingir o número total de itens no vetor.

Iterators cuidam de toda essa lógica para você, reduzindo código repetitivo que você poderia errar. Iterators dão mais flexibilidade para usar a mesma lógica com muitos tipos diferentes de sequências, não apenas estruturas de dados que você pode indexar, como vetores. Vamos examinar como iterators fazem isso.

## A Trait `Iterator` e o Método `next`

Todos os iterators implementam uma trait chamada `Iterator`, definida na biblioteca padrão. A definição da trait se parece com isto:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // métodos com implementações padrão omitidos
}
```

Observe que esta definição usa sintaxe nova: `type Item` e `Self::Item`, que definem um tipo associado com esta trait. Falaremos sobre tipos associados em profundidade no Capítulo 20. Por enquanto, basta saber que este código diz que implementar a trait `Iterator` exige que você também defina um tipo `Item`, e esse tipo `Item` é usado no tipo de retorno do método `next`. Em outras palavras, o tipo `Item` será o tipo retornado pelo iterator.

A trait `Iterator` só exige que os implementadores definam um método: o método `next`, que retorna um item do iterator por vez, embrulhado em `Some`, e, quando a iteração termina, retorna `None`.

Podemos chamar o método `next` em iterators diretamente; a Listagem 13-12 demonstra quais valores são retornados de chamadas repetidas a `next` no iterator criado a partir do vetor.

**Arquivo: src/lib.rs**

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
}
```

[Listagem 13-12](#listagem-13-12): Chamando o método `next` em um iterator

Note que precisamos tornar `v1_iter` mutável: chamar o método `next` em um iterator altera o estado interno que o iterator usa para rastrear onde está na sequência. Em outras palavras, este código _consome_, ou usa, o iterator. Cada chamada a `next` consome um item do iterator. Não precisamos tornar `v1_iter` mutável quando usamos um loop `for`, porque o loop tomou posse de `v1_iter` e o tornou mutável nos bastidores.

Também note que os valores que obtemos das chamadas a `next` são referências imutáveis aos valores no vetor. O método `iter` produz um iterator sobre referências imutáveis. Se quisermos criar um iterator que toma posse de `v1` e retorna valores owned, podemos chamar `into_iter` em vez de `iter`. Da mesma forma, se quisermos iterar sobre referências mutáveis, podemos chamar `iter_mut` em vez de `iter`.

## Métodos que Consomem o Iterator

A trait `Iterator` tem vários métodos diferentes com implementações padrão fornecidas pela biblioteca padrão; você pode conhecer esses métodos consultando a documentação da API da biblioteca padrão para a trait `Iterator`. Alguns desses métodos chamam o método `next` em sua definição, por isso você precisa implementar o método `next` ao implementar a trait `Iterator`.

Métodos que chamam `next` são chamados de _adapters consumidores_ porque chamá-los usa o iterator. Um exemplo é o método `sum`, que toma posse do iterator e itera pelos itens chamando `next` repetidamente, consumindo o iterator. Enquanto itera, adiciona cada item a um total acumulado e retorna o total quando a iteração termina. A Listagem 13-13 tem um teste ilustrando o uso do método `sum`.

**Arquivo: src/lib.rs**

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
}
```

[Listagem 13-13](#listagem-13-13): Chamando o método `sum` para obter o total de todos os itens do iterator

Não podemos usar `v1_iter` depois da chamada a `sum`, porque `sum` toma posse do iterator em que é chamado.

## Métodos que Produzem Outros Iterators

_Adapters de iterator_ são métodos definidos na trait `Iterator` que não consomem o iterator. Em vez disso, produzem iterators diferentes alterando algum aspecto do iterator original.

A Listagem 13-14 mostra um exemplo de chamada ao adapter de iterator `map`, que recebe uma closure para chamar em cada item conforme os itens são iterados. O método `map` retorna um novo iterator que produz os itens modificados. A closure aqui cria um novo iterator em que cada item do vetor será incrementado em 1.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    v1.iter().map(|x| x + 1);
}
```

[Listagem 13-14](#listagem-13-14): Chamando o adapter de iterator `map` para criar um novo iterator

No entanto, este código produz um aviso:

```bash
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: iterators are lazy and do nothing unless consumed
  = note: `#[warn(unused_must_use)]` on by default
help: use `let _ = ...` to ignore the resulting value
  |
4 |     let _ = v1.iter().map(|x| x + 1);
  |     +++++++

warning: `iterators` (bin "iterators") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.47s
     Running `target/debug/iterators`
```

O código na Listagem 13-14 não faz nada; a closure que especificamos nunca é chamada. O aviso nos lembra o porquê: adapters de iterator são lazy, e precisamos consumir o iterator aqui.

Para corrigir este aviso e consumir o iterator, usaremos o método `collect`, que usamos com `env::args` na Listagem 12-1. Este método consome o iterator e coleta os valores resultantes em um tipo de coleção.

Na Listagem 13-15, coletamos os resultados de iterar sobre o iterator retornado pela chamada a `map` em um vetor. Este vetor acabará contendo cada item do vetor original, incrementado em 1.

**Arquivo: src/main.rs**

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
}
```

[Listagem 13-15](#listagem-13-15): Chamando o método `map` para criar um novo iterator e depois o método `collect` para consumir o novo iterator e criar um vetor

Como `map` recebe uma closure, podemos especificar qualquer operação que queiramos executar em cada item. Este é um ótimo exemplo de como closures permitem personalizar algum comportamento enquanto reutilizamos o comportamento de iteração que a trait `Iterator` fornece.

Você pode encadear várias chamadas a adapters de iterator para executar ações complexas de forma legível. Mas como todos os iterators são lazy, você precisa chamar um dos métodos adapter consumidores para obter resultados das chamadas a adapters de iterator.

## Closures que Capturam o Ambiente

Muitos adapters de iterator recebem closures como argumentos, e comumente as closures que especificaremos como argumentos para adapters de iterator serão closures que capturam seu ambiente.

Para este exemplo, usaremos o método `filter`, que recebe uma closure. A closure recebe um item do iterator e retorna um `bool`. Se a closure retornar `true`, o valor será incluído na iteração produzida por `filter`. Se a closure retornar `false`, o valor não será incluído.

Na Listagem 13-16, usamos `filter` com uma closure que captura a variável `shoe_size` de seu ambiente para iterar sobre uma coleção de instâncias da struct `Shoe`. Ela retornará apenas sapatos do tamanho especificado.

**Arquivo: src/lib.rs**

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
```

[Listagem 13-16](#listagem-13-16): Usando o método `filter` com uma closure que captura `shoe_size`

A função `shoes_in_size` toma posse de um vetor de sapatos e um tamanho de sapato como parâmetros. Ela retorna um vetor contendo apenas sapatos do tamanho especificado.

No corpo de `shoes_in_size`, chamamos `into_iter` para criar um iterator que toma posse do vetor. Em seguida, chamamos `filter` para adaptar esse iterator em um novo iterator que contém apenas elementos para os quais a closure retorna `true`.

A closure captura o parâmetro `shoe_size` do ambiente e compara o valor com o tamanho de cada sapato, mantendo apenas sapatos do tamanho especificado. Por fim, chamar `collect` reúne os valores retornados pelo iterator adaptado em um vetor que é retornado pela função.

O teste mostra que, quando chamamos `shoes_in_size`, obtemos de volta apenas sapatos que têm o mesmo tamanho que o valor que especificamos.
