---
chapter_code: 04-01
slug: o-que-e-ownership
---

# O que é ownership?

_Ownership_ é um conjunto de regras que governa como um programa em Rust gerencia a memória. Todos os programas precisam gerenciar a forma como utilizam a memória do computador enquanto estão em execução. Algumas linguagens possuem coletor de lixo (garbage collector), que procura regularmente por memória que não está mais sendo usada durante a execução do programa; em outras linguagens, o programador deve alocar e liberar a memória explicitamente. Rust utiliza uma terceira abordagem: a memória é gerenciada por meio de um sistema de ownership (posse) com um conjunto de regras que o compilador verifica. Se qualquer uma dessas regras for violada, o programa não será compilado. Nenhuma das funcionalidades do sistema de ownership deixa o programa mais lento durante a execução.

Como ownership é um conceito novo para muitos programadores, leva algum tempo para se acostumar com ele. A boa notícia é que, quanto mais experiência você adquire com Rust e com as regras do sistema de ownership, mais fácil se torna desenvolver naturalmente código que seja seguro e eficiente. Continue praticando!

Quando você entende ownership, passa a ter uma base sólida para compreender as funcionalidades que tornam Rust único. Neste capítulo, você aprenderá ownership trabalhando com alguns exemplos que focam em uma estrutura de dados muito comum: _strings_.

---

## A _stack_ e a _heap_

Muitas linguagens de programação não exigem que você pense com frequência sobre a _stack_ e a _heap_. Porém, em uma linguagem de programação de sistemas como Rust, o fato de um valor estar na stack ou na heap afeta o comportamento da linguagem e explica por que certas decisões precisam ser tomadas. Partes do sistema de ownership serão descritas em relação à stack e à heap mais adiante neste capítulo, então aqui vai uma breve explicação como preparação.

Tanto a stack quanto a heap são partes da memória disponíveis para o seu código usar em tempo de execução, mas elas são estruturadas de formas diferentes. A stack armazena valores na ordem em que os recebe e remove os valores na ordem inversa. Isso é conhecido como _last in, first out (LIFO)_, ou "último a entrar, primeiro a sair". Pense em uma pilha de pratos: quando você adiciona mais pratos, coloca-os no topo da pilha, e quando precisa de um prato, tira um do topo. Adicionar ou remover pratos do meio ou da base não funcionaria tão bem! Adicionar dados é chamado de _push_ na stack, e remover dados é chamado de _pop_ da stack. Todos os dados armazenados na stack precisam ter um tamanho conhecido e fixo. Dados com tamanho desconhecido em tempo de compilação ou cujo tamanho pode mudar devem ser armazenados na heap.

A heap é menos organizada: quando você coloca dados na heap, você solicita uma certa quantidade de espaço. O alocador de memória encontra um local vazio na heap que seja grande o suficiente, marca esse espaço como estando em uso e retorna um ponteiro, que é o endereço dessa localização. Esse processo é chamado de alocação na heap e, muitas vezes, é abreviado simplesmente como alocação (push de valores para a stack não é considerado alocação). Como o ponteiro para a heap tem um tamanho conhecido e fixo, você pode armazenar esse ponteiro na stack; porém, quando quiser acessar os dados reais, será necessário seguir o ponteiro. Pense em estar sendo acomodado em um restaurante: ao entrar, você informa o número de pessoas do seu grupo, e o anfitrião encontra uma mesa vazia que comporte todos e o conduz até ela. Se alguém do seu grupo chegar atrasado, essa pessoa pode perguntar onde vocês estão sentados para encontrá-los.

Colocar dados na stack é mais rápido do que alocar na heap, porque o alocador nunca precisa procurar um local para armazenar novos dados; esse local está sempre no topo da stack. Em comparação, alocar espaço na heap exige mais trabalho, pois o alocador primeiro precisa encontrar um espaço grande o suficiente para conter os dados e, depois, realizar tarefas de controle para preparar a próxima alocação.

Acessar dados na heap geralmente é mais lento do que acessar dados na stack, porque é necessário seguir um ponteiro para chegar até eles. Processadores modernos são mais rápidos quando precisam "pular" menos pela memória. Continuando a analogia, imagine um garçom em um restaurante anotando pedidos de várias mesas. É mais eficiente pegar todos os pedidos de uma mesa antes de ir para a próxima. Anotar um pedido da mesa A, depois um da mesa B, depois outro da A e outro da B seria um processo bem mais lento. Da mesma forma, um processador normalmente consegue executar melhor seu trabalho quando lida com dados que estão próximos uns dos outros (como acontece na stack), em vez de dados mais distantes (como pode ocorrer na heap).

Quando o seu código chama uma função, os valores passados para essa função (incluindo, potencialmente, ponteiros para dados na heap) e as variáveis locais da função são colocados na stack. Quando a função termina, esses valores são removidos da stack.

Acompanhar quais partes do código estão usando quais dados na heap, minimizar a quantidade de dados duplicados na heap e limpar dados não utilizados na heap para que você não fique sem espaço são todos problemas que o sistema de ownership resolve. Depois que você entende ownership, não precisará pensar na stack e na heap com muita frequência. Ainda assim, saber que o principal objetivo do ownership é gerenciar dados na heap ajuda a explicar por que ele funciona da maneira que funciona.