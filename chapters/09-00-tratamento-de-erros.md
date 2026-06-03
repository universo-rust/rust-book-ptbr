---
title: "Tratamento de erros"
chapter_code: 09-00
slug: tratamento-de-erros
---

# Tratamento de Erros

Erros fazem parte do software — e o Rust oferece vários recursos para quando algo dá errado. Em muitos casos, o compilador exige que você reconheça a possibilidade de erro e tome alguma providência antes de compilar. Esse requisito deixa o programa mais robusto: você descobre e trata falhas antes de levar o código para produção.

O Rust divide erros em duas categorias: recuperáveis e irrecuperáveis. Um _erro recuperável_ — como _arquivo não encontrado_ — em geral basta reportar ao usuário e tentar de novo. _Erros irrecuperáveis_ indicam bug: acessar além do fim de um array, por exemplo. Nesses casos, o programa deve parar na hora.

A maioria das linguagens não separa os dois tipos e trata tudo igual, com exceções ou mecanismos parecidos. Rust não tem exceções. Para erros recuperáveis, há o tipo `Result<T, E>`; para irrecuperáveis, a macro `panic!`, que interrompe a execução. Neste capítulo, começamos com `panic!` e depois vemos como retornar valores `Result<T, E>`. Por fim, discutimos quando tentar se recuperar de um erro e quando encerrar o programa.
