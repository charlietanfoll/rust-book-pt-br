<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## Transferir dados entre threads com passagem de mensagens

Uma abordagem cada vez mais popular para garantir concorrência segura é a passagem de mensagens, onde threads ou atores se comunicam enviando uns aos outros mensagens contendo dados. Aqui está a ideia resumida em um slogan da [documentação da linguagem Go](https://golang.org/doc/effective_go.html#concurrency):
"Não se comunique compartilhando memória; em vez disso, compartilhe memória comunicando-se."

Para realizar a concorrência por envio de mensagens, a biblioteca padrão do Rust fornece uma implementação de canais. Um _canal_ (channel) é um conceito geral de programação pelo qual os dados são enviados de uma thread para outra.

Você pode imaginar um canal na programação como sendo semelhante a um canal direcional de água, como um riacho ou um rio. Se você colocar algo como um pato de borracha em um rio, ele flutuará rio abaixo até o final do curso d'água.

Um canal tem duas metades: um transmissor e um receptor. A metade do transmissor é o local a montante onde você coloca o pato de borracha no rio, e a metade do receptor é onde o pato de borracha acaba a jusante. Uma parte do seu código chama métodos no transmissor com os dados que você deseja enviar, e outra parte verifica a extremidade receptora em busca de mensagens que chegam. Diz-se que um canal está _fechado_ (closed) se a metade do transmissor ou do receptor for descartada.

Aqui, vamos construir um programa que possui uma thread para gerar valores e enviá-los por um canal, e outra thread que receberá os valores e os imprimirá. Enviaremos valores simples entre threads usando um canal para ilustrar o recurso. Assim que estiver familiarizado com a técnica, você poderá usar canais para quaisquer threads que precisem se comunicar, como um sistema de chat ou um sistema onde várias threads realizam partes de um cálculo e enviam as partes para uma única thread que agrega os resultados.

Primeiro, na Listagem 16-6, criaremos um canal, mas não faremos nada com ele.
Observe que isso ainda não vai compilar porque o Rust não consegue identificar qual tipo de valor queremos enviar pelo canal.

<Listing number="16-6" file-name="src/main.rs" caption="Criando um canal e atribuindo as duas metades a `tx` e `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Criamos um novo canal usando a função `mpsc::channel`; `mpsc` significa _multiple producer, single consumer_ (múltiplos produtores, único consumidor). Em suma, a forma como a biblioteca padrão do Rust implementa os canais significa que um canal pode ter várias extremidades de _envio_ que produzem valores, mas apenas uma extremidade de _recepção_ que consome esses valores. Imagine vários riachos fluindo juntos para um grande rio: tudo o que for enviado por qualquer um dos riachos acabará em um único rio no final. Começaremos com um único produtor por enquanto, mas adicionaremos múltiplos produtores quando fizermos este exemplo funcionar.

A função `mpsc::channel` retorna uma tupla, cujo primeiro elemento é a extremidade de envio — o transmissor — e o segundo elemento é a extremidade de recepção — o receptor. As abreviações `tx` e `rx` são tradicionalmente usadas em muitos campos para _transmissor_ (transmitter) e _receptor_ (receiver), respectivamente, então nomeamos nossas variáveis assim para indicar cada extremidade. Estamos usando uma instrução `let` com um padrão que desestrutura as tuplas; discutiremos o uso de padrões em instruções `let` e a desestruturação no Capítulo 19. Por enquanto, saiba que usar uma instrução `let` dessa forma é uma abordagem conveniente para extrair as partes da tupla retornada por `mpsc::channel`.

Vamos mover a extremidade transmissora para uma thread gerada (_spawned thread_) e fazer com que ela envie uma única string para que a thread gerada se comunique com a thread principal, conforme mostrado na Listagem 16-7. Isso é como colocar um pato de borracha no rio a montante ou enviar uma mensagem de chat de uma thread para outra.

<Listing number="16-7" file-name="src/main.rs" caption='Movendo `tx` para uma thread gerada e enviando `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Novamente, estamos usando `thread::spawn` para criar uma nova thread e, em seguida, usando `move` para mover `tx` para dentro do closure, de modo que a thread gerada seja dona de `tx`. A thread gerada precisa ser dona do transmissor para poder enviar mensagens através do canal.

O transmissor possui um método `send` que recebe o valor que queremos enviar. O método `send` retorna um tipo `Result<T, E>`, portanto, se o receptor já tiver sido descartado e não houver para onde enviar um valor, a operação de envio retornará um erro. Neste exemplo, estamos chamando `unwrap` para causar um _panic_ em caso de erro. Mas em uma aplicação real, lidaríamos com isso adequadamente: retorne ao Capítulo 9 para revisar estratégias de tratamento adequado de erros.

Na Listagem 16-8, obteremos o valor do receptor na thread principal. Isso é como recuperar o pato de borracha da água no final do rio ou receber uma mensagem de chat.

<Listing number="16-8" file-name="src/main.rs" caption='Recebendo o valor `"hi"` na thread principal e imprimindo-o'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

O receptor possui dois métodos úteis: `recv` e `try_recv`. Estamos usando `recv`, abreviação de _receive_ (receber), que bloqueará a execução da thread principal e aguardará até que um valor seja enviado pelo canal. Assim que um valor for enviado, `recv` o retornará em um `Result<T, E>`. Quando o transmissor for fechado, `recv` retornará um erro para sinalizar que não haverá mais valores chegando.

O método `try_recv` não bloqueia; em vez disso, ele retorna um `Result<T, E>` imediatamente: um valor `Ok` contendo uma mensagem, se houver uma disponível, e um valor `Err`, se não houver nenhuma mensagem no momento. Usar `try_recv` é útil se esta thread tiver outro trabalho a fazer enquanto espera por mensagens: poderíamos escrever um loop que chama `try_recv` periodicamente, lida com uma mensagem se houver uma disponível e, caso contrário, executa outras tarefas por um curto período até verificar novamente.

Usamos `recv` neste exemplo por simplicidade; não temos nenhum outro trabalho para a thread principal fazer além de esperar por mensagens, então bloquear a thread principal é apropriado.

Quando executamos o código na Listagem 16-8, veremos o valor impresso a partir da thread principal:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Perfeito!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### Transferência de propriedade por meio de canais

As regras de propriedade desempenham um papel vital no envio de mensagens porque ajudam você a escrever código concorrente seguro. Prevenir erros na programação concorrente é a vantagem de pensar sobre a propriedade em todos os seus programas Rust. Vamos fazer um experimento para mostrar como os canais e a propriedade funcionam juntos para evitar problemas: tentaremos usar o valor `val` na thread gerada _depois_ de tê-lo enviado pelo canal. Tente compilar o código na Listagem 16-9 para ver por que esse código não é permitido.

<Listing number="16-9" file-name="src/main.rs" caption="Tentando usar `val` após tê-lo enviado pelo canal">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Aqui, tentamos imprimir `val` depois de tê-lo enviado pelo canal via `tx.send`. Permitir isso seria uma má ideia: assim que o valor fosse enviado para outra thread, essa thread poderia modificá-lo ou descartá-lo antes que tentássemos usar o valor novamente. Em potencial, as modificações da outra thread poderiam causar erros ou resultados inesperados devido a dados inconsistentes ou inexistentes. No entanto, o Rust nos dá um erro se tentarmos compilar o código da Listagem 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Nosso erro de concorrência causou um erro em tempo de compilação. A função `send` assume a propriedade de seu parâmetro e, quando o valor é movido, o receptor assume a propriedade dele. Isso nos impede de usar acidentalmente o valor novamente após o envio; o sistema de propriedade verifica se está tudo bem.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### Enviando múltiplos valores

O código na Listagem 16-8 compilou e executou, mas não mostrou claramente que duas threads separadas estavam conversando entre si através do canal.

Na Listagem 16-10, fizemos algumas modificações que provarão que o código na Listagem 16-8 está sendo executado concorrentemente: a thread gerada agora enviará várias mensagens e fará uma pausa de um segundo entre cada mensagem.

<Listing number="16-10" file-name="src/main.rs" caption="Enviando várias mensagens e fazendo uma pausa entre cada uma delas">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Desta vez, a thread gerada tem um vetor de strings que queremos enviar para a thread principal. Iteramos sobre elas, enviando cada uma individualmente, e fazemos uma pausa entre cada uma chamando a função `thread::sleep` com um valor `Duration` de um segundo.

Na thread principal, não estamos mais chamando a função `recv` explicitamente: em vez disso, estamos tratando `rx` como um iterador. Para cada valor recebido, nós o imprimimos. Quando o canal for fechado, a iteração terminará.

Ao executar o código na Listagem 16-10, você deverá ver a seguinte saída com uma pausa de um segundo entre cada linha:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Como não temos nenhum código que pause ou atrase no loop `for` na thread principal, podemos dizer que a thread principal está esperando para receber valores da thread gerada.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### Criando múltiplos produtores

Anteriormente mencionamos que `mpsc` era um acrônimo para _multiple producer, single consumer_ (múltiplos produtores, único consumidor). Vamos colocar o `mpsc` em prática e expandir o código da Listagem 16-10 para criar várias threads que enviam valores para o mesmo receptor. Podemos fazer isso clonando o transmissor, conforme mostrado na Listagem 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Enviando múltiplas mensagens a partir de múltiplos produtores">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Desta vez, antes de criarmos a primeira thread gerada, chamamos `clone` no transmissor. Isso nos dará um novo transmissor que podemos passar para a primeira thread gerada. Passamos o transmissor original para uma segunda thread gerada. Isso nos dá duas threads, cada uma enviando mensagens diferentes para o mesmo receptor.

Quando você executa o código, sua saída deve ser semelhante a esta:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Você pode ver os valores em outra ordem, dependendo do seu sistema. É isso que torna a concorrência interessante e também difícil. Se você experimentar o `thread::sleep`, dando a ele vários valores nas diferentes threads, cada execução será mais não determinística e criará uma saída diferente a cada vez.

Agora que vimos como os canais funcionam, vamos analisar um método diferente de concorrência.