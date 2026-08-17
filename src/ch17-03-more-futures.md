<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Cedendo o Controle para o Runtime

Lembre-se da seção [“Nosso Primeiro Programa Assíncrono”][async-program]<!-- ignore -->
que em cada ponto de `await`, o Rust dá ao runtime a chance de pausar a
tarefa e alternar para outra se a future que está sendo aguardada não estiver pronta. O
inverso também é verdadeiro: o Rust _apenas_ pausa blocos assíncronos e devolve o controle
a um runtime em um ponto de `await`. Tudo entre os pontos de `await` é síncrono.

Isso significa que se você fizer muito trabalho em um bloco assíncrono sem um ponto de `await`,
essa future impedirá que quaisquer outras futures progridam. Às vezes, você pode
ouvir isso sendo referenciado como uma future _faminta_ (starving) de recursos em relação a outras futures. Em alguns casos,
isso pode não ser um grande problema. No entanto, se você estiver fazendo algum tipo de configuração custosa
ou trabalho de longa duração, ou se você tiver uma future que continuará executando alguma
tarefa específica indefinidamente, você precisará pensar sobre quando e onde devolver o
controle para o runtime.

Vamos simular uma operação de longa duração para ilustrar o problema de inanição (starvation),
e então explorar como resolvê-lo. A Listagem 17-14 introduz uma função `slow` (lenta).

<Listing number="17-14" caption="Usando `thread::sleep` para simular operações lentas" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Este código usa `std::thread::sleep` em vez de `trpl::sleep` para que a chamada
a `slow` bloqueie a thread atual por um determinado número de milissegundos. Podemos
usar `slow` para representar operações do mundo real que são de longa duração e
bloqueantes.

Na Listagem 17-15, usamos `slow` para emular a execução desse tipo de trabalho vinculado à CPU em
um par de futures.

<Listing number="17-15" caption="Chamando a função `slow` para simular operações lentas" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Cada future devolve o controle para o runtime apenas _após_ realizar um monte
de operações lentas. Se você executar este código, verá esta saída:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Assim como na Listagem 17-5, onde usamos `trpl::select` para colocar futures de busca de duas
URLs em uma corrida, o `select` ainda termina assim que `a` termina. No entanto, não há intercalação
entre as chamadas para `slow` nas duas futures. A future `a` faz todo
o seu trabalho até que a chamada a `trpl::sleep` seja aguardada, então a future `b` faz
todo o seu trabalho até que sua própria chamada a `trpl::sleep` seja aguardada e, finalmente, a
future `a` é concluída. Para permitir que ambas as futures progridam entre suas tarefas lentas,
precisamos de pontos de `await` para podermos devolver o controle ao runtime. Isso
significa que precisamos de algo que possamos aguardar com `await`!

Já podemos ver esse tipo de transferência acontecendo na Listagem 17-15: se
removêssemos o `trpl::sleep` no final da future `a`, ela seria concluída
sem que a future `b` fosse executada _sequer_. Vamos tentar usar a função `trpl::sleep`
como ponto de partida para permitir que as operações alternem a progressão,
conforme mostrado na Listagem 17-16.

<Listing number="17-16" caption="Usando `trpl::sleep` para permitir que as operações alternem a progressão" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Adicionamos chamadas a `trpl::sleep` com pontos de `await` entre cada chamada a `slow`.
Agora o trabalho das duas futures é intercalado:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

A future `a` ainda é executada por um tempo antes de passar o controle para `b`, porque
ela chama `slow` antes de chamar `trpl::sleep`, mas depois disso as futures
alternam de um lado para o outro cada vez que uma delas atinge um ponto de `await`. Nesse caso, nós
fizemos isso após cada chamada a `slow`, mas poderíamos dividir o trabalho da
maneira que fizer mais sentido para nós.

No entanto, nós não queremos realmente _dormir_ (`sleep`) aqui: queremos progredir o mais rápido
possível. Só precisamos devolver o controle ao runtime. Podemos fazer isso
diretamente, usando a função `trpl::yield_now`. Na Listagem 17-17, substituímos
todas essas chamadas a `trpl::sleep` por `trpl::yield_now`.

<Listing number="17-17" caption="Usando `yield_now` para permitir que as operações alternem a progressão" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Este código é mais claro quanto à intenção real e pode ser significativamente
mais rápido do que usar `sleep`, porque temporizadores (timers) como o usado por `sleep` frequentemente
têm limites em quão granulares podem ser. A versão de `sleep` que estamos usando,
por exemplo, sempre dormirá por pelo menos um milissegundo, mesmo se passarmos a ela um
`Duration` de um nanossegundo. Mais uma vez, os computadores modernos são _rápidos_: eles podem fazer
muita coisa em um milissegundo!

Isso significa que o código assíncrono pode ser útil até mesmo para tarefas vinculadas à CPU (compute-bound), dependendo do
que mais o seu programa está fazendo, porque ele fornece uma ferramenta útil para
estruturar as relações entre diferentes partes do programa (mas ao custo da sobrecarga da máquina de estados assíncrona). Esta é uma forma de
_multitarefa cooperativa_, onde cada future tem o poder de determinar quando
ela cede o controle por meio de pontos de `await`. Cada future, portanto, também tem
a responsabilidade de evitar o bloqueio por muito tempo. Em alguns sistemas operacionais embarcados baseados em Rust, este é o
_único_ tipo de multitarefa!

No código do mundo real, você geralmente não estará alternando chamadas de função com pontos de `await` em absolutamente todas as linhas, é claro. Embora ceder o controle dessa maneira seja
relativamente barato, não é de graça. Em muitos casos, tentar dividir uma
tarefa intensiva de CPU pode torná-la significativamente mais lenta, então às vezes é melhor
para o desempenho _geral_ permitir que uma operação bloqueie brevemente. Sempre
meça para ver quais são os reais gargalos de desempenho do seu código. A
dinâmica subjacente é importante de se ter em mente, no entanto, se você _estiver_
vendo muito trabalho acontecendo em série que você esperava que acontecesse concorrentemente!

### Construindo Nossas Próprias Abstrações Assíncronas

Também podemos compor futures juntas para criar novos padrões. Por exemplo, podemos
construir uma função `timeout` com blocos de construção assíncronos que já temos. Quando
terminarmos, o resultado será outro bloco de construção que poderemos usar para criar
ainda mais abstrações assíncronas.

A Listagem 17-18 mostra como esperaríamos que esse `timeout` funcionasse com uma future lenta.

<Listing number="17-18" caption="Usando nosso `timeout` imaginado para executar uma operação lenta com um limite de tempo" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Vamos implementar isso! Para começar, vamos pensar sobre a API para `timeout`:

- Ela precisa ser uma função assíncrona por si só, para que possamos aguardá-la com `await`.
- Seu primeiro parâmetro deve ser uma future para executar. Podemos torná-la genérica para permitir
  que funcione com qualquer future.
- Seu segundo parâmetro será o tempo máximo de espera. Se usarmos um `Duration`,
  isso facilitará o repasse para `trpl::sleep`.
- Ela deve retornar um `Result`. Se a future for concluída com sucesso, o
  `Result` será `Ok` com o valor produzido pela future. Se o tempo limite esgotar primeiro, o
  `Result` será `Err` com a duração que o tempo limite esperou.

A Listagem 17-19 mostra essa declaração.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="Definindo a assinatura de `timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Isso satisfaz nossos objetivos para os tipos. Agora vamos pensar sobre o _comportamento_ que
precisamos: queremos colocar a future passada como argumento em uma corrida contra a duração. Podemos usar
`trpl::sleep` para criar uma future de temporizador a partir da duração, e usar `trpl::select`
para executar esse temporizador junto com a future passada por quem chamou.

Na Listagem 17-20, implementamos `timeout` fazendo pattern matching (correspondência de padrões) no resultado de aguardar
`trpl::select`.

<Listing number="17-20" caption="Definindo `timeout` com `select` e `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

A implementação de `trpl::select` não é justa (fair): ela sempre pesquisa (polls) os argumentos na
ordem em que são passados (outras implementações de `select` escolherão
aleatoriamente qual argumento pesquisar primeiro). Assim, passamos `future_to_try` para
`select` primeiro para que ela tenha a chance de ser concluída mesmo que `max_time` seja uma duração muito
curta. Se `future_to_try` terminar primeiro, `select` retornará `Left`
com o resultado de `future_to_try`. Se o `timer` terminar primeiro, `select` retornará
`Right` com o resultado `()` do temporizador.

Se `future_to_try` tiver sucesso e obtivermos um `Left(output)`, retornamos
`Ok(output)`. Se o temporizador de sono expirar em vez disso e obtivermos um `Right(())`, ignoramos
o `()` com `_` e retornamos `Err(max_time)` em vez disso.

Com isso, temos um `timeout` funcional construído a partir de outros dois auxiliares assíncronos. Se
executarmos nosso código, ele imprimirá o modo de falha após o estouro do tempo limite:

```text
Failed after 2 seconds
```

Como as futures se compõem com outras futures, você pode construir ferramentas realmente poderosas
usando blocos de construção assíncronos menores. Por exemplo, você pode usar essa mesma
abordagem para combinar tempos limites (timeouts) com tentativas (retries), e por sua vez usá-las com
operações como chamadas de rede (como aquelas na Listagem 17-5).

Na prática, você geralmente trabalhará diretamente com `async` e `await`, e
secundariamente com funções como `select` e macros como a macro `join!`
para controlar como as futures mais externas são executadas.

Vimos agora várias maneiras de trabalhar com múltiplas futures ao mesmo tempo.
A seguir, veremos como podemos trabalhar com múltiplas futures em sequência ao longo
do tempo com _fluxos_ (streams).

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
