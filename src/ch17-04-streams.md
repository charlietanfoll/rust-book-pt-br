<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Streams: Futuros em Sequência

Lembre-se de como usamos o receptor para o nosso canal assíncrono anteriormente neste capítulo
na seção [“Passagem de Mensagens”][17-02-messages]<!-- ignore -->. O método assíncrono
`recv` produz uma sequência de itens ao longo do tempo. Esta é uma instância de um
padrão muito mais geral conhecido como _stream_ (fluxo). Muitos conceitos são representados
naturalmente como streams: itens ficando disponíveis em uma fila, pedaços de dados sendo
puxados incrementalmente do sistema de arquivos quando o conjunto completo de dados é grande demais
para a memória do computador, ou dados chegando pela rede ao longo do tempo. Como
os streams são futuros, podemos usá-los com qualquer outro tipo de futuro e combiná-los
de maneiras interessantes. Por exemplo, podemos agrupar eventos em lotes para evitar disparar
chamadas de rede em excesso, definir tempos limite (timeouts) em sequências de operações de longa
duração, ou limitar a frequência de eventos de interface de usuário para evitar trabalho desnecessário.

Vimos uma sequência de itens de volta no Capítulo 13, quando analisamos a trait `Iterator`
na seção [“A Trait `Iterator` e o Método `next`”][iterator-trait]<!-- ignore -->, mas
há duas diferenças entre iteradores e o receptor de canal assíncrono. A primeira diferença
é o tempo: iteradores são síncronos, enquanto o receptor de canal é assíncrono. A segunda diferença
é a API. Ao trabalhar diretamente com `Iterator`, chamamos seu método síncrono `next`.
Com o stream `trpl::Receiver` em particular, chamamos um método assíncrono `recv`.
Caso contrário, essas APIs parecem muito semelhantes, e essa semelhança não é coincidência.
Um stream é como uma forma assíncrona de iteração. No entanto, enquanto o `trpl::Receiver`
espera especificamente para receber mensagens, a API de stream de propósito geral é muito mais
ampla: ela fornece o próximo item da mesma forma que o `Iterator` faz, mas de forma assíncrona.

A semelhança entre iteradores e streams em Rust significa que podemos criar um stream
a partir de qualquer iterador. Assim como com um iterador, podemos trabalhar com um stream
chamando seu método `next` e, em seguida, aguardando (`await`) o resultado, como na Listagem
17-21, que ainda não vai compilar.

<Listing number="17-21" caption="Criando um stream a partir de um iterador e imprimindo seus valores" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Começamos com um array de números, que convertemos em um iterador e, em seguida,
chamamos `map` para dobrar todos os valores. Depois, convertemos o iterador em um
stream usando a função `trpl::stream_from_iter`. Em seguida, iteramos sobre os
itens no stream conforme eles chegam com o loop `while let`.

Infelizmente, quando tentamos executar o código, ele não compila e relata
que não há nenhum método `next` disponível:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Como esta saída explica, o motivo do erro do compilador é que precisamos da trait
correta no escopo para poder usar o método `next`. Dada a nossa discussão
até agora, você poderia esperar razoavelmente que essa trait fosse `Stream`, mas na verdade
é `StreamExt`. Abreviação de _extension_ (extensão), `Ext` é um padrão comum na
comunidade Rust para estender uma trait com outra.

A trait `Stream` define uma interface de baixo nível que efetivamente combina as
traits `Iterator` e `Future`. A `StreamExt` fornece um conjunto de APIs de nível superior
sobre a `Stream`, incluindo o método `next`, bem como outros métodos utilitários
semelhantes aos fornecidos pela trait `Iterator`. `Stream` e `StreamExt` ainda
não fazem parte da biblioteca padrão do Rust, mas a maioria das crates do ecossistema
usa definições semelhantes.

A correção para o erro do compilador é adicionar uma instrução `use` para
`trpl::StreamExt`, como na Listagem 17-22.

<Listing number="17-22" caption="Usando com sucesso um iterador como base para um stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

Com todas essas peças reunidas, este código funciona do jeito que queremos! Além
disso, agora que temos `StreamExt` no escopo, podemos usar todos os seus métodos
utilitários, assim como fazemos com iteradores.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
