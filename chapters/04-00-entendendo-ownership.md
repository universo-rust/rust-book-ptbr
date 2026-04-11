---
title: "Entendendo ownership"
chapter_code: 04-00
slug: entendendo-ownership
---

# Entendendo Ownership

O ownership é a característica mais singular do Rust e tem profundas implicações para o restante da linguagem. Ela permite que o Rust ofereça garantias de segurança de memória sem a necessidade de um coletor de lixo, portanto, é importante entender como o ownership funciona. Neste capítulo, falaremos sobre ownership, bem como sobre vários recursos relacionados: borrowing, slices e como o Rust organiza os dados na memória.