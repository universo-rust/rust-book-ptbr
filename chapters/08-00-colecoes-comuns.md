---
title: "Coleções comuns"
chapter_code: 08-00
slug: colecoes-comuns
---

# Coleções Comuns

A biblioteca padrão do Rust inclui várias estruturas de dados muito úteis chamadas de _coleções_. A maioria dos outros tipos de dados representa um valor específico, mas coleções podem conter vários valores. Diferentemente dos tipos array e tupla embutidos, os dados que essas coleções apontam são armazenados na heap, o que significa que a quantidade de dados não precisa ser conhecida em tempo de compilação e pode crescer ou encolher enquanto o programa executa. Cada tipo de coleção tem capacidades e custos diferentes, e escolher uma apropriada para sua situação atual é uma habilidade que você desenvolverá com o tempo. Neste capítulo, discutiremos três coleções usadas com muita frequência em programas Rust:

- Um _vetor_ permite armazenar um número variável de valores uns ao lado dos outros.
- Uma _string_ é uma coleção de caracteres. Já mencionamos o tipo `String` antes, mas neste capítulo falaremos sobre ela em profundidade.
- Um _hash map_ permite associar um valor a uma chave específica. É uma implementação particular da estrutura de dados mais geral chamada _mapa_.

Para aprender sobre os outros tipos de coleções fornecidos pela biblioteca padrão, consulte a documentação.

Discutiremos como criar e atualizar vetores, strings e hash maps, bem como o que torna cada um especial.
