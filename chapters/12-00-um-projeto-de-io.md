---
title: "Um projeto de I/O: construindo um programa de linha de comando"
chapter_code: 12-00
slug: um-projeto-de-io
---

# Um projeto de I/O: construindo um programa de linha de comando

Este capítulo recapitula muitas das habilidades que você aprendeu até agora e explora mais alguns recursos da biblioteca padrão. Vamos construir uma ferramenta de linha de comando que interage com arquivos e com a entrada e saída da linha de comando, praticando alguns dos conceitos de Rust que você já tem na bagagem.

A velocidade, a segurança, a geração de um único binário e o suporte multiplataforma fazem de Rust uma linguagem ideal para criar ferramentas de linha de comando. Por isso, em nosso projeto, criaremos a nossa própria versão da clássica ferramenta de busca de linha de comando `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint). No caso de uso mais simples, `grep` procura uma determinada string em um determinado arquivo. Para isso, `grep` recebe como argumentos um caminho de arquivo e uma string. Então, ele lê o arquivo, encontra as linhas desse arquivo que contêm a string passada como argumento e imprime essas linhas.

Ao longo do caminho, mostraremos como fazer nossa ferramenta de linha de comando usar recursos do terminal que muitas outras ferramentas de linha de comando também usam. Leremos o valor de uma variável de ambiente para permitir que o usuário configure o comportamento da nossa ferramenta. Também imprimiremos mensagens de erro no fluxo de console de erro padrão (`stderr`), em vez de na saída padrão (`stdout`), para que, por exemplo, o usuário possa redirecionar a saída bem-sucedida para um arquivo e ainda assim ver as mensagens de erro na tela.

Um membro da comunidade Rust, Andrew Gallant, já criou uma versão completa e muito rápida de `grep`, chamada `ripgrep`. Em comparação, nossa versão será bastante simples, mas este capítulo dará a você parte do conhecimento de base necessário para entender um projeto do mundo real como o `ripgrep`.

Nosso projeto `grep` combinará vários conceitos que você aprendeu até agora:

- Organização de código ([Capítulo 7](/livro/cap07-00-gerenciando-projetos-crescentes-com-packages-crates-e-modulos))
- Uso de vetores e strings ([Capítulo 8](/livro/cap08-00-colecoes-comuns))
- Tratamento de erros ([Capítulo 9](/livro/cap09-00-tratamento-de-erros))
- Uso de traits e lifetimes quando apropriado ([Capítulo 10](/livro/cap10-00-tipos-genericos-traits-e-lifetimes))
- Escrita de testes ([Capítulo 11](/livro/cap11-00-escrevendo-testes-automatizados))

Também apresentaremos brevemente closures, iterators e trait objects, que o [Capítulo 13](/livro/cap13-00-recursos-funcionais-iterators-e-closures) e o [Capítulo 18](/livro/cap18-00-recursos-de-oop) abordarão em detalhes.
