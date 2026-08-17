## Definindo um Enum

Onde as *structs* oferecem uma maneira de agrupar campos e dados relacionados,
como um `Rectangle` com sua `width` e `height`, os enums dão a você uma maneira de
dizer que um valor é um de um conjunto possível de valores. Por exemplo,
podemos querer dizer que `Rectangle` é um de um conjunto de formas possíveis que
também inclui `Circle` e `Triangle`. Para fazer isso, o Rust nos permite codificar
essas possibilidades como um enum.

Vamos analisar uma situação que gostaríamos de expressar em código e ver por que
os enums são úteis e mais apropriados do que as *structs* neste caso. Digamos que
precisamos trabalhar com endereços IP. Atualmente, dois padrões principais são
usados para endereços IP: versão quatro e versão seis. Como essas são as únicas
possibilidades para um endereço IP com as quais nosso programa irá se deparar,
podemos _enumerar_ todas as variantes possíveis, que é de onde a enumeração
recebe seu nome.

Qualquer endereço IP pode ser um endereço da versão quatro ou da versão seis,
mas não ambos ao mesmo tempo. Essa propriedade dos endereços IP torna a
estrutura de dados enum apropriada, pois um valor de enum só pode ser uma de
suas variantes. Tanto os endereços da versão quatro quanto os da versão seis
ainda são fundamentalmente endereços IP, portanto, devem ser tratados como o
mesmo tipo quando o código estiver lidando com situações que se aplicam a
qualquer tipo de endereço IP.

Podemos expressar esse conceito em código definindo uma enumeração `IpAddrKind` e
listando os tipos possíveis que um endereço IP pode ser, `V4` e `V6`. Estas são
as variantes do enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` agora é um tipo de dado personalizado que podemos usar em outras
partes do nosso código.

### Valores de Enum

Podemos criar instâncias de cada uma das duas variantes de `IpAddrKind` assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Note que as variantes do enum estão sob o namespace do seu identificador, e nós
usamos dois pontos duplos para separar os dois. Isso é útil porque agora ambos os
valores `IpAddrKind::V4` e `IpAddrKind::V6` são do mesmo tipo: `IpAddrKind`.
Podemos então, por exemplo, definir uma função que aceita qualquer
`IpAddrKind`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

E podemos chamar esta função com qualquer uma das variantes:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Usar enums tem ainda mais vantagens. Pensando mais sobre o nosso tipo de
endereço IP, no momento não temos uma maneira de armazenar os _dados_ reais do
endereço IP; nós apenas sabemos que _tipo_ ele é. Como você acabou de aprender
sobre *structs* no Capítulo 5, você pode ficar tentado a resolver esse problema
com *structs*, conforme mostrado na Listagem 6-1.

<Listing number="6-1" caption="Armazenando os dados e a variante `IpAddrKind` de um endereço IP usando uma `struct`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Aqui, definimos uma *struct* `IpAddr` que possui dois campos: um campo `kind` que
é do tipo `IpAddrKind` (o enum que definimos anteriormente) e um campo `address`
do tipo `String`. Temos duas instâncias desta *struct*. A primeira é `home`, e
ela tem o valor `IpAddrKind::V4` como seu `kind` com os dados de endereço
associados `127.0.0.1`. A segunda instância é `loopback`. Ela tem a outra
variante de `IpAddrKind` como seu valor de `kind`, `V6`, e tem o endereço `::1`
associado a ela. Usamos uma *struct* para agrupar os valores `kind` e `address`,
então agora a variante está associada ao valor.

No entanto, representar o mesmo conceito usando apenas um enum é mais conciso:
Em vez de um enum dentro de uma *struct*, podemos colocar dados diretamente em
cada variante do enum. Esta nova definição do enum `IpAddr` diz que ambas as
variantes `V4` e `V6` terão valores de `String` associados:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Anexamos dados diretamente a cada variante do enum, de modo que não há
necessidade de uma *struct* extra. Aqui, também é mais fácil ver outro detalhe de
como os enums funcionam: o nome de cada variante de enum que definimos também se
torna uma função que constrói uma instância do enum. Ou seja, `IpAddr::V4()` é
uma chamada de função que aceita um argumento `String` e retorna uma instância
do tipo `IpAddr`. Nós obtemos automaticamente essa função construtora definida
como resultado da definição do enum.

Há outra vantagem em usar um enum em vez de uma *struct*: cada variante pode ter
diferentes tipos e quantidades de dados associados. Os endereços IP da versão
quatro sempre terão quatro componentes numéricos que terão valores entre 0 e
255. Se quiséssemos armazenar endereços `V4` como quatro valores `u8`, mas ainda
expressar endereços `V6` como um único valor `String`, não poderíamos fazer isso
com uma *struct*. Os enums lidam com esse caso com facilidade:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Mostramos várias maneiras diferentes de definir estruturas de dados para
armazenar endereços IP da versão quatro e da versão seis. No entanto, acontece
que querer armazenar endereços IP e codificar qual o seu tipo é tão comum que [a
biblioteca padrão tem uma definição que podemos usar!][IpAddr]<!-- ignore -->
Vamos ver como a biblioteca padrão define `IpAddr`. Ela tem exatamente o enum e
as variantes que definimos e usamos, mas ela embute os dados de endereço dentro
das variantes na forma de duas *structs* diferentes, que são definidas de forma
diferente para cada variante:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Este código ilustra que você pode colocar qualquer tipo de dado dentro de uma
variante de enum: strings, tipos numéricos ou *structs*, por exemplo. Você pode
inclusive incluir outro enum! Além disso, os tipos da biblioteca padrão muitas
vezes não são muito mais complicados do que o que você mesmo poderia criar.

Note que, embora a biblioteca padrão contenha uma definição para `IpAddr`, ainda
podemos criar e usar nossa própria definição sem conflito, porque não trouxemos
a definição da biblioteca padrão para o nosso escopo. Falaremos mais sobre como
trazer tipos para o escopo no Capítulo 7.

Vamos olhar para outro exemplo de enum na Listagem 6-2: Este possui uma grande
variedade de tipos embutidos em suas variantes.

<Listing number="6-2" caption="Um enum `Message` cujas variantes armazenam diferentes quantidades e tipos de valores">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Este enum possui quatro variantes com tipos diferentes:

- `Quit`: Não possui nenhum dado associado a ele.
- `Move`: Possui campos nomeados, assim como uma *struct*.
- `Write`: Inclui uma única `String`.
- `ChangeColor`: Inclui três valores `i32`.

Definir um enum com variantes como as da Listagem 6-2 é semelhante a definir
diferentes tipos de definições de *struct*, exceto que o enum não usa a palavra-chave
`struct` e todas as variantes são agrupadas sob o tipo `Message`. As seguintes
*structs* poderiam conter os mesmos dados que as variantes de enum anteriores
contêm:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Mas se usássemos as diferentes *structs*, cada uma tendo seu próprio tipo, não
poderíamos definir tão facilmente uma função para aceitar qualquer um desses tipos
de mensagem como poderíamos com o enum `Message` definido na Listagem 6-2, que é
um único tipo.

Há mais uma semelhança entre enums e *structs*: assim como podemos definir métodos
em *structs* usando `impl`, também podemos definir métodos em enums. Aqui está
um método chamado `call` que poderíamos definir em nosso enum `Message`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

O corpo do método usaria `self` para obter o valor sobre o qual chamamos o
método. Neste exemplo, criamos uma variável `m` que tem o valor
`Message::Write(String::from("hello"))`, e é isso que `self` será no corpo do
método `call` quando `m.call()` for executado.

Vamos examinar outro enum na biblioteca padrão que é muito comum e útil:
`Option`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### O Enum `Option`

Esta seção explora um estudo de caso do `Option`, que é outro enum definido pela
biblioteca padrão. O tipo `Option` codifica o cenário muito comum em que um valor
pode ser alguma coisa ou pode ser nada.

Por exemplo, se você solicitar o primeiro item em uma lista não vazia, você
obterá um valor. Se você solicitar o primeiro item em uma lista vazia, você não
obterá nada. Expressar esse conceito em termos do sistema de tipos significa que
o compilaador pode verificar se você tratou todos os casos que deveria tratar;
essa funcionalidade pode prevenir bugs que são extremamente comuns em outras
linguagens de programação.

A criação de linguagens de programação é frequentemente pensada em termos de
quais recursos você inclui, mas os recursos que você exclui também são
importantes. O Rust não tem o recurso nulo (*null*) que muitas outras linguagens
possuem. *Null* é um valor que significa que não há nenhum valor ali. Em
linguagens com null, as variáveis podem sempre estar em um de dois estados: nulo
ou não nulo.

Em sua apresentação de 2009 “Null References: The Billion Dollar Mistake”, Tony
Hoare, o inventor do null, disse o seguinte:

> Eu chamo isso de meu erro de um bilhão de dólares. Naquela época, eu estava
> projetando o primeiro sistema de tipos abrangente para referências em uma
> linguagem orientada a objetos. Meu objetivo era garantir que todo o uso de
> referências fosse absolutamente seguro, com verificação executada
> automaticamente pelo compilador. Mas eu não pude resistir à tentação de colocar
> uma referência nula, simplesmente porque era muito fácil de implementar. Isso
> levou a inúmeros erros, vulnerabilidades e falhas de sistema, que provavelmente
> causaram um bilhão de dólares de dor e danos nos últimos quarenta anos.

O problema com valores nulos é que, se você tentar usar um valor nulo como um
valor não nulo, você receberá algum tipo de erro. Como essa propriedade de nulo
ou não nulo é onipresente, é extremamente fácil cometer esse tipo de erro.

No entanto, o conceito que o nulo está tentando expressar ainda é útil: um nulo
é um valor que atualmente é inválido ou ausente por algum motivo.

O problema não está realmente no conceito, mas na implementação particular.
Como tal, o Rust não tem nulos (*nulls*), mas tem um enum que pode codificar o
conceito de um valor estar presente ou ausente. Este enum é `Option<T>`, e ele
é [definido pela biblioteca padrão][option]<!-- ignore --> da seguinte forma:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

O enum `Option<T>` é tão útil que está incluído até mesmo no *prelude*; você não
precisa trazê-lo para o escopo explicitamente. Suas variantes também estão
incluídas no *prelude*: você pode usar `Some` e `None` diretamente sem o prefixo
`Option::`. O enum `Option<T>` ainda é apenas um enum comum, e `Some(T)` e `None`
ainda são variantes do tipo `Option<T>`.

A sintaxe `<T>` é um recurso do Rust sobre o qual ainda não falamos. É um
parâmetro de tipo genérico, e abordaremos genéricos com mais detalhes no
Capítulo 10. Por enquanto, tudo o que você precisa saber é que `<T>` significa
que a variante `Some` do enum `Option` pode conter um único dado de qualquer
tipo, e que cada tipo concreto usado no lugar de `T` torna o tipo `Option<T>`
geral um tipo diferente. Aqui estão alguns exemplos de uso de valores `Option`
para conter tipos numéricos e tipos de caractere:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

O tipo de `some_number` é `Option<i32>`. O tipo de `some_char` é `Option<char>`,
que é um tipo diferente. O Rust pode inferir esses tipos porque especificamos um
valor dentro da variante `Some`. Para `absent_number`, o Rust exige que façamos
a anotação do tipo `Option` geral: o compilador não pode inferir o tipo que a
variante `Some` correspondente conterá olhando apenas para um valor `None`.
Aqui, dizemos ao Rust que queremos que `absent_number` seja do tipo
`Option<i32>`.

Quando temos um valor `Some`, sabemos que um valor está presente, e esse valor
está contido dentro do `Some`. Quando temos um valor `None`, de certa forma ele
significa a mesma coisa que null: não temos um valor válido. Então, por que ter
`Option<T>` é melhor do que ter null?

Em suma, porque `Option<T>` e `T` (onde `T` pode ser qualquer tipo) são tipos
diferentes, o compilador não nos deixará usar um valor `Option<T>` como se ele
fosse definitivamente um valor válido. Por exemplo, este código não compila,
porque está tentando adicionar um `i8` a um `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Se executarmos este código, recebemos uma mensagem de erro como esta:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Intenso! Em efeito, esta mensagem de erro significa que o Rust não entende como
adicionar um `i8` e um `Option<i8>`, porque eles são tipos diferentes. Quando
temos um valor de um tipo como `i8` no Rust, o compilador garantirá que sempre
tenhamos um valor válido. Podemos prosseguir com confiança, sem precisar
verificar se há null antes de usar esse valor. Apenas quando temos um
`Option<i8>` (ou qualquer tipo de valor com o qual estejamos trabalhando) é que
temos que nos preocupar com a possibilidade de não termos um valor, e o
compilador garantirá que tratemos esse caso antes de usar o valor.

Em outras palavras, você deve converter um `Option<T>` em um `T` antes de poder
executar operações de `T` com ele. Geralmente, isso ajuda a pegar um dos
problemas mais comuns com null: assumir que algo não é nulo quando na verdade é.

Eliminar o risco de assumir incorretamente um valor não nulo ajuda você a ter
mais confiança no seu código. Para ter um valor que possivelmente possa ser nulo,
você deve optar explicitamente por isso, tornando o tipo desse valor
`Option<T>`. Então, quando você usa esse valor, você é obrigado a tratar
explicitamente o caso em que o valor é nulo. Em todos os lugares onde um valor
tem um tipo que não é um `Option<T>`, você _pode_ assumir com segurança que o
valor não é nulo. Esta foi uma decisão deliberada de design para o Rust limitar
a onipresença do null e aumentar a segurança do código Rust.

Então, como você extrai o valor `T` de uma variante `Some` quando você tem um
valor do tipo `Option<T>` para que possa usar esse valor? O enum `Option<T>` tem
um grande número de métodos que são úteis em uma variedade de situações; você
pode conferi-los [em sua documentação][docs]<!-- ignore -->. Familiarizar-se com
os métodos em `Option<T>` será extremamente útil em sua jornada com o Rust.

Em geral, para usar um valor `Option<T>`, você vai querer ter um código que lide
com cada variante. Você vai querer algum código que seja executado apenas quando
você tiver um valor `Some(T)`, e este código tem permissão para usar o `T` interno.
Você vai querer que algum outro código seja executado apenas se você tiver um
valor `None`, e esse código não tem um valor `T` disponível. A expressão `match`
é uma construção de fluxo de controle que faz exatamente isso quando usada com
enums: ela executa código diferente dependendo de qual variante do enum ela tem,
e esse código pode usar os dados dentro do valor correspondente.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
