---
title: "Introdução"
chapter_code: 00-02
slug: introducao
---

# Introdução

> **Nota:** Esta edição do livro é a mesma de [The Rust Programming Language](https://nostarch.com/rust-programming-language-3rd-edition), disponível em formato impresso e ebook pela [No Starch Press](https://nostarch.com/).

Bem-vindo a *The Rust Programming Language*, um livro introdutório sobre Rust. A linguagem de programação Rust ajuda você a escrever softwares mais rápidos e confiáveis. Ergonomia de alto nível e controle de baixo nível costumam entrar em conflito no design de linguagens de programação; Rust desafia esse conflito. Ao equilibrar um grande poder técnico com uma excelente experiência para o desenvolvedor, Rust oferece a opção de controlar detalhes de baixo nível (como o uso de memória) sem toda a complexidade tradicionalmente associada a esse tipo de controle.

## Para quem o Rust é indicado

Rust é ideal para muitas pessoas por diversos motivos. Vamos analisar alguns dos grupos mais importantes.

### Times de desenvolvedores

Rust tem se mostrado uma ferramenta produtiva para colaboração entre grandes equipes de desenvolvedores, com diferentes níveis de conhecimento em programação de sistemas. Código de baixo nível é propenso a diversos bugs sutis que, na maioria das outras linguagens, só podem ser detectados por meio de testes extensivos e revisões cuidadosas de código feitas por desenvolvedores experientes. Em Rust, o compilador atua como um guardião, recusando-se a compilar códigos com esses bugs difíceis de detectar, incluindo bugs de concorrência. Trabalhando lado a lado com o compilador, a equipe pode focar na lógica do programa em vez de caçar erros.

Rust também traz ferramentas modernas de desenvolvimento para o mundo da programação de sistemas:

- Cargo, o gerenciador de dependências e ferramenta de build, torna simples, consistente e indolor adicionar, compilar e gerenciar dependências em todo o ecossistema Rust.
- A ferramenta de formatação `rustfmt` garante um estilo de código consistente entre desenvolvedores.
- O Rust Language Server oferece integração com ambientes de desenvolvimento (IDEs), com autocomplete de código e mensagens de erro inline.

Ao utilizar essas e outras ferramentas do ecossistema Rust, os desenvolvedores conseguem ser produtivos mesmo escrevendo código de nível de sistema.

### Estudantes

Rust é para estudantes e para quem tem interesse em aprender conceitos de sistemas. Usando Rust, muitas pessoas aprenderam sobre temas como desenvolvimento de sistemas operacionais. A comunidade é muito acolhedora e está sempre disposta a responder perguntas de estudantes. Por meio de iniciativas como este livro, os times do Rust querem tornar conceitos de sistemas mais acessíveis para mais pessoas, especialmente para quem está começando a programar.

### Empresas

Centenas de empresas, grandes e pequenas, utilizam Rust em produção para uma grande variedade de tarefas, incluindo ferramentas de linha de comando, serviços web, ferramentas de DevOps, dispositivos embarcados, análise e transcodificação de áudio e vídeo, criptomoedas, bioinformática, motores de busca, aplicações de internet das coisas, machine learning e até partes importantes do navegador Firefox.

### Desenvolvedores Open Source

Rust é para pessoas que desejam construir a linguagem Rust, sua comunidade, ferramentas de desenvolvimento e bibliotecas. Adoraríamos contar com sua contribuição para a linguagem.

### Pessoas que valorizam velocidade e estabilidade

Rust é para quem busca velocidade e estabilidade em uma linguagem. Por velocidade, queremos dizer tanto o quão rápido o código Rust pode ser executado quanto a rapidez com que Rust permite escrever programas. As verificações do compilador garantem estabilidade durante a adição de funcionalidades e refatorações. Isso contrasta com códigos legados frágeis em linguagens que não possuem essas verificações, nas quais desenvolvedores muitas vezes têm receio de fazer mudanças. Ao buscar abstrações de custo zero — recursos de alto nível que compilam para código de baixo nível tão rápido quanto código escrito manualmente — Rust se esforça para que código seguro também seja código rápido.

A linguagem Rust espera atender muitos outros usuários; os mencionados aqui são apenas alguns dos principais públicos. No geral, a maior ambição do Rust é eliminar os compromissos que programadores aceitaram por décadas, oferecendo segurança *e* produtividade, velocidade *e* ergonomia. Experimente Rust e veja se essas escolhas funcionam para você.

## Para quem é este livro

Este livro assume que você já escreveu código em outra linguagem de programação, mas não faz suposições sobre qual delas. Tentamos tornar o material acessível a pessoas com diferentes formações em programação. Não gastamos muito tempo explicando o que é programação ou como pensar como programador. Se você é totalmente iniciante, talvez seja melhor começar por um livro que ofereça uma introdução específica à programação.

## Como usar este livro

Em geral, este livro pressupõe que você o leia em sequência, do começo ao fim. Os capítulos posteriores se baseiam em conceitos apresentados nos capítulos anteriores, e os capítulos iniciais podem não se aprofundar em detalhes sobre um determinado tópico, mas voltarão a ele em um capítulo posterior.

Você encontrará dois tipos de capítulos neste livro: capítulos conceituais e capítulos de projeto. Nos capítulos conceituais, você aprenderá sobre algum aspecto do Rust. Nos capítulos de projeto, construiremos pequenos programas juntos, aplicando o que você aprendeu até então. O Capítulo 2, o Capítulo 12 e o Capítulo 21 são capítulos de projeto; os demais são capítulos conceituais.

O **Capítulo 1** explica como instalar o Rust, como escrever um programa "Hello, world!" e como usar o Cargo, o gerenciador de pacotes e ferramenta de build do Rust. O **Capítulo 2** é uma introdução prática à escrita de um programa em Rust, fazendo você construir um jogo de adivinhação de números. Aqui, cobrimos os conceitos em um nível mais alto, e capítulos posteriores fornecerão detalhes adicionais. Se você quiser colocar a mão na massa imediatamente, o **Capítulo 2** é o lugar certo para isso. Se você for um aprendiz particularmente meticuloso que prefere aprender cada detalhe antes de passar para o próximo, talvez queira pular o **Capítulo 2** e ir direto para o **Capítulo 3**, que aborda recursos do Rust semelhantes aos de outras linguagens de programação; depois, você pode retornar ao **Capítulo 2** quando quiser trabalhar em um projeto aplicando os detalhes que aprendeu.

No **Capítulo 4**, você aprenderá sobre o sistema de ownership do Rust. O **Capítulo 5** discute structs e métodos. O **Capítulo 6** cobre enums, expressões `match` e as construções de controle de fluxo `if let` e `let...else`. Você usará structs e enums para criar tipos personalizados.

No **Capítulo 7**, você aprenderá sobre o sistema de módulos do Rust e sobre as regras de privacidade para organizar seu código e sua interface de programação de aplicações (API) pública. O **Capítulo 8** discute algumas estruturas de dados de coleção comuns que a biblioteca padrão fornece: vetores, strings e hash maps. O **Capítulo 9** explora a filosofia e as técnicas de tratamento de erros do Rust.

O **Capítulo 10** aprofunda-se em generics, traits e lifetimes, que dão a você o poder de definir código que se aplica a múltiplos tipos. O **Capítulo 11** é inteiramente dedicado a testes, que, mesmo com as garantias de segurança do Rust, são necessários para assegurar que a lógica do seu programa esteja correta. No **Capítulo 12**, construiremos nossa própria implementação de um subconjunto das funcionalidades da ferramenta de linha de comando `grep`, que busca texto dentro de arquivos. Para isso, usaremos muitos dos conceitos discutidos nos capítulos anteriores.

O **Capítulo 13** explora closures e iteradores: recursos do Rust que vêm de linguagens de programação funcional. No **Capítulo 14**, examinaremos o Cargo com mais profundidade e falaremos sobre boas práticas para compartilhar suas bibliotecas com outras pessoas. O **Capítulo 15** discute os smart pointers que a biblioteca padrão fornece e as traits que habilitam sua funcionalidade.

No **Capítulo 16**, percorreremos diferentes modelos de programação concorrente e falaremos sobre como o Rust ajuda você a programar com múltiplas threads sem medo. No **Capítulo 17**, avançamos a partir disso explorando a sintaxe async e await do Rust, juntamente com tarefas, futures e streams, e o modelo de concorrência leve que eles possibilitam.

O **Capítulo 18** analisa como os idiomas do Rust se comparam aos princípios de programação orientada a objetos com os quais você talvez esteja familiarizado. O **Capítulo 19** é uma referência sobre padrões e pattern matching, que são formas poderosas de expressar ideias em programas Rust. O **Capítulo 20** contém um verdadeiro banquete de tópicos avançados de interesse, incluindo Rust unsafe, macros e mais detalhes sobre lifetimes, traits, tipos, funções e closures.

No **Capítulo 21**, concluiremos um projeto no qual implementaremos um servidor web de baixo nível e multithread!

Por fim, alguns apêndices contêm informações úteis sobre a linguagem em um formato mais voltado a referência. O **Apêndice A** cobre as palavras-chave do Rust, o **Apêndice B** cobre os operadores e símbolos do Rust, o **Apêndice C** cobre as traits deriváveis fornecidas pela biblioteca padrão, o **Apêndice D** cobre algumas ferramentas úteis de desenvolvimento, e o **Apêndice E** explica as edições do Rust. No **Apêndice F**, você pode encontrar traduções do livro, e no **Apêndice G** abordaremos como o Rust é desenvolvido e o que é o Rust nightly.

Não existe uma forma errada de ler este livro: se você quiser pular adiante, vá em frente! Talvez seja necessário voltar a capítulos anteriores se surgir alguma confusão. Mas faça o que funcionar melhor para você.

Uma parte importante do processo de aprender Rust é aprender a ler as mensagens de erro exibidas pelo compilador: elas irão guiá-lo em direção a um código que funcione. Assim, forneceremos muitos exemplos que não compilam, juntamente com a mensagem de erro que o compilador mostrará em cada situação. Saiba que, se você digitar e executar um exemplo aleatório, ele pode não compilar! Certifique-se de ler o texto ao redor para ver se o exemplo que você está tentando executar foi feito para gerar erro. Na maioria das situações, nós o conduziremos à versão correta de qualquer código que não compile. Ferris também ajudará você a distinguir códigos que não foram feitos para funcionar:

| Ferris | Significado |
|--------|-------------|
| ![Ferris com interrogação: exemplo que não compila.](https://doc.rust-lang.org/book/img/ferris/does_not_compile.svg) | Este código não compila! |
| ![Ferris em pânico: exemplo que entra em pânico em tempo de execução.](https://doc.rust-lang.org/book/img/ferris/panics.svg) | Este código entra em pânico! |
| ![Ferris encolhendo os ombros: exemplo com comportamento indesejado.](https://doc.rust-lang.org/book/img/ferris/not_desired_behavior.svg) | Este código não produz o comportamento desejado. |

Na maioria das situações, vamos guiá-lo para a versão correta de qualquer código que não compile.

## Código fonte

Os arquivos fonte dos quais este livro é gerado podem ser encontrados no [Github](https://github.com/rust-lang/book/tree/main/src) (em inglês).