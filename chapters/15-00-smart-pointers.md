---
title: "Smart pointers"
chapter_code: 15-00
slug: smart-pointers
---

# Smart Pointers

Um ponteiro é um conceito geral para uma variável que contém um endereço na memória. Esse endereço se refere, ou “aponta para”, algum outro dado. O tipo mais comum de ponteiro em Rust é uma referência, que você aprendeu no Capítulo 4. Referências são indicadas pelo símbolo `&` e emprestam o valor para o qual apontam. Elas não têm capacidades especiais além de se referir a dados, e não têm overhead.

_Smart pointers_, por outro lado, são estruturas de dados que se comportam como um ponteiro, mas também têm metadados e capacidades adicionais. O conceito de smart pointers não é exclusivo do Rust: smart pointers surgiram em C++ e existem em outras linguagens também. Rust tem vários smart pointers definidos na biblioteca padrão que fornecem funcionalidade além da fornecida por referências. Para explorar o conceito geral, veremos alguns exemplos diferentes de smart pointers, incluindo um tipo de smart pointer de _contagem de referências_. Esse ponteiro permite que dados tenham vários donos, rastreando o número de donos e, quando não restarem donos, limpando os dados.

Em Rust, com seu conceito de ownership e borrowing, há uma diferença adicional entre referências e smart pointers: enquanto referências apenas emprestam dados, em muitos casos smart pointers _possuem_ os dados para os quais apontam.

Smart pointers costumam ser implementados usando structs. Diferente de uma struct comum, smart pointers implementam as traits `Deref` e `Drop`. A trait `Deref` permite que uma instância da struct smart pointer se comporte como uma referência, para que você possa escrever código que funcione com referências ou smart pointers. A trait `Drop` permite personalizar o código executado quando uma instância do smart pointer sai de escopo. Neste capítulo, discutiremos ambas as traits e demonstraremos por que são importantes para smart pointers.

Dado que o padrão smart pointer é um padrão de design geral usado frequentemente em Rust, este capítulo não cobrirá todos os smart pointers existentes. Muitas bibliotecas têm seus próprios smart pointers, e você pode até escrever os seus. Cobriremos os smart pointers mais comuns na biblioteca padrão:

- `Box<T>`, para alocar valores no heap
- `Rc<T>`, um tipo de contagem de referências que permite múltipla posse
- `Ref<T>` e `RefMut<T>`, acessados por `RefCell<T>`, um tipo que impõe as regras de borrowing em tempo de execução em vez de tempo de compilação

Além disso, cobriremos o padrão de _interior mutability_, em que um tipo imutável expõe uma API para mutar um valor interior. Também discutiremos ciclos de referência: como eles podem vazar memória e como evitá-los.

Vamos mergulhar!
