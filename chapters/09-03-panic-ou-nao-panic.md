---
title: "Panic ou não panic"
chapter_code: 09-03
slug: panic-ou-nao-panic
---

# Panic ou Não Panic

Como decidir entre `panic!` e `Result`? Quando o código entra em pânico, não há recuperação. Você poderia chamar `panic!` em qualquer erro — recuperável ou não —, mas aí decide pelo chamador que a situação é irrecuperável. Ao retornar `Result`, você deixa a escolha com quem chama: tentar se recuperar da forma que fizer sentido, ou tratar `Err` como irrecuperável e chamar `panic!`, transformando seu erro recuperável em irrecuperável. Por isso, ao definir uma função que pode falhar, retornar `Result` costuma ser o padrão certo.

Em exemplos, protótipos e testes, faz mais sentido deixar o código entrar em pânico do que retornar `Result`. Veremos por quê; depois, casos em que o compilador não prova que a falha é impossível, mas você sabe que é. O capítulo fecha com diretrizes para decidir pânico em código de biblioteca.

### Exemplos, protótipos e testes

Num exemplo que ilustra um conceito, tratamento de erro robusto pode atrapalhar a leitura. Em exemplos, assume-se que um `unwrap` que poderia entrar em pânico é um placeholder — a aplicação real trataria o erro de outro jeito, conforme o restante do código.

`unwrap` e `expect` também ajudam no protótipo, quando você ainda não definiu como tratar erros. Eles marcam pontos no código para revisar quando for tornar o programa mais sólido.

Se uma chamada falha num teste, o teste inteiro deve falhar — mesmo que aquela chamada não seja o foco do teste. Como falha de teste passa por `panic!`, `unwrap` ou `expect` é exatamente o comportamento desejado.

### Quando você sabe mais que o compilador

Também faz sentido usar `expect` quando outra lógica garante um `Ok`, mas o compilador não consegue ver isso. A operação ainda retorna `Result` e, em geral, ainda pode falhar — só que, no seu caso, isso é logicamente impossível. Se, inspecionando o código, você tem certeza de que nunca haverá `Err`, `expect` é aceitável; documente o motivo na mensagem. Exemplo:

**Arquivo: src/main.rs**

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1"
    .parse()
    .expect("Hardcoded IP address should be valid");
```

Criamos um `IpAddr` fazendo parse de uma string fixa no código. `127.0.0.1` é válido — `expect` aqui é razoável. Porém, uma string válida embutida não muda o retorno de `parse`: continua sendo `Result`, e o compilador exige tratar `Err` como possibilidade, porque não infere que esta string sempre será um IP válido. Se o endereço viesse do usuário e pudesse falhar, trataríamos `Result` de forma mais robusta. Mencionar na mensagem que o IP é hardcoded lembra de trocar `expect` por algo melhor se, no futuro, a fonte mudar.

### Diretrizes para tratamento de erros

Entre em pânico quando o código pode cair num _estado inválido_. Neste contexto, estado inválido é violação de suposição, garantia, contrato ou invariante — valores inválidos, contraditórios ou ausentes passados ao seu código — e pelo menos um destes:

- O estado é inesperado, não algo que ocorre de vez em quando (como dados mal formatados digitados pelo usuário).
- Daí em diante o código precisa confiar que não está nesse estado, em vez de checar a cada passo.
- Não há boa forma de expressar isso nos tipos. Veremos um exemplo em [Codificando estados e comportamento como tipos](/livro/cap18-03-implementando-um-padrao-de-design-orientado-a-objetos#codificando-estados-e-comportamento-como-tipos), no Capítulo 18.

Se alguém chama sua API com valores sem sentido, prefira retornar erro quando possível — quem usa a biblioteca decide o que fazer. Se continuar seria inseguro ou prejudicial, `panic!` pode ser melhor: avisa quem usa a biblioteca sobre o bug no código dela, para corrigir em desenvolvimento. `panic!` também costuma servir quando código externo, fora do seu controle, devolve um estado inválido que você não consegue consertar.

Quando a falha é esperada, retorne `Result` em vez de `panic!`. Parser com entrada malformada, HTTP com status de rate limit — casos em que falhar faz parte do fluxo normal. `Result` deixa claro que o chamador deve decidir como reagir.

Se uma operação com valores inválidos coloca o usuário em risco, valide antes e entre em pânico se não passar. Motivo principal: segurança — operar em dados inválidos abre vulnerabilidades. Por isso a biblioteca padrão entra em pânico em acesso fora dos limites: ler memória que não pertence à estrutura é falha grave de segurança. Funções têm _contratos_: o comportamento só é garantido se as entradas cumprem requisitos. Violar o contrato indica bug do chamador — não é erro que o chamador deva tratar explicitamente; não há recuperação razoável, só correção do código. Documente contratos na API, especialmente quando violação causa pânico.

Checar erro em toda função seria verboso. O sistema de tipos do Rust (e a verificação do compilador) faz parte desse trabalho. Com um tipo concreto no parâmetro, você segue sabendo que o valor já foi validado. Com um tipo em vez de `Option`, o programa espera algo, não nada — sem ramos `Some`/`None`; quem tentar passar “nada” nem compila. Outro exemplo: `u32` garante que o parâmetro nunca é negativo.

### Tipos personalizados para validação

Levemos a ideia adiante: um tipo customizado para validação. No jogo de adivinhação do Capítulo 2, pedíamos um número entre 1 e 100, mas só validávamos se o palpite era positivo — não se estava no intervalo. As consequências eram leves (“Muito alto” / “Muito baixo” ainda faziam sentido), mas seria melhor orientar palpites válidos e distinguir número fora do intervalo de entrada inválida (letras, por exemplo).

Uma abordagem: fazer parse como `i32` (não só `u32`, para aceitar negativos) e checar o intervalo:

**Arquivo: src/main.rs**

```rust
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
        // --snip--
    }
}
```

O `if` detecta valor fora do intervalo, avisa o usuário e usa `continue` para pedir outro palpite. Depois disso, comparamos `guess` com o número secreto sabendo que está entre 1 e 100.

Não é ideal: se apenas valores de 1 a 100 fossem aceitos em todo o programa e muitas funções tivessem esse requisito, repetir essa checagem em cada uma seria tedioso (e poderia afetar desempenho).

Melhor: um tipo num módulo dedicado, com validação numa função de construção — em vez de repetir a checagem em todo lugar. Funções passam a usar o tipo na assinatura e confiam nos valores recebidos. A Listagem 9-13 define `Guess`, criado só quando `new` recebe valor entre 1 e 100.

**Arquivo: src/guessing_game.rs**

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

<a id="listagem-9-13"></a>

[Listagem 9-13](#listagem-9-13): Tipo `Guess` que só aceita valores entre 1 e 100

Esse código em _src/guessing_game.rs_ exige `mod guessing_game;` em _src/lib.rs_ (não mostrado aqui). No módulo, `Guess` tem campo privado `value: i32`, onde o número fica armazenado.

Implementamos a função associada `new`, com parâmetro `value: i32` e retorno `Guess`. O corpo testa se `value` está entre 1 e 100. Se não estiver, `panic!` — sinalizando ao autor do código chamador um bug: criar `Guess` fora do intervalo viola o contrato de `Guess::new`. Documente na API pública quando `Guess::new` pode entrar em pânico; convenções de documentação para `panic!` aparecem no Capítulo 14. Se passar no teste, retornamos `Guess { value }`.

Depois, o método `value` empresta `self` e devolve `i32` — um _getter_ para expor o campo privado. `value` precisa ser privado: quem usa `Guess` não pode atribuir diretamente. Fora do módulo `guessing_game`, só `Guess::new` cria instâncias — garantindo que nenhum `Guess` escapa sem validação.

Funções que recebem ou devolvem apenas números de 1 a 100 podem usar `Guess` na assinatura em vez de `i32`, sem checagens extras no corpo.

## Resumo

O tratamento de erros em Rust ajuda a escrever código mais robusto. `panic!` indica estado que o programa não consegue lidar e encerra o processo em vez de seguir com valores inválidos. `Result` usa o sistema de tipos para marcar falhas das quais dá para se recuperar — e avisa quem chama que precisa tratar sucesso ou falha. Usar `panic!` e `Result` nos contextos certos deixa o código mais confiável diante de problemas inevitáveis.

Já vimos como a biblioteca padrão usa generics com `Option` e `Result`. A seguir, como generics funcionam e como usá-los no seu código.
