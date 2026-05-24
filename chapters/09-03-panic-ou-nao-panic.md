---
title: "Panic ou não panic"
chapter_code: 09-03
slug: panic-ou-nao-panic
---

# Panic ou Não Panic

Então, como decidir quando deve chamar `panic!` e quando deve retornar `Result`? Quando o código entra em pânico, não há como recuperar. Você poderia chamar `panic!` para qualquer situação de erro, seja possível recuperar ou não, mas então está tomando a decisão de que uma situação é irrecuperável em nome do código chamador. Quando você escolhe retornar um valor `Result`, dá opções ao código chamador. O código chamador poderia escolher tentar recuperar de uma forma apropriada para sua situação, ou poderia decidir que um valor `Err` neste caso é irrecuperável, então pode chamar `panic!` e transformar seu erro recuperável em irrecuperável. Portanto, retornar `Result` é uma boa escolha padrão quando você está definindo uma função que pode falhar.

Em situações como exemplos, código protótipo e testes, é mais apropriado escrever código que entra em pânico em vez de retornar um `Result`. Vamos explorar por quê, depois discutir situações em que o compilador não pode dizer que a falha é impossível, mas você como humano pode. O capítulo concluirá com algumas diretrizes gerais sobre como decidir se deve entrar em pânico em código de biblioteca.

### Exemplos, código protótipo e testes

Quando você está escrevendo um exemplo para ilustrar algum conceito, incluir também código robusto de tratamento de erros pode tornar o exemplo menos claro. Em exemplos, entende-se que uma chamada a um método como `unwrap` que poderia entrar em pânico é um placeholder para a forma como você gostaria que sua aplicação lidasse com erros, que pode diferir com base no que o restante do seu código está fazendo.

Da mesma forma, os métodos `unwrap` e `expect` são muito úteis quando você está prototipando e ainda não está pronto para decidir como lidar com erros. Eles deixam marcadores claros no seu código para quando estiver pronto para tornar seu programa mais robusto.

Se uma chamada de método falhar em um teste, você quer que o teste inteiro falhe, mesmo que esse método não seja a funcionalidade sob teste. Como `panic!` é como um teste é marcado como falha, chamar `unwrap` ou `expect` é exatamente o que deve acontecer.

### Quando você tem mais informação que o compilador

Também seria apropriado chamar `expect` quando você tem alguma outra lógica que garante que o `Result` terá um valor `Ok`, mas a lógica não é algo que o compilador entende. Você ainda terá um valor `Result` que precisa tratar: qualquer operação que você está chamando ainda tem a possibilidade de falhar em geral, mesmo que seja logicamente impossível na sua situação particular. Se você pode garantir inspecionando manualmente o código que nunca terá uma variante `Err`, é perfeitamente aceitável chamar `expect` e documentar o motivo pelo qual acha que nunca terá uma variante `Err` no texto do argumento. Aqui está um exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
}
```

Estamos criando uma instância de `IpAddr` fazendo parse de uma string hardcoded. Podemos ver que `127.0.0.1` é um endereço IP válido, então é aceitável usar `expect` aqui. Porém, ter uma string hardcoded válida não muda o tipo de retorno do método `parse`: ainda obtemos um valor `Result`, e o compilador ainda nos fará lidar com o `Result` como se a variante `Err` fosse uma possibilidade, porque o compilador não é inteligente o suficiente para ver que esta string é sempre um endereço IP válido. Se a string de endereço IP viesse de um usuário em vez de estar hardcoded no programa e portanto _tivesse_ possibilidade de falha, definitivamente quereríamos lidar com o `Result` de forma mais robusta. Mencionar a suposição de que este endereço IP é hardcoded nos lembrará de mudar `expect` para um código de tratamento de erros melhor se, no futuro, precisarmos obter o endereço IP de alguma outra fonte.

### Diretrizes para tratamento de erros

É aconselhável que seu código entre em pânico quando for possível que seu código acabe em um estado ruim. Neste contexto, um _estado ruim_ é quando alguma suposição, garantia, contrato ou invariante foi violado, como quando valores inválidos, contraditórios ou ausentes são passados ao seu código — mais um ou mais dos seguintes:

- O estado ruim é algo inesperado, em oposição a algo que provavelmente acontecerá ocasionalmente, como um usuário inserir dados no formato errado.
- Seu código depois deste ponto precisa confiar em não estar neste estado ruim, em vez de verificar o problema em cada passo.
- Não há uma boa forma de codificar esta informação nos tipos que você usa. Trabalharemos um exemplo do que queremos dizer em Codificando estados e comportamento como tipos no Capítulo 18.

Se alguém chama seu código e passa valores que não fazem sentido, é melhor retornar um erro se puder para que o usuário da biblioteca possa decidir o que quer fazer nesse caso. Porém, em casos em que continuar poderia ser inseguro ou prejudicial, a melhor escolha pode ser chamar `panic!` e alertar a pessoa que usa sua biblioteca sobre o bug no código dela para que possa corrigi-lo durante o desenvolvimento. Da mesma forma, `panic!` é frequentemente apropriado se você está chamando código externo que está fora do seu controle e retorna um estado inválido que você não tem como corrigir.

Porém, quando a falha é esperada, é mais apropriado retornar um `Result` do que fazer uma chamada a `panic!`. Exemplos incluem um parser recebendo dados malformados ou uma requisição HTTP retornando um status que indica que você atingiu um limite de taxa. Nestes casos, retornar um `Result` indica que a falha é uma possibilidade esperada que o código chamador deve decidir como tratar.

Quando seu código realiza uma operação que poderia colocar um usuário em risco se for chamada com valores inválidos, seu código deve verificar se os valores são válidos primeiro e entrar em pânico se não forem. Isso é principalmente por motivos de segurança: tentar operar em dados inválidos pode expor seu código a vulnerabilidades. Esta é a principal razão pela qual a biblioteca padrão chamará `panic!` se você tentar um acesso à memória fora dos limites: tentar acessar memória que não pertence à estrutura de dados atual é um problema de segurança comum. Funções frequentemente têm _contratos_: seu comportamento só é garantido se as entradas atenderem requisitos particulares. Entrar em pânico quando o contrato é violado faz sentido porque uma violação de contrato sempre indica um bug do lado do chamador, e não é um tipo de erro que você quer que o código chamador precise tratar explicitamente. Na verdade, não há forma razoável de o código chamador se recuperar; os _programadores_ chamadores precisam corrigir o código. Contratos de uma função, especialmente quando uma violação causará pânico, devem ser explicados na documentação da API da função.

Porém, ter muitas verificações de erro em todas as suas funções seria verboso e irritante. Felizmente, você pode usar o sistema de tipos do Rust (e assim a verificação de tipos feita pelo compilador) para fazer muitas das verificações por você. Se sua função tem um tipo particular como parâmetro, você pode prosseguir com a lógica do seu código sabendo que o compilador já garantiu que você tem um valor válido. Por exemplo, se você tem um tipo em vez de um `Option`, seu programa espera ter _algo_ em vez de _nada_. Seu código então não precisa lidar com dois casos para as variantes `Some` e `None`: só terá um caso para definitivamente ter um valor. Código tentando passar nada para sua função nem compilará, então sua função não precisa verificar esse caso em tempo de execução. Outro exemplo é usar um tipo inteiro sem sinal como `u32`, que garante que o parâmetro nunca será negativo.

### Tipos personalizados para validação

Vamos levar a ideia de usar o sistema de tipos do Rust para garantir que temos um valor válido um passo adiante e olhar para criar um tipo personalizado para validação. Lembre-se do jogo de adivinhação no Capítulo 2 em que nosso código pedia ao usuário para adivinhar um número entre 1 e 100. Nunca validamos que o palpite do usuário estava entre esses números antes de compará-lo com nosso número secreto; só validamos que o palpite era positivo. Neste caso, as consequências não foram muito graves: nossa saída de “Muito alto” ou “Muito baixo” ainda estaria correta. Mas seria um aprimoramento útil orientar o usuário em direção a palpites válidos e ter comportamento diferente quando o usuário adivinha um número fora do intervalo versus quando digita, por exemplo, letras.

Uma forma de fazer isso seria fazer parse do palpite como `i32` em vez de apenas `u32` para permitir números potencialmente negativos, e então adicionar uma verificação de que o número está no intervalo, assim:

**Arquivo: src/main.rs**

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: i32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < 1 || guess > 100 {
            println!("The secret number will be between 1 and 100.");
            continue;
        }

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

A expressão `if` verifica se nosso valor está fora do intervalo, informa o usuário sobre o problema e chama `continue` para iniciar a próxima iteração do loop e pedir outro palpite. Depois da expressão `if`, podemos prosseguir com as comparações entre `guess` e o número secreto sabendo que `guess` está entre 1 e 100.

Porém, esta não é uma solução ideal: se fosse absolutamente crítico que o programa operasse apenas em valores entre 1 e 100, e tivesse muitas funções com este requisito, ter uma verificação assim em cada função seria tedioso (e poderia impactar o desempenho).

Em vez disso, podemos fazer um novo tipo em um módulo dedicado e colocar as validações em uma função para criar uma instância do tipo em vez de repetir as validações em todos os lugares. Dessa forma, é seguro para funções usarem o novo tipo em suas assinaturas e usar com confiança os valores que recebem. A Listagem 9-13 mostra uma forma de definir um tipo `Guess` que só criará uma instância de `Guess` se a função `new` receber um valor entre 1 e 100.

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

[Listagem 9-13](#listagem-9-13): Um tipo `Guess` que só continuará com valores entre 1 e 100

Observe que este código em _src/guessing_game.rs_ depende de adicionar uma declaração de módulo `mod guessing_game;` em _src/lib.rs_ que não mostramos aqui. Dentro do arquivo deste novo módulo, definimos uma struct chamada `Guess` que tem um campo chamado `value` que contém um `i32`. É aqui que o número será armazenado.

Em seguida, implementamos uma função associada chamada `new` em `Guess` que cria instâncias de valores `Guess`. A função `new` é definida para ter um parâmetro chamado `value` do tipo `i32` e retornar um `Guess`. O código no corpo da função `new` testa `value` para garantir que está entre 1 e 100. Se `value` não passar neste teste, fazemos uma chamada a `panic!`, que alertará o programador que está escrevendo o código chamador de que tem um bug que precisa corrigir, porque criar um `Guess` com um `value` fora deste intervalo violaria o contrato em que `Guess::new` está confiando. As condições em que `Guess::new` pode entrar em pânico devem ser discutidas em sua documentação de API voltada ao público; cobriremos convenções de documentação indicando a possibilidade de `panic!` na documentação da API que você cria no Capítulo 14. Se `value` passar no teste, criamos um novo `Guess` com seu campo `value` definido para o parâmetro `value` e retornamos o `Guess`.

Em seguida, implementamos um método chamado `value` que faz borrow de `self`, não tem outros parâmetros e retorna um `i32`. Este tipo de método às vezes é chamado de _getter_ porque seu propósito é obter alguns dados de seus campos e retorná-los. Este método público é necessário porque o campo `value` da struct `Guess` é privado. É importante que o campo `value` seja privado para que código usando a struct `Guess` não possa definir `value` diretamente: código fora do módulo `guessing_game` _deve_ usar a função `Guess::new` para criar uma instância de `Guess`, garantindo assim que não há forma de um `Guess` ter um `value` que não foi verificado pelas condições na função `Guess::new`.

Uma função que tem um parâmetro ou retorna apenas números entre 1 e 100 poderia então declarar em sua assinatura que recebe ou retorna um `Guess` em vez de um `i32` e não precisaria fazer verificações adicionais em seu corpo.

## Resumo

Os recursos de tratamento de erros do Rust foram projetados para ajudá-lo a escrever código mais robusto. A macro `panic!` sinaliza que seu programa está em um estado que não pode lidar e permite dizer ao processo para parar em vez de tentar prosseguir com valores inválidos ou incorretos. O enum `Result` usa o sistema de tipos do Rust para indicar que operações podem falhar de uma forma da qual seu código poderia se recuperar. Você pode usar `Result` para dizer ao código que chama seu código que precisa lidar com sucesso ou falha potenciais. Usar `panic!` e `Result` nas situações apropriadas tornará seu código mais confiável diante de problemas inevitáveis.

Agora que você viu formas úteis pelas quais a biblioteca padrão usa generics com os enums `Option` e `Result`, falaremos sobre como generics funcionam e como você pode usá-los no seu código.
