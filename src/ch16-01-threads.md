## Usando Threads para Executar Código Simultaneamente

Na maioria dos sistemas operacionais atuais, o código de um programa executado é
executado em um _processo_, e o sistema operacional gerenciará vários processos ao
mesmo tempo. Dentro de um programa, você também pode ter partes independentes que
são executadas simultaneamente. Os recursos que executam essas partes
independentes são chamados de _threads_. Por exemplo, um servidor Web pode ter
várias threads para poder responder a mais de uma solicitação ao mesmo tempo.

Dividir a computação do seu programa em várias threads para executar várias
tarefas ao mesmo tempo pode melhorar o desempenho, mas também adiciona
complexidade. Como as threads podem ser executadas simultaneamente, não há
garantia inerente sobre a ordem em que partes do seu código em diferentes
threads serão executadas. Isso pode levar a problemas, tais como:

- Condições de corrida (_race conditions_), nas quais as threads estão acessando dados ou recursos
  em uma ordem inconsistente
- Impasses (_deadlocks_), nos quais duas threads estão esperando uma pela outra, impedindo
  que ambas continuem
- Bugs que só acontecem em determinadas situações e são difíceis de reproduzir e
  corrigir de forma confiável

O Rust tenta mitigar os efeitos negativos do uso de threads, mas programar
em um contexto multithread ainda exige um pensamento cuidadoso e requer uma
estrutura de código diferente daquela em programas executados em uma única
thread.

As linguagens de programação implementam threads de algumas maneiras diferentes,
e muitos sistemas operacionais fornecem uma API que a linguagem de programação
pode chamar para criar novas threads. A biblioteca padrão do Rust usa um modelo
de implementação de threads _1:1_, em que um programa usa uma thread do sistema
operacional por uma thread da linguagem. Existem crates que implementam outros
modelos de gerenciamento de threads que fazem compensações (_trade-offs_)
diferentes do modelo 1:1. (O sistema assíncrono do Rust, que veremos no próximo
capítulo, também fornece outra abordagem para concorrência.)

### Criando uma Nova Thread com `spawn`

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos a ela
uma closure (falamos sobre closures no Capítulo 13) contendo o código que queremos
executar na nova thread. O exemplo na Listagem 16-1 imprime algum texto a partir
da thread principal e outro texto a partir de uma nova thread.

<Listing number="16-1" file-name="src/main.rs" caption="Criando uma nova thread para imprimir uma coisa enquanto a thread principal imprime outra">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Observe que, quando a thread principal de um programa Rust é concluída, todas
as threads criadas são encerradas, independentemente de terem terminado de ser
executadas ou não. A saída deste programa pode ser um pouco diferente a cada vez,
mas será semelhante à seguinte:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

As chamadas para `thread::sleep` forçam uma thread a parar sua execução por um
curto período de tempo, permitindo que uma thread diferente seja executada. As
threads provavelmente se alternarão, mas isso não é garantido: depende de como
o seu sistema operacional escalona as threads. Nesta execução, a thread
principal foi impressa primeiro, mesmo que a instrução de impressão da thread
criada apareça primeiro no código. E mesmo tendo dito para a thread criada
imprimir até que `i` seja `9`, ela chegou apenas a `5` antes que a thread
principal fosse encerrada.

Se você executar este código e vir apenas a saída da thread principal, ou não
vir nenhuma sobreposição, tente aumentar os números nos intervalos para criar
mais oportunidades para o sistema operacional alternar entre as threads.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### Esperando Todas as Threads Terminarem

O código na Listagem 16-1 não apenas interrompe a thread criada prematuramente na
maioria das vezes devido ao término da thread principal, mas, como não há
garantia sobre a ordem em que as threads são executadas, também não podemos
garantir que a thread criada conseguirá ser executada!

Podemos resolver o problema da thread criada não ser executada ou de terminar
prematuremente salvando o valor de retorno de `thread::spawn` em uma variável. O
tipo de retorno de `thread::spawn` é `JoinHandle<T>`. Um `JoinHandle<T>` é um
valor possuído (_owned value_) que, quando chamamos o método `join` nele, espera
que sua thread termine. A Listagem 16-2 mostra como usar o `JoinHandle<T>` da
thread que criamos na Listagem 16-1 e como chamar `join` para garantir que a
thread criada termine antes que `main` seja encerrada.

<Listing number="16-2" file-name="src/main.rs" caption="Salvando um `JoinHandle<T>` de `thread::spawn` para garantir que a thread seja executada até o fim">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Chamar `join` no identificador (_handle_) bloqueia a thread atualmente em
execução até que a thread representada pelo identificador termine. _Bloquear_
uma thread significa que a thread é impedida de realizar trabalho ou de sair.
Como colocamos a chamada para `join` após o loop `for` da thread principal, a
execução da Listagem 16-2 deve produzir uma saída semelhante a esta:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

As duas threads continuam se alternando, mas a thread principal espera devido
à chamada para `handle.join()` e não termina até que a thread criada seja
concluída.

Mas vamos ver o que acontece quando movemos `handle.join()` para antes do loop
`for` em `main`, desta forma:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

A thread principal esperará a thread criada terminar e só então executará seu
loop `for`, de modo que a saída não será mais intercalada, conforme mostrado aqui:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Pequenos detalhes, como onde `join` é chamado, podem afetar se suas threads são
executadas ao mesmo tempo ou não.

### Usando Closures `move` com Threads

Frequentemente usaremos a palavra-chave `move` com closures passadas para
`thread::spawn` porque a closure assumirá a propriedade (_ownership_) dos valores
que ela usa do ambiente, transferindo assim a propriedade desses valores de uma
thread para outra. Em [“Capturando Referências ou Movendo a Propriedade”][capture]<!-- ignore
--> no Capítulo 13, discutimos o `move` no contexto de closures. Agora vamos nos
concentrar mais na interação entre `move` e `thread::spawn`.

Note na Listagem 16-1 que a closure que passamos para `thread::spawn` não recebe
nenhum argumento: não estamos usando nenhum dado da thread principal no código da
thread criada. Para usar dados da thread principal na thread criada, a closure
da thread criada deve capturar os valores de que precisa. A Listagem 16-3 mostra
uma tentativa de criar um vetor na thread principal e usá-lo na thread criada.
No entanto, isso ainda não funcionará, como você verá em um momento.

<Listing number="16-3" file-name="src/main.rs" caption="Tentando usar um vetor criado pela thread principal em outra thread">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

A closure usa `v`, portanto, ela capturará `v` e o tornará parte do ambiente da
closure. Como `thread::spawn` executa essa closure em uma nova thread, devemos
ser capazes de acessar `v` dentro dessa nova thread. Mas quando compilamos este
exemplo, obtemos o seguinte erro:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

O Rust _infere_ como capturar `v`, e como o `println!` precisa apenas de uma
referência para `v`, a closure tenta emprestar (`borrow`) `v`. No entanto, há um
problema: o Rust não consegue dizer por quanto tempo a thread criada será
executada, então ele não sabe se a referência a `v` será sempre válida.

A Listagem 16-4 fornece um cenário que tem mais probabilidade de ter uma
referência a `v` que não será válida.

<Listing number="16-4" file-name="src/main.rs" caption="Uma thread com uma closure que tenta capturar uma referência a `v` de uma thread principal que descarta (`drops`) `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Se o Rust nos permitisse executar este código, haveria a possibilidade de a
thread criada ser colocada imediatamente em segundo plano sem ser executada. A
thread criada tem uma referência a `v` dentro dela, mas a thread principal
descartar (`drops`) `v` imediatamente, usando a função `drop` que discutimos no
Capítulo 15. Então, quando a thread criada começa a ser executada, `v` não é
mais válido, portanto, uma referência a ele também é inválida. Oh, não!

Para corrigir o erro de compilação na Listagem 16-3, podemos usar o conselho da
mensagem de erro:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a assumir
a propriedade dos valores que está usando, em vez de permitir que o Rust infira
que deve emprestar os valores. A modificação na Listagem 16-3 mostrada na
Listagem 16-5 será compilada e executada conforme pretendemos.

<Listing number="16-5" file-name="src/main.rs" caption="Usando a palavra-chave `move` para forçar uma closure a assumir a propriedade dos valores que ela usa">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Podemos ficar tentados a tentar a mesma coisa para corrigir o código na Listagem
16-4 onde a thread principal chamou `drop` usando uma closure `move`. No entanto,
essa correção não funcionará porque o que a Listagem 16-4 está tentando fazer é
proibido por um motivo diferente. Se adicionássemos `move` à closure, moveríamos
`v` para o ambiente da closure, e não poderíamos mais chamar `drop` nele na
thread principal. Obteríamos este erro de compilação:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

As regras de propriedade do Rust nos salvaram novamente! Obtivemos um erro do
código na Listagem 16-3 porque o Rust estava sendo conservador e apenas
emprestando `v` para a thread, o que significava que a thread principal poderia
teoricamente invalidar a referência da thread criada. Ao dizer ao Rust para
mover a propriedade de `v` para a thread criada, estamos garantindo ao Rust que
a thread principal não usará mais `v`. Se alterarmos a Listagem 16-4 da mesma
forma, estaremos violando as regras de propriedade ao tentar usar `v` na
thread principal. A palavra-chave `move` substitui o padrão conservador de
empréstimo do Rust; ela não nos permite violar as regras de propriedade.

Agora que cobrimos o que são threads e os métodos fornecidos pela API de
threads, vamos analisar algumas situações em que podemos usar threads.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
