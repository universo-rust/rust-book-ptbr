---
title: "Fundamentos de programação assíncrona: async, await, futures e streams"
chapter_code: 17-00
slug: fundamentos-de-programacao-assincrona-async-await-futures-e-streams
---

# Fundamentos de programação assíncrona: async, await, futures e streams

Muitas operações que pedimos ao computador podem demorar para terminar. Seria bom poder fazer outra coisa enquanto esperamos esses processos de longa duração. Os computadores modernos oferecem duas técnicas para trabalhar em mais de uma operação ao mesmo tempo: paralelismo e concorrência. A lógica dos nossos programas, porém, é escrita de forma em grande parte linear. Gostaríamos de poder especificar as operações que um programa deve realizar e os pontos em que uma função poderia pausar e outra parte do programa executar no lugar, sem precisar definir de antemão exatamente a ordem e a maneira como cada trecho de código deve executar. _Programação assíncrona_ é uma abstração que nos permite expressar o código em termos de possíveis pontos de pausa e resultados futuros, cuidando dos detalhes de coordenação por nós.

Este capítulo se baseia no uso de threads para paralelismo e concorrência do Capítulo 16, introduzindo uma abordagem alternativa de escrita de código: as futures, streams e a sintaxe `async` e `await` do Rust, que nos permitem expressar como as operações podem ser assíncronas, e os crates de terceiros que implementam runtimes assíncronos: código que gerencia e coordena a execução de operações assíncronas.

Pense em um exemplo. Digamos que você está exportando um vídeo que criou de uma celebração em família — uma operação que pode levar de minutos a horas. A exportação usará o máximo de CPU e GPU que puder. Se você tivesse apenas um núcleo de CPU e o sistema operacional não pausasse essa exportação até ela terminar — ou seja, se executasse a exportação de forma _síncrona_ — não poderia fazer mais nada no computador enquanto a tarefa rodasse. Seria uma experiência frustrante. Felizmente, o sistema operacional do seu computador pode, e de fato faz, interromper invisivelmente a exportação com frequência suficiente para você fazer outro trabalho ao mesmo tempo.

Agora digamos que você está baixando um vídeo compartilhado por outra pessoa, o que também pode demorar, mas não consome tanto tempo de CPU. Nesse caso, a CPU precisa esperar os dados chegarem pela rede. Você pode começar a ler os dados quando começam a chegar, mas pode levar algum tempo até tudo aparecer. Mesmo com todos os dados presentes, se o vídeo for grande, pode levar pelo menos um ou dois segundos para carregar tudo. Isso pode não parecer muito, mas é muito tempo para um processador moderno, que pode executar bilhões de operações por segundo. De novo, o sistema operacional interromperá invisivelmente seu programa para a CPU fazer outro trabalho enquanto espera a chamada de rede terminar.

A exportação de vídeo é um exemplo de operação _limitada por CPU_ ou _limitada por computação_. É limitada pela velocidade de processamento de dados potencial da CPU ou GPU e por quanto dessa velocidade pode ser dedicada à operação. O download de vídeo é um exemplo de operação _limitada por E/S_, porque é limitada pela velocidade de _entrada e saída_ do computador: só pode ir tão rápido quanto os dados forem enviados pela rede.

Nos dois casos, as interrupções invisíveis do sistema operacional oferecem uma forma de concorrência. Essa concorrência acontece apenas no nível do programa inteiro: o sistema operacional interrompe um programa para outros fazerem trabalho. Em muitos casos, porque entendemos nossos programas em um nível muito mais granular que o sistema operacional, podemos identificar oportunidades de concorrência que o sistema operacional não consegue ver.

Por exemplo, se estamos construindo uma ferramenta para gerenciar downloads de arquivos, devemos poder escrever o programa de modo que iniciar um download não trave a interface, e os usuários possam iniciar vários downloads ao mesmo tempo. Muitas APIs do sistema operacional para interagir com a rede são _bloqueantes_: bloqueiam o progresso do programa até os dados estarem completamente prontos.

> **Nota:** É assim que a _maioria_ das chamadas de função funciona, se você pensar bem. O termo _bloqueante_, porém, costuma ser reservado para chamadas que interagem com arquivos, rede ou outros recursos do computador, porque são os casos em que um programa individual se beneficiaria de a operação ser _não_ bloqueante.

Poderíamos evitar bloquear a thread principal criando uma thread dedicada para cada download. Porém, a sobrecarga dos recursos do sistema usados por essas threads acabaria virando problema. Seria melhor se a chamada não bloqueasse desde o início e pudéssemos definir um conjunto de tarefas que queremos que o programa complete, deixando o runtime escolher a melhor ordem e maneira de executá-las.

É exatamente isso que a abstração _async_ (abreviação de _asynchronous_) do Rust nos dá. Neste capítulo, você aprenderá tudo sobre async enquanto cobrimos os seguintes tópicos:

- Como usar a sintaxe `async` e `await` do Rust e executar funções assíncronas com um runtime
- Como usar o modelo async para resolver alguns dos mesmos desafios que vimos no Capítulo 16
- Como multithreading e async oferecem soluções complementares que você pode combinar em muitos casos

Antes de ver como async funciona na prática, porém, precisamos de um pequeno desvio para discutir as diferenças entre paralelismo e concorrência.

## Paralelismo e concorrência

Até aqui tratamos paralelismo e concorrência como praticamente intercambiáveis. Agora precisamos distingui-los com mais precisão, porque as diferenças aparecerão conforme avançarmos.

Pense nas formas diferentes de uma equipe dividir o trabalho em um projeto de software. Você pode atribuir a um membro várias tarefas, dar a cada membro uma tarefa, ou usar uma mistura das duas abordagens.

Quando uma pessoa trabalha em várias tarefas diferentes antes de qualquer uma terminar, isso é _concorrência_. Uma forma de implementar concorrência é semelhante a ter dois projetos diferentes abertos no computador: quando você se entedia ou trava em um, muda para o outro. Você é uma só pessoa, então não pode avançar nas duas tarefas exatamente ao mesmo tempo, mas pode multitarefar, progredindo em uma de cada vez alternando entre elas (veja a Figura 17-1).

![Diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com losangos representando subtarefas. Setas de A1 a B1, B1 a A2, A2 a B2, B2 a A3, A3 a A4 e A4 a B3. As setas entre as subtarefas cruzam as caixas entre a Tarefa A e a Tarefa B.](https://doc.rust-lang.org/book/img/trpl17-01.svg)

*Figura 17-1: Um fluxo de trabalho concorrente, alternando entre a Tarefa A e a Tarefa B*

Quando a equipe divide um grupo de tarefas fazendo cada membro assumir uma tarefa e trabalhar sozinho nela, isso é _paralelismo_. Cada pessoa da equipe pode progredir exatamente ao mesmo tempo (veja a Figura 17-2).

![Diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com losangos representando subtarefas. Setas de A1 a A2, A2 a A3, A3 a A4, B1 a B2 e B2 a B3. Nenhuma seta cruza entre as caixas da Tarefa A e da Tarefa B.](https://doc.rust-lang.org/book/img/trpl17-02.svg)

*Figura 17-2: Um fluxo de trabalho paralelo, em que o trabalho na Tarefa A e na Tarefa B ocorre de forma independente*

Nos dois fluxos, pode ser preciso coordenar entre tarefas diferentes. Talvez você achasse que a tarefa de um membro era totalmente independente do trabalho dos outros, mas na verdade depende de outra pessoa terminar a tarefa dela primeiro. Parte do trabalho poderia ser feita em paralelo, mas parte era de fato _serial_: só podia acontecer em série, uma tarefa depois da outra, como na Figura 17-3.

![Diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com losangos representando subtarefas. Na Tarefa A, setas de A1 a A2, de A2 a um par de linhas verticais grossas como símbolo de pausa, e desse símbolo a A3. Na Tarefa B, setas de B1 a B2, B2 a B3, B3 a A3 e B3 a B4.](https://doc.rust-lang.org/book/img/trpl17-03.svg)

*Figura 17-3: Um fluxo de trabalho parcialmente paralelo, em que o trabalho nas Tarefas A e B ocorre de forma independente até A3 ficar bloqueada aguardando os resultados de B3*

Da mesma forma, você pode perceber que uma de suas tarefas depende de outra sua. Agora seu trabalho concorrente também ficou serial.

Paralelismo e concorrência também podem se cruzar. Se você descobre que um colega está travado até você terminar uma tarefa, provavelmente concentrará todos os esforços nela para “desbloquear” o colega. Você e seu colega deixam de trabalhar em paralelo e também deixam de trabalhar de forma concorrente nas próprias tarefas.

As mesmas dinâmicas aparecem em software e hardware. Em uma máquina com um único núcleo de CPU, a CPU só pode executar uma operação por vez, mas ainda pode trabalhar de forma concorrente. Com ferramentas como threads, processos e async, o computador pode pausar uma atividade e mudar para outras antes de eventualmente voltar à primeira. Em uma máquina com vários núcleos, também pode haver paralelismo: um núcleo executa uma tarefa enquanto outro executa outra completamente diferente, e essas operações acontecem ao mesmo tempo.

Executar código async em Rust costuma acontecer de forma concorrente. Dependendo do hardware, do sistema operacional e do runtime async que usamos (falaremos de runtimes em breve), essa concorrência também pode usar paralelismo por baixo dos panos.

Agora vamos ver como a programação assíncrona em Rust realmente funciona.
