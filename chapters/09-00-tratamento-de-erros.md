---
title: "Tratamento de erros"
chapter_code: 09-00
slug: tratamento-de-erros
---

# Tratamento de Erros

Erros são um fato da vida no software, então o Rust tem vários recursos para lidar com situações em que algo dá errado. Em muitos casos, o Rust exige que você reconheça a possibilidade de um erro e tome alguma ação antes que seu código compile. Esse requisito torna seu programa mais robusto ao garantir que você descobrirá erros e os tratará adequadamente antes de implantar seu código em produção!

O Rust agrupa erros em duas categorias principais: erros recuperáveis e irrecuperáveis. Para um _erro recuperável_, como um erro de _arquivo não encontrado_, provavelmente queremos apenas reportar o problema ao usuário e tentar a operação novamente. _Erros irrecuperáveis_ são sempre sintomas de bugs, como tentar acessar uma localização além do fim de um array, e então queremos parar o programa imediatamente.

A maioria das linguagens não distingue entre esses dois tipos de erros e trata ambos da mesma forma, usando mecanismos como exceções. O Rust não tem exceções. Em vez disso, tem o tipo `Result<T, E>` para erros recuperáveis e a macro `panic!` que interrompe a execução quando o programa encontra um erro irrecuperável. Este capítulo cobre primeiro a chamada a `panic!` e depois fala sobre retornar valores `Result<T, E>`. Além disso, exploraremos considerações ao decidir se tentar recuperar de um erro ou interromper a execução.
