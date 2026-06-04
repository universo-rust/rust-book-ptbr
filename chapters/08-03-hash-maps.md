---
title: "Hash maps"
chapter_code: 08-03
slug: hash-maps
challenge_day: 10
reading_minutes: 13
---

# Armazenando Chaves com Valores Associados em Hash Maps

A última das nossas coleções comuns é o hash map. O tipo `HashMap<K, V>` guarda um mapeamento de chaves do tipo `K` para valores do tipo `V`, usando uma _função de hash_ que define como esses pares ficam na memória. Muitas linguagens têm estruturas parecidas, mas com nomes diferentes — _hash_, _map_, _object_, _hash table_, _dictionary_, _array associativo_, entre outros.

Hash maps são úteis quando você quer buscar dados não por índice (como em vetores), mas por uma chave de qualquer tipo. Em um jogo, por exemplo, dá para guardar a pontuação de cada equipe num hash map: a chave é o nome da equipe e o valor é a pontuação. Com o nome em mãos, você recupera a pontuação.

Nesta seção, veremos a API básica de hash maps — mas há muito mais nas funções que a biblioteca padrão define em `HashMap<K, V>`. Como sempre, consulte a documentação para saber mais.

### Criando um novo hash map

Uma forma de criar um hash map vazio é chamar `new` e adicionar elementos com `insert`. Na Listagem 8-20, acompanhamos a pontuação de duas equipes, _Blue_ e _Yellow_: Blue começa com 10 pontos e Yellow com 50.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<a id="listagem-8-20"></a>

[Listagem 8-20](#listagem-8-20): Criando um hash map e inserindo chaves e valores

Repare que precisamos importar `HashMap` com `use`, do módulo `collections` da biblioteca padrão. Entre as três coleções comuns, esta é a menos usada — por isso não entra no _prelude_, que traz itens automaticamente para o escopo. Hash maps também têm menos suporte integrado: não existe macro embutida para construí-los, por exemplo.

Como vetores, hash maps armazenam dados na heap. Este `HashMap` tem chaves `String` e valores `i32`. Também como vetores, hash maps são homogêneos: todas as chaves devem ser do mesmo tipo, e todos os valores também.

### Acessando valores em um hash map

Para obter um valor, passamos a chave ao método `get`, como na Listagem 8-21.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);
```

<a id="listagem-8-21"></a>

[Listagem 8-21](#listagem-8-21): Acessando a pontuação da equipe Blue no hash map

Aqui, `score` recebe o valor associado à equipe Blue — `10`. O método `get` retorna um `Option<&V>`; se não houver valor para aquela chave, retorna `None`. O programa trata o `Option` com `copied`, obtendo `Option<i32>` em vez de `Option<&i32>`, e depois `unwrap_or`, definindo `score` como zero quando `scores` não tem entrada para a chave.

Dá para iterar sobre cada par chave-valor com um loop `for`, como em vetores:

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
}
```

Esse código imprime cada par em ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

### Gerenciando ownership em hash maps

Para tipos que implementam a trait `Copy`, como `i32`, os valores são copiados para o hash map. Para tipos com ownership, como `String`, os valores são movidos — e o hash map passa a ser dono deles, como na Listagem 8-22.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

<a id="listagem-8-22"></a>

[Listagem 8-22](#listagem-8-22): Chaves e valores passam a pertencer ao hash map depois de inseridos

Depois do `insert`, não dá para usar `field_name` e `field_value` — elas foram movidas para o hash map.

Se inserirmos referências, os valores não são movidos. Porém, o que as referências apontam precisa permanecer válido enquanto o hash map existir. Veremos isso com mais detalhe em [Validando Referências com Lifetimes](/livro/cap10-03-validando-referencias-com-lifetimes), no Capítulo 10.

### Atualizando um hash map

O número de pares chave-valor pode crescer, mas cada chave única só pode ter um valor por vez (o contrário não vale: Blue e Yellow podem ter o mesmo valor `10` no hash map `scores`).

Ao alterar dados num hash map, você precisa decidir o que fazer quando a chave já tem valor. Pode substituir o antigo pelo novo, descartando o anterior. Pode manter o antigo e ignorar o novo — inserindo só se a chave _não_ tiver valor. Ou pode combinar os dois. Vejamos cada caso.

#### Sobrescrevendo um valor

Se inserirmos uma chave com um valor e depois inserirmos a mesma chave com outro valor, o valor associado é substituído. Na Listagem 8-23, `insert` é chamado duas vezes, mas o hash map fica com um único par — ambas as vezes para a chave da equipe Blue.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{scores:?}");
```

<a id="listagem-8-23"></a>

[Listagem 8-23](#listagem-8-23): Substituindo o valor de uma chave

Esse código imprime `{"Blue": 25}`. O valor original, `10`, foi sobrescrito.

#### Adicionando chave e valor só se a chave não existir

É comum verificar se uma chave já tem valor e agir assim: se existir, manter o valor atual; se não existir, inserir chave e valor.

Hash maps têm a API `entry` para isso — ela recebe a chave a verificar. `entry` retorna o enum `Entry`, que representa um valor que pode ou não existir. Suponha que queremos checar se a equipe Yellow tem pontuação; se não tiver, inserir `50` — e fazer o mesmo para Blue. Com a API `entry`, o código fica como na Listagem 8-24.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{scores:?}");
```

<a id="listagem-8-24"></a>

[Listagem 8-24](#listagem-8-24): Usando `entry` para inserir só se a chave ainda não tiver valor

`or_insert` em `Entry` retorna uma referência mutável ao valor da chave, se ela existir; caso contrário, insere o parâmetro como novo valor e retorna referência mutável a ele. Essa abordagem é mais limpa do que escrever a lógica na mão — e convive melhor com o borrow checker.

Executar a Listagem 8-24 imprime `{"Yellow": 50, "Blue": 10}`. A primeira chamada a `entry` insere Yellow com `50`, porque ainda não havia valor. A segunda não altera nada: Blue já tinha `10`.

#### Atualizando um valor com base no antigo

Outro uso frequente é ler o valor de uma chave e atualizá-lo a partir do anterior. A Listagem 8-25 conta quantas vezes cada palavra aparece num texto: palavras são chaves e o valor é incrementado a cada ocorrência. Na primeira vez que vemos uma palavra, inserimos `0`.

**Arquivo: src/main.rs**

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{map:?}");
```

<a id="listagem-8-25"></a>

[Listagem 8-25](#listagem-8-25): Contando ocorrências de palavras com um hash map

Esse código imprime `{"world": 2, "hello": 1, "wonderful": 1}`. A ordem pode variar — lembre que iterar sobre um hash map é arbitrário, como vimos em Acessando valores em um hash map.

`split_whitespace` retorna um iterador sobre subslices de `text`, separadas por espaço em branco. `or_insert` retorna uma referência mutável (`&mut V`) ao valor da chave. Guardamos essa referência em `count`; para alterar o valor, desreferenciamos `count` com `*`. A referência mutável sai de escopo no fim do loop `for`, então as alterações respeitam as regras de borrowing.

### Funções de hash

Por padrão, `HashMap` usa _SipHash_, que oferece resistência a ataques de negação de serviço (DoS) em tabelas hash. Não é o algoritmo mais rápido, mas a troca — um pouco menos de desempenho por mais segurança — costuma valer a pena. Se, ao analisar o desempenho do código, a função padrão for lenta demais, dá para trocar especificando outro hasher. Um _hasher_ é um tipo que implementa a trait `BuildHasher`. Veremos traits e implementação no Capítulo 10. Não é preciso implementar um hasher do zero: em [crates.io](https://crates.io/) há crates com hashers para vários algoritmos comuns.

## Resumo

Vetores, strings e hash maps cobrem boa parte do que você precisa para armazenar, acessar e modificar dados. Alguns exercícios que você já pode encarar:

1. Dada uma lista de inteiros, use um vetor e retorne a mediana (valor na posição central, com a lista ordenada) e a moda (valor mais frequente — um hash map ajuda) da lista.
2. Converta strings para Pig Latin. A primeira consoante de cada palavra vai para o final e se acrescenta _ay_; _first_ vira _irst-fay_. Palavras que começam com vogal recebem _hay_ no final (_apple_ vira _apple-hay_). Lembre dos detalhes de UTF-8!
3. Com hash map e vetores, crie uma interface de texto para adicionar funcionários a departamentos — por exemplo, “Add Sally to Engineering” ou “Add Amir to Sales”. Depois, permita listar todos de um departamento ou todos da empresa por departamento, em ordem alfabética.

A documentação da API da biblioteca padrão descreve métodos úteis em vetores, strings e hash maps para esses exercícios.

Os programas estão ficando mais complexos — e operações podem falhar. É hora de falar sobre tratamento de erros. Vamos a isso em seguida!
