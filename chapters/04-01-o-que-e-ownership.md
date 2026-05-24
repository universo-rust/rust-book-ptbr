---
title: "O que é ownership?"
chapter_code: 04-01
slug: o-que-e-ownership
---

# O que é ownership?

_Ownership_ é um conjunto de regras que governa como um programa em Rust gerencia a memória. Todos os programas precisam gerenciar a forma como utilizam a memória do computador enquanto estão em execução. Algumas linguagens possuem coletor de lixo (garbage collector), que procura regularmente por memória que não está mais sendo usada durante a execução do programa; em outras linguagens, o programador deve alocar e liberar a memória explicitamente. Rust utiliza uma terceira abordagem: a memória é gerenciada por meio de um sistema de ownership (posse) com um conjunto de regras que o compilador verifica. Se qualquer uma dessas regras for violada, o programa não será compilado. Nenhuma das funcionalidades do sistema de ownership deixa o programa mais lento durante a execução.

Como ownership é um conceito novo para muitos programadores, leva algum tempo para se acostumar com ele. A boa notícia é que, quanto mais experiência você adquire com Rust e com as regras do sistema de ownership, mais fácil se torna desenvolver naturalmente código que seja seguro e eficiente. Continue praticando!

Quando você entende ownership, passa a ter uma base sólida para compreender as funcionalidades que tornam Rust único. Neste capítulo, você aprenderá ownership trabalhando com alguns exemplos que focam em uma estrutura de dados muito comum: _strings_.

> ### A _stack_ e a _heap_
>
> Muitas linguagens de programação não exigem que você pense com frequência sobre a _stack_ e a _heap_. Porém, em uma linguagem de programação de sistemas como Rust, o fato de um valor estar na stack ou na heap afeta o comportamento da linguagem e explica por que certas decisões precisam ser tomadas. Partes do sistema de ownership serão descritas em relação à stack e à heap mais adiante neste capítulo, então aqui vai uma breve explicação como preparação.
>
> Tanto a stack quanto a heap são partes da memória disponíveis para o seu código usar em tempo de execução, mas elas são estruturadas de formas diferentes. A stack armazena valores na ordem em que os recebe e remove os valores na ordem inversa. Isso é conhecido como _last in, first out (LIFO)_, ou "último a entrar, primeiro a sair". Pense em uma pilha de pratos: quando você adiciona mais pratos, coloca-os no topo da pilha, e quando precisa de um prato, tira um do topo. Adicionar ou remover pratos do meio ou da base não funcionaria tão bem! Adicionar dados é chamado de _push_ na stack, e remover dados é chamado de _pop_ da stack. Todos os dados armazenados na stack precisam ter um tamanho conhecido e fixo. Dados com tamanho desconhecido em tempo de compilação ou cujo tamanho pode mudar devem ser armazenados na heap.
>
> A heap é menos organizada: quando você coloca dados na heap, você solicita uma certa quantidade de espaço. O alocador de memória encontra um local vazio na heap que seja grande o suficiente, marca esse espaço como estando em uso e retorna um ponteiro, que é o endereço dessa localização. Esse processo é chamado de alocação na heap e, muitas vezes, é abreviado simplesmente como alocação (push de valores para a stack não é considerado alocação). Como o ponteiro para a heap tem um tamanho conhecido e fixo, você pode armazenar esse ponteiro na stack; porém, quando quiser acessar os dados reais, será necessário seguir o ponteiro. Pense em estar sendo acomodado em um restaurante: ao entrar, você informa o número de pessoas do seu grupo, e o anfitrião encontra uma mesa vazia que comporte todos e o conduz até ela. Se alguém do seu grupo chegar atrasado, essa pessoa pode perguntar onde vocês estão sentados para encontrá-los.
>
> Colocar dados na stack é mais rápido do que alocar na heap, porque o alocador nunca precisa procurar um local para armazenar novos dados; esse local está sempre no topo da stack. Em comparação, alocar espaço na heap exige mais trabalho, pois o alocador primeiro precisa encontrar um espaço grande o suficiente para conter os dados e, depois, realizar tarefas de controle para preparar a próxima alocação.
>
> Acessar dados na heap geralmente é mais lento do que acessar dados na stack, porque é necessário seguir um ponteiro para chegar até eles. Processadores modernos são mais rápidos quando precisam "pular" menos pela memória. Continuando a analogia, imagine um garçom em um restaurante anotando pedidos de várias mesas. É mais eficiente pegar todos os pedidos de uma mesa antes de ir para a próxima. Anotar um pedido da mesa A, depois um da mesa B, depois outro da A e outro da B seria um processo bem mais lento. Da mesma forma, um processador normalmente consegue executar melhor seu trabalho quando lida com dados que estão próximos uns dos outros (como acontece na stack), em vez de dados mais distantes (como pode ocorrer na heap).
>
> Quando o seu código chama uma função, os valores passados para essa função (incluindo, potencialmente, ponteiros para dados na heap) e as variáveis locais da função são colocados na stack. Quando a função termina, esses valores são removidos da stack.
>
> Acompanhar quais partes do código estão usando quais dados na heap, minimizar a quantidade de dados duplicados na heap e limpar dados não utilizados na heap para que você não fique sem espaço são todos problemas que o sistema de ownership resolve. Depois que você entende ownership, não precisará pensar na stack e na heap com muita frequência. Ainda assim, saber que o principal objetivo do ownership é gerenciar dados na heap ajuda a explicar por que ele funciona da maneira que funciona.

## Regras de ownership

Primeiro, vamos dar uma olhada nas regras de ownership. Mantenha estas regras em mente enquanto trabalhamos com os exemplos que as ilustram:

- Cada valor em Rust tem um _dono_ (_owner_).
- Só pode haver um dono por vez.
- Quando o dono sai de escopo, o valor será descartado (_dropped_).

## Escopo de variáveis

Agora que já passamos pela sintaxe básica de Rust, não incluiremos todo o código `fn main() {` nos exemplos; se você estiver acompanhando, certifique-se de colocar os exemplos a seguir dentro de uma função `main` manualmente. Assim, nossos exemplos ficarão um pouco mais concisos, permitindo focar nos detalhes reais em vez de código repetitivo.

Como primeiro exemplo de ownership, vamos olhar o escopo de algumas variáveis. Um _escopo_ é o intervalo dentro de um programa no qual um item é válido. Considere a variável a seguir:

```rust
let s = "hello";
```

A variável `s` se refere a um literal de string, em que o valor da string está embutido no texto do programa. A variável é válida a partir do ponto em que é declarada até o fim do escopo atual. O programa a seguir mostra comentários indicando onde a variável `s` seria válida.

**Arquivo: src/main.rs**

```rust
fn main() {
    {                      // s não é válida aqui, pois ainda não foi declarada
        let s = "hello";   // s é válida a partir deste ponto

        // faça algo com s
    }                      // este escopo terminou; s não é mais válida
}
```

[Listagem 4-1](#listagem-4-1): Uma variável e o escopo no qual ela é válida

Em outras palavras, há dois momentos importantes aqui:

- Quando `s` _entra_ em escopo, ela é válida.
- Ela permanece válida até _sair_ de escopo.

Neste ponto, a relação entre escopos e quando as variáveis são válidas é semelhante à de outras linguagens de programação. Agora vamos construir sobre esse entendimento introduzindo o tipo `String`.

## O tipo `String`

Para ilustrar as regras de ownership, precisamos de um tipo de dado mais complexo do que os que cobrimos na seção Tipos de dados do Capítulo 3. Os tipos abordados anteriormente têm tamanho conhecido, podem ser armazenados na stack e removidos da stack quando seu escopo termina, e podem ser copiados de forma rápida e trivial para criar uma nova instância independente se outra parte do código precisar usar o mesmo valor em um escopo diferente. Mas queremos examinar dados armazenados na heap e explorar como o Rust sabe quando limpar esses dados — e o tipo `String` é um ótimo exemplo.

Vamos nos concentrar nas partes de `String` que se relacionam com ownership. Esses aspectos também se aplicam a outros tipos de dados complexos, sejam fornecidos pela biblioteca padrão ou criados por você. Discutiremos aspectos de `String` que não envolvem ownership no Capítulo 8.

Já vimos literais de string, em que um valor de string está embutido no programa. Literais de string são convenientes, mas não servem para todas as situações em que podemos querer usar texto. Um motivo é que eles são imutáveis. Outro é que nem todo valor de string pode ser conhecido quando escrevemos o código: por exemplo, e se quisermos receber entrada do usuário e armazená-la? Para essas situações, Rust tem o tipo `String`. Esse tipo gerencia dados alocados na heap e, portanto, consegue armazenar uma quantidade de texto desconhecida em tempo de compilação. Você pode criar uma `String` a partir de um literal de string usando a função `from`, assim:

```rust
let s = String::from("hello");
```

O operador de duplo dois-pontos `::` permite colocar essa função `from` particular no namespace do tipo `String`, em vez de usar algum nome como `string_from`. Discutiremos essa sintaxe com mais detalhes na seção Métodos do Capítulo 5 e quando falarmos de namespaces com módulos no Capítulo 7.

Esse tipo de string _pode_ ser mutado:

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() acrescenta um literal a uma String

    println!("{s}"); // isso imprimirá `hello, world!`
}
```

Então, qual é a diferença aqui? Por que `String` pode ser mutada, mas literais não? A diferença está em como esses dois tipos lidam com a memória.

## Memória e alocação

No caso de um literal de string, sabemos o conteúdo em tempo de compilação, então o texto é embutido diretamente no executável final. É por isso que literais de string são rápidos e eficientes. Mas essas propriedades vêm apenas da imutabilidade do literal. Infelizmente, não podemos colocar um bloco de memória no binário para cada trecho de texto cujo tamanho é desconhecido em tempo de compilação e cujo tamanho pode mudar enquanto o programa está em execução.

Com o tipo `String`, para suportar um trecho de texto mutável e expansível, precisamos alocar na heap uma quantidade de memória desconhecida em tempo de compilação para armazenar o conteúdo. Isso significa:

- A memória deve ser solicitada ao alocador de memória em tempo de execução.
- Precisamos de uma forma de devolver essa memória ao alocador quando terminarmos de usar nossa `String`.

A primeira parte é feita por nós: quando chamamos `String::from`, sua implementação solicita a memória de que precisa. Isso é praticamente universal em linguagens de programação.

Porém, a segunda parte é diferente. Em linguagens com _coletor de lixo (GC)_, o GC acompanha e limpa memória que não está mais sendo usada, e não precisamos nos preocupar com isso. Na maioria das linguagens sem GC, é nossa responsabilidade identificar quando a memória não está mais sendo usada e chamar código para liberá-la explicitamente, assim como fizemos para solicitá-la. Fazer isso corretamente historicamente foi um problema difícil de programação. Se esquecermos, desperdiçamos memória. Se fizermos cedo demais, teremos uma variável inválida. Se fizermos duas vezes, isso também é um bug. Precisamos parear exatamente um `allocate` com exatamente um `free`.

Rust segue um caminho diferente: a memória é devolvida automaticamente quando a variável que é dona dela sai de escopo. Aqui está uma versão do nosso exemplo de escopo da Listagem 4-1 usando uma `String` em vez de um literal de string:

**Arquivo: src/main.rs**

```rust
fn main() {
    {
        let s = String::from("hello"); // s é válida a partir deste ponto

        // faça algo com s
    } // este escopo terminou; s não é mais válida
}
```

Há um ponto natural em que podemos devolver ao alocador a memória que nossa `String` precisa: quando `s` sai de escopo. Quando uma variável sai de escopo, Rust chama uma função especial para nós. Essa função se chama `drop`, e é onde o autor de `String` pode colocar o código para devolver a memória. Rust chama `drop` automaticamente na chave de fechamento.

> **Nota:** Em C++, esse padrão de desalocar recursos no fim da vida útil de um item às vezes é chamado de _Resource Acquisition Is Initialization (RAII)_. A função `drop` em Rust será familiar se você já usou padrões RAII.

Esse padrão tem um impacto profundo na forma como o código Rust é escrito. Pode parecer simples agora, mas o comportamento do código pode ser inesperado em situações mais complicadas, quando queremos que várias variáveis usem os dados que alocamos na heap. Vamos explorar algumas dessas situações agora.

### Variáveis e dados interagindo com _move_

Várias variáveis podem interagir com os mesmos dados de formas diferentes em Rust. O exemplo a seguir usa um inteiro.

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;
    let y = x;
}
```

[Listagem 4-2](#listagem-4-2): Atribuindo o valor inteiro da variável `x` a `y`

Provavelmente podemos adivinhar o que isso faz: "associe o valor `5` a `x`; depois, faça uma cópia do valor em `x` e associe-a a `y`." Agora temos duas variáveis, `x` e `y`, e ambas são iguais a `5`. É exatamente isso que acontece, porque inteiros são valores simples com tamanho conhecido e fixo, e esses dois valores `5` são colocados na stack.

Agora vejamos a versão com `String`:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
}
```

Isso parece muito semelhante, então podemos supor que funciona da mesma forma: ou seja, a segunda linha faria uma cópia do valor em `s1` e a associaria a `s2`. Mas não é exatamente isso que acontece.

Observe a Figura 4-1 para ver o que acontece com `String` por baixo dos panos. Uma `String` é composta por três partes, mostradas à esquerda: um ponteiro para a memória que contém o conteúdo da string, um comprimento (_length_) e uma capacidade (_capacity_). Esse grupo de dados fica armazenado na stack. À direita está a memória na heap que contém o conteúdo.

*Figura 4-1: A representação na memória de uma `String` com o valor `"hello"` associada a `s1`. Duas tabelas: a primeira contém a representação de s1 na stack, com comprimento (5), capacidade (5) e um ponteiro para o primeiro valor da segunda tabela. A segunda tabela contém a representação dos dados da string na heap, byte a byte.*

O comprimento é quanta memória, em bytes, o conteúdo da `String` está usando atualmente. A capacidade é a quantidade total de memória, em bytes, que a `String` recebeu do alocador. A diferença entre comprimento e capacidade importa, mas não neste contexto; por enquanto, podemos ignorar a capacidade.

Quando atribuímos `s1` a `s2`, os dados da `String` são copiados — ou seja, copiamos o ponteiro, o comprimento e a capacidade que estão na stack. Não copiamos os dados na heap aos quais o ponteiro se refere. Em outras palavras, a representação dos dados na memória fica como na Figura 4-2.

*Figura 4-2: A representação na memória da variável `s2`, que tem uma cópia do ponteiro, do comprimento e da capacidade de `s1`. Três tabelas: as tabelas s1 e s2 representam essas strings na stack, respectivamente, e ambas apontam para os mesmos dados da string na heap.*

A representação _não_ fica como na Figura 4-3, que é como a memória seria se Rust copiasse também os dados na heap. Se Rust fizesse isso, a operação `s2 = s1` poderia ser muito custosa em tempo de execução se os dados na heap fossem grandes.

*Figura 4-3: Outra possibilidade do que `s2 = s1` poderia fazer se Rust copiasse também os dados na heap. Quatro tabelas: duas tabelas representam os dados na stack de s1 e s2, e cada uma aponta para sua própria cópia dos dados da string na heap.*

Como dissemos antes, quando uma variável sai de escopo, Rust chama automaticamente a função `drop` e limpa a memória na heap dessa variável. Mas a Figura 4-2 mostra ambos os ponteiros de dados apontando para o mesmo local. Isso é um problema: quando `s2` e `s1` saírem de escopo, ambos tentarão liberar a mesma memória. Isso é conhecido como erro de _double free_ e é um dos bugs de segurança de memória que mencionamos antes. Liberar memória duas vezes pode levar à corrupção de memória, o que pode resultar em vulnerabilidades de segurança.

Para garantir segurança de memória, depois da linha `let s2 = s1;`, Rust considera `s1` como não mais válida. Portanto, Rust não precisa liberar nada quando `s1` sai de escopo. Veja o que acontece quando você tenta usar `s1` depois que `s2` foi criada; não funcionará:

**Arquivo: src/main.rs (Este código não compila!)**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{s1}, world!");
}
```

Você receberá um erro como este, porque o Rust impede que você use a referência invalidada:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:16
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{s1}, world!");
  |                ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let s2 = s1.clone();
  |                ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` (bin "ownership") due to 1 previous error
```

Se você já ouviu os termos _cópia superficial_ (_shallow copy_) e _cópia profunda_ (_deep copy_) em outras linguagens, o conceito de copiar o ponteiro, o comprimento e a capacidade sem copiar os dados provavelmente soa como uma cópia superficial. Mas, como o Rust também invalida a primeira variável, em vez de ser chamada de cópia superficial, isso é conhecido como _move_. Neste exemplo, diríamos que `s1` foi _movida_ (_moved_) para `s2`. O que realmente acontece é mostrado na Figura 4-4.

*Figura 4-4: A representação na memória depois que `s1` foi invalidada. Três tabelas: as tabelas s1 e s2 representam essas strings na stack, respectivamente, e ambas apontam para os mesmos dados da string na heap. A tabela s1 está acinzentada porque s1 não é mais válida; apenas s2 pode ser usada para acessar os dados na heap.*

Isso resolve nosso problema! Com apenas `s2` válida, quando ela sair de escopo liberará a memória sozinha, e pronto.

Além disso, há uma escolha de design implícita: Rust nunca criará automaticamente cópias "profundas" dos seus dados. Portanto, qualquer cópia _automática_ pode ser assumida como barata em termos de desempenho em tempo de execução.

### Escopo e atribuição

O inverso também é verdadeiro para a relação entre escopo, ownership e memória liberada pela função `drop`. Quando você atribui um valor completamente novo a uma variável existente, Rust chamará `drop` e liberará imediatamente a memória do valor original. Por exemplo:

**Arquivo: src/main.rs**

```rust
fn main() {
    let mut s = String::from("hello");
    s = String::from("ahoy");

    println!("{s}, world!");
}
```

Inicialmente declaramos uma variável `s` e a associamos a uma `String` com o valor `"hello"`. Em seguida, criamos imediatamente uma nova `String` com o valor `"ahoy"` e a atribuímos a `s`. Nesse ponto, nada mais se refere ao valor original na heap. A Figura 4-5 ilustra os dados na stack e na heap agora:

*Figura 4-5: A representação na memória depois que o valor inicial foi substituído por completo. Uma tabela representa o valor da string na stack, apontando para o segundo trecho de dados da string (ahoy) na heap, com os dados originais da string (hello) acinzentados porque não podem mais ser acessados.*

A string original assim sai de escopo imediatamente. Rust executa a função `drop` nela e sua memória é liberada na hora. Quando imprimimos o valor no final, será `"ahoy, world!"`.

### Variáveis e dados interagindo com _clone_

Se _quisermos_ copiar profundamente os dados na heap da `String`, e não apenas os dados na stack, podemos usar um método comum chamado `clone`. Discutiremos sintaxe de métodos no Capítulo 5, mas como métodos são um recurso comum em muitas linguagens, você provavelmente já os viu antes.

Aqui está um exemplo do método `clone` em ação:

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {s1}, s2 = {s2}");
}
```

Isso funciona perfeitamente e produz explicitamente o comportamento em que os dados na heap _são_ copiados — cada variável tem sua própria cópia na heap.

Quando você vê uma chamada a `clone`, sabe que algum código arbitrário está sendo executado e que esse código pode ser custoso. É um indicador visual de que algo diferente está acontecendo.

### Dados somente na stack: _Copy_

Há outro detalhe que ainda não discutimos. Este código usando inteiros — parte do qual apareceu na Listagem 4-2 — funciona e é válido:

**Arquivo: src/main.rs**

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("x = {x}, y = {y}");
}
```

Mas este código parece contradizer o que acabamos de aprender: não temos uma chamada a `clone`, mas `x` ainda é válido e não foi movido para `y`.

O motivo é que tipos como inteiros, que têm tamanho conhecido em tempo de compilação, são armazenados inteiramente na stack, então cópias dos valores reais são rápidas de fazer. Isso significa que não há motivo para impedir que `x` continue válido depois de criarmos a variável `y`. Em outras palavras, aqui não há diferença entre cópia profunda e superficial, então chamar `clone` não faria nada diferente da cópia superficial usual, e podemos omiti-la.

Rust tem uma anotação especial chamada trait `Copy` que podemos colocar em tipos armazenados na stack, como os inteiros (falaremos mais sobre traits no Capítulo 10). Se um tipo implementa a trait `Copy`, variáveis que o usam não fazem _move_, mas sim são copiadas trivialmente, permanecendo válidas após atribuição a outra variável.

Rust não nos deixa anotar um tipo com `Copy` se o tipo, ou qualquer parte dele, implementou a trait `Drop`. Se o tipo precisa de algo especial quando o valor sai de escopo e adicionamos a anotação `Copy` a esse tipo, obteremos um erro em tempo de compilação. Para aprender como adicionar a anotação `Copy` ao seu tipo para implementar a trait, veja o [Apêndice C](/livro/cap22-03-traits-derivaveis).

Então, quais tipos implementam a trait `Copy`? Você pode consultar a documentação do tipo em questão para ter certeza, mas, como regra geral, qualquer grupo de valores escalares simples pode implementar `Copy`, e nada que exija alocação ou seja alguma forma de recurso pode implementar `Copy`. Estes são alguns dos tipos que implementam `Copy`:

- Todos os tipos inteiros, como `u32`.
- O tipo booleano `bool`, com os valores `true` e `false`.
- Todos os tipos de ponto flutuante, como `f64`.
- O tipo caractere `char`.
- Tuplas, se contiverem apenas tipos que também implementam `Copy`. Por exemplo, `(i32, i32)` implementa `Copy`, mas `(i32, String)` não.

## Ownership e funções

A mecânica de passar um valor para uma função é semelhante à de atribuir um valor a uma variável. Passar uma variável para uma função fará _move_ ou _copy_, assim como a atribuição. O exemplo a seguir tem anotações mostrando onde as variáveis entram e saem de escopo.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s = String::from("hello");  // s entra em escopo

    takes_ownership(s);             // o valor de s é movido para a função...
                                    // ... e s deixa de ser válida aqui

    let x = 5;                      // x entra em escopo

    makes_copy(x);                  // Como i32 implementa a trait Copy,
                                    // x NÃO é movido para a função,
                                    // então podemos usar x depois.

} // aqui, x sai de escopo, depois s. Porém, como o valor de s foi movido,
  // nada especial acontece.

fn takes_ownership(some_string: String) { // some_string entra em escopo
    println!("{some_string}");
} // aqui, some_string sai de escopo e `drop` é chamado. A memória é liberada.

fn makes_copy(some_integer: i32) { // some_integer entra em escopo
    println!("{some_integer}");
} // aqui, some_integer sai de escopo. Nada especial acontece.
```

[Listagem 4-3](#listagem-4-3): Funções com ownership e escopo anotados

Se tentássemos usar `s` depois da chamada a `takes_ownership`, Rust geraria um erro em tempo de compilação. Essas verificações estáticas nos protegem de erros. Tente adicionar código em `main` que use `s` e `x` para ver onde você pode usá-los e onde as regras de ownership impedem isso.

## Valores de retorno e escopo

Retornar valores também pode transferir ownership. O exemplo a seguir mostra uma função que retorna algum valor, com anotações semelhantes às da Listagem 4-3.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership move seu valor
                                        // de retorno para s1

    let s2 = String::from("hello");     // s2 entra em escopo

    let s3 = takes_and_gives_back(s2); // s2 é movida para
                                        // takes_and_gives_back, que também
                                        // move seu valor de retorno para s3
} // aqui, s3 sai de escopo e é descartada. s2 foi movida, então nada
  // acontece. s1 sai de escopo e é descartada.

fn gives_ownership() -> String { // gives_ownership moverá seu
                                 // valor de retorno para a função chamadora

    let some_string = String::from("yours"); // some_string entra em escopo

    some_string // some_string é retornada e
                // move para a função chamadora
}

// Esta função recebe uma String e retorna uma String.
fn takes_and_gives_back(a_string: String) -> String { // a_string entra
                                                      // em escopo

    a_string // a_string é retornada e move para a função chamadora
}
```

[Listagem 4-4](#listagem-4-4): Transferindo ownership de valores de retorno

O ownership de uma variável segue o mesmo padrão sempre: atribuir um valor a outra variável o move. Quando uma variável que inclui dados na heap sai de escopo, o valor será limpo por `drop`, a menos que o ownership dos dados tenha sido movido para outra variável.

Embora isso funcione, tomar ownership e depois devolver ownership em cada função é um pouco tedioso. E se quisermos deixar uma função usar um valor sem tomar ownership? É bem irritante ter que passar de volta tudo o que passamos, se quisermos usá-lo novamente, além de qualquer dado resultante do corpo da função que também queiramos retornar.

Rust permite retornar vários valores usando uma tupla, como mostra o exemplo a seguir.

**Arquivo: src/main.rs**

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() retorna o comprimento de uma String

    (s, length)
}
```

[Listagem 4-5](#listagem-4-5): Devolvendo ownership dos parâmetros

Mas isso é cerimônia demais e muito trabalho para um conceito que deveria ser comum. Felizmente, Rust tem um recurso para usar um valor sem transferir ownership: referências.
