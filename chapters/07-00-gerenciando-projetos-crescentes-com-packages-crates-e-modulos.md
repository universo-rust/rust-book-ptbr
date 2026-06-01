---
title: "Packages, crates e módulos"
chapter_code: 07-00
slug: packages-crates-e-modulos
challenge_day: 8
reading_minutes: 4
---

# Packages, Crates e Módulos

À medida que você escreve programas maiores, organizar o código se torna cada vez mais importante. Ao agrupar funcionalidades relacionadas e separar o código em partes com responsabilidades distintas, fica mais claro onde encontrar a implementação de um recurso específico e onde fazer mudanças no funcionamento desse recurso.

Os programas que escrevemos até aqui ficaram em um único módulo, em um único arquivo. Conforme um projeto cresce, você deve organizar o código dividindo-o em vários módulos e, depois, em vários arquivos. Um package pode conter vários crates binários e, opcionalmente, um crate de biblioteca. À medida que um package cresce, você pode extrair partes para crates separados, que passam a ser dependências externas. Este capítulo cobre todas essas técnicas. Em projetos muito grandes, formados por um conjunto de packages inter-relacionados que evoluem juntos, o Cargo oferece workspaces, que veremos em Cargo Workspaces no Capítulo 14.

Também vamos discutir como encapsular detalhes de implementação, o que permite reutilizar código em um nível mais alto: depois de implementar uma operação, outros trechos de código podem chamá-la por meio da interface pública sem precisar saber como a implementação funciona internamente. A forma como você escreve o código define quais partes ficam públicas para uso externo e quais partes permanecem como detalhes privados de implementação, que você se reserva o direito de alterar. Essa é outra maneira de limitar a quantidade de detalhes que você precisa manter na cabeça.

Um conceito relacionado é escopo: o contexto aninhado em que o código é escrito tem um conjunto de nomes definidos como “em escopo”. Ao ler, escrever e compilar código, programadores e compiladores precisam saber se um nome específico, em um ponto específico, se refere a uma variável, função, struct, enum, módulo, constante ou outro item, e o que esse item significa. Você pode criar escopos e alterar quais nomes estão dentro ou fora deles. Não é possível ter dois itens com o mesmo nome no mesmo escopo; há ferramentas para resolver conflitos de nomes.

Rust tem vários recursos que permitem gerenciar a organização do seu código, incluindo quais detalhes são expostos, quais permanecem privados e quais nomes estão em cada escopo dos seus programas. Esses recursos, às vezes chamados coletivamente de _sistema de módulos_, incluem:

* **Packages**: um recurso do Cargo que permite compilar, testar e compartilhar crates
* **Crates**: uma árvore de módulos que gera uma biblioteca ou um executável
* **Módulos e `use`**: permitem controlar a organização, o escopo e a privacidade de caminhos
* **Caminhos**: uma forma de nomear um item, como uma struct, uma função ou um módulo

Neste capítulo, vamos cobrir todos esses recursos, discutir como eles interagem e explicar como usá-los para gerenciar escopo. Ao final, você deve ter uma compreensão sólida do sistema de módulos e ser capaz de trabalhar com escopos como um profissional!
