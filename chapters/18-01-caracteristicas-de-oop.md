---
title: "Características de linguagens orientadas a objetos"
chapter_code: 18-01
slug: caracteristicas-de-linguagens-orientadas-a-objetos
---

# Características de Linguagens Orientadas a Objetos

Não há consenso na comunidade sobre quais recursos uma linguagem precisa ter para ser considerada orientada a objetos. Rust é influenciada por muitos paradigmas, incluindo POO; por exemplo, exploramos recursos vindos da programação funcional no Capítulo 13. Argumentavelmente, linguagens POO compartilham certas características — objetos, encapsulamento e herança. Vejamos o que cada uma significa e se Rust as suporta.

## Objetos contêm dados e comportamento

O livro _Design Patterns: Elements of Reusable Object-Oriented Software_, de Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides (Addison-Wesley, 1994), conhecido como livro da _Gang of Four_, é um catálogo de padrões de design orientados a objetos. Define POO assim:

> Programas orientados a objetos são feitos de objetos. Um **objeto** empacota dados e os procedimentos que operam sobre esses dados. Os procedimentos costumam ser chamados de **métodos** ou **operações**.

Por essa definição, Rust é orientada a objetos: structs e enums têm dados, e blocos `impl` fornecem métodos. Embora structs e enums com métodos não sejam _chamados_ de objetos, oferecem a mesma funcionalidade segundo a definição da Gang of Four.

## Encapsulamento que esconde detalhes de implementação

Outro aspecto associado à POO é _encapsulamento_: detalhes de implementação de um objeto não são acessíveis ao código que o usa. A interação só pode ocorrer pela API pública; código externo não deve alterar dados ou comportamento internos diretamente. Isso permite refatorar o interior sem mudar quem usa o objeto.

Vimos como controlar encapsulamento no Capítulo 7: `pub` decide o que é público; o resto é privado por padrão. Podemos definir `AveragedCollection` com um vetor de `i32` e um campo com a média em cache. A Listagem 18-1 define a struct.

**Arquivo: src/lib.rs**

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

<a id="listagem-18-1"></a>

[Listagem 18-1](#listagem-18-1): Struct `AveragedCollection` que mantém lista de inteiros e média dos itens

A struct é `pub`, mas os campos permanecem privados. Queremos que, ao adicionar ou remover da lista, a média seja atualizada. Implementamos `add`, `remove` e `average` na Listagem 18-2.

**Arquivo: src/lib.rs**

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

<a id="listagem-18-2"></a>

[Listagem 18-2](#listagem-18-2): Implementações públicas de `add`, `remove` e `average` em `AveragedCollection`

`add`, `remove` e `average` são as únicas formas de acessar ou modificar dados. `add` e `remove` chamam o método privado `update_average`.

Deixamos `list` e `average` privados para código externo não alterar `list` diretamente — senão `average` poderia dessincronizar. `average` só lê o campo.

Com encapsulamento, podemos mudar a estrutura interna (ex.: `HashSet<i32>` em vez de `Vec<i32>`) mantendo as assinaturas públicas. Se `list` fosse público, quem alterasse `list` diretamente precisaria mudar porque `HashSet` e `Vec` têm APIs diferentes.

Se encapsulamento for requisito para POO, Rust atende. `pub` ou não em cada parte permite encapsular detalhes.

## Herança como sistema de tipos e compartilhamento de código

_Herança_ é um mecanismo pelo qual um objeto herda elementos da definição de outro, ganhando dados e comportamento do pai sem redefini-los.

Se herança for obrigatória para POO, Rust não é orientada a objetos: não há como definir struct que herde campos e métodos do pai sem macro.

Se você está acostumado à herança, há alternativas em Rust conforme o motivo.

Um motivo é reutilizar código: implementar comportamento num tipo e reutilizar noutro. Em Rust, de forma limitada, com implementações padrão de métodos de trait — como `summarize` na trait `Summary` na Listagem 10-14. Qualquer tipo que implemente `Summary` tem `summarize` sem código extra, parecido com classe pai e filha. Podemos sobrescrever ao implementar a trait, como filho sobrescrevendo método herdado.

Outro motivo é o sistema de tipos: filho usado onde o pai é esperado — _polimorfismo_, substituir objetos em tempo de execução se compartilham características.

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinônimo de herança. É um conceito mais geral: código que funciona com dados de vários tipos. Em herança, esses tipos são em geral subclasses.
>
> Rust usa generics para abstrair tipos possíveis e trait bounds para impor o que devem fornecer — às vezes chamado de _polimorfismo paramétrico limitado_.

Rust escolheu trade-offs diferentes ao não oferecer herança. Herança pode compartilhar mais código do que o necessário; subclasses herdam tudo do pai, reduzindo flexibilidade, e podem expor métodos que não fazem sentido no filho. Algumas linguagens só permitem _herança simples_, limitando ainda mais o design.

Por isso Rust usa trait objects em vez de herança para polimorfismo em tempo de execução. Vejamos como funcionam.
