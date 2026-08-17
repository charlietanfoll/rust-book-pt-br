<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## De um Servidor de Uma Única Thread para um Servidor Multi-thread

No momento, o servidor processará cada requisição em sequência, o que significa que
ele não processará uma segunda conexão até que a primeira conexão termine de ser processada.
Se o servidor receber cada vez mais requisições, essa execução serial será
cada vez menos ideal. Se o servidor receber uma requisição que demore muito
para ser processada, as requisições subsequentes terão que esperar até que a requisição longa
termine, mesmo que as novas requisições pudessem ser processadas rapidamente. Precisamos corrigir
isso, mas primeiro vamos ver o problema em ação.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Simulando uma Requisição Lenta

Vamos ver como uma requisição de processamento lento pode afetar outras requisições feitas à
nossa implementação atual do servidor. A Listagem 21-10 implementa o tratamento de uma requisição
para _/sleep_ com uma resposta simulada lenta que fará o servidor dormir
por cinco segundos antes de responder.

<Listing number="21-10" file-name="src/main.rs" caption="Simulando uma requisição lenta fazendo o servidor dormir por cinco segundos">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Mudamos de `if` para `match` agora que temos três casos. Precisamos
fazer correspondência explícita em uma fatia (slice) de `request_line` para fazer pattern-matching contra os
valores de string literal; o `match` não faz referenciamento e
desreferenciamento automáticos como o método de igualdade faz.

O primeiro braço é o mesmo que o bloco `if` da Listagem 21-9. O segundo braço
faz correspondência com uma requisição para _/sleep_. Quando essa requisição é recebida, o servidor irá
dormir por cinco segundos antes de renderizar a página HTML de sucesso. O terceiro braço
é o mesmo que o bloco `else` da Listagem 21-9.

Você pode ver quão primitivo é o nosso servidor: Bibliotecas reais lidariam com
o reconhecimento de múltiplas requisições de uma maneira muito menos verbosa!

Inicie o servidor usando `cargo run`. Em seguida, abra duas janelas do navegador: uma para
_http://127.0.0.1:7878_ e a outra para _http://127.0.0.1:7878/sleep_. Se você
inserir a URI _/_ algumas vezes, como antes, você a verá responder rapidamente. Mas se
você inserir _/sleep_ e depois carregar _/_, você verá que _/_ espera até que `sleep`
tenha dormido por seus cinco segundos completos antes de carregar.

Existem várias técnicas que poderíamos usar para evitar que requisições fiquem acumuladas atrás
de uma requisição lenta, incluindo o uso de async como fizemos no Capítulo 17; a técnica que vamos
implementar é um pool de threads.

### Melhorando a Vazão (Throughput) com um Pool de Threads

Um _pool de threads_ (conjunto de threads) é um grupo de threads geradas que estão prontas e esperando para
lidar com uma tarefa. Quando o programa recebe uma nova tarefa, ele atribui uma das
threads do pool à tarefa, e essa thread processará a tarefa. As
threads restantes no pool ficam disponíveis para lidar com quaisquer outras tarefas que chegarem
enquanto a primeira thread estiver processando. Quando a primeira thread terminar
de processar sua tarefa, ela é devolvida ao pool de threads ociosas, pronta para lidar com
uma nova tarefa. Um pool de threads permite processar conexões concorrentemente,
aumentando a vazão do seu servidor.

Limitaremos o número de threads no pool a um número pequeno para nos proteger
contra ataques DoS; se fizéssemos nosso programa criar uma nova thread para cada requisição conforme
ela chegasse, alguém fazendo 10 milhões de requisições ao nosso servidor poderia causar estragos
esgotando todos os recursos do nosso servidor e paralisando o processamento das requisições.

Em vez de gerar threads ilimitadas, portanto, teremos um número fixo de
threads esperando no pool. As requisições que chegam são enviadas ao pool para
processamento. O pool manterá uma fila de requisições de entrada. Cada uma das
threads no pool retirará uma requisição desta fila, lidará com a requisição,
e então pedirá outra requisição à fila. Com esse design, podemos processar até
_`N`_ requisições concorrentemente, onde _`N`_ é o número de threads. Se cada
thread estiver respondendo a uma requisição de longa duração, as requisições subsequentes ainda poderão
se acumular na fila, mas aumentamos o número de requisições de longa duração
que podemos manipular antes de atingir esse ponto.

Esta técnica é apenas uma das muitas maneiras de melhorar a vazão de um servidor web.
Outras opções que você pode explorar são o modelo fork/join, o
modelo de E/S assíncrona de thread única e o modelo de E/S assíncrona multi-thread. Se
você estiver interessado neste tópico, poderá ler mais sobre outras soluções e
tentar implementá-las; com uma linguagem de baixo nível como o Rust, todas essas
opções são possíveis.

Antes de começarmos a implementar um pool de threads, vamos falar sobre como o uso
do pool deve ser. Quando você está tentando projetar código, escrever a interface
do cliente primeiro pode ajudar a guiar seu design. Escreva a API do código de forma
que ela seja estruturada da maneira que você deseja chamá-la; então, implemente a
funcionalidade dentro dessa estrutura em vez de implementar a funcionalidade
e depois projetar a API pública.

Da mesma forma que usamos o desenvolvimento orientado a testes (TDD) no projeto do Capítulo 12,
usaremos aqui o desenvolvimento orientado pelo compilador. Escreveremos o código que chama as
funções que queremos e, em seguida, analisaremos os erros do compilador para determinar
o que devemos mudar a seguir para fazer o código funcionar. Antes de fazer isso, no entanto,
vamos explorar a técnica que não vamos usar como ponto de partida.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Gerando uma Thread para Cada Requisição

Primeiro, vamos explorar como nosso código poderia parecer se ele criasse uma nova thread para
cada conexão. Como mencionado anteriormente, este não é o nosso plano final devido aos
problemas de potencialmente gerar um número ilimitado de threads, mas é um
ponto de partida para obter um servidor multi-thread funcional primeiro. Depois, adicionaremos o
pool de threads como uma melhoria, e contrastar as duas soluções será mais fácil.

A Listagem 21-11 mostra as alterações a serem feitas em `main` para gerar uma nova thread para
lidar com cada stream dentro do loop `for`.

<Listing number="21-11" file-name="src/main.rs" caption="Gerando uma nova thread para cada stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Como você aprendeu no Capítulo 16, `thread::spawn` criará uma nova thread e executará
o código no closure nessa nova thread. Se você executar este código e carregar
_/sleep_ no seu navegador, e então _/_ em outras duas abas do navegador, você verá
que as requisições para _/_ não precisam esperar _/sleep_ terminar. No entanto, como
mencionamos, isso eventualmente sobrecarregará o sistema porque você estaria criando
novas threads sem nenhum limite.

Você também deve se lembrar do Capítulo 17 que este é exatamente o tipo de situação
onde async e await realmente brilham! Lembre-se disso enquanto construímos o pool
de threads e pense em como as coisas seriam diferentes ou iguais com async.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Criando um Número Finito de Threads

Queremos que nosso pool de threads funcione de maneira semelhante e familiar, para que a troca
de threads para um pool de threads não exija grandes alterações no código que
usa nossa API. A Listagem 21-12 mostra a interface hipotética para uma struct `ThreadPool`
que queremos usar em vez de `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Nossa interface ideal para o `ThreadPool`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Usamos `ThreadPool::new` para criar um novo pool de threads com um número configurável
de threads, neste caso quatro. Em seguida, no loop `for`, `pool.execute` tem uma
interface semelhante a `thread::spawn` no sentido de que ela recebe um closure que o pool
deve executar para cada stream. Precisamos implementar `pool.execute` para que ele
receba o closure e o entregue a uma thread no pool para execução. Este código ainda não
compila, mas vamos tentar para que o compilador possa nos guiar sobre como consertá-lo.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Construindo `ThreadPool` Usando o Desenvolvimento Orientado pelo Compilador

Faça as alterações da Listagem 21-12 em _src/main.rs_ e, em seguida, usemos os
erros de compilação de `cargo check` para guiar nosso desenvolvimento. Aqui está o primeiro
erro que obtemos:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Ótimo! Este erro nos diz que precisamos de um tipo ou módulo `ThreadPool`, então vamos
construir um agora. Nossa implementação de `ThreadPool` será independente do tipo
de trabalho que nosso servidor web está fazendo. Portanto, vamos mudar a crate `hello` de
uma crate binária para uma crate de biblioteca para conter nossa implementação de `ThreadPool`. Depois
de mudarmos para uma crate de biblioteca, também poderemos usar a biblioteca de pool de threads separada
para qualquer trabalho que quisermos fazer usando um pool de threads, não apenas para atender a requisições web.

Crie um arquivo _src/lib.rs_ que contenha o seguinte, que é a definição mais simples
de uma struct `ThreadPool` que podemos ter por enquanto:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


Em seguida, edite o arquivo _main.rs_ para trazer `ThreadPool` para o escopo da crate de biblioteca
adicionando o seguinte código no topo de _src/main.rs_:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Este código ainda não funcionará, mas vamos verificá-lo novamente para obter o próximo erro que
precisamos resolver:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Este erro indica que, em seguida, precisamos criar uma função associada chamada
`new` para `ThreadPool`. Também sabemos que `new` precisa ter um parâmetro
que possa aceitar `4` como argumento e deve retornar uma instância de `ThreadPool`.
Vamos implementar a função `new` mais simples que terá essas
características:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Escolhemos `usize` como o tipo do parâmetro `size` porque sabemos que um
número negativo de threads não faz sentido. Também sabemos que usaremos este
`4` como o número de elementos em uma coleção de threads, que é para o que o
tipo `usize` serve, conforme discutido na seção [“Tipos Inteiros”][integer-types]<!--
ignore --> no Capítulo 3.

Vamos verificar o código novamente:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Agora o erro ocorre porque não temos um método `execute` em `ThreadPool`.
Lembre-se da seção [“Criando um Número Finito de
Threads”](#creating-a-finite-number-of-threads)<!-- ignore --> que
decidimos que nosso pool de threads deve ter uma interface semelhante a `thread::spawn`. Além
disso, implementaremos a função `execute` para que ela receba o closure fornecido
e o entregue a uma thread ociosa no pool para execução.

Definiremos o método `execute` em `ThreadPool` para receber um closure como
parâmetro. Lembre-se da seção [“Movendo Valores Capturados para Fora de Closures”][moving-out-of-closures]<!-- ignore --> no Capítulo 13 que podemos
receber closures como parâmetros com três traits diferentes: `Fn`, `FnMut` e
`FnOnce`. Precisamos decidir qual tipo de closure usar aqui. Sabemos que
terminaremos fazendo algo semelhante à implementação padrão da biblioteca `thread::spawn`,
então podemos observar quais limites a assinatura de `thread::spawn`
impõe ao seu parâmetro. A documentação nos mostra o seguinte:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

O parâmetro de tipo `F` é aquele com o qual nos preocupamos aqui; o parâmetro de tipo `T`
está relacionado ao valor de retorno, e não estamos preocupados com isso. Nós
podemos ver que `spawn` usa `FnOnce` como o trait bound em `F`. Isto é
provavelmente o que queremos também, porque eventualmente passaremos o argumento que obtemos em
`execute` para `spawn`. Podemos ter ainda mais certeza de que `FnOnce` é o trait que
queremos usar porque a thread para executar uma requisição executará o closure dessa requisição apenas uma vez, o que corresponde ao `Once` em `FnOnce`.

O parâmetro de tipo `F` também possui o trait bound `Send` e o limite de tempo de vida (lifetime bound)
`'static`, que são úteis em nossa situação: Precisamos de `Send` para transferir o
closure de uma thread para outra e `'static` porque não sabemos quanto tempo
a thread levará para executar. Vamos criar um método `execute` em
`ThreadPool` que receberá um parâmetro genérico do tipo `F` com estes limites:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Ainda usamos os `()` após `FnOnce` porque este `FnOnce` representa um closure
que não aceita parâmetros e retorna o tipo unitário `()`. Assim como as definições
de funções, o tipo de retorno pode ser omitido da assinatura, mas mesmo se
não tivermos parâmetros, ainda precisamos dos parênteses.

Novamente, esta é a implementação mais simples do método `execute`: Ele não faz
nada, mas estamos apenas tentando fazer nosso código compilar. Vamos verificá-lo novamente:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Ele compila! Mas note que se você tentar `cargo run` e fizer uma requisição no
navegador, você verá os erros no navegador que vimos no início do
capítulo. Nossa biblioteca ainda não está chamando o closure passado para `execute`!

> Nota: Um ditado que você pode ouvir sobre linguagens com compiladores estritos, como
> Haskell e Rust, é “Se o código compila, ele funciona.” Mas esse ditado não é
> universalmente verdadeiro. Nosso projeto compila, mas não faz absolutamente nada! Se
> estivéssemos construindo um projeto real e completo, este seria um bom momento para começar
> a escrever testes unitários para verificar se o código compila _e_ tem o comportamento que
> queremos.

Considere: O que seria diferente aqui se fôssemos executar um future
em vez de um closure?

#### Validando o Número de Threads em `new`

Não estamos fazendo nada com os parâmetros para `new` e `execute`. Vamos
implementar os corpos dessas funções com o comportamento que queremos. Para começar,
vamos pensar sobre `new`. Anteriormente, escolhemos um tipo sem sinal para o parâmetro
`size` porque um pool com um número negativo de threads não faz sentido.
No entanto, um pool com zero threads também não faz sentido, mas zero é um `usize`
perfeitamente válido. Adicionaremos código para verificar se `size` é maior que zero antes
de retornarmos uma instância de `ThreadPool`, e faremos o programa entrar em pânico se ele
receber um zero usando a macro `assert!`, como mostrado na Listagem 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Implementando `ThreadPool::new` para entrar em pânico se `size` for zero">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Também adicionamos alguma documentação para o nosso `ThreadPool` com comentários de documentação.
Note que seguimos boas práticas de documentação adicionando uma seção que
destaca as situações em que nossa função pode entrar em pânico, conforme discutido no
Capítulo 14. Tente executar `cargo doc --open` e clique na struct `ThreadPool`
para ver como a documentação gerada para `new` se parece!

Em vez de adicionar a macro `assert!` como fizemos aqui, poderíamos transformar `new`
em `build` e retornar um `Result` como fizemos com `Config::build` no projeto
de E/S na Listagem 12-9. Mas decidimos neste caso que tentar criar um
pool de threads sem nenhuma thread deve ser um erro irrecuperável. Se você estiver
se sentindo ambicioso, tente escrever uma função chamada `build` com a seguinte
assinatura para comparar com a função `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Criando Espaço para Armazenar as Threads

Agora que temos uma maneira de saber que temos um número válido de threads para armazenar no
pool, podemos criar essas threads e armazená-las na struct `ThreadPool` antes de retornar
a struct. Mas como nós “armazenamos” uma thread? Vamos dar outra
olhada na assinatura de `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

A função `spawn` retorna um `JoinHandle<T>`, onde `T` é o tipo que o
closure retorna. Vamos tentar usar `JoinHandle` também e ver o que acontece. Em nosso
caso, os closures que estamos passando para o pool de threads lidarão com a conexão
e não retornarão nada, então `T` será o tipo unitário `()`.

O código na Listagem 21-14 compilará, mas ainda não cria nenhuma thread.
Mudamos a definição de `ThreadPool` para conter um vetor de
instâncias de `thread::JoinHandle<()>`, inicializamos o vetor com uma capacidade de
`size`, configuramos um loop `for` que executará algum código para criar as threads, e
retornamos uma instância de `ThreadPool` contendo-as.

<Listing number="21-14" file-name="src/lib.rs" caption="Criando um vetor para o `ThreadPool` armazenar as threads">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Trouxemos `std::thread` para o escopo na crate de biblioteca porque estamos
usando `thread::JoinHandle` como o tipo dos itens no vetor em
`ThreadPool`.

Assim que um tamanho válido é recebido, nosso `ThreadPool` cria um novo vetor que pode
conter `size` itens. A função `with_capacity` executa a mesma tarefa que
`Vec::new`, mas com uma diferença importante: Ela pré-aloca espaço no
vetor. Como sabemos que precisamos armazenar `size` elementos no vetor, fazer
essa alocação antecipadamente é slightly (ligeiramente) mais eficiente do que usar `Vec::new`,
que redimensiona a si mesmo conforme os elementos são inseridos.

Quando você executa `cargo check` novamente, ele deve ser bem-sucedido.

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Enviando Código do `ThreadPool` para uma Thread

Deixamos um comentário no loop `for` na Listagem 21-14 sobre a criação de
threads. Aqui, veremos como realmente criamos threads. A biblioteca padrão
fornece `thread::spawn` como uma maneira de criar threads, e
`thread::spawn` espera receber algum código que a thread deve executar assim que a
thread for criada. No entanto, em nosso caso, queremos criar as threads e fazer
com que elas _esperem_ pelo código que enviaremos mais tarde. A implementação de threads da biblioteca
padrão não inclui nenhuma maneira de fazer isso; temos que
implementá-la manualmente.

Implementaremos esse comportamento introduzindo uma nova estrutura de dados entre o
`ThreadPool` e as threads que gerenciará esse novo comportamento. Chamaremos
esta estrutura de dados de _Worker_ (trabalhador), que é um termo comum em implementações
de pooling. O `Worker` pega o código que precisa ser executado e executa o
código em sua thread.

Pense em pessoas trabalhando na cozinha de um restaurante: Os trabalhadores esperam até
que os pedidos cheguem dos clientes, e então eles são responsáveis por pegar esses
pedidos e prepará-los.

Em vez de armazenar um vetor de instâncias de `JoinHandle<()>` no pool de threads,
armazenaremos instâncias da struct `Worker`. Cada `Worker` armazenará uma única
instância de `JoinHandle<()>`. Em seguida, implementaremos um método no `Worker` que
pegará um closure de código para executar e o enviará para a thread já em execução
para execução. Também daremos a cada `Worker` um `id` para que possamos distinguir
entre as diferentes instâncias de `Worker` no pool ao registrar logs ou
depurar.

Aqui está o novo processo que acontecerá quando criarmos um `ThreadPool`. Implementaremos
o código que envia o closure para a thread depois de termos o `Worker`
configurado desta maneira:

1. Defina uma struct `Worker` que contenha um `id` e um `JoinHandle<()>`.
2. Altere `ThreadPool` para conter um vetor de instâncias de `Worker`.
3. Defina uma função `Worker::new` que aceite um número `id` e retorne uma
   instância de `Worker` que contenha o `id` e uma thread gerada com um closure vazio.
4. Em `ThreadPool::new`, use o contador do loop `for` para gerar um `id`, crie
   um novo `Worker` com esse `id` e armazene o `Worker` no vetor.

Se você estiver pronto para um desafio, tente implementar essas alterações por conta própria antes
de olhar o código na Listagem 21-15.

Pronto? Aqui está a Listagem 21-15 com uma maneira de fazer as modificações anteriores.

<Listing number="21-15" file-name="src/lib.rs" caption="Modificando o `ThreadPool` para conter instâncias de `Worker` em vez de conter threads diretamente">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Mudamos o nome do campo em `ThreadPool` de `threads` para `workers`
porque ele agora contém instâncias de `Worker` em vez de instâncias de `JoinHandle<()>`.
Usamos o contador no loop `for` como um argumento para
`Worker::new`, e armazenamos cada novo `Worker` no vetor chamado `workers`.

Código externo (como nosso servidor em _src/main.rs_) não precisa saber os
detalhes de implementação referentes ao uso de uma struct `Worker` dentro do `ThreadPool`,
então tornamos a struct `Worker` e sua função `new` privadas. A
função `Worker::new` usa o `id` que fornecemos e armazena uma instância de `JoinHandle<()>`
que é criada gerando uma nova thread usando um closure vazio.

> Nota: Se o sistema operacional não puder criar uma thread porque não há
> recursos de sistema suficientes, `thread::spawn` entrará em pânico. Isso fará com que
> todo o nosso servidor entre em pânico, mesmo que a criação de algumas threads possa
> ser bem-sucedida. Por simplicidade, esse comportamento é aceitável, mas em uma implementação de pool de
> threads de produção, você provavelmente iria querer usar
> [`std::thread::Builder`][builder]<!-- ignore --> e seu
> método [`spawn`][builder-spawn]<!-- ignore --> que retorna `Result` em vez disso.

Este código compilará e armazenará o número de instâncias de `Worker` que
especificamos como um argumento para `ThreadPool::new`. Mas _ainda_ não estamos processando
o closure que recebemos em `execute`. Vejamos como fazer isso a seguir.

#### Enviando Requisições para Threads por Meio de Canais (Channels)

O próximo problema que abordaremos é que os closures fornecidos a `thread::spawn` não fazem
absolutamente nada. Atualmente, obtemos o closure que queremos executar no
método `execute`. Mas precisamos fornecer a `thread::spawn` um closure para executar quando
criamos cada `Worker` durante a criação do `ThreadPool`.

Queremos que as structs `Worker` que acabamos de criar busquem o código a ser executado a partir
de uma fila mantida no `ThreadPool` e enviem esse código para sua thread para execução.

Os canais sobre os quais aprendemos no Capítulo 16 — uma maneira simples de se comunicar entre
duas threads — seriam perfeitos para este caso de uso. Usaremos um canal para funcionar
como a fila de trabalhos, e `execute` enviará um trabalho do `ThreadPool` para as
instâncias de `Worker`, que enviarão o trabalho para sua thread. Aqui está o plano:

1. O `ThreadPool` criará um canal e manterá o remetente (sender).
2. Cada `Worker` manterá o receptor (receiver).
3. Criaremos uma nova struct `Job` que conterá os closures que queremos enviar
   pelo canal.
4. O método `execute` enviará o trabalho que deseja executar através do
   remetente.
5. Em sua thread, o `Worker` fará um loop sobre seu receptor e executará os
   closures de quaisquer trabalhos que receber.

Vamos começar criando um canal em `ThreadPool::new` e mantendo o remetente
na instância de `ThreadPool`, como mostrado na Listagem 21-16. A struct `Job`
não contém nada por enquanto, mas será o tipo de item que enviaremos pelo
canal.

<Listing number="21-16" file-name="src/lib.rs" caption="Modificando o `ThreadPool` para armazenar o remetente de um canal que transmite instâncias de `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Em `ThreadPool::new`, criamos nosso novo canal e fazemos o pool manter o
remetente. Isso compilará com sucesso.

Vamos tentar passar um receptor do canal para cada `Worker` conforme o pool de
threads cria o canal. Sabemos que queremos usar o receptor na thread que
as instâncias de `Worker` geram, então faremos referência ao parâmetro `receiver` no
closure. O código na Listagem 21-17 ainda não vai compilar direito.

<Listing number="21-17" file-name="src/lib.rs" caption="Passando o receptor para cada `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Fizemos algumas alterações pequenas e diretas: Passamos o receptor para
`Worker::new` e, em seguida, o usamos dentro do closure.

Quando tentamos verificar este código, obtemos este erro:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

O código está tentando passar `receiver` para várias instâncias de `Worker`. Isso
não vai funcionar, como você deve se lembrar do Capítulo 16: A implementação de canal que
o Rust fornece é de múltiplos _produtores_ (producers), único _consumidor_ (consumer). Isso significa que não podemos
simplesmente clonar a extremidade consumidora do canal para consertar este código. Nós também não
queremos enviar uma mensagem várias vezes para vários consumidores; queremos uma lista
de mensagens com múltiplas instâncias de `Worker` de modo que cada mensagem seja
processada uma vez.

Além disso, retirar um trabalho da fila do canal envolve mutar o
`receiver`, então as threads precisam de uma maneira segura de compartilhar e modificar o `receiver`;
caso contrário, podemos ter condições de corrida (race conditions, conforme abordado no Capítulo 16).

Lembre-se dos ponteiros inteligentes seguros para threads discutidos no Capítulo 16: Para compartilhar
a propriedade entre várias threads e permitir que as threads mudem o valor, nós
precisamos usar `Arc<Mutex<T>>`. O tipo `Arc` permitirá que várias instâncias de `Worker`
possuam o receptor, e o `Mutex` garantirá que apenas um `Worker` obtenha um trabalho do
receptor por vez. A Listagem 21-18 mostra as alterações que precisamos fazer.

<Listing number="21-18" file-name="src/lib.rs" caption="Compartilhando o receptor entre as instâncias de `Worker` usando `Arc` e `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Em `ThreadPool::new`, colocamos o receptor em um `Arc` e um `Mutex`. Para cada
novo `Worker`, clonamos o `Arc` para incrementar a contagem de referências para que as
instâncias de `Worker` possam compartilhar a propriedade do receptor.

Com essas alterações, o código compila! Estamos chegando lá!

#### Implementando o Método `execute`

Vamos finalmente implementar o método `execute` em `ThreadPool`. Também mudaremos
`Job` de uma struct para um alias de tipo para um trait object que contém o tipo de
closure que `execute` recebe. Conforme discutido na seção [“Sinônimos de Tipos e Aliases de Tipos”][type-aliases]<!-- ignore --> no Capítulo 20, aliases de tipos
nos permitem tornar tipos longos mais curtos para facilitar o uso. Veja a Listagem 21-19.

<Listing number="21-19" file-name="src/lib.rs" caption="Criando um alias de tipo `Job` para um `Box` que contém cada closure e, em seguida, enviando o trabalho pelo canal">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Após criar uma nova instância de `Job` usando o closure que obtemos em `execute`, nós
enviamos esse trabalho pela extremidade remetente do canal. Estamos chamando `unwrap` em
`send` para o caso de o envio falhar. Isso pode acontecer se, por exemplo,
pararmos todas as nossas threads de executar, o que significa que a extremidade receptora parou
de receber novas mensagens. No momento, não podemos parar nossas threads de
executar: Nossas threads continuam executando enquanto o pool existir. O
motivo pelo qual usamos `unwrap` é que sabemos que o caso de falha não acontecerá, mas o
compilador não sabe disso.

Mas ainda não terminamos! No `Worker`, nosso closure sendo passado para
`thread::spawn` ainda apenas _faz referência_ à extremidade receptora do canal.
Em vez disso, precisamos que o closure faça um loop para sempre, pedindo à extremidade receptora do
canal um trabalho e executando o trabalho quando obtiver um. Vamos fazer a alteração
mostrada na Listagem 21-20 em `Worker::new`.

<Listing number="21-20" file-name="src/lib.rs" caption="Recebendo e executando os trabalhos na thread da instância de `Worker`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Aqui, primeiro chamamos `lock` no `receiver` para adquirir o mutex e, em seguida,
chamamos `unwrap` para entrar em pânico em caso de quaisquer erros. Adquirir um bloqueio (lock) pode falhar se o mutex
estiver em um estado _envenenado_ (poisoned), o que pode acontecer se alguma outra thread entrou em pânico enquanto
mantinha o bloqueio em vez de liberá-lo. Nesta situação, chamar
`unwrap` para fazer esta thread entrar em pânico é a ação correta a tomar. Sinta-se à vontade para
alterar este `unwrap` para um `expect` com uma mensagem de erro que seja significativa para
você.

Se obtivermos o bloqueio no mutex, chamamos `recv` para receber um `Job` do
canal. Um `unwrap` final passa por cima de quaisquer erros aqui também, o que pode ocorrer
se a thread que detém o remetente foi encerrada, de forma semelhante a como o método `send`
retorna `Err` se o receptor for encerrado.

A chamada para `recv` bloqueia, então se ainda não houver nenhum trabalho, a thread atual
esperará até que um trabalho fique disponível. O `Mutex<T>` garante que apenas uma
thread `Worker` por vez esteja tentando solicitar um trabalho.

Nosso pool de threads está agora em um estado funcional! Dê a ele um `cargo run` e faça algumas
requisições:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Sucesso! Agora temos um pool de threads que executa conexões de forma assíncrona.
Nunca há mais do que quatro threads criadas, portanto nosso sistema não ficará
sobrecarregado se o servidor receber muitas requisições. Se fizermos uma requisição para
_/sleep_, o servidor poderá atender a outras requisições fazendo com que outra
thread as execute.

> Nota: Se você abrir _/sleep_ em várias janelas do navegador simultaneamente, elas
> podem carregar uma de cada vez em intervalos de cinco segundos. Alguns navegadores web executam
> múltiplas instâncias da mesma requisição sequencialmente por motivos de cache. Essa
> limitação não é causada pelo nosso servidor web.

Este é um bom momento para pausar e considerar como o código nas Listagens 21-18, 21-19
e 21-20 seria diferente se estivéssemos usando futures em vez de um closure para
o trabalho a ser feito. Que tipos mudariam? Como as assinaturas dos métodos seriam
diferentes, se é que seriam? Que partes do código permaneceriam iguais?

Depois de aprender sobre o loop `while let` no Capítulo 17 e no Capítulo 19, você
pode estar se perguntando por que não escrevemos o código da thread `Worker` conforme mostrado na
Listagem 21-21.

<Listing number="21-21" file-name="src/lib.rs" caption="Uma implementação alternativa de `Worker::new` usando `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Este código compila e é executado, mas não resulta no comportamento de threading
desejado: Uma requisição lenta ainda fará com que outras requisições esperem para ser
processadas. O motivo é um tanto sutil: A struct `Mutex` não tem um método público
`unlock` porque a propriedade do bloqueio é baseada no tempo de vida (lifetime) do
`MutexGuard<T>` dentro do `LockResult<MutexGuard<T>>` que o método `lock`
retorna. Em tempo de compilação, o verificador de empréstimos (borrow checker) pode então impor a regra
de que um recurso protegido por um `Mutex` não pode ser acessado a menos que mantenhamos o
bloqueio. No entanto, esta implementação também pode fazer com que o bloqueio seja mantido
por mais tempo do que o pretendido se não formos cautelosos com o tempo de vida do
`MutexGuard<T>`.

O código na Listagem 21-20 que usa `let job =
receiver.lock().unwrap().recv().unwrap();` funciona porque com `let`, quaisquer
valores temporários usados na expressão no lado direito do sinal de igual
são descartados imediatamente quando a instrução `let` termina. No entanto, `while
let` (e `if let` e `match`) não descarta valores temporários até o final do bloco
associado. Na Listagem 21-21, o bloqueio permanece retido durante a duração da chamada
a `job()`, o que significa que outras instâncias de `Worker` não podem receber trabalhos.

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
