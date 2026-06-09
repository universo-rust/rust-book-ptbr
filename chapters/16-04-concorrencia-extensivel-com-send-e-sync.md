---
title: "Concorrência extensível com `Send` e `Sync`"
chapter_code: 16-04
slug: concorrencia-extensivel-com-send-e-sync
challenge_day: 21
reading_minutes: 6
---

# Concorrência extensível com `Send` e `Sync`

Curiosamente, quase todo recurso de concorrência que falamos neste capítulo fez parte da biblioteca padrão, não da linguagem. Suas opções para lidar com concorrência não se limitam à linguagem ou à biblioteca padrão; você pode escrever seus próprios recursos de concorrência ou usar os escritos por outros.

No entanto, entre os conceitos-chave de concorrência embutidos na linguagem em vez da biblioteca padrão estão as traits `Send` e `Sync` do módulo `std::marker`.

## Transferindo ownership entre threads

A trait marcador `Send` indica que a ownership de valores do tipo que implementa `Send` pode ser transferida entre threads. Quase todo tipo Rust implementa `Send`, mas há algumas exceções, incluindo `Rc<T>`: este não pode implementar `Send` porque, se você clonasse um valor `Rc<T>` e tentasse transferir a ownership do clone para outra thread, ambas as threads poderiam atualizar a contagem de referências ao mesmo tempo. Por isso, `Rc<T>` é implementado para uso em situações de thread única, onde você não quer pagar a penalidade de desempenho de thread-safety.

Portanto, o sistema de tipos e os trait bounds do Rust garantem que você nunca envie acidentalmente um valor `Rc<T>` entre threads de forma insegura. Quando tentamos fazer isso na Listagem 16-14, obtivemos o erro ``the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. Quando mudamos para `Arc<T>`, que implementa `Send`, o código compilou.

Qualquer tipo composto inteiramente de tipos `Send` é automaticamente marcado como `Send` também. Quase todos os tipos primitivos são `Send`, exceto ponteiros brutos, que discutiremos no Capítulo 20.

## Acessando a partir de várias threads

A trait marcador `Sync` indica que é seguro que o tipo que implementa `Sync` seja referenciado a partir de várias threads. Em outras palavras, qualquer tipo `T` implementa `Sync` se `&T` (uma referência imutável a `T`) implementa `Send`, o que significa que a referência pode ser enviada com segurança para outra thread. Semelhante a `Send`, tipos primitivos implementam `Sync`, e tipos compostos inteiramente de tipos que implementam `Sync` também implementam `Sync`.

O smart pointer `Rc<T>` também não implementa `Sync` pelas mesmas razões pelas quais não implementa `Send`. O tipo `RefCell<T>` (que falamos no Capítulo 15) e a família de tipos relacionados `Cell<T>` não implementam `Sync`. A implementação de verificação de borrowing que `RefCell<T>` faz em tempo de execução não é thread-safe. O smart pointer `Mutex<T>` implementa `Sync` e pode ser usado para compartilhar acesso com várias threads, como você viu em Acesso compartilhado a `Mutex<T>`.

## Implementar `Send` e `Sync` manualmente é unsafe

Como tipos compostos inteiramente de outros tipos que implementam as traits `Send` e `Sync` também implementam automaticamente `Send` e `Sync`, não precisamos implementar essas traits manualmente. Como traits marcador, elas nem têm métodos para implementar. São apenas úteis para impor invariantes relacionados à concorrência.

Implementar essas traits manualmente envolve implementar código Rust unsafe. Falaremos sobre usar código Rust unsafe no Capítulo 20; por agora, a informação importante é que construir novos tipos concorrentes não feitos de partes `Send` e `Sync` exige pensamento cuidadoso para manter as garantias de segurança. [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/) tem mais informação sobre essas garantias e como mantê-las.

## Resumo

Este não é o último que você verá de concorrência neste livro: o próximo capítulo foca em programação async, e o projeto do Capítulo 21 usará os conceitos deste capítulo em uma situação mais realista do que os exemplos menores discutidos aqui.

Como mencionado anteriormente, como muito pouco de como o Rust lida com concorrência faz parte da linguagem, muitas soluções de concorrência são implementadas como crates. Elas evoluem mais rapidamente que a biblioteca padrão, então procure online pelos crates atuais e de ponta para usar em situações multithread.

A biblioteca padrão do Rust fornece channels para passagem de mensagens e tipos smart pointer, como `Mutex<T>` e `Arc<T>`, que são seguros para usar em contextos concorrentes. O sistema de tipos e o borrow checker garantem que o código que usa essas soluções não acabará com data races ou referências inválidas. Depois que seu código compilar, você pode ficar tranquilo de que ele executará feliz em várias threads sem os tipos de bugs difíceis de rastrear comuns em outras linguagens. Programação concorrente não é mais um conceito para temer: vá em frente e torne seus programas concorrentes, sem medo!
