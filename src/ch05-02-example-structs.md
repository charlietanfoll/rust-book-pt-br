## Um Exemplo de Programa Usando Structs

Para entender quando podemos querer usar structs, vamos escrever um programa que calcula a área de um retângulo. Começaremos usando variáveis individuais e, em seguida, refatoraremos o programa até usarmos structs.

Vamos criar um novo projeto binário com o Cargo chamado _rectangles_ que receberá a largura e a altura de um retângulo especificadas em pixels e calculará a área do retângulo. A Listagem 5-8 mostra um programa curto com uma maneira de fazer exatamente isso no arquivo _src/main.rs_ do nosso projeto.

<Listing number="5-8" file-name="src/main.rs" caption="Calculando a área de um retângulo especificado por variáveis separadas de largura e altura">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Agora, execute este programa usando `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Este código consegue descobrir a área do retângulo chamando a função `area` com cada dimensão, mas podemos fazer mais para tornar este código claro e legível.

O problema com este código é evidente na assinatura de `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

A função `area` deveria calcular a área de um retângulo, mas a função que escrevemos tem dois parâmetros, e não fica claro em nenhum lugar do nosso programa que os parâmetros estão relacionados. Seria mais legível e mais gerenciável agrupar largura e altura. Já discutimos uma maneira de fazer isso na seção [“O Tipo Tupla”][the-tuple-type]<!-- ignore --> do Capítulo 3: usando tuplas.

### Refatorando com Tuplas

A Listagem 5-9 mostra outra versão do nosso programa que usa tuplas.

<Listing number="5-9" file-name="src/main.rs" caption="Especificando a largura e a altura do retângulo com uma tupla">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

De certa forma, este programa é melhor. As tuplas nos permitem adicionar um pouco de estrutura, e agora estamos passando apenas um argumento. Mas, por outro lado, esta versão é menos clara: as tuplas não nomeiam seus elementos, então temos que acessar as partes da tupla por índice, tornando nosso cálculo menos óbvio.

Misturar a largura e a altura não faria diferença para o cálculo da área, mas se quisermos desenhar o retângulo na tela, faria! Teríamos que nos lembrar de que `width` (largura) é o índice `0` da tupla e `height` (altura) é o índice `1` da tupla. Isso seria ainda mais difícil para outra pessoa descobrir e manter em mente se ela fosse usar nosso código. Como não transmitimos o significado dos nossos dados no código, agora é mais fácil introduzir erros.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Refatorando com Structs

Usamos structs para adicionar significado rotulando os dados. Podemos transformar a tupla que estamos usando em uma struct com um nome para o todo, bem como nomes para as partes, conforme mostrado na Listagem 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Definindo uma struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Aqui, definimos uma struct e a nomeamos como `Rectangle`. Dentro das chaves, definimos os campos como `width` e `height`, ambos com o tipo `u32`. Então, em `main`, criamos uma instância específica de `Rectangle` que tem largura `30` e altura `50`.

Nossa função `area` agora está definida com um parâmetro, que chamamos de `rectangle`, cujo tipo é um empréstimo imutável de uma instância da struct `Rectangle`. Conforme mencionado no Capítulo 4, queremos pegar a struct emprestada em vez de tomar sua propriedade. Dessa forma, `main` retém sua propriedade e pode continuar usando `rect1`, que é o motivo de usarmos o `&` na assinatura da função e onde chamamos a função.

A função `area` acessa os campos `width` e `height` da instância de `Rectangle` (observe que acessar campos de uma instância de struct emprestada não move os valores dos campos, razão pela qual você frequentemente vê empréstimos de structs). A assinatura da nossa função para `area` agora diz exatamente o que queremos dizer: Calcular a área de `Rectangle`, usando seus campos `width` e `height`. Isso transmite que a largura e a altura estão relacionadas entre si e dá nomes descritivos aos valores em vez de usar os valores de índice de tupla `0` e `1`. Isso é uma vitória para a clareza.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### Adicionando Funcionalidade com Traits Derivadas

Seria útil poder imprimir uma instância de `Rectangle` enquanto depuramos nosso programa e ver os valores de todos os seus campos. A Listagem 5-11 tenta usar a [macro `println!`][println]<!-- ignore --> como usamos em capítulos anteriores. No entanto, isso não funcionará.

<Listing number="5-11" file-name="src/main.rs" caption="Tentando imprimir uma instância de `Rectangle`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Quando compilamos este código, obtemos um erro com esta mensagem principal:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

A macro `println!` pode fazer vários tipos de formatação e, por padrão, as chaves dizem para o `println!` usar uma formatação conhecida como `Display`: saída destinada ao consumo direto do usuário final. Os tipos primitivos que vimos até agora implementam `Display` por padrão porque há apenas uma maneira de querer mostrar um `1` ou qualquer outro tipo primitivo a um usuário. Mas com structs, a maneira como o `println!` deve formatar a saída é menos clara porque há mais possibilidades de exibição: Você quer vírgulas ou não? Você quer imprimir as chaves? Todos os campos devem ser mostrados? Devido a essa ambiguidade, o Rust não tenta adivinhar o que queremos, e as structs não têm uma implementação fornecida de `Display` para usar com o `println!` e o marcador `{}`.

Se continuarmos lendo os erros, encontraremos esta nota útil:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Vamos tentar! A chamada da macro `println!` agora se parecerá com `println!("rect1 is {rect1:?}");`. Colocar o especificador `:?` dentro das chaves diz ao `println!` que queremos usar um formato de saída chamado `Debug`. A trait `Debug` nos permite imprimir nossa struct de uma forma útil para desenvolvedores, para que possamos ver seu valor enquanto depuramos nosso código.

Compile o código com esta alteração. Droga! Ainda recebemos um erro:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Mas, novamente, o compilador nos dá uma nota útil:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

O Rust _inclui_ funcionalidade para imprimir informações de depuração, mas temos que optar explicitamente por disponibilizar essa funcionalidade para nossa struct. Para fazer isso, adicionamos o atributo externo `#[derive(Debug)]` logo antes da definição da struct, conforme mostrado na Listagem 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Adicionando o atributo para derivar a trait `Debug` e imprimindo a instância de `Rectangle` usando a formatação de depuração">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Agora, quando executarmos o programa, não teremos nenhum erro e veremos a seguinte saída:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Legal! Não é a saída mais bonita, mas mostra os valores de todos os campos para esta instância, o que definitivamente ajudaria durante a depuração. Quando temos structs maiores, é útil ter uma saída um pouco mais fácil de ler; nesses casos, podemos usar `{:#?}` em vez de `{:?}` na string do `println!`. Neste exemplo, usar o estilo `{:#?}` gerará a seguinte saída:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Outra maneira de imprimir um valor usando o formato `Debug` é usar a [macro `dbg!`][dbg]<!-- ignore -->, que toma posse de uma expressão (em oposição ao `println!`, que pega uma referência), imprime o arquivo e o número da linha onde aquela chamada da macro `dbg!` ocorre no seu código junto com o valor resultante dessa expressão, e retorna a propriedade do valor.

> Nota: Chamar a macro `dbg!` imprime no fluxo do console de erro padrão (`stderr`), em oposição ao `println!`, que imprime no fluxo do console de saída padrão (`stdout`). Falaremos mais sobre `stderr` e `stdout` na seção [“Redirecionando Erros para o Erro Padrão” no Capítulo 12][err]<!-- ignore -->.

Aqui está um exemplo onde estamos interessados no valor que é atribuído ao campo `width`, bem como no valor de toda a struct em `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Podemos colocar `dbg!` ao redor da expressão `30 * scale` e, como `dbg!` retorna a propriedade do valor da expressão, o campo `width` terá o mesmo valor de se não tivéssemos a chamada `dbg!` ali. Não queremos que `dbg!` tome a propriedade de `rect1`, então usamos uma referência a `rect1` na próxima chamada. Aqui está a aparência da saída deste exemplo:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Podemos ver que a primeira parte da saída veio de _src/main.rs_ na linha 10, onde estamos depurando a expressão `30 * scale`, e seu valor resultante é `60` (a formatação `Debug` implementada para inteiros é imprimir apenas o seu valor). A chamada `dbg!` na linha 14 de _src/main.rs_ gera o valor de `&rect1`, que é a struct `Rectangle`. Esta saída usa a formatação `Debug` bonita (`pretty`) do tipo `Rectangle`. A macro `dbg!` pode ser muito útil quando você está tentando descobrir o que seu código está fazendo!

Além da trait `Debug`, o Rust forneceu várias traits para usarmos com o atributo `derive` que podem adicionar comportamento útil aos nossos tipos personalizados. Essas traits e seus comportamentos estão listados no [Apêndice C][app-c]<!-- ignore -->. Abordaremos como implementar essas traits com comportamento personalizado, bem como como criar suas próprias traits no Capítulo 10. Há também muitos atributos além do `derive`; para mais informações, consulte [a seção “Atributos” da Referência do Rust][attributes].

Nossa função `area` é muito específica: ela apenas calcula a área de retângulos. Seria útil vincular esse comportamento mais de perto à nossa struct `Rectangle` porque ela não funcionará com nenhum outro tipo. Vamos ver como podemos continuar refatorando este código transformando a função `area` em um método `area` definido em nosso tipo `Rectangle`.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
