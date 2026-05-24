---
title: "Packages, crates e módulos"
chapter_code: 07-00
slug: packages-crates-e-modulos
---

# Packages, Crates e Módulos

À medida que você escreve programas grandes, organizar seu código se tornará cada vez mais importante. Ao agrupar funcionalidades relacionadas e separar código com recursos distintos, você esclarece onde encontrar o código que implementa um recurso particular e onde ir para alterar como um recurso funciona.

Os programas que escrevemos até agora estavam em um módulo em um arquivo. Conforme um projeto cresce, você deve organizar o código dividindo-o em vários módulos e depois em vários arquivos. Um package pode conter vários binary crates e, opcionalmente, um library crate. Conforme um package cresce, você pode extrair partes em crates separados que se tornam dependências externas. Este capítulo cobre todas essas técnicas. Para projetos muito grandes compostos por um conjunto de packages interrelacionados que evoluem juntos, o Cargo fornece workspaces, que cobriremos em Cargo Workspaces no Capítulo 14.

Também discutiremos encapsular detalhes de implementação, o que permite reutilizar código em um nível mais alto: depois de implementar uma operação, outro código pode chamar seu código via sua interface pública sem precisar saber como a implementação funciona. A forma como você escreve o código define quais partes são públicas para outro código usar e quais partes são detalhes de implementação privados que você se reserva o direito de alterar. Essa é outra forma de limitar a quantidade de detalhes que você precisa manter em mente.

Um conceito relacionado é escopo: o contexto aninhado em que o código é escrito tem um conjunto de nomes definidos como “no escopo”. Ao ler, escrever e compilar código, programadores e compiladores precisam saber se um nome particular em um ponto específico se refere a uma variável, função, struct, enum, módulo, constante ou outro item e o que esse item significa. Você pode criar escopos e alterar quais nomes estão dentro ou fora do escopo. Não é possível ter dois itens com o mesmo nome no mesmo escopo; existem ferramentas para resolver conflitos de nomes.

O Rust tem vários recursos que permitem gerenciar a organização do seu código, incluindo quais detalhes são expostos, quais são privados e quais nomes estão em cada escopo dos seus programas. Esses recursos, às vezes chamados coletivamente de _sistema de módulos_, incluem:

* **Packages**: um recurso do Cargo que permite compilar, testar e compartilhar crates
* **Crates**: uma árvore de módulos que produz uma biblioteca ou executável
* **Módulos e `use`**: permitem controlar a organização, o escopo e a privacidade de caminhos
* **Caminhos**: uma forma de nomear um item, como uma struct, função ou módulo

Neste capítulo, cobriremos todos esses recursos, discutiremos como interagem e explicaremos como usá-los para gerenciar escopo. Ao final, você deve ter uma compreensão sólida do sistema de módulos e ser capaz de trabalhar com escopos como um profissional!
