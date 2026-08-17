## Métodos

Métodos são semelhantes a funções: Nós os declaramos com a palavra-chave `fn` e um
nome, eles podem ter parâmetros e um valor de retorno, e contêm algum código
que é executado quando o método é chamado de algum outro lugar. Ao contrário das funções,
os métodos são definidos dentro do contexto de uma struct (ou de um enum ou de um objeto
trait, que abordamos no [Capítulo 6][enums]<!-- ignore --> e no [Capítulo
18][trait-objects]<!-- ignore -->, respectivamente), e seu primeiro parâmetro é
sempre `self`, que representa a instância da struct na qual o método está sendo
chamado.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### Sintaxe de Métodos

Vamos alterar a função `area` que tem uma instância de `Rectangle` como parâmetro
e, em vez disso, criar um método `area` definido na struct `Rectangle`, conforme mostrado
na Listagem 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Definindo um método `area` na struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Para definir a função dentro do contexto de `Rectangle`, iniciamos um bloco `impl`
(implementação) para `Rectangle`. Tudo dentro deste bloco `impl`
será associado ao tipo `Rectangle`. Em seguida, movemos a função `area`
para dentro das chaves do `impl` e alteramos o primeiro (e neste caso, único)
parâmetro para ser `self` na assinatura e em todo o corpo. Em
`main`, onde chamávamos a função `area` e passávamos `rect1` como argumento,
podemos em vez disso usar a _sintaxe de método_ para chamar o método `area` em nossa
instância de `Rectangle`. A sintaxe de método vem após uma instância: Adicionamos um ponto seguido pelo
nome do método, parênteses e quaisquer argumentos.

Na assinatura de `area`, usamos `&self` em vez de `rectangle: &Rectangle`.
O `&self` é na verdade uma abreviação para `self: &Self`. Dentro de um bloco `impl`, o
tipo `Self` é um apelido para o tipo ao qual o bloco `impl` se refere. Os métodos devem
ter um parâmetro chamado `self` do tipo `Self` como seu primeiro parâmetro, então o Rust
permite que você abrevie isso usando apenas o nome `self` no espaço do primeiro parâmetro.
Note que ainda precisamos usar o `&` na frente do atalho `self` para
indicar que este método empresta a instância `Self`, assim como fizemos em
`rectangle: &Rectangle`. Os métodos podem assumir a propriedade de `self`, emprestar `self`
de forma imutável, como fizemos aqui, ou emprestar `self` de forma mutável, assim como podem
fazer com qualquer outro parâmetro.

Escolhemos `&self` aqui pelo mesmo motivo que usamos `&Rectangle` na versão em função:
Não queremos assumir a propriedade, e queremos apenas ler os dados na struct,
não escrever neles. Se quiséssemos alterar a instância na qual chamamos
o método como parte do que o método faz, usaríamos `&mut self` como
o primeiro parâmetro. Ter um método que assume a propriedade da instância usando
apenas `self` como o primeiro parâmetro é raro; essa técnica é geralmente
usada quando o método transforma `self` em outra coisa e você quer
impedir que quem o chamou use a instância original após a transformação.

O principal motivo para usar métodos em vez de funções, além de
fornecer a sintaxe de método e não ter que repetir o tipo de `self` na assinatura de cada método,
é a organização. Colocamos todas as coisas que podemos fazer
com uma instância de um tipo em um único bloco `impl` em vez de fazer com que futuros usuários
do nosso código procurem por capacidades de `Rectangle` em vários lugares na
biblioteca que fornecemos.

Note que podemos escolher dar a um método o mesmo nome de um dos campos da struct.
Por exemplo, podemos definir um método em `Rectangle` que também se chama
`width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Aqui, estamos escolhendo fazer com que o método `width` retorne `true` se o valor no
campo `width` da instância for maior que `0` e `false` se o valor for
`0`: Podemos usar um campo dentro de um método de mesmo nome para qualquer propósito. Em
`main`, quando colocamos parênteses após `rect1.width`, o Rust sabe que queremos dizer o
método `width`. Quando não usamos parênteses, o Rust sabe que queremos dizer o campo
`width`.

Frequentemente, mas nem sempre, quando damos a um método o mesmo nome de um campo, queremos
que ele apenas retorne o valor do campo e não faça mais nada. Métodos como este
são chamados de _getters_ (getters/consultores), e o Rust não os implementa automaticamente para campos de structs
como algumas outras linguagens fazem. Getters são úteis porque você pode tornar o
campo privado e o método público, permitindo assim o acesso somente leitura a esse
campo como parte da API pública do tipo. Discutiremos o que são público e privado
e como designar um campo ou método como público ou privado no [Capítulo
7][public]<!-- ignore -->.

> ### Onde está o operador `->`?
>
> Em C e C++, dois operadores diferentes são usados para chamar métodos: Você usa
> `.` se estiver chamando um método diretamente no objeto e `->` se estiver
> chamando o método em um ponteiro para o objeto e precisar desreferenciar o ponteiro
> primeiro. Em outras palavras, se `object` é um ponteiro,
> `object->something()` é semelhante a `(*object).something()`.
>
> O Rust não tem um equivalente ao operador `->`; em vez disso, o Rust tem um
> recurso chamado _referenciação e desreferenciação automáticas_. Chamar métodos é
> um dos poucos lugares no Rust com esse comportamento.
>
> Funciona assim: Quando você chama um método com `object.something()`, o Rust
> adiciona automaticamente `&`, `&mut` ou `*` para que `object` corresponda à
> assinatura do método. Em outras palavras, os exemplos a seguir são equivalentes:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> O primeiro parece muito mais limpo. Esse comportamento de referenciação automática funciona
> porque os métodos têm um receptor claro — o tipo de `self`. Dado o receptor
> e o nome de um método, o Rust pode determinar definitivamente se o método está
> lindo (`&self`), mutando (`&mut self`) ou consumindo (`self`). O fato de
> o Rust tornar o empréstimo implícito para receptores de métodos é uma parte muito importante
> para tornar a propriedade ergonômica na prática.

### Métodos com Mais Parâmetros

Vamos praticar o uso de métodos implementando um segundo método na struct
`Rectangle`. Desta vez, queremos que uma instância de `Rectangle` aceite outra instância
de `Rectangle` e retorne `true` se o segundo `Rectangle` couber completamente
dentro do primeiro (o primeiro `Rectangle`); caso contrário, ele deve retornar `false`.
Ou seja, uma vez que tenhamos definido o método `can_hold`, queremos poder escrever
o programa mostrado na Listagem 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Usando o método ainda não escrito `can_hold`">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

A saída esperada seria semelhante à seguinte porque ambas as dimensões de
`rect2` são menores que as dimensões de `rect1`, mas `rect3` é mais largo que
`rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir um método, então ele estará dentro do bloco `impl Rectangle`.
O nome do método será `can_hold`, e ele aceitará um empréstimo imutável de outro
`Rectangle` como parâmetro. Podemos descobrir qual será o tipo do
parâmetro olhando para o código que chama o método:
`rect1.can_hold(&rect2)` passa `&rect2`, que é um empréstimo imutável de
`rect2`, uma instância de `Rectangle`. Isso faz sentido porque precisamos apenas
ler `rect2` (em vez de escrever, o que significaria que precisaríamos de um empréstimo mutável),
e queremos que `main` retenha a propriedade de `rect2` para que possamos usá-lo novamente
após chamar o método `can_hold`. O valor de retorno de `can_hold` será um
booleano, e a implementação verificará se a largura e a altura de
`self` são maiores que a largura e a altura do outro `Rectangle`,
respectivamente. Vamos adicionar o novo método `can_hold` ao bloco `impl` da
Listagem 5-13, mostrado na Listagem 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Implementando o método `can_hold` em `Rectangle` que aceita outra instância de `Rectangle` como parâmetro">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Quando executamos este código com a função `main` da Listagem 5-14, obteremos a
saída desejada. Os métodos podem aceitar múltiplos parâmetros que adicionamos à
assinatura após o parâmetro `self`, e esses parâmetros funcionam exatamente como
parâmetros em funções.

### Funções Associadas

Todas as funções definidas dentro de um bloco `impl` são chamadas de _funções associadas_
porque estão associadas ao tipo nomeado após o `impl`. Podemos definir
funções associadas que não possuem `self` como seu primeiro parâmetro (e, portanto,
não são métodos) porque elas não precisam de uma instância do tipo para funcionar.
Já usamos uma função assim antes: a função `String::from` que está
definida no tipo `String`.

Funções associadas que não são métodos são frequentemente usadas para construtores que
retornam uma nova instância da struct. Elas geralmente são chamadas de `new`, mas
`new` não é um nome especial e não é embutido na linguagem. Por exemplo, poderíamos
optar por fornecer uma função associada chamada `square` (quadrado) que teria
um parâmetro de dimensão e o usaria tanto como largura quanto como altura, facilitando
assim a criação de um `Rectangle` quadrado em vez de ter que especificar o mesmo
valor duas vezes:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

As palavras-chave `Self` no tipo de retorno e no corpo da função são
apelidos para o tipo que aparece após a palavra-chave `impl`, que neste caso
é `Rectangle`.

Para chamar essa função associada, usamos a sintaxe `::` com o nome da struct;
`let sq = Rectangle::square(3);` é um exemplo. Esta função está sob o namespace da
struct: A sintaxe `::` é usada tanto para funções associadas quanto para
namespaces criados por módulos. Discutiremos módulos no [Capítulo
7][modules]<!-- ignore -->.

### Múltiplos Blocos `impl`

Cada struct tem permissão para ter múltiplos blocos `impl`. Por exemplo, a Listagem
5-15 é equivalente ao código mostrado na Listagem 5-16, que tem cada método em
seu próprio bloco `impl`.

<Listing number="5-16" caption="Reescrevendo a Listagem 5-15 usando múltiplos blocos `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Não há razão para separar esses métodos em múltiplos blocos `impl` aqui,
mas esta é uma sintaxe válida. Veremos um caso em que múltiplos blocos `impl` são
úteis no Capítulo 10, onde discutiremos tipos genéricos e traits.

## Resumo

As structs permitem criar tipos personalizados que fazem sentido para o seu domínio. Ao
usar structs, você pode manter partes de dados relacionadas conectadas umas às outras
e nomear cada parte para tornar seu código claro. Em blocos `impl`, você pode definir
funções que estão associadas ao seu tipo, e os métodos são um tipo de função associada que
permite especificar o comportamento que as instâncias das suas structs possuem.

Mas as structs não são a única maneira de criar tipos personalizados: Vamos voltar
ao recurso de enum do Rust para adicionar outra ferramenta à sua caixa de ferramentas.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
