---
title: "Concorrência sem medo"
chapter_code: 16-00
slug: concorrencia-sem-medo
---

# Concorrência sem medo

Um dos grandes objetivos do Rust é tornar a programação concorrente segura e eficiente. Em _programação concorrente_, diferentes partes do código podem executar de forma independente; em _programação paralela_, elas podem executar simultaneamente. Com a popularização de computadores com múltiplos processadores, esses modelos de programação se tornaram cada vez mais importantes. No entanto, historicamente eles sempre foram difíceis de implementar corretamente e propensos a erros. O Rust foi criado para ajudar a resolver esse problema.

Inicialmente, a equipe do Rust pensou que garantir segurança de memória e prevenir problemas de concorrência eram dois desafios separados a serem resolvidos com métodos diferentes. Com o tempo, a equipe descobriu que o sistema de ownership e de tipos é um conjunto poderoso de ferramentas para ajudar a gerenciar segurança de memória _e_ problemas de concorrência! Ao aproveitar ownership e verificação de tipos, muitos erros de concorrência são erros em tempo de compilação em Rust em vez de erros em tempo de execução. Portanto, em vez de fazer você gastar muito tempo tentando reproduzir as circunstâncias exatas em que um bug de concorrência em tempo de execução ocorre, código incorreto se recusará a compilar e apresentará um erro explicando o problema. Como resultado, você pode corrigir seu código enquanto trabalha nele, em vez de potencialmente depois que foi enviado para produção. Apelidamos este aspecto do Rust de _concorrência sem medo_. Concorrência sem medo permite escrever código livre de bugs sutis e fácil de refatorar sem introduzir novos bugs.

> **Nota:** Por simplicidade, nos referiremos a muitos dos problemas como _concorrentes_ em vez de ser mais precisos dizendo _concorrentes e/ou paralelos_. Para este capítulo, substitua mentalmente _concorrente e/ou paralelo_ sempre que usarmos _concorrente_. No próximo capítulo, onde a distinção importa mais, seremos mais específicos.

Muitas linguagens são dogmáticas sobre as soluções que oferecem para lidar com problemas concorrentes. Por exemplo, Erlang tem funcionalidade elegante para concorrência por passagem de mensagens, mas tem apenas formas obscuras de compartilhar estado entre threads. Oferecer apenas um subconjunto de soluções possíveis é uma estratégia razoável para linguagens de alto nível, porque uma linguagem de alto nível promete benefícios ao abrir mão de algum controle para ganhar abstrações. No entanto, linguagens de baixo nível devem fornecer a solução com o melhor desempenho em qualquer situação e têm menos abstrações sobre o hardware. Portanto, o Rust oferece uma variedade de ferramentas para modelar problemas da forma apropriada para sua situação e requisitos.

Estes são os tópicos que cobriremos neste capítulo:

- Como criar threads para executar vários trechos de código ao mesmo tempo
- Concorrência por _passagem de mensagens_, em que channels enviam mensagens entre threads
- Concorrência de _estado compartilhado_, em que várias threads têm acesso a algum dado
- As traits `Sync` e `Send`, que estendem as garantias de concorrência do Rust a tipos definidos pelo usuário, bem como tipos fornecidos pela biblioteca padrão
