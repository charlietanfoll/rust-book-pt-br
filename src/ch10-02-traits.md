<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## Definindo Comportamento Compartilhado com Traits

Uma _trait_ (traço) define a funcionalidade que um tipo particular possui e pode
compartilhar com outros tipos. Podemos usar traits para definir comportamento
compartilhado de maneira abstrata. Podemos usar _limites de trait_ (trait
bounds) para especificar que um tipo genérico pode ser qualquer tipo que possua
um determinado comportamento.

> Nota: Traits são semelhantes a um recurso frequentemente chamado de
> _interfaces_ em outras linguagens, embora com algumas diferenças.

### Definindo uma Trait

O comportamento de um tipo consiste nos métodos que podemos chamar nesse tipo.
Diferentes tipos compartilham o mesmo comportamento se pudermos chamar os mesmos
métodos em todos esses tipos. Definições de trait são uma forma de agrupar
assinaturas de métodos para definir um conjunto de comportamentos necessários
para realizar algum propósito.

Por exemplo, digamos que temos várias estruturas (_structs_) que armazenam
vários tipos e quantidades de texto: uma struct `NewsArticle` que armazena uma
notícia arquivada em um local específico e um `SocialPost` que pode ter, no
máximo, 280 caracteres junto com metadados que indicam se foi uma nova postagem,
um repost ou uma resposta a outra postagem.

Queremos criar uma biblioteca agregadora de mídia chamada `aggregator` que possa
exibir resumos de dados que podem estar armazenados em uma instância de
`NewsArticle` ou `SocialPost`. Para fazer isso, precisamos de um resumo de cada
tipo, e solicitaremos esse resumo chamando um método `summarize` em uma
instância. A Listagem 10-12 mostra a definição de uma trait pública `Summary`
que expressa esse comportamento.

<Listing number="10-12" file-name="src/lib.rs" caption="Uma trait `Summary` que consiste no comportamento fornecido por um método `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Aqui, declaramos uma trait usando a palavra-chave `trait` e depois o nome da
trait, que é `Summary` neste caso. Também declaramos a trait como `pub` para que
crates que dependam desta crate também possam fazer uso dela, como veremos em
alguns exemplos. Dentro das chaves, declaramos as assinaturas dos métodos que
descrevem os comportamentos dos tipos que implementam esta trait, que neste caso
é `fn summarize(&self) -> String`.

Após a assinatura do método, em vez de fornecer uma implementação dentro de
chaves, usamos um ponto e vírgula. Cada tipo que implementa esta trait deve
fornecer seu próprio comportamento personalizado para o corpo do método. O
compilador garantirá que qualquer tipo que possua a trait `Summary` tenha o
método `summarize` definido exatamente com esta assinatura.

Uma trait pode ter múltiplos métodos em seu corpo: as assinaturas dos métodos
são listadas uma por linha, e cada linha termina com um ponto e vírgula.

### Implementando uma Trait em um Tipo

Agora que definimos as assinaturas desejadas dos métodos da trait `Summary`,
podemos implementá-la nos tipos do nosso agregador de mídia. A Listagem 10-13
mostra uma implementação da trait `Summary` na struct `NewsArticle` que usa a
manchete, o autor e o local para criar o valor de retorno de `summarize`. Para a
struct `SocialPost`, definimos `summarize` como o nome de usuário seguido pelo
texto inteiro da postagem, assumindo que o conteúdo da postagem já esteja
limitado a 280 caracteres.

<Listing number="10-13" file-name="src/lib.rs" caption="Implementando a trait `Summary` nos tipos `NewsArticle` e `SocialPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Implementar uma trait em um tipo é semelhante a implementar métodos comuns. A
diferença é que, após `impl`, colocamos o nome da trait que queremos
implementar, depois usamos a palavra-chave `for` e, em seguida, especificamos o
nome do tipo para o qual queremos implementar a trait. Dentro do bloco `impl`,
colocamos as assinaturas de métodos que a definição da trait definiu. Em vez de
adicionar um ponto e vírgula após cada assinatura, usamos chaves e preenchemos o
corpo do método com o comportamento específico que queremos que os métodos da
trait tenham para aquele tipo em particular.

Agora que a biblioteca implementou a trait `Summary` em `NewsArticle` e
`SocialPost`, os usuários da crate podem chamar os métodos da trait em
instâncias de `NewsArticle` e `SocialPost` da mesma forma que chamamos métodos
comuns. A única diferença é que o usuário deve trazer a trait para o escopo
além dos tipos. Aqui está um exemplo de como uma crate binária poderia usar
nossa crate de biblioteca `aggregator`:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Este código imprime `1 new post: horse_ebooks: of course, as you probably already
know, people`.

Outras crates que dependem da crate `aggregator` também podem trazer a trait
`Summary` para o escopo para implementar `Summary` em seus próprios tipos. Uma
restrição a ser observada é que só podemos implementar uma trait em um tipo se
a trait, o tipo ou ambos forem locais à nossa crate. Por exemplo, podemos
implementar traits da biblioteca padrão como `Display` em um tipo personalizado
como `SocialPost` como parte da funcionalidade da nossa crate `aggregator`
porque o tipo `SocialPost` é local à nossa crate `aggregator`. Também podemos
implementar `Summary` em `Vec<T>` em nossa crate `aggregator` porque a trait
`Summary` é local à nossa crate `aggregator`.

Mas não podemos implementar traits externas em tipos externos. Por exemplo, não
podemos implementar a trait `Display` em `Vec<T>` dentro da nossa crate
`aggregator`, porque tanto `Display` quanto `Vec<T>` são definidos na biblioteca
padrão e não são locais à nossa crate `aggregator`. Esta restrição faz parte de
uma propriedade chamada _coerência_ (coherence) e, mais especificamente, da
_regra da órfã_ (orphan rule), chamada assim porque o tipo pai não está presente.
Esta regra garante que o código de outras pessoas não possa quebrar o seu código
e vice-versa. Sem a regra, duas crates poderiam implementar a mesma trait para
o mesmo tipo, e o Rust não saberia qual implementação usar.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### Usando Implementações Padrão

Às vezes, é útil ter um comportamento padrão para alguns ou todos os métodos em
uma trait, em vez de exigir implementações para todos os métodos em todos os
tipos. Então, conforme implementamos a trait em um tipo particular, podemos
manter ou substituir (_override_) o comportamento padrão de cada método.

Na Listagem 10-14, especificamos uma string padrão para o método `summarize` da
trait `Summary` em vez de apenas definir a assinatura do método, como fizemos
na Listagem 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Definindo uma trait `Summary` com uma implementação padrão do método `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Para usar uma implementação padrão para resumir instâncias de `NewsArticle`,
especificamos um bloco `impl` vazio com `impl Summary for NewsArticle {}`.

Embora não estejamos mais definindo o método `summarize` diretamente em
`NewsArticle`, fornecemos uma implementação padrão e especificamos que
`NewsArticle` implementa a trait `Summary`. Como resultado, ainda podemos
chamar o método `summarize` em uma instância de `NewsArticle`, desta forma:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Este código imprime `New article available! (Read more...)`.

Criar uma implementação padrão não exige que mudemos nada sobre a
implementação de `Summary` em `SocialPost` na Listagem 10-13. O motivo é que a
sintaxe para substituir uma implementação padrão é a mesma que a sintaxe para
implementar um método de trait que não possui uma implementação padrão.

Implementações padrão podem chamar outros métodos na mesma trait, mesmo que
esses outros métodos não tenham uma implementação padrão. Dessa forma, uma trait
pode fornecer muita funcionalidade útil e exigir apenas que os implementadores
especifiquem uma pequena parte dela. Por exemplo, poderíamos definir a trait
`Summary` para ter um método `summarize_author` cuja implementação seja
obrigatória, e então definir um método `summarize` que tenha uma implementação
padrão que chama o método `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Para usar esta versão de `Summary`, só precisamos definir `summarize_author`
quando implementamos a trait em um tipo:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Depois de definirmos `summarize_author`, podemos chamar `summarize` em
instâncias da struct `SocialPost`, e a implementação padrão de `summarize`
chamará a definição de `summarize_author` que fornecemos. Como implementamos
`summarize_author`, a trait `Summary` nos deu o comportamento do método
`summarize` sem exigir que escrevêssemos mais código. É assim que fica:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Este código imprime `1 new post: (Read more from @horse_ebooks...)`.

Note que não é possível chamar a implementação padrão a partir de uma
implementação que substitui esse mesmo método.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### Usando Traits como Parâmetros

Agora que você sabe como definir e implementar traits, podemos explorar como usar
traits para definir funções que aceitam muitos tipos diferentes. Usaremos a
trait `Summary` que implementamos nos tipos `NewsArticle` e `SocialPost` na
Listagem 10-13 para definir uma função `notify` que chama o método `summarize`
em seu parâmetro `item`, que é de algum tipo que implementa a trait `Summary`.
Para fazer isso, usamos a sintaxe `impl Trait`, desta forma:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Em vez de um tipo concreto para o parâmetro `item`, especificamos a
palavra-chave `impl` e o nome da trait. Este parâmetro aceita qualquer tipo que
implemente a trait especificada. No corpo de `notify`, podemos chamar qualquer
método em `item` que venha da trait `Summary`, como `summarize`. Podemos chamar
`notify` e passar qualquer instância de `NewsArticle` ou `SocialPost`. O código
que chama a função com qualquer outro tipo, como uma `String` ou um `i32`, não
vai compilar, porque esses tipos não implementam `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Sintaxe de Limite de Trait (Trait Bound)

A sintaxe `impl Trait` funciona para casos simples, mas na verdade é açúcar
sintático para uma forma mais longa conhecida como _limite de trait_ (trait
bound); ela se parece com isto:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Esta forma mais longa é equivalente ao exemplo na seção anterior, mas é mais
verbosa. Colocamos limites de trait junto com a declaração do parâmetro de tipo
genérico após dois pontos e dentro de colchetes angulares.

A sintaxe `impl Trait` é conveniente e resulta em um código mais conciso em
casos simples, enquanto a sintaxe de limite de trait mais completa pode expressar
mais complexidade em outros casos. Por exemplo, podemos ter dois parâmetros que
implementam `Summary`. Fazer isso com a sintaxe `impl Trait` se parece com isto:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Usar `impl Trait` é apropriado se quisermos que esta função permita que `item1`
e `item2` tenham tipos diferentes (desde que ambos os tipos implementem
`Summary`). Se quisermos forçar ambos os parâmetros a terem o mesmo tipo, no
entanto, devemos usar um limite de trait, desta forma:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

O tipo genérico `T` especificado como o tipo dos parâmetros `item1` e `item2`
restringe a função de forma que o tipo concreto do valor passado como argumento
para `item1` e `item2` deve ser o mesmo.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### Múltiplos Limites de Trait com a Sintaxe `+`

Também podemos especificar mais de um limite de trait. Digamos que quiséssemos
que `notify` usasse formatação de exibição (`Display`) bem como `summarize` em
`item`: especificamos na definição de `notify` que `item` deve implementar tanto
`Display` quanto `Summary`. Podemos fazer isso usando a sintaxe `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

A sintaxe `+` também é válida com limites de trait em tipos genéricos:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Com os dois limites de trait especificados, o corpo de `notify` pode chamar
`summarize` e usar `{}` para formatar `item`.

#### Limites de Trait Mais Claros com Cláusulas `where`

Usar muitos limites de trait tem suas desvantagens. Cada genérico tem seus
próprios limites de trait, então funções com múltiplos parâmetros de tipo
genérico podem conter muitas informações de limites de trait entre o nome da
função e sua lista de parâmetros, tornando a assinatura da função difícil de
ler. Por esse motivo, o Rust tem uma sintaxe alternativa para especificar
limites de trait dentro de uma cláusula `where` após a assinatura da função.
Então, em vez de escrever isto:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

podemos usar uma cláusula `where`, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

A assinatura desta função é menos poluída: o nome da função, a lista de
parâmetros e o tipo de retorno estão próximos, de forma semelhante a uma função
sem muitos limites de trait.

### Retornando Tipos Que Implementam Traits

Também podemos usar a sintaxe `impl Trait` na posição de retorno para retornar
um valor de algum tipo que implementa uma trait, como mostrado aqui:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Ao usar `impl Summary` para o tipo de retorno, especificamos que a função
`returns_summarizable` retorna algum tipo que implementa a trait `Summary` sem
nomear o tipo concreto. Neste caso, `returns_summarizable` retorna um
`SocialPost`, mas o código que chama esta função não precisa saber disso.

A capacidade de especificar um tipo de retorno apenas pela trait que ele
implementa é especialmente útil no contexto de closures e iteradores, que
abordamos no Capítulo 13. Closures e iteradores criam tipos que apenas o
compilador conhece ou tipos que são muito longos de especificar. A sintaxe
`impl Trait` permite que você especifique de forma concisa que uma função
retorna algum tipo que implementa a trait `Iterator` sem precisar escrever um
tipo muito longo.

No entanto, você só pode usar `impl Trait` se estiver retornando um único tipo.
Por exemplo, este código que retorna um `NewsArticle` ou um `SocialPost` com o
tipo de retorno especificado como `impl Summary` não funcionaria:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Retornar um `NewsArticle` ou um `SocialPost` não é permitido devido a restrições
sobre como a sintaxe `impl Trait` é implementada no compilador. Abordaremos como
escrever uma função com esse comportamento na seção [“Usando Objetos Trait para Abstrair sobre Comportamento Compartilhado”][trait-objects]<!-- ignore --> do Capítulo 18.

### Usando Limites de Trait para Implementar Métodos Condicionalmente

Usando um limite de trait com um bloco `impl` que usa parâmetros de tipo
genérico, podemos implementar métodos condicionalmente para tipos que
implementam as traits especificadas. Por exemplo, o tipo `Pair<T>` na Listagem
10-15 sempre implementa a função `new` para retornar uma nova instância de
`Pair<T>` (lembre-se da seção [“Sintaxe de Métodos”][methods]<!-- ignore --> do
Capítulo 5 que `Self` é um apelido de tipo para o tipo do bloco `impl`, que
neste caso é `Pair<T>`). Mas no próximo bloco `impl`, `Pair<T>` só implementa o
método `cmp_display` se seu tipo interno `T` implementar a trait `PartialOrd`
que permite comparação _e_ a trait `Display` que permite impressão.

<Listing number="10-15" file-name="src/lib.rs" caption="Implementando métodos condicionalmente em um tipo genérico dependendo de limites de trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Também podemos implementar condicionalmente uma trait para qualquer tipo que
implemente outra trait. Implementações de uma trait em qualquer tipo que
satisfaça os limites de trait são chamadas de _implementações abrangentes_
(_blanket implementations_) e são usadas extensivamente na biblioteca padrão do
Rust. Por exemplo, a biblioteca padrão implementa a trait `ToString` em
qualquer tipo que implemente a trait `Display`. O bloco `impl` na biblioteca
padrão se parece com este código:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Como a biblioteca padrão tem essa implementação abrangente, podemos chamar o
método `to_string` definido pela trait `ToString` em qualquer tipo que
implemente a trait `Display`. Por exemplo, podemos transformar inteiros em seus
valores de `String` correspondentes desta forma porque inteiros implementam
`Display`:

```rust
let s = 3.to_string();
```

Implementações abrangentes aparecem na documentação da trait na seção
“Implementors” (Implementadores).

Traits e limites de trait nos permitem escrever código que usa parâmetros de
tipo genérico para reduzir duplicação, mas também especificar para o
compilador que queremos que o tipo genérico tenha um comportamento particular.
O compilador pode então usar as informações de limite de trait para verificar
se todos os tipos concretos usados com nosso código fornecem o comportamento
correto. Em linguagens tipadas dinamicamente, obteríamos um erro em tempo de
execução se chamássemos um método em um tipo que não definiu o método. Mas o
Rust move esses erros para o tempo de compilação para que sejamos forçados a
corrigir os problemas antes mesmo que nosso código possa ser executado. Além
disso, não precisamos escrever código que verifica o comportamento em tempo de
execução, porque já verificamos em tempo de compilação. Fazer isso melhora o
desempenho sem ter que abrir mão da flexibilidade dos genéricos.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax