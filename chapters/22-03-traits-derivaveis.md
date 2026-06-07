---
title: "Traits deriváveis"
chapter_code: 22-03
slug: traits-derivaveis
---

# Apêndice C: Traits deriváveis

Em vários lugares do livro, discutimos o atributo `derive`, que você pode aplicar a uma definição de struct ou enum. O atributo `derive` gera código que implementará uma trait com sua implementação padrão no tipo que você anotou com a sintaxe `derive`.

Neste apêndice, fornecemos uma referência de todas as traits na biblioteca padrão que você pode usar com `derive`. Cada seção cobre:

- Quais operadores e métodos derivar esta trait habilitará
- O que a implementação da trait fornecida por `derive` faz
- O que implementar a trait significa sobre o tipo
- As condições em que você pode ou não pode implementar a trait
- Exemplos de operações que exigem a trait

Se você quiser comportamento diferente do fornecido pelo atributo `derive`, consulte a [documentação da biblioteca padrão](https://doc.rust-lang.org/std/index.html) de cada trait para detalhes sobre como implementá-las manualmente.

As traits listadas aqui são as únicas definidas pela biblioteca padrão que podem ser implementadas nos seus tipos usando `derive`. Outras traits definidas na biblioteca padrão não têm comportamento padrão sensato, então cabe a você implementá-las da forma que faça sentido para o que você está tentando realizar.

Um exemplo de trait que não pode ser derivada é `Display`, que lida com formatação para usuários finais. Você deve sempre considerar a forma apropriada de exibir um tipo para um usuário final. Quais partes do tipo um usuário final deve poder ver? Quais partes seriam relevantes para ele? Qual formato dos dados seria mais relevante? O compilador Rust não tem essa percepção, então não pode fornecer comportamento padrão apropriado para você.

A lista de traits deriváveis fornecida neste apêndice não é exaustiva: bibliotecas podem implementar `derive` para suas próprias traits, tornando a lista de traits com as quais você pode usar `derive` verdadeiramente aberta. Implementar `derive` envolve usar uma macro procedural, que é abordada na seção [“Macros `derive` customizadas”](/livro/cap20-05-macros#macros-derive-personalizadas) do Capítulo 20.

### `Debug` para saída do programador

A trait `Debug` habilita formatação de depuração em strings de formato, que você indica adicionando `:?` dentro de placeholders `{}`.

A trait `Debug` permite imprimir instâncias de um tipo para fins de depuração, para que você e outros programadores que usem seu tipo possam inspecionar uma instância em um ponto específico da execução de um programa.

A trait `Debug` é necessária, por exemplo, no uso da macro `assert_eq!`. Esta macro imprime os valores das instâncias fornecidas como argumentos se a asserção de igualdade falhar, para que os programadores possam ver por que as duas instâncias não eram iguais.

### `PartialEq` e `Eq` para comparações de igualdade

A trait `PartialEq` permite comparar instâncias de um tipo para verificar igualdade e habilita o uso dos operadores `==` e `!=`.

Derivar `PartialEq` implementa o método `eq`. Quando `PartialEq` é derivada em structs, duas instâncias são iguais apenas se _todos_ os campos forem iguais, e as instâncias não são iguais se _qualquer_ campo não for igual. Quando derivada em enums, cada variante é igual a si mesma e não é igual às outras variantes.

A trait `PartialEq` é necessária, por exemplo, com o uso da macro `assert_eq!`, que precisa poder comparar duas instâncias de um tipo quanto à igualdade.

A trait `Eq` não tem métodos. Seu propósito é sinalizar que, para todo valor do tipo anotado, o valor é igual a si mesmo. A trait `Eq` só pode ser aplicada a tipos que também implementam `PartialEq`, embora nem todos os tipos que implementam `PartialEq` possam implementar `Eq`. Um exemplo disso são tipos de ponto flutuante: a implementação de números de ponto flutuante afirma que duas instâncias do valor not-a-number (`NaN`) não são iguais entre si.

Um exemplo de quando `Eq` é necessária é para chaves em um `HashMap<K, V>`, para que o `HashMap<K, V>` possa dizer se duas chaves são iguais.

### `PartialOrd` e `Ord` para comparações de ordenação

A trait `PartialOrd` permite comparar instâncias de um tipo para fins de ordenação. Um tipo que implementa `PartialOrd` pode ser usado com os operadores `<`, `>`, `<=` e `>=`. Você só pode aplicar a trait `PartialOrd` a tipos que também implementam `PartialEq`.

Derivar `PartialOrd` implementa o método `partial_cmp`, que retorna um `Option<Ordering>` que será `None` quando os valores fornecidos não produzirem uma ordenação. Um exemplo de valor que não produz ordenação, embora a maioria dos valores desse tipo possa ser comparada, é o valor de ponto flutuante `NaN`. Chamar `partial_cmp` com qualquer número de ponto flutuante e o valor de ponto flutuante `NaN` retornará `None`.

Quando derivada em structs, `PartialOrd` compara duas instâncias comparando o valor em cada campo na ordem em que os campos aparecem na definição da struct. Quando derivada em enums, variantes do enum declaradas antes na definição do enum são consideradas menores que as variantes listadas depois.

A trait `PartialOrd` é necessária, por exemplo, para o método `gen_range` do crate `rand`, que gera um valor aleatório no intervalo especificado por uma expressão de intervalo.

A trait `Ord` permite saber que, para quaisquer dois valores do tipo anotado, existirá uma ordenação válida. A trait `Ord` implementa o método `cmp`, que retorna um `Ordering` em vez de um `Option<Ordering>` porque uma ordenação válida sempre será possível. Você só pode aplicar a trait `Ord` a tipos que também implementam `PartialOrd` e `Eq` (e `Eq` exige `PartialEq`). Quando derivada em structs e enums, `cmp` se comporta da mesma forma que a implementação derivada de `partial_cmp` com `PartialOrd`.

Um exemplo de quando `Ord` é necessária é ao armazenar valores em um `BTreeSet<T>`, uma estrutura de dados que armazena dados com base na ordem de classificação dos valores.

### `Clone` e `Copy` para duplicar valores

A trait `Clone` permite criar explicitamente uma cópia profunda de um valor, e o processo de duplicação pode envolver executar código arbitrário e copiar dados na heap. Consulte a seção [“Variáveis e dados interagindo com Clone”](/livro/cap04-01-o-que-e-ownership#variáveis-e-dados-interagindo-com-clone) do Capítulo 4 para mais informações sobre `Clone`.

Derivar `Clone` implementa o método `clone`, que, quando implementado para o tipo inteiro, chama `clone` em cada uma das partes do tipo. Isso significa que todos os campos ou valores no tipo também devem implementar `Clone` para derivar `Clone`.

Um exemplo de quando `Clone` é necessária é ao chamar o método `to_vec` em uma fatia. A fatia não tem ownership das instâncias do tipo que contém, mas o vetor retornado por `to_vec` precisará ter ownership de suas instâncias, então `to_vec` chama `clone` em cada item. Assim, o tipo armazenado na fatia deve implementar `Clone`.

A trait `Copy` permite duplicar um valor apenas copiando bits armazenados na pilha; nenhum código arbitrário é necessário. Consulte a seção [“Dados somente na pilha: Copy”](/livro/cap04-01-o-que-e-ownership#dados-somente-na-pilha-copy) do Capítulo 4 para mais informações sobre `Copy`.

A trait `Copy` não define nenhum método para impedir que programadores sobrecarreguem esses métodos e violem a suposição de que nenhum código arbitrário está sendo executado. Dessa forma, todos os programadores podem assumir que copiar um valor será muito rápido.

Você pode derivar `Copy` em qualquer tipo cujas partes implementem todas `Copy`. Um tipo que implementa `Copy` também deve implementar `Clone`, porque um tipo que implementa `Copy` tem uma implementação trivial de `Clone` que executa a mesma tarefa que `Copy`.

A trait `Copy` raramente é exigida; tipos que implementam `Copy` têm otimizações disponíveis, o que significa que você não precisa chamar `clone`, tornando o código mais conciso.

Tudo que é possível com `Copy` também pode ser feito com `Clone`, mas o código pode ser mais lento ou ter que usar `clone` em alguns lugares.

### `Hash` para mapear um valor a um valor de tamanho fixo

A trait `Hash` permite pegar uma instância de um tipo de tamanho arbitrário e mapear essa instância para um valor de tamanho fixo usando uma função de hash. Derivar `Hash` implementa o método `hash`. A implementação derivada do método `hash` combina o resultado de chamar `hash` em cada uma das partes do tipo, o que significa que todos os campos ou valores também devem implementar `Hash` para derivar `Hash`.

Um exemplo de quando `Hash` é necessária é ao armazenar chaves em um `HashMap<K, V>` para armazenar dados com eficiência.

### `Default` para valores padrão

A trait `Default` permite criar um valor padrão para um tipo. Derivar `Default` implementa a função `default`. A implementação derivada da função `default` chama a função `default` em cada parte do tipo, o que significa que todos os campos ou valores no tipo também devem implementar `Default` para derivar `Default`.

A função `Default::default` é comumente usada em combinação com a sintaxe de atualização de struct discutida na seção [“Criando instâncias com sintaxe de atualização de struct”](/livro/cap05-01-definindo-structs#criando-instâncias-com-sintaxe-de-atualização-de-struct) do Capítulo 5. Você pode personalizar alguns campos de uma struct e então definir e usar um valor padrão para o restante dos campos usando `..Default::default()`.

A trait `Default` é necessária quando você usa o método `unwrap_or_default` em instâncias de `Option<T>`, por exemplo. Se o `Option<T>` for `None`, o método `unwrap_or_default` retornará o resultado de `Default::default` para o tipo `T` armazenado no `Option<T>`.
