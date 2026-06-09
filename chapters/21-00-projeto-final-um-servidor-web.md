---
title: "Projeto final: um servidor web"
chapter_code: 21-00
slug: projeto-final-um-servidor-web
---

# Projeto final: construindo um servidor web multithread

Foi uma longa jornada, mas chegamos ao fim do livro. Neste capítulo, construiremos mais um projeto juntos para demonstrar alguns dos conceitos que cobrimos nos capítulos finais, além de recapitular algumas lições anteriores.

Para nosso projeto final, faremos um servidor web que diz “Hello!” e se parece com a Figura 21-1 em um navegador web.

Este é o nosso plano para construir o servidor web:

1. Aprender um pouco sobre TCP e HTTP.
2. Escutar conexões TCP em um socket.
3. Analisar um pequeno número de requisições HTTP.
4. Criar uma resposta HTTP adequada.
5. Melhorar a taxa de transferência do nosso servidor com um pool de threads.

![Captura de tela de navegador em 127.0.0.1:7878 com página Hello! Hi from Rust](https://doc.rust-lang.org/book/img/trpl21-01.png)

*Figura 21-1: Nosso projeto final compartilhado*

Antes de começarmos, devemos mencionar dois detalhes. Primeiro, o método que usaremos não será a melhor forma de construir um servidor web com Rust. Membros da comunidade publicaram vários crates prontos para produção disponíveis em [crates.io](https://crates.io/) que fornecem implementações de servidor web e pool de threads mais completas do que a que construiremos. No entanto, nossa intenção neste capítulo é ajudá-lo a aprender, não tomar o caminho fácil. Como Rust é uma linguagem de programação de sistemas, podemos escolher o nível de abstração com o qual queremos trabalhar e podemos ir a um nível mais baixo do que seria possível ou prático em outras linguagens.

Segundo, não usaremos async e await aqui. Construir um pool de threads já é um desafio grande por si só, sem incluir a construção de um runtime async! No entanto, observaremos como async e await poderiam ser aplicáveis a alguns dos mesmos problemas que veremos neste capítulo. Em última análise, como observamos de volta no Capítulo 17, muitos runtimes async usam pools de threads para gerenciar seu trabalho.

Portanto, escreveremos manualmente o servidor HTTP básico e o pool de threads para que você possa aprender as ideias e técnicas gerais por trás dos crates que poderá usar no futuro.
