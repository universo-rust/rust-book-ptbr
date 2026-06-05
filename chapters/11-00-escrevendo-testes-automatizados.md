---
title: "Escrevendo testes automatizados"
chapter_code: 11-00
slug: escrevendo-testes-automatizados
challenge_day: 14
reading_minutes: 3
---

# Escrevendo testes automatizados

Em seu ensaio de 1972, “The Humble Programmer”, Edsger W. Dijkstra disse que “testes de programas podem ser uma maneira muito eficaz de mostrar a presença de bugs, mas são irremediavelmente inadequados para mostrar sua ausência.” Isso não significa que não devamos tentar testar tanto quanto pudermos!

_Correção_ em nossos programas é a medida em que nosso código faz aquilo que pretendemos que ele faça. O Rust foi projetado com um alto grau de preocupação com a correção dos programas, mas correção é um assunto complexo e não é fácil de provar. O sistema de tipos do Rust assume uma enorme parte desse peso, mas o sistema de tipos não consegue detectar tudo. Por isso, o Rust inclui suporte para escrever testes de software automatizados.

Digamos que escrevamos uma função `add_two` que adiciona 2 a qualquer número passado para ela. A assinatura dessa função aceita um inteiro como parâmetro e retorna um inteiro como resultado. Quando implementamos e compilamos essa função, o Rust faz toda a verificação de tipos e a verificação de empréstimos que você aprendeu até aqui para garantir que, por exemplo, não estejamos passando um valor `String` ou uma referência inválida para essa função. Mas o Rust _não consegue_ verificar se essa função fará exatamente o que pretendemos, que é retornar o parâmetro mais 2 em vez de, digamos, o parâmetro mais 10 ou o parâmetro menos 50! É aí que entram os testes.

Podemos escrever testes que afirmam, por exemplo, que, quando passamos `3` para a função `add_two`, o valor retornado é `5`. Podemos executar esses testes sempre que fizermos alterações no código para garantir que qualquer comportamento correto já existente não tenha mudado.

Testar é uma habilidade complexa: embora não possamos abordar em um único capítulo todos os detalhes sobre como escrever bons testes, neste capítulo discutiremos a mecânica dos recursos de teste do Rust. Falaremos sobre as anotações e macros disponíveis para você ao escrever seus testes, sobre o comportamento padrão e as opções fornecidas para executá-los, e sobre como organizar testes em testes unitários e testes de integração.
