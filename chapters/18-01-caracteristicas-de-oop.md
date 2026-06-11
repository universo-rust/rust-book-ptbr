---
title: "Características de linguagens orientadas a objetos"
chapter_code: 18-01
slug: caracteristicas-de-linguagens-orientadas-a-objetos
---

# Características de linguagens orientadas a objetos

Não há consenso na comunidade de programação sobre quais recursos uma linguagem precisa ter para ser considerada orientada a objetos. Rust é influenciada por muitos paradigmas de programação, incluindo POO; por exemplo, no Capítulo 13 exploramos recursos vindos da programação funcional. De modo geral, linguagens orientadas a objetos costumam compartilhar algumas características: objetos, encapsulamento e herança. Vamos ver o que cada uma dessas características significa e se Rust oferece suporte a elas.

## Objetos contêm dados e comportamento

O livro _Design Patterns: Elements of Reusable Object-Oriented Software_, de Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides (Addison-Wesley, 1994), conhecido informalmente como o livro da _Gang of Four_, é um catálogo de padrões de design orientados a objetos. Ele define POO desta forma:

> Programas orientados a objetos são formados por objetos. Um **objeto** empacota tanto dados quanto os procedimentos que operam sobre esses dados. Esses procedimentos normalmente são chamados de **métodos** ou **operações**.

Usando essa definição, Rust é orientada a objetos: structs e enums têm dados, e blocos `impl` fornecem métodos para structs e enums. Embora structs e enums com métodos não sejam _chamados_ de objetos, eles oferecem a mesma funcionalidade segundo a definição de objeto da Gang of Four.

## Encapsulamento que esconde detalhes de implementação

Outro aspecto frequentemente associado à POO é a ideia de _encapsulamento_, que significa que os detalhes de implementação de um objeto não ficam acessíveis ao código que usa esse objeto. Portanto, a única forma de interagir com um objeto deve ser por meio de sua API pública; o código que usa o objeto não deveria conseguir alcançar seus detalhes internos e alterar dados ou comportamento diretamente. Isso permite que a pessoa programadora altere e refatore o interior do objeto sem precisar mudar o código que o utiliza.

Discutimos como controlar encapsulamento no Capítulo 7: podemos usar a palavra-chave `pub` para decidir quais módulos, tipos, funções e métodos do nosso código devem ser públicos, enquanto todo o resto é privado por padrão. Por exemplo, podemos definir uma struct `AveragedCollection` com um campo que contém um vetor de valores `i32`. A struct também pode ter um campo que contém a média dos valores no vetor, o que significa que a média não precisa ser calculada sob demanda sempre que alguém precisar dela. Em outras palavras, `AveragedCollection` manterá a média calculada em cache para nós. A Listagem 18-1 mostra a definição da struct `AveragedCollection`.

**Arquivo: src/lib.rs**

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

<a id="listagem-18-1"></a>

[Listagem 18-1](#listagem-18-1): Struct `AveragedCollection` que mantém uma lista de inteiros e a média dos itens na coleção

A struct é marcada como `pub`, para que outros códigos possam usá-la, mas seus campos continuam privados. Isso é importante neste caso porque queremos garantir que, sempre que um valor for adicionado à lista ou removido dela, a média também seja atualizada. Fazemos isso implementando os métodos `add`, `remove` e `average` na struct, como mostra a Listagem 18-2.

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

[Listagem 18-2](#listagem-18-2): Implementações públicas dos métodos `add`, `remove` e `average` em `AveragedCollection`

Os métodos públicos `add`, `remove` e `average` são as únicas formas de acessar ou modificar os dados de uma instância de `AveragedCollection`. Quando um item é adicionado a `list` com o método `add`, ou removido com o método `remove`, as implementações desses métodos chamam o método privado `update_average`, que também atualiza o campo `average`.

Mantemos os campos `list` e `average` privados para que código externo não consiga adicionar nem remover itens diretamente do campo `list`; caso contrário, o campo `average` poderia ficar fora de sincronia quando `list` mudasse. O método `average` retorna o valor do campo `average`, permitindo que código externo leia a média, mas não a modifique.

Como encapsulamos os detalhes de implementação da struct `AveragedCollection`, podemos mudar aspectos internos no futuro com facilidade, como a estrutura de dados usada. Por exemplo, poderíamos usar um `HashSet<i32>` em vez de um `Vec<i32>` para o campo `list`. Enquanto as assinaturas dos métodos públicos `add`, `remove` e `average` continuassem as mesmas, o código que usa `AveragedCollection` não precisaria mudar. Se `list` fosse público, isso não seria necessariamente verdade: `HashSet<i32>` e `Vec<i32>` têm métodos diferentes para adicionar e remover itens, então o código externo provavelmente teria que mudar se estivesse modificando `list` diretamente.

Se encapsulamento for uma característica obrigatória para que uma linguagem seja considerada orientada a objetos, então Rust atende a esse requisito. A possibilidade de usar ou não `pub` em diferentes partes do código permite encapsular detalhes de implementação.

## Herança como sistema de tipos e compartilhamento de código

_Herança_ é um mecanismo pelo qual um objeto pode herdar elementos da definição de outro objeto, ganhando os dados e o comportamento do objeto pai sem que você precise defini-los novamente.

Se uma linguagem precisa ter herança para ser orientada a objetos, então Rust não é essa linguagem. Não há uma forma de definir uma struct que herde os campos e as implementações de métodos de uma struct pai sem usar uma macro.

No entanto, se você está acostumado a ter herança na sua caixa de ferramentas de programação, pode usar outras soluções em Rust, dependendo do motivo que o levaria a buscar herança em primeiro lugar.

Você escolheria herança por dois motivos principais. Um deles é reutilização de código: você implementa um comportamento específico para um tipo, e a herança permite reutilizar essa implementação em outro tipo. Em Rust, dá para fazer isso de forma limitada usando implementações padrão de métodos de traits, como vimos na Listagem 10-14 ao adicionar uma implementação padrão do método `summarize` à trait `Summary`. Qualquer tipo que implementasse a trait `Summary` teria o método `summarize` disponível sem código adicional. Isso é parecido com uma classe pai que tem a implementação de um método e uma classe filha que herda essa implementação. Também podemos sobrescrever a implementação padrão de `summarize` ao implementar a trait `Summary`, o que é parecido com uma classe filha sobrescrevendo a implementação de um método herdado de uma classe pai.

O outro motivo para usar herança está relacionado ao sistema de tipos: permitir que um tipo filho seja usado nos mesmos lugares que o tipo pai. Isso também é chamado de _polimorfismo_, que significa poder substituir vários objetos uns pelos outros em tempo de execução se eles compartilharem certas características.

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinônimo de herança. Mas, na verdade, polimorfismo é um conceito mais geral: código que consegue trabalhar com dados de múltiplos tipos. No caso da herança, esses tipos normalmente são subclasses.
>
> Rust usa generics para abstrair diferentes tipos possíveis e trait bounds para impor restrições sobre o que esses tipos precisam fornecer. Às vezes, isso é chamado de _polimorfismo paramétrico limitado_.

Rust escolheu um conjunto diferente de tradeoffs ao não oferecer herança. Herança muitas vezes corre o risco de compartilhar mais código do que o necessário. Subclasses nem sempre deveriam compartilhar todas as características da classe pai, mas com herança acabam compartilhando. Isso pode tornar o design de um programa menos flexível. Também abre a possibilidade de chamar em subclasses métodos que não fazem sentido ou que causam erros porque não se aplicam àquela subclasse. Além disso, algumas linguagens permitem apenas _herança simples_, ou seja, uma subclasse só pode herdar de uma classe, o que restringe ainda mais a flexibilidade do design.

Por esses motivos, Rust adota uma abordagem diferente: usa trait objects em vez de herança para alcançar polimorfismo em tempo de execução. Vamos ver como trait objects funcionam.
