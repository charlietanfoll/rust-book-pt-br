## Definindo e Instanciando Structs

Structs são semelhantes às tuplas, discutidas na seção [“O Tipo Tupla”][tuples]<!--
ignore -->, pois ambas contêm múltiplos valores relacionados. Assim como as tuplas,
as partes de uma struct podem ser de tipos diferentes. Ao contrário das tuplas,
em uma struct você dá um nome a cada pedaço de dado, para que fique claro o que os
valores significam. Adicionar esses nomes torna as structs mais flexíveis do que
as tuplas: você não precisa depender da ordem dos dados para especificar ou
acessar os valores de uma instância.

Para definir uma struct, usamos a palavra-chave `struct` seguida do nome da struct
inteira. O nome de uma struct deve descrever o significado dos pedaços de dados
que estão sendo agrupados. Em seguida, entre chaves, definimos os nomes e os
tipos dos pedaços de dados, que chamamos de _campos_ (_fields_). Por exemplo, a
Listagem 5-1 mostra uma struct que armazena informações sobre uma conta de
usuário.

<Listing number="5-1" file-name="src/main.rs" caption="Uma definição de struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Para usar uma struct após tê-la definido, criamos uma _instância_ dessa struct
especificando valores concretos para cada um dos campos. Criamos uma instância
escrevendo o nome da struct seguido de chaves contendo pares de _`chave:
valor`_ (`key: value`), onde as chaves são os nomes dos campos e os valores são
os dados que queremos armazenar nesses campos. Não precisamos especificar os
campos na mesma ordem em que os declaramos na struct. Em outras palavras, a
definição da struct é como um modelo geral para o tipo, e as instâncias preenchem
esse modelo com dados específicos para criar valores desse tipo. Por exemplo,
podemos declarar um usuário específico conforme mostrado na Listagem 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Criando uma instância da struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Para obter um valor específico de uma struct, usamos a notação de ponto. Por
exemplo, para acessar o endereço de e-mail deste usuário, usamos `user1.email`.
Se a instância for mutável, podemos alterar um valor usando a notação de ponto
e atribuindo a um campo específico. A Listagem 5-3 mostra como alterar o valor
no campo `email` de uma instância mutável de `User`.

<Listing number="5-3" file-name="src/main.rs" caption="Alterando o valor no campo `email` de uma instância de `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Note que a instância inteira deve ser mutável; o Rust não permite que marquemos
apenas determinados campos como mutáveis. Como em qualquer expressão, podemos
construir uma nova instância da struct como a última expressão no corpo da função
para retornar implicitamente essa nova instância.

A Listagem 5-4 mostra uma função `build_user` que retorna uma instância de `User`
com o e-mail e o nome de usuário fornecidos. O campo `active` recebe o valor
`true`, e o campo `sign_in_count` recebe o valor `1`.

<Listing number="5-4" file-name="src/main.rs" caption="Uma função `build_user` que recebe um e-mail e um nome de usuário e retorna uma instância de `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Faz sentido nomear os parâmetros da função com o mesmo nome dos campos da struct,
mas ter que repetir os nomes dos campos e variáveis `email` e `username` é um
tanto entediante. Se a struct tivesse mais campos, repetir cada nome se tornaria
ainda mais irritante. Felizmente, existe um atalho conveniente!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Usando o Atalho de Inicialização de Campo

Como os nomes dos parâmetros e os nomes dos campos da struct são exatamente os
mesmos na Listagem 5-4, podemos usar a sintaxe do _atalho de inicialização de campo_
(_field init shorthand_) para reescrever `build_user` de modo que ela se comporte
exatamente da mesma forma, mas sem a repetição de `username` e `email`, como
mostrado na Listagem 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="Uma função `build_user` que usa o atalho de inicialização de campo porque os parâmetros `username` e `email` têm o mesmo nome que os campos da struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Aqui, estamos criando uma nova instância da struct `User`, que possui um campo
chamado `email`. Queremos definir o valor do campo `email` com o valor presente
no parâmetro `email` da função `build_user`. Como o campo `email` e o parâmetro
`email` têm o mesmo nome, precisamos apenas escrever `email` em vez de
`email: email`.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### Criando Instâncias a Partir de Outras Instâncias com a Sintaxe de Atualização de Struct

Muitas vezes é útil criar uma nova instância de uma struct que inclua a maioria
dos valores de outra instância do mesmo tipo, mas altere alguns deles. Você pode
fazer isso usando a sintaxe de atualização de struct.

Primeiro, na Listagem 5-6, mostramos como criar uma nova instância de `User` em
`user2` da maneira normal, sem a sintaxe de atualização. Definimos um novo valor
para `email`, mas usamos os mesmos valores de `user1` que criamos na
Listagem 5-2 para o restante.

<Listing number="5-6" file-name="src/main.rs" caption="Criando uma nova instância de `User` usando todos os valores de `user1`, exceto um">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Usando a sintaxe de atualização de struct, podemos obter o mesmo efeito com menos
código, como mostrado na Listagem 5-7. A sintaxe `..` especifica que os campos
restantes que não foram explicitamente definidos devem ter o mesmo valor dos
campos na instância fornecida.

<Listing number="5-7" file-name="src/main.rs" caption="Usando a sintaxe de atualização de struct para definir um novo valor de `email` para uma instância de `User`, mas usando o restante dos valores de `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

O código na Listagem 5-7 também cria uma instância em `user2` que possui um valor
diferente para `email`, mas possui os mesmos valores para os campos `username`,
`active` e `sign_in_count` vindos de `user1`. O `..user1` deve vir por último para
especificar que quaisquer campos restantes devem obter seus valores dos campos
correspondentes em `user1`, mas podemos optar por especificar valores para quantos
campos quisermos em qualquer ordem, independentemente da ordem dos campos na
definição da struct.

Note que a sintaxe de atualização de struct usa `=` como uma atribuição; isso
ocorre porque ela move os dados, exatamente como vimos na seção [“Variáveis e Dados
Interagindo com Move”][move]<!-- ignore -->. Neste exemplo, não podemos mais usar
`user1` após criar `user2` porque a `String` no campo `username` de `user1` foi
movida para `user2`. Se tivéssemos dado a `user2` novos valores de `String` tanto
para `email` quanto para `username`, usando assim apenas os valores `active` e
`sign_in_count` de `user1`, então `user1` ainda seria válido após a criação de
`user2`. Tanto `active` quanto `sign_in_count` são tipos que implementam o trait
`Copy`, portanto o comportamento que discutimos na seção [“Dados Apenas na Pilha:
Copy”][copy]<!-- ignore --> se aplicaria. Também ainda podemos usar
`user1.email` neste exemplo, porque seu valor não foi movido para fora de
`user1`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### Criando Tipos Diferentes com Tuple Structs

O Rust também suporta structs que se parecem com tuplas, chamadas de _tuple
structs_ (structs de tupla). As tuple structs têm o significado adicional que o
nome da struct fornece, mas não possuem nomes associados aos seus campos; em vez
disso, elas possuem apenas os tipos dos campos. Tuple structs são úteis quando
você quer dar um nome à tupla inteira e tornar a tupla um tipo diferente de
outras tuplas, e quando nomear cada campo como em uma struct normal seria
verboso ou redundante.

Para definir uma tuple struct, comece com a palavra-chave `struct` e o nome da
struct seguido pelos tipos na tupla. Por exemplo, aqui definimos e usamos duas
tuple structs chamadas `Color` e `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Note que os valores `black` e `origin` são tipos diferentes porque são instâncias
de tuple structs diferentes. Cada struct que você define é seu próprio tipo,
mesmo que os campos dentro da struct possam ter os mesmos tipos. Por exemplo,
uma função que aceita um parâmetro do tipo `Color` não pode receber um `Point`
como argumento, mesmo que ambos os tipos sejam compostos por três valores `i32`.
Caso contrário, instâncias de tuple structs são semelhantes às tuplas no sentido
de que você pode desestruturá-las em suas partes individuais, e você pode usar um
`_` (`.`) seguido pelo índice para acessar um valor individual. Ao contrário das
tuplas, as tuple structs exigem que você nomeie o tipo da struct ao
desestruturá-las. Por exemplo, escreveríamos `let Point(x, y, z) = origin;` para
desestruturar os valores no ponto `origin` em variáveis chamadas `x`, `y` e `z`.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### Definindo Structs Semelhantes a Unit (Unit-Like Structs)

Você também pode definir structs que não possuem nenhum campo! Elas são
chamadas de _unit-like structs_ (structs semelhantes a unit) porque se comportam
de maneira semelhante a `()`, o tipo unit que mencionamos na seção [“O Tipo
Tupla”][tuples]<!-- ignore -->. Unit-like structs podem ser úteis quando você
precisa implementar um trait em algum tipo, mas não tem nenhum dado que queira
armazenar no próprio tipo. Discutiremos traits no Capítulo 10. Aqui está um
exemplo de declaração e instanciação de uma unit struct chamada `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Para definir `AlwaysEqual`, usamos a palavra-chave `struct`, o nome que queremos
e, em seguida, um ponto e vírgula. Não são necessárias chaves nem parênteses!
Depois, podemos obter uma instância de `AlwaysEqual` na variável `subject` de
maneira semelhante: usando o nome que definimos, sem chaves ou parênteses.
Imagine que mais tarde implementaremos um comportamento para este tipo de forma
que cada instância de `AlwaysEqual` seja sempre igual a cada instância de
qualquer outro tipo, talvez para ter um resultado conhecido para fins de teste.
Não precisaríamos de nenhum dado para implementar esse comportamento! Você verá no
Capítulo 10 como definir traits e implementá-los em qualquer tipo, incluindo
unit-like structs.

> ### Propriedade dos Dados da Struct
>
> Na definição da struct `User` na Listagem 5-1, usamos o tipo proprietário
> `String` em vez do tipo de fatia de string `&str`. Esta é uma escolha deliberada
> porque queremos que cada instância desta struct possua todos os seus dados e
> que esses dados sejam válidos enquanto toda a struct for válida.
>
> Também é possível que structs armazenem referências a dados de propriedade de
> outra coisa, mas para fazer isso é necessário o uso de _lifetimes_ (tempos de
> vida), um recurso do Rust que discutiremos no Capítulo 10. Os lifetimes garantem
> que os dados referenciados por uma struct sejam válidos pelo mesmo tempo que a
> struct. Digamos que você tente armazenar uma referência em uma struct sem
> especificar os lifetimes, como no exemplo a seguir em *src/main.rs*; isso não
> vai funcionar:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> O compilador vai reclamar que precisa de especificadores de lifetime:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> No Capítulo 10, discutiremos como corrigir esses erros para que você possa
> armazenar referências em structs, mas por enquanto, corrigiremos erros como
> esses usando tipos proprietários como `String` em vez de referências como
> `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
