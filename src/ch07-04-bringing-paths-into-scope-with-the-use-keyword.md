## Trazendo Caminhos para o Escopo com a Palavra-chave `use`

Ter que escrever os caminhos para chamar funções pode parecer inconveniente e
repetitivo. Na Listagem 7-7, independentemente de termos escolhido o caminho
absoluto ou relativo para a função `add_to_waitlist`, toda vez que queríamos
chamar `add_to_waitlist` tínhamos que especificar `front_of_house` e `hosting`
também. Felizmente, há uma maneira de simplificar esse processo: podemos criar um
atalho para um caminho com a palavra-chave `use` uma vez e, em seguida, usar o
nome mais curto em todos os outros lugares do escopo.

Na Listagem 7-11, trazemos o módulo `crate::front_of_house::hosting` para o
escopo da função `eat_at_restaurant` para que precisemos especificar apenas
`hosting::add_to_waitlist` para chamar a função `add_to_waitlist` em
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Trazendo um módulo para o escopo com `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Adicionar `use` e um caminho em um escopo é semelhante a criar um link simbólico
no sistema de arquivos. Ao adicionar `use crate::front_of_house::hosting` na raiz
da crate, `hosting` agora é um nome válido nesse escopo, exatamente como se o
módulo `hosting` tivesse sido definido na raiz da crate. Os caminhos trazidos
para o escopo com `use` também verificam a privacidade, como qualquer outro
caminho.

Note que `use` cria o atalho apenas para o escopo particular no qual o `use`
ocorre. A Listagem 7-12 move a função `eat_at_restaurant` para um novo módulo
filho chamado `customer`, que passa a ser um escopo diferente da instrução `use`,
então o corpo da função não compilará.

<Listing number="7-12" file-name="src/lib.rs" caption="Uma instrução `use` se aplica apenas no escopo em que está.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

O erro do compilador mostra que o atalho não se aplica mais dentro do módulo
`customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Note que também há um aviso de que o `use` não é mais usado em seu escopo! Para
corrigir esse problema, mova o `use` para dentro do módulo `customer` também, ou
referencie o atalho no módulo pai com `super::hosting` dentro do módulo filho
`customer`.

### Criando Caminhos `use` Idiomáticos

Na Listagem 7-11, você pode ter se perguntado por que especificamos `use
crate::front_of_house::hosting` e depois chamamos `hosting::add_to_waitlist` em
`eat_at_restaurant`, em vez de especificar o caminho `use` até a função
`add_to_waitlist` para alcançar o mesmo resultado, como na Listagem 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Trazendo a função `add_to_waitlist` para o escopo com `use`, o que não é idiomático">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Embora tanto a Listagem 7-11 quanto a Listagem 7-13 realizem a mesma tarefa, a
Listagem 7-11 é a maneira idiomática de trazer uma função para o escopo com
`use`. Trazer o módulo pai da função para o escopo com `use` significa que temos
que especificar o módulo pai ao chamar a função. Especificar o módulo pai ao
chamar a função deixa claro que a função não está definida localmente, ao mesmo
tempo em que minimiza a repetição do caminho completo. O código na Listagem 7-13
deixa incerto onde `add_to_waitlist` está definido.

Por outro lado, ao trazer structs, enums e outros itens com `use`, é idiomático
especificar o caminho completo. A Listagem 7-14 mostra a maneira idiomática de
trazer a struct `HashMap` da biblioteca padrão para o escopo de uma binary
crate (crate binária).

<Listing number="7-14" file-name="src/main.rs" caption="Trazendo `HashMap` para o escopo de maneira idiomática">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Não há uma razão forte por trás desse idioma: é apenas a convenção que emergiu,
e as pessoas se acostumaram a ler e escrever código Rust dessa forma.

A exceção a essa regra idiomática é se estivermos trazendo dois itens com o
mesmo nome para o escopo com instruções `use`, porque o Rust não permite isso. A
Listagem 7-15 mostra como trazer dois tipos `Result` para o escopo que têm o
mesmo nome, mas módulos pai diferentes, e como se referir a eles.

<Listing number="7-15" file-name="src/lib.rs" caption="Trazer dois tipos com o mesmo nome para o mesmo escopo requer o uso de seus módulos pai.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Como você pode ver, usar os módulos pai diferencia os dois tipos `Result`. Se
em vez disso especificássemos `use std::fmt::Result` e `use std::io::Result`,
teríamos dois tipos `Result` no mesmo escopo, e o Rust não saberia a qual deles
estaríamos nos referindo quando usássemos `Result`.

### Fornecendo Novos Nomes com a Palavra-chave `as`

Há outra solução para o problema de trazer dois tipos de mesmo nome para o
mesmo escopo com `use`: após o caminho, podemos especificar `as` e um novo nome
local, ou _apelido_ (alias), para o tipo. A Listagem 7-16 mostra outra maneira
de escrever o código da Listagem 7-15 renomeando um dos dois tipos `Result`
usando `as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Renomeando um tipo quando ele é trazido para o escopo com a palavra-chave `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Na segunda instrução `use`, escolhemos o novo nome `IoResult` para o tipo
`std::io::Result`, que não entrará em conflito com o `Result` de `std::fmt` que
também trouxemos para o escopo. Tanto a Listagem 7-15 quanto a Listagem 7-16 são
consideradas idiomáticas, então a escolha depende de você!

### Reexportando Nomes com `pub use`

Quando trazemos um nome para o escopo com a palavra-chave `use`, o nome é privado
ao escopo para o qual o importamos. Para permitir que o código fora desse escopo
se refira a esse nome como se ele tivesse sido definido nesse escopo, podemos
combinar `pub` e `use`. Essa técnica é chamada de _reexportação_
(re-exporting) porque estamos trazendo um item para o escopo e também tornando
esse item disponível para que outros o tragam para o escopo deles.

A Listagem 7-17 mostra o código da Listagem 7-11 com o `use` no módulo raiz
alterado para `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Tornando um nome disponível para qualquer código usar a partir de um novo escopo com `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Antes dessa alteração, o código externo teria que chamar a função
`add_to_waitlist` usando o caminho
`restaurant::front_of_house::hosting::add_to_waitlist()`, o que também teria
exigido que o módulo `front_of_house` fosse marcado como `pub`. Agora que este
`pub use` reexportou o módulo `hosting` a partir do módulo raiz, o código
externo pode usar o caminho `restaurant::hosting::add_to_waitlist()` em vez
disso.

A reexportação é útil quando a estrutura interna do seu código é diferente de
como os programadores que chamam seu código pensariam sobre o domínio. Por
exemplo, nesta metáfora de restaurante, as pessoas que administram o
restaurante pensam em "salão" (`front of house`) e "cozinha" (`back of house`).
Mas os clientes que visitam um restaurante provavelmente não pensarão nas partes
do restaurante nesses termos. Com `pub use`, podemos escrever nosso código com
uma estrutura, mas expor uma estrutura diferente. Fazer isso torna nossa
biblioteca bem organizada para os programadores que trabalham na biblioteca e
para os programadores que a chamam. Veremos outro exemplo de `pub use` e como
isso afeta a documentação da sua crate em [“Exportando uma API Pública
Conveniente”][ch14-pub-use]<!-- ignore --> no Capítulo 14.

### Usando Pacotes Externos

No Capítulo 2, programamos um projeto de jogo de adivinhação que usava um pacote
externo chamado `rand` para obter números aleatórios. Para usar `rand` em nosso
projeto, adicionamos esta linha ao _Cargo.toml_:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Adicionar `rand` como uma dependência no _Cargo.toml_ diz ao Cargo para baixar o
pacote `rand` e quaisquer dependências de [crates.io](https://crates.io/) e
tornar o `rand` disponível para o nosso projeto.

Então, para trazer as definições de `rand` para o escopo do nosso pacote,
adicionamos uma linha `use` começando com o nome da crate, `rand`, e listamos
os itens que queríamos trazer para o escopo. Lembre-se de que em [“Gerando um
Número Aleatório”][rand]<!-- ignore --> no Capítulo 2, trouxemos itens no
módulo `rand::prelude` para o escopo e chamamos a função `rand::rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Membros da comunidade Rust disponibilizaram muitos pacotes em
[crates.io](https://crates.io/), e puxar qualquer um deles para o seu pacote
envolve estes mesmos passos: listá-los no arquivo _Cargo.toml_ do seu pacote e
usar `use` para trazer itens de suas crates para o escopo.

Note que a biblioteca padrão `std` também é uma crate externa ao nosso pacote.
Como a biblioteca padrão é distribuída com a linguagem Rust, não precisamos
alterar o _Cargo.toml_ para incluir o `std`. Mas precisamos nos referir a ela
com `use` para trazer itens de lá para o escopo do nosso pacote. Por exemplo,
com o `HashMap` usaríamos esta linha:

```rust
use std::collections::HashMap;
```

Este é um caminho absoluto começando com `std`, o nome da crate da biblioteca
padrão.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### Usando Caminhos Aninhados para Limpar Grandes Listas de `use`

Se estivermos usando múltiplos itens definidos na mesma crate ou no mesmo
módulo, listar cada item em sua própria linha pode ocupar muito espaço vertical
em nossos arquivos. Por exemplo, estas duas instruções `use` que tínhamos no
jogo de adivinhação na Listagem 2-4 trazem itens de `std` para o escopo:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Em vez disso, podemos usar caminhos aninhados para trazer os mesmos itens para
o escopo em uma única linha. Fazemos isso especificando a parte comum do
caminho, seguida por dois pontos duplos e, em seguida, chaves envolvendo uma
lista das partes dos caminhos que diferem, conforme mostrado na Listagem 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Especificando um caminho aninhado para trazer vários itens com o mesmo prefixo para o escopo">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Em programas maiores, trazer muitos itens para o escopo da mesma crate ou
módulo usando caminhos aninhados pode reduzir bastante o número de instruções
`use` separadas necessárias!

Podemos usar um caminho aninhado em qualquer nível de um caminho, o que é útil
ao combinar duas instruções `use` que compartilham um subcaminho. Por exemplo, a
Listagem 7-19 mostra duas instruções `use`: uma que traz `std::io` para o escopo
e outra que traz `std::io::Write` para o escopo.

<Listing number="7-19" file-name="src/lib.rs" caption="Duas instruções `use` onde uma é um subcaminho da outra">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

A parte comum desses dois caminhos é `std::io`, e esse é o primeiro caminho
completo. Para mesclar esses dois caminhos em uma única instrução `use`,
podemos usar `self` no caminho aninhado, conforme mostrado na Listagem 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Combinando os caminhos da Listagem 7-19 em uma única instrução `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Esta linha traz `std::io` e `std::io::Write` para o escopo.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### Importando Itens com o Operador Glob

Se quisermos trazer _todos_ os itens públicos definidos em um caminho para o
escopo, podemos especificar esse caminho seguido pelo operador glob `*`:

```rust
use std::collections::*;
```

Esta instrução `use` traz todos os itens públicos definidos em
`std::collections` para o escopo atual. Tome cuidado ao usar o operador glob!
O Glob pode tornar mais difícil dizer quais nomes estão no escopo e onde um
nome usado em seu programa foi definido. Além disso, se a dependência alterar
suas definições, o que você importou também mudará, o que pode levar a erros de
compilação ao atualizar a dependência caso ela adicione uma definição com o
mesmo nome que uma definição sua no mesmo escopo, por exemplo.

O operador glob é frequentemente usado em testes para trazer tudo o que está
sendo testado para o módulo `tests`; falaremos sobre isso em [“Como Escrever
Testes”][writing-tests]<!-- ignore --> no Capítulo 11. O operador glob também é
às vezes usado como parte do padrão prelude: Veja [a documentação da biblioteca
padrão](../std/prelude/index.html#other-preludes)<!-- ignore --> para mais
informações sobre esse padrão.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests