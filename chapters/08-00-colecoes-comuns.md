---
title: "Coleções comuns"
chapter_code: 08-00
slug: colecoes-comuns
---

# Coleções Comuns

A biblioteca padrão do Rust inclui várias estruturas de dados muito úteis, chamadas de _coleções_. Enquanto a maioria dos outros tipos representa um valor específico, as coleções podem conter vários valores. Diferentemente dos tipos `array` e tupla integrados à linguagem, os dados referenciados por essas coleções ficam armazenados na heap — ou seja, a quantidade de dados não precisa ser conhecida em tempo de compilação e pode crescer ou diminuir enquanto o programa executa. Cada tipo de coleção tem capacidades e custos distintos; escolher a mais adequada para a situação é uma habilidade que você desenvolve com o tempo. Neste capítulo, veremos três coleções muito usadas em programas Rust:

- Um _vetor_ permite armazenar um número variável de valores, lado a lado.
- Uma _string_ é uma coleção de caracteres. Já mencionamos o tipo `String` antes, mas aqui vamos falar dela em profundidade.
- Um _hash map_ permite associar um valor a uma chave específica. É uma implementação particular da estrutura de dados mais geral chamada _mapa_.

Para conhecer os demais tipos de coleção disponíveis na biblioteca padrão, consulte a documentação.

Veremos como criar e atualizar vetores, strings e hash maps, e o que torna cada uma especial.
