## Tipos de Dados Genéricos

Usamos genéricos para criar definições para itens como assinaturas de funções ou estruturas (`structs`), que podemos usar com vários tipos de dados concretos diferentes. Primeiro, vamos ver como definir funções, estruturas, enums e métodos usando genéricos. Depois, discutiremos como os genéricos afetam o desempenho do código.

### Em Definições de Funções

Ao definir uma função que usa genéricos, colocamos os genéricos na assinatura da função onde normalmente especificaríamos os tipos de dados dos parâmetros e do valor de retorno. Fazer isso torna nosso código mais flexível e oferece mais funcionalidade para quem chama nossa função, ao mesmo tempo em que evita a duplicação de código.

Continuando com nossa função `largest` (maior), a Listagem 10-4 mostra duas funções que encontram o maior valor em uma fatia (`slice`). Em seguida, combinaremos essas funções em uma única função que usa genéricos.

<Listing number="10-4" file-name="src/main.rs" caption="Duas funções que diferem apenas em seus nomes e nos tipos em suas assinaturas">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

A função `largest_i32` é a que extraímos na Listagem 10-3 para encontrar o maior `i32` em uma fatia. A função `largest_char` encontra o maior `char` em uma fatia. Os corpos das funções contêm o mesmo código, então vamos eliminar a duplicação introduzindo um parâmetro de tipo genérico em uma única função.

Para parametrizar os tipos em uma nova função única, precisamos nomear o parâmetro de tipo, assim como fazemos para os parâmetros de valor de uma função. Você pode usar qualquer identificador como nome de parâmetro de tipo. Mas usaremos `T` porque, por convenção, os nomes de parâmetros de tipo em Rust são curtos, geralmente apenas uma letra, e a convenção de nomenclatura de tipos do Rust é `UpperCamelCase`. Abreviação de _type_ (tipo), `T` é a escolha padrão da maioria dos programadores Rust.

Quando usamos um parâmetro no corpo da função, temos que declarar o nome do parâmetro na assinatura para que o compilador saiba o que esse nome significa. Da mesma forma, quando usamos o nome de um parâmetro de tipo na assinatura de uma função, temos que declarar o nome do parâmetro de tipo antes de usá-lo. Para definir a função genérica `largest`, colocamos declarações de nomes de tipos entre colchetes angulares, `<>`, entre o nome da função e a lista de parâmetros, assim:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Lemos esta definição como: “A função `largest` é genérica sobre algum tipo `T`”. Esta função tem um parâmetro chamado `list`, que é uma fatia de valores do tipo `T`. A função `largest` retornará uma referência a um valor do mesmo tipo `T`.

A Listagem 10-5 mostra a definição combinada da função `largest` usando o tipo de dado genérico em sua assinatura. A listagem também mostra como podemos chamar a função com uma fatia de valores `i32` ou valores `char`. Note que este código ainda não compila.

<Listing number="10-5" file-name="src/main.rs" caption="A função `largest` usando parâmetros de tipo genérico; isto ainda não compila">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Se compilarmos este código agora, receberemos este erro:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

O texto de ajuda menciona `std::cmp::PartialOrd`, que é um *trait*, e vamos falar sobre *traits* na próxima seção. Por enquanto, saiba que este erro afirma que o corpo de `largest` não funcionará para todos os tipos possíveis que `T` poderia ser. Como queremos comparar valores do tipo `T` no corpo, só podemos usar tipos cujos valores possam ser ordenados. Para permitir comparações, a biblioteca padrão possui o *trait* `std::cmp::PartialOrd` que você pode implementar em tipos (veja o Apêndice C para mais informações sobre este *trait*). Para corrigir a Listagem 10-5, podemos seguir a sugestão do texto de ajuda e restringir os tipos válidos para `T` apenas à daqueles que implementam `PartialOrd`. A listagem então compilará, porque a biblioteca padrão implementa `PartialOrd` tanto em `i32` quanto em `char`.

### Em Definições de Estruturas (`Structs`)

Também podemos definir estruturas para usar um parâmetro de tipo genérico em um ou mais campos usando a sintaxe `<>`. A Listagem 10-6 define uma estrutura `Point<T>` para armazenar valores de coordenadas `x` e `y` de qualquer tipo.

<Listing number="10-6" file-name="src/main.rs" caption="Uma estrutura `Point<T>` que armazena valores `x` e `y` do tipo `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

A sintaxe para usar genéricos em definições de estruturas é semelhante à usada em definições de funções. Primeiro, declaramos o nome do parâmetro de tipo entre colchetes angulares logo após o nome da estrutura. Em seguida, usamos o tipo genérico na definição da estrutura onde, de outra forma, especificaríamos tipos de dados concretos.

Note que, como usamos apenas um tipo genérico para definir `Point<T>`, esta definição diz que a estrutura `Point<T>` é genérica sobre algum tipo `T`, e os campos `x` e `y` são *ambos* desse mesmo tipo, seja ele qual for. Se criarmos uma instância de um `Point<T>` que tenha valores de tipos diferentes, como na Listagem 10-7, nosso código não compilará.

<Listing number="10-7" file-name="src/main.rs" caption="Os campos `x` e `y` devem ser do mesmo tipo porque ambos têm o mesmo tipo de dado genérico `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Neste exemplo, quando atribuímos o valor inteiro `5` a `x`, deixamos o compilador saber que o tipo genérico `T` será um inteiro para esta instância de `Point<T>`. Então, quando especificamos `4.0` para `y`, que definimos para ter o mesmo tipo que `x`, receberemos um erro de incompatibilidade de tipos como este:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Para definir uma estrutura `Point` onde `x` e `y` são ambos genéricos, mas podem ter tipos diferentes, podemos usar múltiplos parâmetros de tipo genérico. Por exemplo, na Listagem 10-8, alteramos a definição de `Point` para ser genérica sobre os tipos `T` e `U`, onde `x` é do tipo `T` e `y` é do tipo `U`.

<Listing number="10-8" file-name="src/main.rs" caption="Um `Point<T, U>` genérico sobre dois tipos para que `x` e `y` possam ser valores de tipos diferentes">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Agora todas as instâncias mostradas de `Point` são permitidas! Você pode usar quantos parâmetros de tipo genérico quiser em uma definição, mas usar mais do que alguns torna seu código difícil de ler. Se você perceber que precisa de muitos tipos genéricos em seu código, isso pode indicar que seu código precisa ser reestruturado em pedaços menores.

### Em Definições de Enums

Assim como fizemos com estruturas, podemos definir enums para armazenar tipos de dados genéricos em suas variantes. Vamos dar outra olhada no enum `Option<T>` que a biblioteca padrão fornece, o qual usamos no Capítulo 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Esta definição agora deve fazer mais sentido para você. Como você pode ver, o enum `Option<T>` é genérico sobre o tipo `T` e tem duas variantes: `Some`, que armazena um valor do tipo `T`, e uma variante `None`, que não armazena nenhum valor. Ao usar o enum `Option<T>`, podemos expressar o conceito abstrato de um valor opcional e, como `Option<T>` é genérico, podemos usar essa abstração independentemente do tipo do valor opcional.

Enums também podem usar múltiplos tipos genéricos. A definição do enum `Result` que usamos no Capítulo 9 é um exemplo:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

O enum `Result` é genérico sobre dois tipos, `T` e `E`, e tem duas variantes: `Ok`, que armazena um valor do tipo `T`, e `Err`, que armazena um valor do tipo `E`. Esta definição torna conveniente usar o enum `Result` em qualquer lugar onde temos uma operação que pode ser bem-sucedida (retornar um valor de algum tipo `T`) ou falhar (retornar um erro de algum tipo `E`). De fato, foi isso que usamos para abrir um arquivo na Listagem 9-3, onde `T` foi preenchido com o tipo `std::fs::File` quando o arquivo foi aberto com sucesso e `E` foi preenchido com o tipo `std::io::Error` quando houve problemas ao abrir o arquivo.

Quando você reconhecer situações em seu código com várias definições de estruturas ou enums que diferem apenas nos tipos dos valores que armazenam, você pode evitar a duplicação usando tipos genéricos em vez disso.

### Em Definições de Métodos

Podemos implementar métodos em estruturas e enums (como fizemos no Capítulo 5) e usar tipos genéricos em suas definições também. A Listagem 10-9 mostra a estrutura `Point<T>` que definimos na Listagem 10-6 com um método chamado `x` implementado nela.

<Listing number="10-9" file-name="src/main.rs" caption="Implementando um método chamado `x` na estrutura `Point<T>` que retornará uma referência ao campo `x` do tipo `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Aqui, definimos um método chamado `x` em `Point<T>` que retorna uma referência aos dados no campo `x`.

Note que temos que declarar `T` logo após `impl` para que possamos usar `T` para especificar que estamos implementando métodos no tipo `Point<T>`. Ao declarar `T` como um tipo genérico após `impl`, o Rust pode identificar que o tipo entre colchetes angulares em `Point` é um tipo genérico em vez de um tipo concreto. Poderíamos ter escolhido um nome diferente para este parâmetro genérico do que o parâmetro genérico declarado na definição da estrutura, mas usar o mesmo nome é uma convenção. Se você escrever um método dentro de um `impl` que declara um tipo genérico, esse método será definido em qualquer instância do tipo, não importa qual tipo concreto acabe substituindo o tipo genérico.

Também podemos especificar restrições em tipos genéricos ao definir métodos no tipo. Poderíamos, por exemplo, implementar métodos apenas em instâncias de `Point<f32>` em vez de em instâncias de `Point<T>` com qualquer tipo genérico. Na Listagem 10-10, usamos o tipo concreto `f32`, o que significa que não declaramos nenhum tipo após `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="Um bloco `impl` que se aplica apenas a uma estrutura com um tipo concreto específico para o parâmetro de tipo genérico `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Este código significa que o tipo `Point<f32>` terá um método `distance_from_origin`; outras instâncias de `Point<T>` onde `T` não é do tipo `f32` não terão este método definido. O método mede quão longe nosso ponto está do ponto nas coordenadas (0.0, 0.0) e usa operações matemáticas que estão disponíveis apenas para tipos de ponto flutuante.

Parâmetros de tipo genérico em uma definição de estrutura nem sempre são os mesmos que você usa nas assinaturas de métodos dessa mesma estrutura. A Listagem 10-11 usa os tipos genéricos `X1` e `Y1` para a estrutura `Point` e `X2` e `Y2` para a assinatura do método `mixup` para tornar o exemplo mais claro. O método cria uma nova instância de `Point` com o valor `x` do `Point` `self` (do tipo `X1`) e o valor `y` do `Point` passado como argumento (do tipo `Y2`).

<Listing number="10-11" file-name="src/main.rs" caption="Um método que usa tipos genéricos que são diferentes da definição de sua estrutura">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

Em `main`, definimos um `Point` que tem um `i32` para `x` (com valor `5`) e um `f64` para `y` (com valor `10.4`). A variável `p2` é uma estrutura `Point` que tem uma fatia de string (*string slice*) para `x` (com valor `"Hello"`) e um `char` para `y` (com valor `c`). Chamar `mixup` em `p1` com o argumento `p2` nos dá `p3`, que terá um `i32` para `x` porque `x` veio de `p1`. A variável `p3` terá um `char` para `y` porque `y` veio de `p2`. A chamada da macro `println!` imprimirá `p3.x = 5, p3.y = c`.

O propósito deste exemplo é demonstrar uma situação em que alguns parâmetros genéricos são declarados com `impl` e alguns são declarados com a definição do método. Aqui, os parâmetros genéricos `X1` e `Y1` são declarados após `impl` porque eles acompanham a definição da estrutura. Os parâmetros genéricos `X2` e `Y2` são declarados após `fn mixup` porque eles são relevantes apenas para o método.

### Desempenho do Código Usando Genéricos

Você pode estar se perguntando se há um custo em tempo de execução ao usar parâmetros de tipo genérico. A boa notícia é que usar tipos genéricos não fará seu programa rodar mais devagar do que faria com tipos concretos.

O Rust consegue isso realizando a monomorfização (*monomorphization*) do código que usa genéricos em tempo de compilação. *Monomorfização* é o processo de transformar código genérico em código específico, preenchendo os tipos concretos que são usados quando compilado. Nesse processo, o compilador faz o oposto das etapas que usamos para criar a função genérica na Listagem 10-5: o compilador olha para todos os lugares onde o código genérico é chamado e gera código para os tipos concretos com os quais o código genérico é chamado.

Vamos ver como isso funciona usando o enum genérico `Option<T>` da biblioteca padrão:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Quando o Rust compila este código, ele realiza a monomorfização. Durante esse processo, o compilador lê os valores que foram usados nas instâncias de `Option<T>` e identifica dois tipos de `Option<T>`: um é `i32` e o outro é `f64`. Como tal, ele expande a definição genérica de `Option<T>` em duas definições especializadas para `i32` e `f64`, substituindo assim a definição genérica pelas específicas.

A versão monomorfizada do código se parece com o seguinte (o compilador usa nomes diferentes dos que estamos usando aqui para ilustração):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

O `Option<T>` genérico é substituído pelas definições específicas criadas pelo compilador. Como o Rust compila código genérico em código que especifica o tipo em cada instância, não pagamos nenhum custo em tempo de execução pelo uso de genéricos. Quando o código é executado, ele tem o mesmo desempenho que teria se tivéssemos duplicado cada definição à mão. O processo de monomorfização torna os genéricos do Rust extremamente eficientes em tempo de execução.
