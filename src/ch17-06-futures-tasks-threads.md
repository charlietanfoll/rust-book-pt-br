## Juntando Tudo: Futures, Tasks e Threads

Como vimos no [Capítulo 16][ch16]<!-- ignore -->, as threads fornecem uma abordagem
para concorrência. Vimos outra abordagem neste capítulo: usar async com
futures e streams. Se você está se perguntando quando escolher um método em
detrimento do outro, a resposta é: depende! E, em muitos casos, a escolha não é
threads _ou_ async, mas sim threads _e_ async.

Muitos sistemas operacionais fornecem modelos de concorrência baseados em
threads há décadas, e muitas linguagens de programação os suportam como resultado.
No entanto, esses modelos não vêm sem suas compensações (trade-offs). Em muitos
sistemas operacionais, eles consomem uma boa quantidade de memória para cada
thread. As threads também são uma opção apenas quando seu sistema operacional e
hardware as suportam. Ao contrário dos computadores de desktop e celulares
comuns, alguns sistemas embarcados não possuem um SO, portanto também não
possuem threads.

O modelo async fornece um conjunto diferente — e ultimamente complementar — de
compensações. No modelo async, operações concorrentes não exigem suas próprias
threads. Em vez disso, elas podem rodar em tarefas (tasks), como quando usamos
`trpl::spawn_task` para iniciar um trabalho a partir de uma função síncrona na
seção de streams. Uma tarefa é semelhante a uma thread, mas em vez de ser
gerenciada pelo sistema operacional, ela é gerenciada por código em nível de
biblioteca: o runtime.

Há um motivo para as APIs de criação de threads (`spawn` de threads) e de criação
de tarefas (`spawn` de tarefas) serem tão semelhantes. As threads atuam como um
limite para conjuntos de operações síncronas; a concorrência é possível _entre_
threads. As tarefas atuam como um limite para conjuntos de operações
_assíncronas_; a concorrência é possível tanto _entre_ quanto _dentro_ das
tarefas, porque uma tarefa pode alternar entre futures em seu corpo. Por fim, as
futures são a unidade de concorrência mais granular do Rust, e cada future pode
representar uma árvore de outras futures. O runtime — especificamente, seu
executor — gerencia as tarefas, e as tarefas gerenciam as futures. Nesse
sentido, as tarefas são semelhantes a threads leves, gerenciadas pelo runtime,
com capacidades adicionais que vêm de serem gerenciadas por um runtime em vez
de pelo sistema operacional.

Isso não significa que as tarefas assíncronas sejam sempre melhores do que as
threads (ou vice-versa). A concorrência com threads é, de certa forma, um modelo
de programação mais simples do que a concorrência com `async`. Isso pode ser uma
força ou uma fraqueza. As threads são um pouco do tipo "dispare e esqueça" (*fire
and forget*); elas não têm equivalente nativo a uma future, então simplesmente
executam até a conclusão sem serem interrompidas, exceto pelo próprio sistema
operacional.

E acaba que threads e tarefas frequentemente funcionam muito bem juntas, porque
as tarefas podem (pelo menos em alguns runtimes) ser movidas entre threads. De
fato, por baixo dos panos, o runtime que temos usado — incluindo as funções
`spawn_blocking` e `spawn_task` — é multithreaded por padrão! Muitos runtimes
usam uma abordagem chamada *work stealing* (roubo de trabalho) para mover
transparentemente tarefas entre threads, com base em como as threads estão sendo
utilizadas no momento, para melhorar o desempenho geral do sistema. Essa
abordagem realmente requer threads _e_ tarefas, e portanto, futures.

Ao pensar sobre qual método usar e quando, considere estas regras gerais:

- Se o trabalho for _muito paralelizável_ (ou seja, limitado por CPU), como
  processar um monte de dados onde cada parte pode ser processada separadamente,
  as threads são uma escolha melhor.
- Se o trabalho for _muito concorrente_ (ou seja, limitado por E/S ou I/O), como
  lidar com mensagens de várias fontes diferentes que podem chegar em intervalos
  diferentes ou taxas diferentes, o async é uma escolha melhor.

E se você precisar tanto de paralelismo quanto de concorrência, não precisa
escolher entre threads e async. Você pode usá-los juntos livremente, deixando cada
um desempenhar o papel no qual é melhor. Por exemplo, a Listagem 17-25 mostra
um exemplo bastante comum desse tipo de mistura em código Rust do mundo real.

<Listing number="17-25" caption="Enviando mensagens com código bloqueante em uma thread e aguardando (awaiting) as mensagens em um bloco async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Começamos criando um canal assíncrono, depois criamos uma thread que assume a
propriedade do lado do transmissor (sender) do canal usando a palavra-chave
`move`. Dentro da thread, enviamos os números de 1 a 10, dormindo por um segundo
entre cada um. Finalmente, executamos uma future criada com um bloco async passado
para `trpl::block_on`, exatamente como fizemos ao longo do capítulo. Nessa
future, aguardamos por essas mensagens, assim como nos outros exemplos de envio de
mensagens que vimos.

Para retornar ao cenário com o qual abrimos o capítulo, imagine executar um
conjunto de tarefas de codificação de vídeo usando uma thread dedicada (porque a
codificação de vídeo é limitada por processamento) mas notificando a interface
de usuário (UI) de que essas operações foram concluídas com um canal assíncrono.
Há incontáveis exemplos desses tipos de combinações em casos de uso do mundo
real.

## Resumo

Esta não é a última vez que você verá concorrência neste livro. O projeto no
[Capítulo 21][ch21]<!-- ignore --> aplicará esses conceitos em uma situação mais
realista do que os exemplos mais simples discutidos aqui e comparará a resolução
de problemas usando threads versus tarefas e futures de forma mais direta.

Não importa qual dessas abordagens você escolha, o Rust oferece as ferramentas de
que você precisa para escrever código seguro, rápido e concorrente — seja para
um servidor web de alto rendimento ou para um sistema operacional embarcado.

Em seguida, falaremos sobre maneiras idiomáticas de modelar problemas e estruturar
soluções à medida que seus programas em Rust crescem. Além disso, discutiremos
como os idiomas do Rust se relacionam com aqueles que você talvez já conheça da
programação orientada a objetos.

[ch16]: ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
