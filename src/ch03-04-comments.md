## Comentários

Todos os programadores se esforçam para tornar seu código fácil de entender, mas às vezes
uma explicação extra é necessária. nesses casos, os programadores deixam _comentários_ em
seu código-fonte que o compilador ignorará, mas que as pessoas que leem o
código-fonte podem achar úteis.

Aqui está um comentário simples:

```rust
// olá, mundo
```

Em Rust, o estilo idiomático de comentário começa um comentário com duas barras, e o
comentário continua até o fim da linha. Para comentários que se estendem além de uma
única linha, você precisará incluir `//` em cada linha, assim:

```rust
// Então estamos fazendo algo complicado aqui, longo o suficiente para que precisemos
// de várias linhas de comentários para fazê-lo! Ufa! Esperamos que este comentário
// explique o que está acontecendo.
```

Comentários também podem ser colocados no final de linhas que contêm código:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Mas você os verá mais frequentemente usados neste formato, com o comentário em uma
linha separada acima do código que ele está anotando:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

O Rust também tem outro tipo de comentário, os comentários de documentação, sobre os quais
vamos discutir na seção [“Publicando um Crate no Crates.io”][publishing]<!-- ignore -->
do Capítulo 14.

[publishing]: ch14-02-publishing-to-crates-io.html
