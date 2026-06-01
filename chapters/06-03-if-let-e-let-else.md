---
title: "Controle de fluxo conciso com `if let` e `let...else`"
chapter_code: 06-03
slug: if-let-e-let-else
challenge_day: 8
reading_minutes: 11
---

# Controle de Fluxo Conciso com `if let` e `let...else`

A sintaxe `if let` permite combinar `if` e `let` de uma forma menos verbosa para lidar com valores que correspondem a um padrão enquanto ignora o restante. Considere o programa da Listagem 6-6 que faz `match` em um valor `Option<u8>` na variável `config_max`, mas só quer executar código se o valor for a variante `Some`.

**Arquivo: src/main.rs**

```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {max}"),
    _ => (),
}
```

<a id="listagem-6-6"></a>

[Listagem 6-6](#listagem-6-6): Um `match` que só se importa em executar código quando o valor é `Some`

Se o valor for `Some`, imprimimos o valor na variante `Some` fazendo bind do valor à variável `max` no padrão. Não queremos fazer nada com o valor `None`. Para satisfazer a expressão `match`, temos que adicionar `_ => ()` depois de processar apenas uma variante, o que é código boilerplate irritante de adicionar.

Em vez disso, poderíamos escrever isso de forma mais curta usando `if let`. O código a seguir se comporta da mesma forma que o `match` da Listagem 6-6:

**Arquivo: src/main.rs**

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {max}");
}
```

A sintaxe `if let` recebe um padrão e uma expressão separados por um sinal de igual. Funciona da mesma forma que um `match`, em que a expressão é dada ao `match` e o padrão é seu primeiro braço. Neste caso, o padrão é `Some(max)`, e `max` faz bind ao valor dentro do `Some`. Podemos então usar `max` no corpo do bloco `if let` da mesma forma que usamos `max` no braço correspondente do `match`. O código no bloco `if let` só é executado se o valor corresponder ao padrão.

Usar `if let` significa menos digitação, menos indentação e menos código boilerplate. Porém, você perde a verificação exaustiva que o `match` impõe e que garante que você não está esquecendo de tratar algum caso. Escolher entre `match` e `if let` depende do que você está fazendo na sua situação particular e se ganhar concisão é uma troca apropriada por perder a verificação exaustiva.

Em outras palavras, o `if let` é uma forma simplificada de um `match` que executa código quando o valor corresponde a um padrão específico e ignora os demais.

Podemos incluir um `else` com um `if let`. O bloco de código que acompanha o `else` é o mesmo que iria com o caso `_` na expressão `match` equivalente ao `if let` e `else`. Lembre-se da definição do enum `Coin` na [Listagem 6-4](/livro/cap06-01-definindo-um-enum#listagem-6-4), em que a variante `Quarter` também continha um valor `UsState`. Se quiséssemos contar todas as moedas que não são quarters que vemos enquanto também anunciamos o estado das quarters, poderíamos fazer isso com uma expressão `match`, assim:

**Arquivo: src/main.rs**

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {state:?}!"),
    _ => count += 1,
}
```

Ou poderíamos usar uma expressão `if let` e `else`, assim:

**Arquivo: src/main.rs**

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {state:?}!");
} else {
    count += 1;
}
```

## Permanecendo no “caminho feliz” com `let...else`

O padrão comum é realizar algum cálculo quando um valor está presente e retornar um valor padrão caso contrário. Continuando com nosso exemplo de moedas com um valor `UsState`, se quiséssemos dizer algo engraçado dependendo de quão antigo é o estado na moeda de 25 centavos, poderíamos introduzir um método em `UsState` para verificar a idade de um estado, assim:

**Arquivo: src/main.rs**

```rust
impl UsState {
    fn existed_in(&self, year: u16) -> bool {
        match self {
            UsState::Alabama => year >= 1819,
            UsState::Alaska => year >= 1959,
            // --snip--
        }
    }
}
```

Em seguida, poderíamos usar `if let` para fazer `match` no tipo de moeda, introduzindo uma variável `state` dentro do corpo da condição, como na Listagem 6-7.

**Arquivo: src/main.rs**

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    if let Coin::Quarter(state) = coin {
        if state.existed_in(1900) {
            Some(format!("{state:?} is pretty old, for America!"))
        } else {
            Some(format!("{state:?} is relatively new."))
        }
    } else {
        None
    }
}
```

<a id="listagem-6-7"></a>

[Listagem 6-7](#listagem-6-7): Verificando se um estado existia em 1900 usando condicionais aninhados dentro de um `if let`

Isso resolve o problema, mas empurrou o trabalho para o corpo da declaração `if let`, e se o trabalho a ser feito for mais complicado, pode ser difícil acompanhar exatamente como os ramos de nível superior se relacionam. Também poderíamos aproveitar o fato de que expressões produzem um valor para produzir o `state` a partir do `if let` ou retornar cedo, como na Listagem 6-8. (Você poderia fazer algo semelhante com um `match` também.)

**Arquivo: src/main.rs**

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(state) = coin {
        state
    } else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```

<a id="listagem-6-8"></a>

[Listagem 6-8](#listagem-6-8): Usando `if let` para produzir um valor ou retornar cedo

Isso também é um pouco irritante de acompanhar à sua maneira! Um braço do `if let` produz um valor, e o outro retorna da função inteiramente.

Para tornar esse padrão comum mais agradável de expressar, o Rust tem `let...else`. A sintaxe `let...else` recebe um padrão à esquerda e uma expressão à direita, muito semelhante a `if let`, mas não tem um braço `if`, apenas um braço `else`. Se o padrão corresponder, fará bind ao valor do padrão no escopo externo. Se o padrão _não_ corresponder, o programa fluirá para o braço `else`, que deve retornar da função.

Na Listagem 6-9, você pode ver como a Listagem 6-8 fica ao usar `let...else` no lugar de `if let`.

**Arquivo: src/main.rs**

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let Coin::Quarter(state) = coin else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```

<a id="listagem-6-9"></a>

[Listagem 6-9](#listagem-6-9): Usando `let...else` para esclarecer o fluxo pela função

Observe que, desta forma, permanece no “caminho feliz” no corpo principal da função, sem ter fluxo de controle significativamente diferente para dois braços como o `if let` tinha.

Se você tiver uma situação em que a lógica do seu programa é verbosa demais para expressar com um `match`, lembre-se de que `if let` e `let...else` também estão na sua caixa de ferramentas Rust.

## Resumo

Agora cobrimos como usar enums para criar tipos personalizados que podem ser um de um conjunto de valores enumerados. Mostramos como o tipo `Option<T>` da biblioteca padrão ajuda você a usar o sistema de tipos para evitar erros. Quando valores de enum têm dados dentro deles, você pode usar `match` ou `if let` para extrair e usar esses valores, dependendo de quantos casos você precisa tratar.

Seus programas Rust agora podem expressar conceitos no seu domínio usando structs e enums. Criar tipos personalizados para usar na sua API garante segurança de tipos: o compilador terá certeza de que suas funções só recebem valores do tipo que cada função espera.

Para fornecer uma API bem organizada aos seus usuários, direta de usar e que exponha exatamente o que seus usuários precisarão, vamos agora passar para os [módulos do Rust](/livro/cap07-00-gerenciando-projetos-crescentes-com-packages-crates-e-modulos).
