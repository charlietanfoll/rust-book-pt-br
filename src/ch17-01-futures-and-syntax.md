## Futures e a Sintaxe Async

Os elementos principais da programação assíncrona em Rust são as *futures* (futuros) e as
palavras-chave `async` e `await` do Rust.

Uma *future* é um valor que pode não estar pronto agora, mas que ficará pronto em algum
momento no futuro. (Esse mesmo conceito aparece em muitas linguagens, às vezes
sob outros nomes, como *task* (tarefa) ou *promise* (promessa).) O Rust fornece a trait `Future` como um bloco de construção para que diferentes operações assíncronas possam ser implementadas com
diferentes estruturas de dados, mas com uma interface comum. Em Rust, futures são
tipos que implementam a trait `Future`. Cada future armazena suas próprias informações
sobre o progresso que foi feito e o que "pronto" significa.

Você pode aplicar a palavra-chave `async` a blocos e funções para especificar que eles
podem ser interrompidos e retomados. Dentro de um bloco ou função `async`, você
pode usar a palavra-chave `await` para *aguardar uma future* (ou seja, esperar que ela fique
pronta). Qualquer ponto onde você aguarde uma future dentro de um bloco ou função async é
um ponto potencial para esse bloco ou função pausar e retomar. O processo de
verificar com uma future para ver se seu valor já está disponível é chamado de *polling* (pesquisa/escuta).

Algumas outras linguagens, como C# e JavaScript, também usam as palavras-chave `async` e `await`
para programação assíncrona. Se você está familiarizado com essas linguagens,
pode notar algumas diferenças significativas na forma como o Rust lida com a sintaxe. Isso
tem um bom motivo, como veremos!

Ao escrever código assíncrono em Rust, usamos as palavras-chave `async` e `await` na maior
parte do tempo. O Rust os compila em código equivalente usando a trait `Future`, assim
como compila loops `for` em código equivalente usando a trait `Iterator`.
No entanto, como o Rust fornece a trait `Future`, você também pode implementá-la para
seus próprios tipos de dados quando precisar. Muitas das funções que veremos
ao longo deste capítulo retornam tipos com suas próprias implementações de
`Future`. Retornaremos à definição da trait no final do capítulo
e nos aprofundaremos mais em como ela funciona, mas isso é detalhe suficiente para nos manter
avançando.

Tudo isso pode parecer um pouco abstrato, então vamos escrever nosso primeiro programa assíncrono: um
pequeno web scraper (coletor web). Vamos passar duas URLs pela linha de comando, buscar ambas
concorrentemente e retornar o resultado daquela que terminar primeiro. Este
exemplo terá um bocado de sintaxe nova, mas não se preocupe—explicaremos
tudo o que você precisa saber à medida que formos avançando.

## Nosso Primeiro Programa Assíncrono

Para manter o foco deste capítulo em aprender assincronismo em vez de equilibrar partes
do ecossistema, criamos a crate `trpl` (`trpl` é uma abreviação para “The Rust
Programming Language” / A Linguagem de Programação Rust). Ela reexporta todos os tipos, traits e funções
de que você precisará, principalmente das crates [`futures`][futures-crate]<!-- ignore --> e
[`tokio`][tokio]<!-- ignore -->. A crate `futures` é o lar oficial
de experimentos em Rust para código assíncrono, e é de fato onde a trait `Future`
foi originalmente projetada. O Tokio é o runtime assíncrono mais amplamente utilizado em
Rust hoje, especialmente para aplicações web. Existem outros ótimos runtimes
por aí, e eles podem ser mais adequados aos seus propósitos. Usamos a crate
`tokio` por baixo dos panos para o `trpl` porque ela é bem testada e amplamente utilizada.

Em alguns casos, o `trpl` também renomeia ou encapsula as APIs originais para mantê-lo
focado nos detalhes relevantes para este capítulo. Se você quiser entender o que
a crate faz, incentivamos você a conferir [o seu código-fonte][crate-source].
Você poderá ver de qual crate cada reexportação vem, e deixamos
comentários extensivos explicando o que a crate faz.

Crie um novo projeto binário chamado `hello-async` e adicione a crate `trpl` como uma
dependência:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Agora podemos usar as várias peças fornecidas pelo `trpl` para escrever nosso primeiro
programa assíncrono. Vamos construir uma pequena ferramenta de linha de comando que busca duas páginas web,
extrai o elemento `<title>` de cada uma e imprime o título de qual
página terminar todo esse processo primeiro.

### Definindo a Função page_title

Vamos começar escrevendo uma função que aceita a URL de uma página como parâmetro, faz
uma solicitação para ela e retorna o texto do elemento `<title>` (veja a Listagem
17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Definindo uma função async para obter o elemento title de uma página HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Primeiro, definimos uma função chamada `page_title` e a marcamos com a palavra-chave
`async`. Em seguida, usamos a função `trpl::get` para buscar qualquer URL que seja passada
e adicionamos a palavra-chave `await` para aguardar a resposta. Para obter o texto da
`response` (resposta), chamamos seu método `text` e mais uma vez a aguardamos com
a palavra-chave `await`. Ambas as etapas são assíncronas. Para a função `get`, temos
que esperar o servidor enviar de volta a primeira parte de sua resposta, o que
incluirá cabeçalhos HTTP, cookies e assim por diante, e pode ser entregue separadamente do
corpo da resposta. Especialmente se o corpo for muito grande, pode levar algum tempo
até que ele chegue por completo. Como temos que esperar a *totalidade* da
resposta chegar, o método `text` também é async.

Precisamos aguardar explicitamente ambas as futures, porque as futures em Rust são
*preguiçosas* (*lazy*): elas não fazem nada até que você peça a elas com a palavra-chave `await`.
(Na verdade, o Rust mostrará um aviso do compilador se você não usar uma future.) Isso
pode lembrá-lo da discussão sobre iteradores na seção
[“Processando uma Série de Itens com Iteradores”][iterators-lazy]<!-- ignore --> no Capítulo 13.
Os iteradores não fazem nada a menos que você chame o método `next` deles — seja diretamente ou usando
loops `for` ou métodos como `map` que usam `next` por baixo dos panos.
Da mesma forma, as futures não fazem nada a menos que você peça explicitamente a elas. Essa preguiça
permite que o Rust evite executar código assíncrono até que ele seja realmente necessário.

> Nota: Isso é diferente do comportamento que vimos ao usar `thread::spawn`
> na seção [“Criando uma Nova Thread com spawn”][thread-spawn]<!-- ignore -->
> no Capítulo 16, onde o closure que passamos para outra thread começou
> a rodar imediatamente. Também é diferente de como muitas outras linguagens
> abordam o assincronismo. Mas isso é importante para que o Rust possa fornecer suas
> garantias de desempenho, assim como acontece com os iteradores.

Assim que temos o `response_text`, podemos analisá-lo em uma instância do tipo `Html`
usando `Html::parse`. Em vez de uma string crua (raw string), agora temos um tipo de dado que
podemos usar para trabalhar com o HTML como uma estrutura de dados mais rica. Em particular, podemos
usar o método `select_first` para encontrar a primeira ocorrência de um seletor CSS
fornecido. Ao passar a string `"title"`, obteremos o primeiro elemento
`<title>` no documento, caso exista um. Como pode não haver nenhum
elemento correspondente, `select_first` retorna um `Option<ElementRef>`. Finalmente, usamos o
método `Option::map`, que nos permite trabalhar com o item no `Option` se ele estiver
presente, e não fazer nada se não estiver. (Poderíamos usar uma expressão
`match` aqui, mas `map` é mais idiomático.) No corpo da função que fornecemos para
`map`, chamamos `inner_html` no `title` para obter seu conteúdo, que é uma
`String`. No fim das contas, temos um `Option<String>`.

Note que a palavra-chave `await` do Rust fica *após* a expressão que você está aguardando,
e não antes dela. Ou seja, é uma palavra-chave *pós-fixada* (*postfix*). Isso pode diferir do que
você está acostumado se já usou `async` em outras linguagens, mas em Rust isso torna
cadeias de métodos muito mais agradáveis de se trabalhar. Como resultado, poderíamos alterar o
corpo de `page_title` para encadear as chamadas de função `trpl::get` e `text`
juntas com `await` entre elas, conforme mostrado na Listagem 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Encadeando com a palavra-chave `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Com isso, escrevemos com sucesso nossa primeira função assíncrona! Antes de adicionarmos
algum código em `main` para chamá-la, vamos falar um pouco mais sobre o que escrevemos
e o que isso significa.

Quando o Rust vê um *bloco* marcado com a palavra-chave `async`, ele o compila em um
tipo de dado único e anônimo que implementa a trait `Future`. Quando o Rust vê
uma *função* marcada com `async`, ele a compila em uma função não-async
cujo corpo é um bloco async. O tipo de retorno de uma função async é o tipo de
dado anônimo que o compilador cria para esse bloco async.

Portanto, escrever `async fn` é equivalente a escrever uma função que retorna uma
*future* do tipo de retorno. Para o compilador, uma definição de função como a
`async fn page_title` na Listagem 17-1 é aproximadamente equivalente a uma função não-async
definida assim:

```rust
# extern crate trpl; // necessário para o teste do mdbook
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Vamos passar por cada parte da versão transformada:

- Ela usa a sintaxe `impl Trait` que discutimos no Capítulo 10 na
  seção [“Traits como Parâmetros”][impl-trait]<!-- ignore -->.
- O valor retornado implementa a trait `Future` com um tipo associado de
  `Output`. Note que o tipo `Output` é `Option<String>`, que é o
  mesmo do tipo de retorno original da versão `async fn` de `page_title`.
- Todo o código chamado no corpo da função original é encapsulado em
  um bloco `async move`. Lembre-se de que blocos são expressões. Todo esse bloco
  é a expressão retornada pela função.
- Este bloco async produz um valor com o tipo `Option<String>`, conforme
  descrito. Esse valor corresponde ao tipo `Output` no tipo de retorno. Isso é
  exatamente como outros blocos que você já viu.
- O corpo da nova função é um bloco `async move` por causa de como ele usa o
  parâmetro `url`. (Falaremos muito mais sobre `async` versus `async move`
  mais adiante no capítulo.)

Agora podemos chamar `page_title` em `main`.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Executando uma Função Assíncrona com um Runtime

Para começar, vamos obter o título de uma única página, mostrado na Listagem 17-3.
Infelizmente, este código ainda não compila.

<Listing number="17-3" file-name="src/main.rs" caption="Chamando a função `page_title` a partir de `main` com um argumento fornecido pelo usuário">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Seguimos o mesmo padrão que usamos para obter argumentos de linha de comando na
seção [“Aceitando Argumentos de Linha de Comando”][cli-args]<!-- ignore --> no
Capítulo 12. Em seguida, passamos o argumento da URL para `page_title` e aguardamos o resultado.
Como o valor produzido pela future é um `Option<String>`, usamos uma
expressão `match` para imprimir mensagens diferentes para lidar com o caso de a página
ter ou não um `<title>`.

O único lugar onde podemos usar a palavra-chave `await` é em funções ou blocos async,
e o Rust não nos permite marcar a função especial `main` como `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

O motivo pelo qual `main` não pode ser marcado como `async` é que o código assíncrono precisa de um *runtime*:
uma crate Rust que gerencia os detalhes de execução de código assíncrono. A
função `main` de um programa pode *inicializar* um runtime, mas ela própria
não é um runtime. (Veremos mais sobre o porquê disso daqui a pouco.) Todo programa
Rust que executa código assíncrono tem pelo menos um lugar onde ele configura um
runtime que executa as futures.

A maioria das linguagens que suportam assincronismo agrupa um runtime, mas o Rust não. Em vez disso,
existem vários runtimes assíncronos diferentes disponíveis, cada um dos quais faz diferentes
compromissos (*tradeoffs*) adequados ao caso de uso que visa. Por exemplo, um servidor web
de alto rendimento com muitos núcleos de CPU e uma grande quantidade de RAM tem necessidades
muito diferentes de um microcontrolador com um único núcleo, uma pequena quantidade de RAM e
nenhuma capacidade de alocação no heap. As crates que fornecem esses runtimes também geralmente
fornecem versões assíncronas de funcionalidades comuns, como E/S de arquivos ou rede.

Aqui, e ao longo do resto deste capítulo, usaremos a função `block_on`
da crate `trpl`, que aceita uma future como argumento e bloqueia a thread
atual até que essa future seja executada até a conclusão. Por trás dos panos,
chamar `block_on` configura um runtime usando a crate `tokio` que é usado para rodar
a future passada (o comportamento de `block_on` da crate `trpl` é semelhante às
funções `block_on` de outras crates de runtime). Uma vez que a future é concluída,
`block_on` retorna qualquer valor que a future tenha produzido.

Poderíamos passar a future retornada por `page_title` diretamente para `block_on` e,
uma vez concluída, poderíamos fazer o pattern matching no `Option<String>` resultante como tentamos
fazer na Listagem 17-3. No entanto, para a maioria dos exemplos do capítulo (e
a maioria do código assíncrono no mundo real), faremos mais do que apenas uma chamada de função
assíncrona, então, em vez disso, passaremos um bloco `async` e aguardaremos explicitamente o
resultado da chamada de `page_title`, como na Listagem 17-4.

<Listing number="17-4" caption="Aguardando um bloco async com `trpl::block_on`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Quando executamos este código, obtemos o comportamento que esperávamos inicialmente:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Ufa — finalmente temos algum código assíncrono funcionando! Mas antes de adicionarmos o código para
fazer duas URLs disputarem entre si, vamos voltar nossa atenção brevemente para como
as futures funcionam.

Cada *ponto de await* — ou seja, cada lugar onde o código usa a palavra-chave
`await` — representa um lugar onde o controle é devolvido ao runtime. Para fazer
isso funcionar, o Rust precisa acompanhar o estado envolvido no bloco assíncrono para
que o runtime possa iniciar algum outro trabalho e depois voltar quando estiver
pronto para tentar avançar o primeiro novamente. Esta é uma máquina de estados invisível,
como se você tivesse escrito um enum como este para salvar o estado atual em cada ponto de await:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

No entanto, escrever o código para fazer a transição entre cada estado manualmente seria tedioso e
suscetível a erros, especialmente quando você precisa adicionar mais funcionalidades e
mais estados ao código mais tarde. Felizmente, o compilador do Rust cria e
gerencia as estruturas de dados da máquina de estados para código assíncrono automaticamente. As
regras normais de empréstimo e propriedade em torno de estruturas de dados ainda se aplicam,
e felizmente o compilador também lida com a verificação delas para nós e fornece
mensagens de erro úteis. Trabalharemos em algumas delas mais adiante no capítulo.

Em última análise, algo tem que executar esta máquina de estados, e esse algo é
um runtime. (É por isso que você pode encontrar menções a *executores* [*executors*] ao
pesquisar sobre runtimes: um executor é a parte de um runtime responsável por
executar o código assíncrono.)

Agora você pode ver por que o compilador nos impediu de tornar o próprio `main` uma
função assíncrona na Listagem 17-3. Se `main` fosse uma função assíncrona, algo mais
precisaria gerenciar a máquina de estados para qualquer future que `main` retornasse, mas
`main` é o ponto de partida para o programa! Em vez disso, chamamos a
função `trpl::block_on` em `main` para configurar um runtime e executar a future
retornada pelo bloco `async` até que ela termine.

> Nota: Alguns runtimes fornecem macros para que você *possa* escrever uma função `main`
> assíncrona. Essas macros reescrevem `async fn main() { ... }` para ser uma `fn
> main` normal, que faz a mesma coisa que fizemos manualmente na Listagem 17-4: chama uma
> função que executa uma future até a conclusão da maneira que `trpl::block_on` faz.

Agora vamos juntar essas peças e ver como podemos escrever código concorrente.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### Colocando Duas URLs para Disputar entre Si Concorrentemente

Na Listagem 17-5, chamamos `page_title` com duas URLs diferentes passadas a partir da
linha de comando e as colocamos em disputa selecionando qual future termina primeiro.

<Listing number="17-5" caption="Chamando `page_title` para duas URLs para ver qual retorna primeiro" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Começamos chamando `page_title` para cada uma das URLs fornecidas pelo usuário. Salvamos
as futures resultantes como `title_fut_1` e `title_fut_2`. Lembre-se, elas ainda não
fazem nada, porque as futures são preguiçosas e ainda não as aguardamos. Em seguida,
passamos as futures para `trpl::select`, que retorna um valor para indicar qual
das futures passadas a ela termina primeiro.

> Nota: Por baixo dos panos, `trpl::select` é construído sobre uma função `select`
> mais geral definida na crate `futures`. A função `select` da crate `futures`
> pode fazer muitas coisas que a função `trpl::select` não pode, mas
> ela também tem alguma complexidade adicional que podemos ignorar por enquanto.

Qualquer uma das futures pode legitimamente "vencer", então não faz sentido retornar um
`Result`. Em vez disso, `trpl::select` retorna um tipo que não vimos antes,
`trpl::Either`. O tipo `Either` (Ou) é um pouco semelhante a um `Result` por ter
dois casos. Diferente de `Result`, no entanto, não há noção de sucesso ou
falha embutida em `Either`. Em vez disso, ele usa `Left` (Esquerda) e `Right` (Direita) para indicar
"um ou outro":

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

A função `select` retorna `Left` com a saída dessa future se o primeiro
argumento vencer, e `Right` com a saída do segundo argumento future se *esse*
vencer. Isso corresponde à ordem em que os argumentos aparecem ao chamar a
função: o primeiro argumento está à esquerda do segundo argumento.

Também atualizamos `page_title` para retornar a mesma URL passada. Dessa forma, se a
página que retornar primeiro não tiver um `<title>` que possamos resolver, ainda podermos
imprimir uma mensagem significativa. Com essa informação disponível, encerramos
atualizando nossa saída `println!` para indicar tanto qual URL terminou primeiro
quanto qual é o `<title>` (se houver) para a página web nessa URL.

Agora você construiu um pequeno web scraper funcional! Escolha algumas URLs e execute a
ferramenta de linha de comando. Você pode descobrir que alguns sites são consistentemente mais rápidos
que outros, enquanto em outros casos o site mais rápido varia de uma execução para outra. O que é mais
importante, você aprendeu o básico de trabalhar com futures, então agora podemos
nos aprofundar no que podemos fazer com o assincronismo.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
