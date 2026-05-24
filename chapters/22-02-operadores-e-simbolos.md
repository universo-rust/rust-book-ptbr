---
title: "Operadores e símbolos"
chapter_code: 22-02
slug: operadores-e-simbolos
---

# Apêndice B: Operadores e símbolos

Este apêndice contém um glossário da sintaxe do Rust, incluindo operadores e outros símbolos que aparecem sozinhos ou no contexto de caminhos, genéricos, trait bounds, macros, atributos, comentários, tuplas e colchetes.

### Operadores

A Tabela B-1 contém os operadores em Rust, um exemplo de como o operador apareceria em contexto, uma breve explicação e se esse operador é sobrecarregável. Se um operador for sobrecarregável, a trait relevante para sobrecarregar esse operador está listada.

| Operador | Exemplo | Explicação | Sobrecarregável? |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!` | `ident!(...)`, `ident!{...}`, `ident![...]` | Expansão de macro | |
| `!` | `!expr` | Complemento bit a bit ou lógico | `Not` |
| `!=` | `expr != expr` | Comparação de desigualdade | `PartialEq` |
| `%` | `expr % expr` | Resto aritmético | `Rem` |
| `%=` | `var %= expr` | Resto aritmético e atribuição | `RemAssign` |
| `&` | `&expr`, `&mut expr` | Borrow | |
| `&` | `&type`, `&mut type`, `&'a type`, `&'a mut type` | Tipo de ponteiro emprestado | |
| `&` | `expr & expr` | AND bit a bit | `BitAnd` |
| `&=` | `var &= expr` | AND bit a bit e atribuição | `BitAndAssign` |
| `&&` | `expr && expr` | AND lógico de curto-circuito | |
| `*` | `expr * expr` | Multiplicação aritmética | `Mul` |
| `*=` | `var *= expr` | Multiplicação aritmética e atribuição | `MulAssign` |
| `*` | `*expr` | Desreferência | `Deref` |
| `*` | `*const type`, `*mut type` | Ponteiro bruto | |
| `+` | `trait + trait`, `'a + trait` | Restrição de tipo composto | |
| `+` | `expr + expr` | Adição aritmética | `Add` |
| `+=` | `var += expr` | Adição aritmética e atribuição | `AddAssign` |
| `,` | `expr, expr` | Separador de argumentos e elementos | |
| `-` | `- expr` | Negação aritmética | `Neg` |
| `-` | `expr - expr` | Subtração aritmética | `Sub` |
| `-=` | `var -= expr` | Subtração aritmética e atribuição | `SubAssign` |
| `->` | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipo de retorno de função e closure | |
| `.` | `expr.ident` | Acesso a campo | |
| `.` | `expr.ident(expr, ...)` | Chamada de método | |
| `.` | `expr.0`, `expr.1`, e assim por diante | Indexação de tupla | |
| `..` | `..`, `expr..`, `..expr`, `expr..expr` | Literal de intervalo com extremidade direita exclusiva | `PartialOrd` |
| `..=` | `..=expr`, `expr..=expr` | Literal de intervalo com extremidade direita inclusiva | `PartialOrd` |
| `..` | `..expr` | Sintaxe de atualização de struct literal | |
| `..` | `variant(x, ..)`, `struct_type { x, .. }` | Binding de padrão “e o resto” | |
| `...` | `expr...expr` | (Obsoleto; use `..=` em vez disso) Em um padrão: padrão de intervalo inclusivo | |
| `/` | `expr / expr` | Divisão aritmética | `Div` |
| `/=` | `var /= expr` | Divisão aritmética e atribuição | `DivAssign` |
| `:` | `pat: type`, `ident: type` | Restrições | |
| `:` | `ident: expr` | Inicializador de campo de struct | |
| `:` | `'a: loop {...}` | Rótulo de loop | |
| `;` | `expr;` | Terminador de instrução e item | |
| `;` | `[...; len]` | Parte da sintaxe de array de tamanho fixo | |
| `<<` | `expr << expr` | Deslocamento à esquerda | `Shl` |
| `<<=` | `var <<= expr` | Deslocamento à esquerda e atribuição | `ShlAssign` |
| `<` | `expr < expr` | Comparação menor que | `PartialOrd` |
| `<=` | `expr <= expr` | Comparação menor ou igual a | `PartialOrd` |
| `=` | `var = expr`, `ident = type` | Atribuição/equivalência | |
| `==` | `expr == expr` | Comparação de igualdade | `PartialEq` |
| `=>` | `pat => expr` | Parte da sintaxe de braço de match | |
| `>` | `expr > expr` | Comparação maior que | `PartialOrd` |
| `>=` | `expr >= expr` | Comparação maior ou igual a | `PartialOrd` |
| `>>` | `expr >> expr` | Deslocamento à direita | `Shr` |
| `>>=` | `var >>= expr` | Deslocamento à direita e atribuição | `ShrAssign` |
| `@` | `ident @ pat` | Binding de padrão | |
| `^` | `expr ^ expr` | OR exclusivo bit a bit | `BitXor` |
| `^=` | `var ^= expr` | OR exclusivo bit a bit e atribuição | `BitXorAssign` |
| <code>&vert;</code> | <code>pat &vert; pat</code> | Alternativas de padrão | |
| <code>&vert;</code> | <code>expr &vert; expr</code> | OR bit a bit | `BitOr` |
| <code>&vert;=</code> | <code>var &vert;= expr</code> | OR bit a bit e atribuição | `BitOrAssign` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | OR lógico de curto-circuito | |
| `?` | `expr?` | Propagação de erro | |

[Tabela B-1](#tabela-b-1): Operadores

### Símbolos que não são operadores

As tabelas a seguir contêm todos os símbolos que não funcionam como operadores; ou seja, não se comportam como uma chamada de função ou método.

A Tabela B-2 mostra símbolos que aparecem sozinhos e são válidos em uma variedade de locais.

| Símbolo | Explicação |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident` | Lifetime nomeado ou rótulo de loop |
| Dígitos imediatamente seguidos por `u8`, `i32`, `f64`, `usize`, e assim por diante | Literal numérico de tipo específico |
| `"..."` | Literal de string |
| `r"..."`, `r#"..."#`, `r##"..."##`, e assim por diante | Literal de string bruta; caracteres de escape não são processados |
| `b"..."` | Literal de byte string; constrói um array de bytes em vez de uma string |
| `br"..."`, `br#"..."#`, `br##"..."##`, e assim por diante | Literal de byte string bruta; combinação de literal de string bruta e byte string |
| `'...'` | Literal de caractere |
| `b'...'` | Literal de byte ASCII |
| <code>&vert;...&vert; expr</code> | Closure |
| `!` | Tipo bottom sempre vazio para funções divergentes |
| `_` | Binding de padrão “ignorado”; também usado para tornar literais inteiros legíveis |

[Tabela B-2](#tabela-b-2): Sintaxe isolada

A Tabela B-3 mostra símbolos que aparecem no contexto de um caminho pela hierarquia de módulos até um item.

| Símbolo | Explicação |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------|
| `ident::ident` | Caminho de namespace |
| `::path` | Caminho relativo à raiz do crate (ou seja, um caminho explicitamente absoluto) |
| `self::path` | Caminho relativo ao módulo atual (ou seja, um caminho explicitamente relativo) |
| `super::path` | Caminho relativo ao pai do módulo atual |
| `type::ident`, `<type as trait>::ident` | Constantes, funções e tipos associados |
| `<type>::...` | Item associado para um tipo que não pode ser nomeado diretamente (por exemplo, `<&T>::...`, `<[T]>::...`, e assim por diante) |
| `trait::method(...)` | Desambiguação de chamada de método nomeando a trait que o define |
| `type::method(...)` | Desambiguação de chamada de método nomeando o tipo para o qual está definido |
| `<type as trait>::method(...)` | Desambiguação de chamada de método nomeando a trait e o tipo |

[Tabela B-3](#tabela-b-3): Sintaxe relacionada a caminhos

A Tabela B-4 mostra símbolos que aparecem no contexto do uso de parâmetros de tipo genérico.

| Símbolo | Explicação |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>` | Especifica parâmetros para um tipo genérico em um tipo (por exemplo, `Vec<u8>`) |
| `path::<...>`, `method::<...>` | Especifica parâmetros para um tipo genérico, função ou método em uma expressão; frequentemente chamado de _turbofish_ (por exemplo, `"42".parse::<i32>()`) |
| `fn ident<...> ...` | Define função genérica |
| `struct ident<...> ...` | Define estrutura genérica |
| `enum ident<...> ...` | Define enumeração genérica |
| `impl<...> ...` | Define implementação genérica |
| `for<...> type` | Limites de lifetime de ordem superior |
| `type<ident=type>` | Um tipo genérico em que um ou mais tipos associados têm atribuições específicas (por exemplo, `Iterator<Item=T>`) |

[Tabela B-4](#tabela-b-4): Genéricos

A Tabela B-5 mostra símbolos que aparecem no contexto de restringir parâmetros de tipo genérico com trait bounds.

| Símbolo | Explicação |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U` | Parâmetro genérico `T` restringido a tipos que implementam `U` |
| `T: 'a` | Tipo genérico `T` deve sobreviver ao lifetime `'a` (ou seja, o tipo não pode conter transitivamente referências com lifetimes mais curtos que `'a`) |
| `T: 'static` | Tipo genérico `T` não contém referências emprestadas além das `'static` |
| `'b: 'a` | Lifetime genérico `'b` deve sobreviver ao lifetime `'a` |
| `T: ?Sized` | Permite que o parâmetro de tipo genérico seja um tipo de tamanho dinâmico |
| `'a + trait`, `trait + trait` | Restrição de tipo composto |

[Tabela B-5](#tabela-b-5): Restrições de trait bound

A Tabela B-6 mostra símbolos que aparecem no contexto de chamar ou definir macros e especificar atributos em um item.

| Símbolo | Explicação |
| ------------------------------------------- | ------------------ |
| `#[meta]` | Atributo externo |
| `#![meta]` | Atributo interno |
| `$ident` | Substituição de macro |
| `$ident:kind` | Metavariável de macro |
| `$(...)...` | Repetição de macro |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Invocação de macro |

[Tabela B-6](#tabela-b-6): Macros e atributos

A Tabela B-7 mostra símbolos que criam comentários.

| Símbolo | Explicação |
| ---------- | ----------------------- |
| `//` | Comentário de linha |
| `//!` | Comentário de documentação de linha interno |
| `///` | Comentário de documentação de linha externo |
| `/*...*/` | Comentário de bloco |
| `/*!...*/` | Comentário de documentação de bloco interno |
| `/**...*/` | Comentário de documentação de bloco externo |

[Tabela B-7](#tabela-b-7): Comentários

A Tabela B-8 mostra os contextos em que parênteses são usados.

| Símbolo | Explicação |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()` | Tupla vazia (também chamada de unit), tanto literal quanto tipo |
| `(expr)` | Expressão entre parênteses |
| `(expr,)` | Expressão de tupla de um elemento |
| `(type,)` | Tipo de tupla de um elemento |
| `(expr, ...)` | Expressão de tupla |
| `(type, ...)` | Tipo de tupla |
| `expr(expr, ...)` | Expressão de chamada de função; também usada para inicializar `struct`s tupla e variantes de `enum` tupla |

[Tabela B-8](#tabela-b-8): Parênteses

A Tabela B-9 mostra os contextos em que chaves são usadas.

| Contexto | Explicação |
| ------------ | ---------------- |
| `{...}` | Expressão de bloco |
| `Type {...}` | Struct literal |

[Tabela B-9](#tabela-b-9): Chaves

A Tabela B-10 mostra os contextos em que colchetes são usados.

| Contexto | Explicação |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]` | Literal de array |
| `[expr; len]` | Literal de array contendo `len` cópias de `expr` |
| `[type; len]` | Tipo de array contendo `len` instâncias de `type` |
| `expr[expr]` | Indexação de coleção; sobrecarregável (`Index`, `IndexMut`) |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexação de coleção simulando fatiamento de coleção, usando `Range`, `RangeFrom`, `RangeTo` ou `RangeFull` como o “índice” |

[Tabela B-10](#tabela-b-10): Colchetes
