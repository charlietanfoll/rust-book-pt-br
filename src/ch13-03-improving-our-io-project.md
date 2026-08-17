## Melhorando Nosso Projeto de E/S

Com esse novo conhecimento sobre iteradores, podemos melhorar o projeto de E/S do
Capítulo 12 usando iteradores para tornar trechos de código mais claros e
concisos. Vamos ver como os iteradores podem melhorar nossa implementação da
função `Config::build` e da função `search`.

### Removendo um `clone` Usando um Iterador

Na Listagem 12-6, adicionamos código que recebia uma fatia (slice) de valores `String` e criava
uma instância da struct `Config` indexando a fatia e clonando os valores,
permitindo que a struct `Config` seja dona desses valores. Na Listagem 13-17,
reproduzimos a implementação da função `Config::build` como estava na
Listagem 12-23.

<Listing number="13-17" file-name="src/main.rs" caption="Reprodução da função `Config::build` da Listagem 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

Na época, dissemos para não se preocupar com as chamadas ineficientes de `clone`
porque as removeríamos no futuro. Pois bem, esse futuro chegou!

Precisávamos de `clone` aqui porque temos uma fatia com elementos `String` no
parâmetro `args`, mas a função `build` não é dona de `args`. Para retornar a
posse de uma instância de `Config`, tivemos que clonar os valores dos campos
`query` e `file_path` de `Config` para que a instância de `Config` possa ser
dona de seus próprios valores.

Com nosso novo conhecimento sobre iteradores, podemos alterar a função `build`
para receber a posse de um iterador como argumento em vez de emprestar uma fatia.
Usaremos a funcionalidade do iterador em vez do código que verifica o comprimento
da fatia e faz a indexação em locais específicos. Isso deixará claro o que a
função `Config::build` está fazendo, pois o iterador acessará os valores.

Uma vez que `Config::build` assume a posse do iterador e para de usar operações
de indexação que emprestam dados, podemos mover os valores `String` do iterador para
`Config` em vez de chamar `clone` e fazer uma nova alocação.

#### Usando o Iterador Retornado Diretamente

Abra o arquivo _src/main.rs_ do seu projeto de E/S, que deve se parecer com isto:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Primeiro, vamos alterar o início da função `main` que tínhamos na Listagem
12-24 para o código da Listagem 13-18, que desta vez usa um iterador. Isso
não vai compilar até que atualizemos `Config::build` também.

<Listing number="13-18" file-name="src/main.rs" caption="Passando o valor de retorno de `env::args` para `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

A função `env::args` retorna um iterador! Em vez de coletar os valores do
iterador em um vetor e então passar uma fatia para `Config::build`, agora
estamos passando a propriedade do iterador retornado de `env::args` para
`Config::build` diretamente.

Em seguida, precisamos atualizar a definição de `Config::build`. Vamos alterar a
assinatura de `Config::build` para ficar como na Listagem 13-19. Isso ainda não vai
compilar, porque precisamos atualizar o corpo da função.

<Listing number="13-19" file-name="src/main.rs" caption="Atualizando a assinatura de `Config::build` para esperar um iterador">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

A documentação da biblioteca padrão para a função `env::args` mostra que o
tipo do iterador que ela retorna é `std::env::Args`, e esse tipo implementa
o trait `Iterator` e retorna valores `String`.

Atualizamos a assinatura da função `Config::build` para que o parâmetro `args`
tenha um tipo genérico com a restrição de trait `impl Iterator<Item =
String>` em vez de `&[String]`. Esse uso da sintaxe `impl Trait` que
discutimos na seção [“Usando Traits como Parâmetros”][impl-trait]<!-- ignore -->
do Capítulo 10 significa que `args` pode ser qualquer tipo que implemente o
trait `Iterator` e retorne itens `String`.

Como estamos assumindo a propriedade de `args` e vamos mutar `args` ao
iterar sobre ele, podemos adicionar a palavra-chave `mut` na especificação do
parâmetro `args` para torná-lo mutável.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### Usando Métodos do Trait `Iterator`

Em seguida, vamos corrigir o corpo de `Config::build`. Como `args` implementa o
trait `Iterator`, sabemos que podemos chamar o método `next` nele! A Listagem 13-20
atualiza o código da Listagem 12-23 para usar o método `next`.

<Listing number="13-20" file-name="src/main.rs" caption="Alterando o corpo de `Config::build` para usar métodos de iterador">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

Lembre-se de que o primeiro valor no retorno de `env::args` é o nome do
programa. Queremos ignorar isso e ir para o próximo valor, então primeiro chamamos
`next` e não fazemos nada com o valor de retorno. Em seguida, chamamos `next` para
obter o valor que queremos colocar no campo `query` de `Config`. Se `next` retornar
`Some`, usamos um `match` para extrair o valor. Se retornar `None`, significa
que não foram fornecidos argumentos suficientes, e retornamos antecipadamente com um
valor `Err`. Fazemos a mesma coisa para o valor de `file_path`.

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### Esclarecendo Código com Adaptadores de Iterador

Também podemos aproveitar iteradores na função `search` em nosso projeto de
E/S, que está reproduzida aqui na Listagem 13-21 como estava na Listagem 12-19.

<Listing number="13-21" file-name="src/lib.rs" caption="A implementação da função `search` da Listagem 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Podemos escrever este código de forma mais concisa usando métodos adaptadores de
iterador. Fazer isso também nos permite evitar ter um vetor `results` intermediário
mutável. O estilo de programação funcional prefere minimizar a quantidade de estado
mutável para tornar o código mais claro. A remoção do estado mutável pode
possibilitar uma melhoria futura para fazer a busca acontecer em paralelo, pois não
teríamos que gerenciar o acesso concorrente ao vetor `results`. A Listagem 13-22
mostra essa mudança.

<Listing number="13-22" file-name="src/lib.rs" caption="Usando métodos adaptadores de iterador na implementação da função `search`">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Lembre-se de que o objetivo da função `search` é retornar todas as linhas em
`contents` que contêm a `query`. Semelhante ao exemplo de `filter` na Listagem
13-16, este código usa o adaptador `filter` para manter apenas as linhas para as
quais `line.contains(query)` retorna `true`. Em seguida, coletamos as linhas
correspondentes em outro vetor com `collect`. Muito mais simples! Sinta-se à vontade
para fazer a mesma alteração para usar métodos de iterador na função
`search_case_insensitive` também.

Para uma melhoria adicional, retorne um iterador da função `search` removendo
a chamada para `collect` e alterando o tipo de retorno para `impl Iterator<Item =
&'a str>` para que a função se torne um adaptador de iterador. Observe que você
também precisará atualizar os testes! Faça buscas em um arquivo grande usando sua
ferramenta `minigrep` antes e depois de fazer essa alteração para observar a
diferença de comportamento. Antes dessa alteração, o programa não imprimirá nenhum
resultado até que tenha coletado todos os resultados, mas após a alteração, os
resultados serão impressos à medida que cada linha correspondente for encontrada,
porque o loop `for` na função `run` é capaz de aproveitar a preGUIça (*laziness*)
do iterador.

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### Escolhendo Entre Loops e Iteradores

A próxima pergunta lógica é qual estilo você deve escolher em seu próprio código e
por quê: a implementação original na Listagem 13-21 ou a versão usando
iteradores na Listagem 13-22 (assumindo que estamos coletando todos os resultados
antes de retorná-los em vez de retornar o iterador). A maioria dos programadores
Rust prefere usar o estilo de iterador. É um pouco mais difícil de pegar o jeito no
início, mas assim que você pega o jeito dos vários adaptadores de iterador e o que
eles fazem, os iteradores podem ser mais fáceis de entender. Em vez de mexer nos
vários detalhes de loops e construção de novos vetores, o código foca no objetivo
de alto nível do loop. Isso abstrai parte do código corriqueiro para que seja
mais fácil ver os conceitos exclusivos deste código, como a condição de
filtragem pela qual cada elemento no iterador deve passar.

Mas as duas implementações são realmente equivalentes? A suposição intuitiva
pode ser que o loop de baixo nível seja mais rápido. Vamos falar sobre desempenho.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
