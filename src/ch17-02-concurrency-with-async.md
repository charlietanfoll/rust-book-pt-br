<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Aplicando Concorrência com Async

Nesta seção, aplicaremos async a alguns dos mesmos desafios de concorrência
que enfrentamos com threads no Capítulo 16. Como já discutimos muitas das
ideias principais lá, nesta seção focaremos no que é diferente entre
threads e *futures*.

Em muitos casos, as APIs para trabalhar com concorrência usando async são muito
semelhantes àquelas para usar threads. Em outros casos, elas acabam sendo bem
diferentes. Mesmo quando as APIs _parecem_ semelhantes entre threads e async, elas
geralmente têm comportamentos diferentes — e quase sempre têm características de
desempenho diferentes.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Criando uma Nova Tarefa com `spawn_task`

A primeira operação que abordamos na seção [“Criando uma Nova Thread com
`spawn`”][thread-spawn]<!-- ignore --> no Capítulo 16 foi contar em
duas threads separadas. Vamos fazer o mesmo usando async. O *crate* `trpl` fornece
uma função `spawn_task` que se parece muito com a API `thread::spawn`, e
uma função `sleep` que é uma versão async da API `thread::sleep`. Nós podemos
usar ambas juntas para implementar o exemplo de contagem, conforme mostrado na Listagem 17-6.

<Listing number="17-6" caption="Criando uma nova tarefa para imprimir uma coisa enquanto a tarefa principal imprime outra" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Como ponto de partida, configuramos nossa função `main` com `trpl::block_on` para
que nossa função de nível superior possa ser async.

> Nota: Daqui em diante neste capítulo, cada exemplo incluirá este
> exato mesmo código de envelopamento com `trpl::block_on` no `main`, então muitas vezes o omitiremos
> assim como fazemos com o `main`. Lembre-se de incluí-lo no seu código!

Em seguida, escrevemos dois loops dentro desse bloco, cada um contendo uma chamada
`trpl::sleep`, que espera por meio segundo (500 milissegundos) antes de enviar a próxima
mensagem. Colocamos um loop no corpo de um `trpl::spawn_task` e o outro em um
loop `for` de nível superior. Também adicionamos um `await` após as chamadas de `sleep`.

Este código se comporta de maneira semelhante à implementação baseada em threads — incluindo o
fato de que você pode ver as mensagens aparecerem em uma ordem diferente no seu
próprio terminal quando executá-lo:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Esta versão para assim que o loop `for` no corpo do bloco async principal
termina, porque a tarefa gerada por `spawn_task` é encerrada quando a
função `main` termina. Se você quiser que ela execute até a
conclusão da tarefa, precisará usar um identificador de junção (*join handle*) para esperar a primeira tarefa
ser concluída. Com threads, usamos o método `join` para “bloquear” até que
a thread terminasse de ser executada. Na Listagem 17-7, podemos usar `await` para fazer a mesma coisa,
porque o identificador da tarefa em si é uma *future*. Seu tipo `Output` é um `Result`, então
também o desempacotamos (*unwrap*) após aguardá-lo.

<Listing number="17-7" caption="Usando `await` com um identificador de junção para executar uma tarefa até a conclusão" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Esta versão atualizada é executada até que _ambos_ os loops terminem:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Até agora, parece que async e threads nos dão resultados semelhantes, apenas com
sintaxe diferente: usar `await` em vez de chamar `join` no identificador de junção,
e aguardar pelas chamadas de `sleep`.

A maior diferença é que não precisamos gerar outra thread do sistema
operacional para fazer isso. Na verdade, nem precisamos gerar uma tarefa aqui. Como
blocos async compilam para *futures* anônimas, podemos colocar cada loop em um bloco
async e fazer o ambiente de execução (*runtime*) executar ambos até a conclusão usando a função
`trpl::join`.

Na seção [“Esperando Todas as Threads Terminarem”][join-handles]<!-- ignore -->
no Capítulo 16, mostramos como usar o método `join` no tipo
`JoinHandle` retornado quando você chama `std::thread::spawn`. A função `trpl::join`
é semelhante, mas para *futures*. Quando você passa duas *futures* para ela, ela produz
uma única nova *future* cuja saída é uma tupla contendo a saída de cada
*future* que você passou assim que _ambas_ são concluídas. Portanto, na Listagem 17-8, usamos
`trpl::join` para esperar tanto `fut1` quanto `fut2` terminarem. Nós _não_ aguardamos
`fut1` e `fut2`, mas sim a nova *future* produzida por `trpl::join`. Nós
ignoramos a saída, porque é apenas uma tupla contendo dois valores vazios (unit).

<Listing number="17-8" caption="Usando `trpl::join` para aguardar duas *futures* anônimas" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Quando executamos isso, vemos ambas as *futures* rodarem até a conclusão:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Agora, você verá exatamente a mesma ordem todas as vezes, o que é muito diferente do
que vimos com threads e com `trpl::spawn_task` na Listagem 17-7. Isso ocorre
porque a função `trpl::join` é _justa_ (*fair*), o que significa que ela verifica cada *future*
igualmente a cada vez, alternando entre elas, e nunca deixa uma disparar na frente se
a outra estiver pronta. Com threads, o sistema operacional decide qual thread
verificar e por quanto tempo deixá-la rodar. Com o Rust assíncrono, o *runtime* decide qual
tarefa verificar. (Na prática, os detalhes se tornam complicados porque um *runtime* assíncrono
pode usar threads do sistema operacional por baixo dos panos como parte de como ele
gerencia concorrência, então garantir justiça pode dar mais trabalho para um
*runtime* — mas ainda é possível!) Os *runtimes* não precisam garantir justiça para
nenhuma operação específica, e eles frequentemente oferecem diferentes APIs para permitir que você escolha
se quer ou não justiça.

Tente algumas destas variações ao aguardar pelas *futures* e veja o que elas fazem:

- Remova o bloco async de um ou de ambos os loops.
- Aguarde cada bloco async imediatamente após defini-lo.
- Envolva apenas o primeiro loop em um bloco async e aguarde a *future* resultante
  após o corpo do segundo loop.

Para um desafio extra, veja se você consegue descobrir qual será a saída em
cada caso _antes_ de executar o código!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### Enviando Dados entre Duas Tarefas Usando Passagem de Mensagens

Compartilhar dados entre *futures* também será familiar: usaremos a passagem de
mensagens novamente, mas desta vez com versões assíncronas dos tipos e funções. Seguiremos
um caminho um pouco diferente do que fizemos na seção [“Transferindo Dados entre Threads
com Passagem de Mensagens”][message-passing-threads]<!-- ignore --> no
Capítulo 16 para ilustrar algumas das principais diferenças entre a concorrência baseada em threads e
baseada em *futures*. Na Listagem 17-9, começaremos com apenas um único
bloco async — _sem_ gerar uma tarefa separada da mesma forma que geramos uma thread separada.

<Listing number="17-9" caption="Criando um canal assíncrono e atribuindo as duas metades a `tx` e `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Aqui, usamos `trpl::channel`, uma versão assíncrona da API de canal de múltiplos produtores e
consumidor único que usamos com threads no Capítulo 16. A versão assíncrona
da API é apenas um pouco diferente da versão baseada em threads: ela
usa um receptor `rx` mutável em vez de imutável, e seu método `recv`
produz uma *future* que precisamos aguardar em vez de produzir o valor diretamente.
Agora podemos enviar mensagens do remetente para o receptor. Note que não
precisamos gerar uma thread separada ou mesmo uma tarefa; apenas precisamos aguardar a
chamada `rx.recv`.

O método síncrono `Receiver::recv` em `std::mpsc::channel` bloqueia até que ele
receba uma mensagem. O método `trpl::Receiver::recv` não faz isso, porque ele é
assíncrono. Em vez de bloquear, ele devolve o controle ao *runtime* até que uma
mensagem seja recebida ou o lado de envio do canal feche. Em contraste, nós
não aguardamos a chamada `send`, porque ela não bloqueia. Ela não precisa bloquear,
porque o canal para o qual estamos enviando não tem limite de tamanho (*unbounded*).

> Nota: Como todo este código assíncrono é executado em um bloco async em uma
> chamada `trpl::block_on`, tudo dentro dele pode evitar o bloqueio. No entanto, o
> código _fora_ dele irá bloquear até que a função `block_on` retorne. Esse é
> todo o objetivo da função `trpl::block_on`: ela permite que você _escolha_ onde
> bloquear em algum conjunto de código assíncrono e, portanto, onde fazer a transição entre
> código síncrono e assíncrono.

Note duas coisas sobre este exemplo. Primeiro, a mensagem chegará imediatamente.
Segundo, embora usemos uma *future* aqui, ainda não há concorrência.
Tudo na listagem acontece em sequência, exatamente como aconteceria se não
houvesse *futures* envolvidas.

Vamos abordar a primeira parte enviando uma série de mensagens e pausando (*sleeping*)
entre elas, conforme mostrado na Listagem 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Enviando e recebendo várias mensagens pelo canal assíncrono e pausando com um `await` entre cada mensagem" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Além de enviar as mensagens, precisamos recebê-las. Neste caso,
como sabemos quantas mensagens estão chegando, poderíamos fazer isso manualmente
chamando `rx.recv().await` quatro vezes. No mundo real, porém, geralmente
estaremos esperando por um número _desconhecido_ de mensagens, então precisamos continuar esperando
até determinarmos que não há mais mensagens.

Na Listagem 16-10, usamos um loop `for` para processar todos os itens recebidos de um
canal síncrono. O Rust ainda não tem uma maneira de usar um loop `for` com uma
série de itens _produzida assincronamente_, no entanto, precisamos usar um loop que
ainda não vimos: o loop condicional `while let`. Esta é a versão de loop do
construto `if let` que vimos na seção [“Controle de Fluxo Conciso com `if
let` e `let...else`”][if-let]<!-- ignore --> no Capítulo 6. O loop
continuará executando enquanto o padrão especificado continuar correspondendo
ao valor.

A chamada `rx.recv` produz uma *future*, a qual aguardamos. O *runtime* pausará
a *future* até que ela esteja pronta. Assim que uma mensagem chegar, a *future* será resolvida
para `Some(message)` quantas vezes uma mensagem chegar. Quando o canal fechar,
independentemente de _qualquer_ mensagem ter chegado, a *future* será resolvida
para `None` para indicar que não há mais valores e, portanto, devemos parar de sondar (*poll*)
— ou seja, parar de aguardar.

O loop `while let` junta tudo isso. Se o resultado de chamar
`rx.recv().await` for `Some(message)`, obtemos acesso à mensagem e podemos
usá-la no corpo do loop, assim como faríamos com `if let`. Se o resultado for
`None`, o loop termina. Cada vez que o loop é concluído, ele atinge o ponto de espera
novamente, de modo que o *runtime* o pausa novamente até que outra mensagem chegue.

O código agora envia e recebe com sucesso todas as mensagens.
Infelizmente, ainda há alguns problemas. Por um lado, as
mensagens não chegam em intervalos de meio segundo. Elas chegam todas de uma vez, 2
segundos (2.000 milissegundos) depois que iniciamos o programa. Por outro lado, este
programa também nunca sai! Em vez disso, ele espera para sempre por novas mensagens. Você
precisará encerrá-lo usando <kbd>ctrl</kbd>-<kbd>C</kbd>.

#### O Código Dentro de um Bloco Async É Executado Linearmente

Vamos começar examinando por que as mensagens chegam todas de uma vez após o atraso total,
em vez de chegarem com atrasos entre cada uma. Dentro de um determinado bloco async,
a ordem em que as palavras-chave `await` aparecem no código também é a ordem
em que elas são executadas quando o programa é executado.

Há apenas um bloco async na Listagem 17-10, então tudo nele é executado
linearmente. Ainda não há concorrência. Todas as chamadas `tx.send` acontecem,
interspersed (intercaladas) com todas as chamadas `trpl::sleep` e seus pontos de espera associados.
Só então o loop `while let` consegue passar por qualquer um dos
pontos de espera nas chamadas `recv`.

Para obter o comportamento que queremos, onde o atraso de pausa acontece entre cada
mensagem, precisamos colocar as operações `tx` e `rx` em seus próprios blocos async,
conforme mostrado na Listagem 17-11. Então o *runtime* pode executar cada um deles separadamente
usando `trpl::join`, assim como na Listagem 17-8. Mais uma vez, aguardamos o resultado da
chamada de `trpl::join`, e não as *futures* individuais. Se aguardássemos as
*futures* individuais em sequência, acabaríamos voltando a um fluxo sequencial — exatamente
o que estamos tentando _não_ fazer.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Separando `send` e `recv` em seus próprios blocos `async` e aguardando as *futures* para esses blocos" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Com o código atualizado na Listagem 17-11, as mensagens são impressas em
intervalos de 500 milissegundos, em vez de todas de uma vez após 2 segundos.

#### Movendo a Posse para Dentro de um Bloco Async

O programa ainda nunca sai, no entanto, por causa da maneira como o loop `while let`
interage com `trpl::join`:

- A *future* retornado de `trpl::join` é concluída apenas uma vez que _ambas_ as *futures*
  passadas para ela tenham sido concluídas.
- A *future* `tx_fut` é concluída assim que termina de dormir após enviar a última
  mensagem em `vals`.
- A *future* `rx_fut` não será concluída até que o loop `while let` termine.
- O loop `while let` não terminará até que aguardar `rx.recv` produza `None`.
- Aguardar `rx.recv` retornará `None` apenas quando a outra ponta do canal
  for fechada.
- O canal fechará apenas se chamarmos `rx.close` ou quando o lado remetente,
  `tx`, for descartado (*dropped*).
- Nós não chamamos `rx.close` em lugar nenhum, e `tx` não será descartado até que o
  bloco async mais externo passado para `trpl::block_on` termine.
- O bloco não pode terminar porque está bloqueado aguardando `trpl::join` ser concluído, o
  que nos leva de volta ao topo desta lista.

No momento, o bloco async onde enviamos as mensagens apenas _empresta_ (`borrows`) `tx`
porque enviar uma mensagem não requer posse (*ownership*), mas se pudéssemos _mover_
`tx` para dentro desse bloco async, ele seria descartado assim que esse bloco terminasse. Na
seção [“Capturando Referências ou Movendo a Posse”][capture-or-move]<!-- ignore -->
no Capítulo 13, você aprendeu como usar a palavra-chave `move` com *closures* e,
conforme discutido na seção [“Usando *Closures* `move` com
Threads”][move-threads]<!-- ignore --> no Capítulo 16, frequentemente precisamos
mover dados para dentro de *closures* ao trabalhar com threads. A mesma dinâmica básica
se aplica a blocos async, de modo que a palavra-chave `move` funciona com blocos async assim
como funciona com *closures*.

Na Listagem 17-12, alteramos o bloco usado para enviar mensagens de `async` para
`async move`.

<Listing number="17-12" caption="Uma revisão do código da Listagem 17-11 que é encerrado corretamente quando concluído" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Quando executamos _esta_ versão do código, ele é encerrado de forma graciosa após a última
mensagem ser enviada e recebida. Em seguida, vamos ver o que precisaria mudar para enviar
dados de mais de uma *future*.

#### Unindo Várias *Futures* com a Macro `join!`

Este canal assíncrono também é um canal de múltiplos produtores, então podemos chamar `clone`
em `tx` se quisermos enviar mensagens de várias *futures*, conforme mostrado na Listagem
17-13.

<Listing number="17-13" caption="Usando múltiplos produtores com blocos async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Primeiro, clonamos `tx`, criando `tx1` fora do primeiro bloco async. Nós movemos
`tx1` para dentro desse bloco assim como fizemos antes com `tx`. Então, mais tarde, movemos o
`tx` original para um _novo_ bloco async, onde enviamos mais mensagens com um
atraso um pouco mais lento. Por acaso colocamos este novo bloco async após o bloco
async para receber mensagens, mas ele poderia ir antes sem problemas. A chave é
a ordem em que as *futures* são aguardadas, e não aquela em que são criadas.

Ambos os blocos async para envio de mensagens precisam ser blocos `async move` para
que tanto `tx` quanto `tx1` sejam descartados quando esses blocos terminarem. Caso contrário,
acabaremos voltando para o mesmo loop infinito em que começamos.

Finalmente, mudamos de `trpl::join` para `trpl::join!` para lidar com a *future*
adicional: a macro `join!` aguarda um número arbitrário de *futures* onde conhecemos
o número de *futures* em tempo de compilação. Discutiremos a espera de uma coleção de
um número desconhecido de *futures* mais adiante neste capítulo.

Agora vemos todas as mensagens de ambas as *futures* remetentes e, como as *futures*
remetentes usam atrasos ligeiramente diferentes após o envio, as mensagens também são
recebidas nesses intervalos diferentes:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Exploramos como usar a passagem de mensagens para enviar dados entre *futures*, como o
código dentro de um bloco async é executado sequencialmente, como mover a posse para
dentro de um bloco async e como juntar várias *futures*. Em seguida, vamos discutir como e por que
dizer ao *runtime* que ele pode mudar para outra tarefa.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
