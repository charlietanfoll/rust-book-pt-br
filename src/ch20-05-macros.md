## Macros

Usamos macros como `println!` ao longo deste livro, mas ainda não exploramos totalmente o que é uma macro e como ela funciona. O termo _macro_ refere-se a uma família de recursos em Rust — macros declarativas com `macro_rules!` e três tipos de macros procedurais:

- Macros `#[derive]` personalizadas que especificam código adicionado com o atributo `derive` usado em structs e enums
- Macros semelhantes a atributos que definem atributos personalizados utilizáveis em qualquer item
- Macros semelhantes a funções que se parecem com chamadas de função, mas operam sobre os tokens especificados como seu argumento

Falaremos sobre cada uma delas por sua vez, mas primeiro, vamos ver por que precisamos de macros quando já temos funções.

### A Diferença entre Macros e Funções

Fundamentalmente, macros são uma forma de escrever código que escreve outro código, o que é conhecido como _metaprogramação_. No Apêndice C, discutimos o atributo `derive`, que gera uma implementação de vários traits para você. Também usamos as macros `println!` e `vec!` ao longo do livro. Todas essas macros _expandem_ para produzir mais código do que o código que você escreveu manualmente.

A metaprogramação é útil para reduzir a quantidade de código que você precisa escrever e manter, o que também é um dos papéis das funções. No entanto, as macros têm alguns poderes adicionais que as funções não têm.

A assinatura de uma função deve declarar o número e o tipo de parâmetros que a função possui. As macros, por outro lado, podem aceitar um número variável de parâmetros: podemos chamar `println!("hello")` com um argumento ou `println!("hello {}", name)` com dois argumentos. Além disso, as macros são expandidas antes que o compilador interprete o significado do código, de modo que uma macro pode, por exemplo, implementar um trait em um determinado tipo. Uma função não pode, porque ela é chamada em tempo de execução e um trait precisa ser implementado em tempo de compilação.

A desvantagem de implementar uma macro em vez de uma função é que as definições de macros são mais complexas do que as definições de funções porque você está escrevendo código Rust que escreve código Rust. Devido a essa indireção, as definições de macros são geralmente mais difíceis de ler, entender e manter do que as definições de funções.

Outra diferença importante entre macros e funções é que você deve definir macros ou trazê-las para o escopo _antes_ de chamá-las em um arquivo, ao contrário de funções, que você pode definir em qualquer lugar e chamar em qualquer lugar.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### Macros Declarativas para Metaprogramação Geral

A forma de macros mais amplamente utilizada em Rust é a _macro declarativa_. Elas também são às vezes chamadas de "macros por exemplo", "macros `macro_rules!`" ou simplesmente "macros". Em sua essência, as macros declarativas permitem que você escreva algo semelhante a uma expressão `match` em Rust. Conforme discutido no Capítulo 6, as expressões `match` são estruturas de controle que pegam uma expressão, comparam o valor resultante da expressão com padrões e, em seguida, executam o código associado ao padrão correspondente. As macros também comparam um valor com padrões associados a um código específico: nessa situação, o valor é o código-fonte literal em Rust passado para a macro; os padrões são comparados com a estrutura desse código-fonte; e o código associado a cada padrão, quando correspondido, substitui o código passado para a macro. Tudo isso acontece durante a compilação.

Para definir uma macro, você usa o constructo `macro_rules!`. Vamos explorar como usar `macro_rules!` observando como a macro `vec!` é definida. O Capítulo 8 abordou como podemos usar a macro `vec!` para criar um novo vetor com valores específicos. Por exemplo, a seguinte macro cria um novo vetor contendo três inteiros:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Também poderíamos usar a macro `vec!` para criar um vetor de dois inteiros ou um vetor de cinco fatias de string (_string slices_). Não poderíamos usar uma função para fazer o mesmo porque não saberíamos o número ou o tipo de valores antecipadamente.

A Listagem 20-35 mostra uma definição ligeiramente simplificada da macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="Uma versão simplificada da definição da macro `vec!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Nota: A definição real da macro `vec!` na biblioteca padrão inclui código para pré-alocar a quantidade correta de memória antecipadamente. Esse código é uma otimização que não incluímos aqui, para tornar o exemplo mais simples.

A anotação `#[macro_export]` indica que esta macro deve ser disponibilizada sempre que o crate no qual a macro é definida for trazido para o escopo. Sem essa anotação, a macro não pode ser trazida para o escopo.

Em seguida, iniciamos a definição da macro com `macro_rules!` e o nome da macro que estamos definindo _sem_ o ponto de exclamação. O nome, neste caso `vec`, é seguido por chaves que denotam o corpo da definição da macro.

A estrutura no corpo de `vec!` é semelhante à estrutura de uma expressão `match`. Aqui temos um braço com o padrão `( $( $x:expr ),* )`, seguido por `=>` e o bloco de código associado a este padrão. Se o padrão corresponder, o bloco de código associado será emitido. Dado que este é o único padrão nesta macro, há apenas uma maneira válida de corresponder; qualquer outro padrão resultará em um erro. Macros mais complexas terão mais de um braço.

A sintaxe de padrão válida em definições de macros é diferente da sintaxe de padrão abordada no Capítulo 19 porque os padrões de macros são correspondidos contra a estrutura do código Rust em vez de valores. Vamos passar pelo significado das peças de padrão na Listagem 20-29; para a sintaxe completa de padrões de macros, consulte a [Referência do Rust][ref].

Primeiro, usamos um par de parênteses para abranger todo o padrão. Usamos um cifrão (`$`) para declarar uma variável no sistema de macros que conterá o código Rust correspondente ao padrão. O cifrão deixa claro que esta é uma variável de macro em oposição a uma variável regular do Rust. Em seguida vem um par de parênteses que captura valores que correspondem ao padrão dentro dos parênteses para uso no código de substituição. Dentro de `$()` está `$x:expr`, que corresponde a qualquer expressão Rust e dá à expressão o nome `$x`.

A vírgula após `$()` indica que um caractere separador de vírgula literal deve aparecer entre cada instância do código que corresponde ao código em `$()`. O `*` especifica que o padrão corresponde a zero ou mais ocorrências do que precede o `*`.

Quando chamamos esta macro com `vec![1, 2, 3];`, o padrão `$x` corresponde três vezes com as três expressões `1`, `2` e `3`.

Agora vamos olhar para o padrão no corpo do código associado a este braço: `temp_vec.push()` dentro de `$()*` é gerado para cada parte que corresponde a `$()` no padrão zero ou mais vezes, dependendo de quantas vezes o padrão corresponde. O `$x` é substituído por cada expressão correspondida. Quando chamamos esta macro com `vec![1, 2, 3];`, o código gerado que substitui esta chamada de macro será o seguinte:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Definimos uma macro que pode aceitar qualquer número de argumentos de qualquer tipo e pode gerar código para criar um vetor contendo os elementos especificados.

Para saber mais sobre como escrever macros, consulte a documentação online ou outros recursos, como [“The Little Book of Rust Macros”][tlborm] iniciado por Daniel Keep e continuado por Lukas Wirth.

### Macros Procedurais para Geração de Código a Partir de Atributos

A segunda forma de macros é a macro procedural, que age mais como uma função (e é um tipo de procedimento). _Macros procedurais_ aceitam algum código como entrada, operam sobre esse código e produzem algum código como saída, em vez de corresponder a padrões e substituir o código por outro código como fazem as macros declarativas. Os três tipos de macros procedurais são `derive` personalizado, semelhantes a atributos e semelhantes a funções, e todas funcionam de maneira semelhante.

Ao criar macros procedurais, as definições devem residir em seu próprio crate com um tipo de crate especial. Isso ocorre por razões técnicas complexas que esperamos eliminar no futuro. Na Listagem 20-36, mostramos como definir uma macro procedural, onde `some_attribute` é um espaço reservado para o uso de uma variedade específica de macro.

<Listing number="20-36" file-name="src/lib.rs" caption="Um exemplo de definição de uma macro procedural">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

A função que define uma macro procedural recebe um `TokenStream` como entrada e produz um `TokenStream` como saída. O tipo `TokenStream` é definido pelo crate `proc_macro` que vem incluído com o Rust e representa uma sequência de tokens. Este é o núcleo da macro: o código-fonte sobre o qual a macro está operando compõe o `TokenStream` de entrada, e o código que a macro produz é o `TokenStream` de saída. A função também possui um atributo anexado a ela que especifica qual tipo de macro procedural estamos criando. Podemos ter vários tipos de macros procedurais no mesmo crate.

Vamos analisar os diferentes tipos de macros procedurais. Começaremos com uma macro `derive` personalizada e, em seguida, explicaremos as pequenas dissimilaridades que tornam as outras formas diferentes.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### Macros `derive` Personalizadas

Vamos criar um crate chamado `hello_macro` que define um trait chamado `HelloMacro` com uma função associada chamada `hello_macro`. Em vez de fazer nossos usuários implementarem o trait `HelloMacro` para cada um de seus tipos, forneceremos uma macro procedural para que os usuários possam anotar seu tipo com `#[derive(HelloMacro)]` para obter uma implementação padrão da função `hello_macro`. A implementação padrão imprimirá `Hello, Macro! My name is TypeName!` onde `TypeName` é o nome do tipo no qual este trait foi definido. Em outras palavras, escreveremos um crate que permite a outro programador escrever código como a Listagem 20-37 usando nosso crate.

<Listing number="20-37" file-name="src/main.rs" caption="O código que um usuário do nosso crate poderá escrever ao usar nossa macro procedural">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Este código imprimirá `Hello, Macro! My name is Pancakes!` quando terminarmos. O primeiro passo é criar um novo crate de biblioteca, assim:

```console
$ cargo new hello_macro --lib
```

Em seguida, na Listagem 20-38, definiremos o trait `HelloMacro` e sua função associada.

<Listing file-name="src/lib.rs" number="20-38" caption="Um trait simples que usaremos com a macro `derive`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Temos um trait e sua função. A este ponto, o usuário do nosso crate poderia implementar o trait para obter a funcionalidade desejada, como na Listagem 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="Como seria se os usuários escrevessem uma implementação manual do trait `HelloMacro`">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

No entanto, eles precisariam escrever o bloco de implementação para cada tipo que quisessem usar com `hello_macro`; queremos poupá-los de ter que fazer esse trabalho.

Além disso, ainda não podemos fornecer à função `hello_macro` uma implementação padrão que imprima o nome do tipo em que o trait está implementado: o Rust não possui recursos de reflexão, portanto, não pode consultar o nome do tipo em tempo de execução. Precisamos de uma macro para gerar código em tempo de compilação.

O próximo passo é definir a macro procedural. No momento em que escrevo isto, as macros procedurais precisam estar em seu próprio crate. Com o tempo, essa restrição poderá ser suspensa. A convenção para estruturar crates e crates de macros é a seguinte: Para um crate chamado `foo`, um crate de macro procedural `derive` personalizado é chamado de `foo_derive`. Vamos iniciar um novo crate chamado `hello_macro_derive` dentro do nosso projeto `hello_macro`:

```console
$ cargo new hello_macro_derive --lib
```

Nossos dois crates estão intimamente relacionados, então criamos o crate de macro procedural dentro do diretório do nosso crate `hello_macro`. Se alterarmos a definição do trait em `hello_macro`, também teremos que alterar a implementação da macro procedural em `hello_macro_derive`. Os dois crates precisarão ser publicados separadamente, e os programadores que usarem esses crates precisarão adicionar ambos como dependências e trazê-los para o escopo. Poderíamos, em vez disso, fazer com que o crate `hello_macro` use `hello_macro_derive` como dependência e reexporte o código da macro procedural. No entanto, a forma como estruturamos o projeto torna possível que os programadores usem `hello_macro` mesmo que não queiram a funcionalidade de `derive`.

Precisamos declarar o crate `hello_macro_derive` como um crate de macro procedural. Também precisaremos de funcionalidades dos crates `syn` e `quote`, como você verá em um momento, então precisamos adicioná-los como dependências. Adicione o seguinte ao arquivo _Cargo.toml_ para `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Para começar a definir a macro procedural, coloque o código na Listagem 20-40 no seu arquivo _src/lib.rs_ para o crate `hello_macro_derive`. Note que este código não compilará até adicionarmos uma definição para a função `impl_hello_macro`.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Código que a maioria dos crates de macros procedurais exigirá para processar código Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Note que dividimos o código na função `hello_macro_derive`, que é responsável por analisar o `TokenStream`, e na função `impl_hello_macro`, que é responsável por transformar a árvore de sintaxe: isso torna a escrita de uma macro procedural mais conveniente. O código na função externa (`hello_macro_derive` neste caso) será o mesmo para quase todos os crates de macros procedurais que você vir ou criar. O código que você especificar no corpo da função interna (`impl_hello_macro` neste caso) será diferente dependendo do propósito da sua macro procedural.

Introduzimos três novos crates: `proc_macro`, [`syn`][syn]<!-- ignore --> e [`quote`][quote]<!-- ignore -->. O crate `proc_macro` vem com o Rust, então não precisamos adicioná-lo às dependências em _Cargo.toml_. O crate `proc_macro` é a API do compilador que nos permite ler e manipular código Rust a partir do nosso código.

O crate `syn` analisa o código Rust a partir de uma string em uma estrutura de dados sobre a qual podemos realizar operações. O crate `quote` transforma as estruturas de dados do `syn` de volta em código Rust. Esses crates tornam muito mais simples analisar qualquer tipo de código Rust que possamos querer manipular: escrever um analisador completo para o código Rust não é uma tarefa simples.

A função `hello_macro_derive` será chamada quando um usuário da nossa biblioteca especificar `#[derive(HelloMacro)]` em um tipo. Isso é possível porque anotamos a função `hello_macro_derive` aqui com `proc_macro_derive` e especificamos o nome `HelloMacro`, que corresponde ao nome do nosso trait; esta é a convenção que a maioria das macros procedurais segue.

A função `hello_macro_derive` primeiro converte a `input` de um `TokenStream` em uma estrutura de dados que podemos interpretar e realizar operações. É aqui que o `syn` entra em ação. A função `parse` em `syn` pega um `TokenStream` e retorna uma struct `DeriveInput` representando o código Rust analisado. A Listagem 20-41 mostra as partes relevantes da struct `DeriveInput` que obtemos ao analisar a string `struct Pancakes;`.

<Listing number="20-41" caption="A instância de `DeriveInput` que obtemos ao analisar o código que possui o atributo da macro na Listagem 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Os campos desta struct mostram que o código Rust que analisamos é uma struct unitária (`unit struct`) com o `ident` (_identificador_, significando o nome) de `Pancakes`. Há mais campos nesta struct para descrever todos os tipos de código Rust; consulte a [documentação do `syn` para `DeriveInput`][syn-docs] para obter mais informações.

Em breve definiremos a função `impl_hello_macro`, que é onde construiremos o novo código Rust que queremos incluir. Mas antes de fazermos isso, note que a saída para nossa macro `derive` também é um `TokenStream`. O `TokenStream` retornado é adicionado ao código que os usuários do nosso crate escrevem, então quando eles compilam seu crate, eles obtêm a funcionalidade extra que fornecemos no `TokenStream` modificado.

Você pode ter notado que estamos chamando `unwrap` para fazer a função `hello_macro_derive` entrar em pânico (_panic_) se a chamada para a função `syn::parse` falhar aqui. É necessário que nossa macro procedural entre em pânico em caso de erros porque as funções `proc_macro_derive` devem retornar `TokenStream` em vez de `Result` para cumprir a API de macros procedurais. Simplificamos este exemplo usando `unwrap`; em código de produção, você deve fornecer mensagens de erro mais específicas sobre o que deu errado usando `panic!` ou `expect`.

Agora que temos o código para transformar o código Rust anotado de um `TokenStream` em uma instância de `DeriveInput`, vamos gerar o código que implementa o trait `HelloMacro` no tipo anotado, como mostrado na Listagem 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Implementando o trait `HelloMacro` usando o código Rust analisado">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Obtemos uma instância da struct `Ident` contendo o nome (identificador) do tipo anotado usando `ast.ident`. A struct na Listagem 20-41 mostra que, quando executamos a função `impl_hello_macro` no código da Listagem 20-37, o `ident` que obtemos terá o campo `ident` com o valor `"Pancakes"`. Assim, a variável `name` na Listagem 20-42 conterá uma instância da struct `Ident` que, quando impressa, será a string `"Pancakes"`, o nome da struct na Listagem 20-37.

A macro `quote!` nos permite definir o código Rust que queremos retornar. O compilador espera algo diferente do resultado direto da execução da macro `quote!`, então precisamos convertê-lo para um `TokenStream`. Fazemos isso chamando o método `into`, que consome esta representação intermediária e retorna um valor do tipo `TokenStream` necessário.

A macro `quote!` também fornece alguns mecanismos de template muito legais: podemos inserir `#name`, e o `quote!` o substituirá pelo valor na variável `name`. Você pode até fazer alguma repetição semelhante à forma como as macros normais funcionam. Confira [a documentação do crate `quote`][quote-docs] para uma introdução completa.

Queremos que nossa macro procedural gere uma implementação do nosso trait `HelloMacro` para o tipo que o usuário anotou, o qual podemos obter usando `#name`. A implementação do trait possui a função `hello_macro`, cujo corpo contém a funcionalidade que queremos fornecer: imprimir `Hello, Macro! My name is` e, em seguida, o nome do tipo anotado.

A macro `stringify!` usada aqui é integrada ao Rust. Ela pega uma expressão Rust, como `1 + 2`, e em tempo de compilação transforma a expressão em um literal de string, como `"1 + 2"`. Isso é diferente de `format!` ou `println!`, que são macros que avaliam a expressão e depois transformam o resultado em uma `String`. Há a possibilidade de que a entrada `#name` possa ser uma expressão para ser impressa literalmente, então usamos `stringify!`. O uso de `stringify!` também evita uma alocação ao converter `#name` em um literal de string em tempo de compilação.

Neste ponto, `cargo build` deve ser concluído com sucesso em ambos `hello_macro` e `hello_macro_derive`. Vamos conectar esses crates ao código da Listagem 20-37 para ver a macro procedural em ação! Crie um novo projeto binário em seu diretório _projects_ usando `cargo new pancakes`. Precisamos adicionar `hello_macro` e `hello_macro_derive` como dependências no _Cargo.toml_ do crate `pancakes`. Se você estiver publicando suas versões de `hello_macro` e `hello_macro_derive` no [crates.io](https://crates.io/)<!-- ignore -->, elas seriam dependências normais; caso contrário, você pode especificá-las como dependências de caminho (`path`) da seguinte forma:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Coloque o código da Listagem 20-37 em _src/main.rs_ e execute `cargo run`: Ele deve imprimir `Hello, Macro! My name is Pancakes!`. A implementação do trait `HelloMacro` da macro procedural foi incluída sem que o crate `pancakes` precisasse implementá-la; o `#[derive(HelloMacro)]` adicionou a implementação do trait.

Em seguida, vamos explorar como os outros tipos de macros procedurais diferem das macros `derive` personalizadas.

### Macros Semelhantes a Atributos

Macros semelhantes a atributos são semelhantes às macros `derive` personalizadas, mas em vez de gerar código para o atributo `derive`, elas permitem que você crie novos atributos. Elas também são mais flexíveis: `derive` funciona apenas para structs e enums; atributos também podem ser aplicados a outros itens, como funções. Aqui está um exemplo de uso de uma macro semelhante a um atributo. Digamos que você tenha um atributo chamado `route` que anota funções ao usar um framework de aplicação web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Este atributo `#[route]` seria definido pelo framework como uma macro procedural. A assinatura da função de definição da macro seria parecida com esta:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Aqui, temos dois parâmetros do tipo `TokenStream`. O primeiro é para o conteúdo do atributo: a parte `GET, "/"`. O segundo é o corpo do item ao qual o atributo está anexado: neste caso, `fn index() {}` e o resto do corpo da função.

Fora isso, as macros semelhantes a atributos funcionam da mesma maneira que as macros `derive` personalizadas: você cria um crate com o tipo de crate `proc-macro` e implementa uma função que gera o código desejado!

### Macros Semelhantes a Funções

Macros semelhantes a funções definem macros que se parecem com chamadas de função. Semelhantemente às macros `macro_rules!`, elas são mais flexíveis do que as funções; por exemplo, podem aceitar um número desconhecido de argumentos. No entanto, as macros `macro_rules!` só podem ser definidas usando a sintaxe semelhante a match que discutimos na seção [“Macros Declarativas para Metaprogramação Geral”][decl]<!-- ignore --> anterior. As macros semelhantes a funções aceitam um parâmetro `TokenStream`, e sua definição manipula esse `TokenStream` usando código Rust, assim como os outros dois tipos de macros procedurais. Um exemplo de macro semelhante a uma função é uma macro `sql!` que pode ser chamada assim:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Esta macro analisaria a instrução SQL dentro dela e verificaria se está sintaticamente correta, o que é um processamento muito mais complexo do que uma macro `macro_rules!` pode fazer. A macro `sql!` seria definida assim:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Esta definição é semelhante à assinatura da macro `derive` personalizada: recebemos os tokens que estão dentro dos parênteses e retornamos o código que queríamos gerar.

## Resumo

Ufa! Agora você tem alguns recursos do Rust em sua caixa de ferramentas que provavelmente não usará com frequência, mas saberá que estão disponíveis em circunstâncias muito específicas. Introduzimos vários tópicos complexos para que, quando você os encontrar em sugestões de mensagens de erro ou no código de outras pessoas, seja capaz de reconhecer esses conceitos e sintaxe. Use este capítulo como referência para guiá-lo em direção a soluções.

Em seguida, colocaremos tudo o que discutimos ao longo do livro em prática e faremos mais um projeto!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
