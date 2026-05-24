---
title: "Hash maps"
chapter_code: 08-03
slug: hash-maps
---

# Armazenando Chaves com Valores Associados em Hash Maps

A última de nossas coleções comuns é o hash map. O tipo `HashMap<K, V>` armazena um mapeamento de chaves do tipo `K` para valores do tipo `V` usando uma _função de hash_, que determina como coloca essas chaves e valores na memória. Muitas linguagens de programação suportam esse tipo de estrutura de dados, mas frequentemente usam um nome diferente, como _hash_, _map_, _object_, _hash table_, _dictionary_ ou _array associativo_, só para citar alguns.

Hash maps são úteis quando você quer consultar dados não por índice, como pode com vetores, mas usando uma chave que pode ser de qualquer tipo. Por exemplo, em um jogo, você poderia acompanhar a pontuação de cada equipe em um hash map em que cada chave é o nome de uma equipe e os valores são a pontuação de cada equipe. Dado um nome de equipe, você pode recuperar sua pontuação.

Passaremos pela API básica de hash maps nesta seção, mas muitos outros recursos estão escondidos nas funções definidas em `HashMap<K, V>` pela biblioteca padrão. Como sempre, consulte a documentação da biblioteca padrão para mais informações.

### Criando um novo hash map

Uma forma de criar um hash map vazio é usar `new` e adicionar elementos com `insert`. Na Listagem 8-20, estamos acompanhando as pontuações de duas equipes cujos nomes são _Blue_ e _Yellow_. A equipe Blue começa com 10 pontos, e a equipe Yellow começa com 50.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```

[Listagem 8-20](#listagem-8-20): Criando um novo hash map e inserindo algumas chaves e valores

Observe que precisamos primeiro fazer `use` de `HashMap` da parte de coleções da biblioteca padrão. Das nossas três coleções comuns, esta é a menos usada, então não está incluída nos recursos trazidos automaticamente para o escopo no prelude. Hash maps também têm menos suporte da biblioteca padrão; não há macro embutida para construí-los, por exemplo.

Assim como vetores, hash maps armazenam seus dados na heap. Este `HashMap` tem chaves do tipo `String` e valores do tipo `i32`. Como vetores, hash maps são homogêneos: todas as chaves devem ter o mesmo tipo, e todos os valores devem ter o mesmo tipo.

### Acessando valores em um hash map

Podemos obter um valor do hash map fornecendo sua chave ao método `get`, como mostrado na Listagem 8-21.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
}
```

[Listagem 8-21](#listagem-8-21): Acessando a pontuação da equipe Blue armazenada no hash map

Aqui, `score` terá o valor associado à equipe Blue, e o resultado será `10`. O método `get` retorna um `Option<&V>`; se não houver valor para essa chave no hash map, `get` retornará `None`. Este programa lida com o `Option` chamando `copied` para obter um `Option<i32>` em vez de um `Option<&i32>`, e então `unwrap_or` para definir `score` como zero se `scores` não tiver uma entrada para a chave.

Podemos iterar sobre cada par chave-valor em um hash map de forma semelhante a como fazemos com vetores, usando um loop `for`:

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
}
```

Este código imprimirá cada par em uma ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

### Gerenciando ownership em hash maps

Para tipos que implementam a trait `Copy`, como `i32`, os valores são copiados para o hash map. Para valores com ownership, como `String`, os valores serão movidos e o hash map será o dono desses valores, como demonstrado na Listagem 8-22.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name e field_value são inválidos neste ponto; tente usá-los e
    // veja qual erro do compilador você obtém!
}
```

[Listagem 8-22](#listagem-8-22): Mostrando que chaves e valores pertencem ao hash map depois de inseridos

Não podemos usar as variáveis `field_name` e `field_value` depois que foram movidas para o hash map com a chamada a `insert`.

Se inserirmos referências a valores no hash map, os valores não serão movidos para o hash map. Os valores aos quais as referências apontam devem ser válidos por pelo menos o tempo que o hash map for válido. Falaremos mais sobre esses problemas em Validando referências com lifetimes no Capítulo 10.

### Atualizando um hash map

Embora o número de pares chave-valor seja expansível, cada chave única só pode ter um valor associado a ela por vez (mas não vice-versa: por exemplo, tanto a equipe Blue quanto a Yellow poderiam ter o valor `10` armazenado no hash map `scores`).

Quando você quer alterar os dados em um hash map, precisa decidir como lidar com o caso em que uma chave já tem um valor atribuído. Você poderia substituir o valor antigo pelo novo, desconsiderando completamente o valor antigo. Poderia manter o valor antigo e ignorar o novo, adicionando o novo valor apenas se a chave _não_ tiver um valor. Ou poderia combinar o valor antigo e o novo. Vamos ver como fazer cada um desses!

#### Sobrescrevendo um valor

Se inserirmos uma chave e um valor em um hash map e então inserirmos a mesma chave com um valor diferente, o valor associado a essa chave será substituído. Embora o código na Listagem 8-23 chame `insert` duas vezes, o hash map conterá apenas um par chave-valor porque estamos inserindo o valor para a chave da equipe Blue ambas as vezes.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{scores:?}");
}
```

[Listagem 8-23](#listagem-8-23): Substituindo um valor armazenado com uma chave particular

Este código imprimirá `{"Blue": 25}`. O valor original de `10` foi sobrescrito.

#### Adicionando uma chave e valor apenas se a chave não estiver presente

É comum verificar se uma chave particular já existe no hash map com um valor e então tomar as seguintes ações: se a chave existir no hash map, o valor existente deve permanecer como está; se a chave não existir, inseri-la e um valor para ela.

Hash maps têm uma API especial para isso chamada `entry` que recebe a chave que você quer verificar como parâmetro. O valor de retorno do método `entry` é um enum chamado `Entry` que representa um valor que pode ou não existir. Digamos que queremos verificar se a chave da equipe Yellow tem um valor associado. Se não tiver, queremos inserir o valor `50`, e o mesmo para a equipe Blue. Usando a API `entry`, o código fica como na Listagem 8-24.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{scores:?}");
}
```

[Listagem 8-24](#listagem-8-24): Usando o método `entry` para inserir apenas se a chave ainda não tiver um valor

O método `or_insert` em `Entry` é definido para retornar uma referência mutável ao valor para a chave `Entry` correspondente se essa chave existir, e se não, insere o parâmetro como o novo valor para esta chave e retorna uma referência mutável ao novo valor. Esta técnica é muito mais limpa que escrever a lógica nós mesmos e, além disso, funciona melhor com o borrow checker.

Executar o código da Listagem 8-24 imprimirá `{"Yellow": 50, "Blue": 10}`. A primeira chamada a `entry` inserirá a chave da equipe Yellow com o valor `50` porque a equipe Yellow ainda não tem um valor. A segunda chamada a `entry` não alterará o hash map, porque a equipe Blue já tem o valor `10`.

#### Atualizando um valor com base no valor antigo

Outro caso de uso comum para hash maps é consultar o valor de uma chave e então atualizá-lo com base no valor antigo. Por exemplo, a Listagem 8-25 mostra código que conta quantas vezes cada palavra aparece em algum texto. Usamos um hash map com as palavras como chaves e incrementamos o valor para acompanhar quantas vezes vimos aquela palavra. Se for a primeira vez que vemos uma palavra, primeiro inserimos o valor `0`.

**Arquivo: src/main.rs**

```rust
fn main() {
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{map:?}");
}
```

[Listagem 8-25](#listagem-8-25): Contando ocorrências de palavras usando um hash map que armazena palavras e contagens

Este código imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. Você pode ver os mesmos pares chave-valor impressos em uma ordem diferente: lembre-se de que iterar sobre um hash map acontece em uma ordem arbitrária.

O método `split_whitespace` retorna um iterador sobre subslices, separadas por espaço em branco, do valor em `text`. O método `or_insert` retorna uma referência mutável (`&mut V`) ao valor para a chave especificada. Aqui, armazenamos essa referência mutável na variável `count`, então para atribuir a esse valor, precisamos primeiro desreferenciar `count` usando o asterisco (`*`). A referência mutável sai de escopo no final do loop `for`, então todas essas alterações são seguras e permitidas pelas regras de borrowing.

### Funções de hash

Por padrão, `HashMap` usa uma função de hash chamada _SipHash_ que pode fornecer resistência a ataques de negação de serviço (DoS) envolvendo tabelas hash. Este não é o algoritmo de hash mais rápido disponível, mas a troca por melhor segurança que vem com a queda de desempenho vale a pena. Se você fizer profiling do seu código e descobrir que a função de hash padrão é lenta demais para seus propósitos, pode mudar para outra função especificando um hasher diferente. Um _hasher_ é um tipo que implementa a trait `BuildHasher`. Falaremos sobre traits e como implementá-las no Capítulo 10. Você não precisa necessariamente implementar seu próprio hasher do zero; [crates.io](https://crates.io/) tem bibliotecas compartilhadas por outros usuários Rust que fornecem hashers implementando muitos algoritmos de hash comuns.

## Resumo

Vetores, strings e hash maps fornecerão uma grande quantidade de funcionalidade necessária em programas quando você precisar armazenar, acessar e modificar dados. Aqui estão alguns exercícios que você agora está equipado para resolver:

1. Dada uma lista de inteiros, use um vetor e retorne a mediana (quando ordenada, o valor na posição do meio) e a moda (o valor que ocorre com mais frequência; um hash map será útil aqui) da lista.
2. Converta strings para Pig Latin. A primeira consoante de cada palavra é movida para o final da palavra e _ay_ é adicionado, então _first_ vira _irst-fay_. Palavras que começam com vogal têm _hay_ adicionado ao final em vez disso (_apple_ vira _apple-hay_). Tenha em mente os detalhes sobre codificação UTF-8!
3. Usando um hash map e vetores, crie uma interface de texto para permitir que um usuário adicione nomes de funcionários a um departamento em uma empresa; por exemplo, “Add Sally to Engineering” ou “Add Amir to Sales”. Em seguida, deixe o usuário recuperar uma lista de todas as pessoas em um departamento ou todas as pessoas na empresa por departamento, ordenadas alfabeticamente.

A documentação da API da biblioteca padrão descreve métodos que vetores, strings e hash maps têm que serão úteis para esses exercícios!

Estamos entrando em programas mais complexos em que operações podem falhar, então é o momento perfeito para discutir tratamento de erros. Faremos isso a seguir!
