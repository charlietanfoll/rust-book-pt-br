## Traits Avançados

Nós abordamos os traits pela primeira vez na seção [“Definindo Comportamento Compartilhado com Traits”][traits]<!-- ignore --> no Capítulo 10, mas não discutimos os detalhes mais avançados. Agora que você sabe mais sobre o Rust, podemos entrar a fundo nos detalhes técnicos.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### Definindo Traits com Tipos Associados

Os _tipos associados_ conectam um marcador de tipo (placeholder) a um trait, de modo que as definições de métodos do trait possam usar esses tipos marcadores em suas assinaturas. O implementador de um trait especificará o tipo concreto a ser usado no lugar do tipo marcador para aquela implementação específica. Dessa forma, podemos definir um trait que usa alguns tipos sem precisar saber exatamente quais são esses tipos até que o trait seja implementado.

Descrevemos a maioria dos recursos avançados neste capítulo como sendo raramente necessários. Os tipos associados estão em uma posição intermediária: eles são usados de forma mais rara do que os recursos explicados no restante do livro, mas de forma mais comum do que muitos dos outros recursos discutidos neste capítulo.

Um exemplo de trait com um tipo associado é o trait `Iterator` fornecido pela biblioteca padrão. O tipo associado se chama `Item` e representa o tipo dos valores sobre os quais o tipo que implementa o trait `Iterator` está iterando. A definição do trait `Iterator` é mostrada na Listagem 20-13.

<Listing number="20-13" caption="A definição do trait `Iterator` que possui um tipo associado `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

O tipo `Item` é um marcador, e a definição do método `next` mostra que ele retornará valores do tipo `Option<Self::Item>`. Os implementadores do trait `Iterator` especificarão o tipo concreto para `Item`, e o método `next` retornará um `Option` contendo um valor desse tipo concreto.

Os tipos associados podem parecer um conceito semelhante aos genéricos, uma vez que estes últimos nos permitem definir uma função sem especificar quais tipos ela pode manipular. Para examinar a diferença entre os dois conceitos, veremos uma implementação do trait `Iterator` em um tipo chamado `Counter` que especifica que o tipo `Item` é `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Esta sintaxe parece comparável à dos genéricos. Então, por que não simplesmente definir o trait `Iterator` com genéricos, como mostrado na Listagem 20-14?

<Listing number="20-14" caption="Uma definição hipotética do trait `Iterator` usando genéricos">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

A diferença é que, ao usar genéricos, como na Listagem 20-14, devemos anotar os tipos em cada implementação; como também podemos implementar `Iterator<String> for Counter` ou qualquer outro tipo, poderíamos ter múltiplas implementações de `Iterator` para `Counter`. Em outras palavras, quando um trait tem um parâmetro genérico, ele pode ser implementado para um tipo várias vezes, alterando os tipos concretos dos parâmetros de tipo genérico a cada vez. Quando usamos o método `next` em `Counter`, teríamos que fornecer anotações de tipo para indicar qual implementação de `Iterator` queremos usar.

Com tipos associados, não precisamos anotar tipos, porque não podemos implementar um trait em um tipo várias vezes. Na Listagem 20-13, com a definição que usa tipos associados, podemos escolher qual será o tipo de `Item` apenas uma vez, pois pode haver apenas um `impl Iterator for Counter`. Não precisamos especificar que queremos um iterador de valores `u32` em todos os lugares onde chamamos `next` em `Counter`.

Os tipos associados também passam a fazer parte do contrato do trait: os implementadores do trait devem fornecer um tipo para substituir o marcador de tipo associado. Os tipos associados geralmente têm um nome que descreve como o tipo será usado, e documentar o tipo associado na documentação da API é uma boa prática.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### Usando Parâmetros Genéricos Padrão e Sobrecarga de Operadores

Quando usamos parâmetros de tipo genérico, podemos especificar um tipo concreto padrão para o tipo genérico. Isso elimina a necessidade de os implementadores do trait especificarem um tipo concreto se o tipo padrão funcionar. Você especifica um tipo padrão ao declarar um tipo genérico com a sintaxe `<PlaceholderType=ConcreteType>`.

Um ótimo exemplo de situação em que essa técnica é útil é a _sobrecarga de operadores_ (operator overloading), na qual você personaliza o comportamento de um operador (como `+`) em situações específicas.

O Rust não permite que você crie seus próprios operadores ou sobrecarregue operadores arbitrários. Mas você pode sobrecarregar as operações e os traits correspondentes listados em `std::ops` implementando os traits associados ao operador. Por exemplo, na Listagem 20-15, sobrecarregamos o operador `+` para somar duas instâncias de `Point`. Fazemos isso implementando o trait `Add` em uma struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Implementando o trait `Add` para sobrecarregar o operador `+` para instâncias de `Point`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

O método `add` soma os valores `x` de duas instâncias de `Point` e os valores `y` de duas instâncias de `Point` para criar um novo `Point`. O trait `Add` possui um tipo associado chamado `Output` que determina o tipo retornado pelo método `add`.

O tipo genérico padrão neste código está dentro do trait `Add`. Aqui está a sua definição:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Este código deve parecer geralmente familiar: um trait com um método e um tipo associado. A parte nova é `Rhs=Self`: esta sintaxe é chamada de _parâmetros de tipo padrão_ (default type parameters). O parâmetro de tipo genérico `Rhs` (abreviação para “right-hand side” ou lado direito) define o tipo do parâmetro `rhs` no método `add`. Se não especifirmamos um tipo concreto para `Rhs` quando implementamos o trait `Add`, o tipo de `Rhs` usará `Self` por padrão, que será o tipo no qual estamos implementando o `Add`.

Quando implementamos `Add` para `Point`, usamos o padrão para `Rhs` porque queríamos somar duas instâncias de `Point`. Vejamos um exemplo de implementação do trait `Add` onde queremos personalizar o tipo `Rhs` em vez de usar o padrão.

Temos duas structs, `Millimeters` e `Meters`, que armazenam valores em unidades diferentes. Esse empacotamento leve de um tipo existente em outra struct é conhecido como o _padrão newtype_, que descrevemos com mais detalhes na seção [“Implementando Traits Externos com o Padrão Newtype”][newtype]<!-- ignore -->. Queremos somar valores em milímetros a valores em metros e fazer com que a implementação de `Add` faça a conversão corretamente. Podemos implementar `Add` para `Millimeters` com `Meters` como o `Rhs`, conforme mostrado na Listagem 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Implementando o trait `Add` em `Millimeters` para somar `Millimeters` e `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Para somar `Millimeters` e `Meters`, especificamos `impl Add<Meters>` para definir o valor do parâmetro de tipo `Rhs` em vez de usar o padrão `Self`.

Você usará parâmetros de tipo padrão de duas maneiras principais:

1. Para estender um tipo sem quebrar o código existente
2. Para permitir personalização em casos específicos que a maioria dos usuários não precisará

O trait `Add` da biblioteca padrão é um exemplo do segundo propósito: geralmente, você somará dois tipos iguais, mas o trait `Add` oferece a capacidade de personalizar além disso. Usar um parâmetro de tipo padrão na definição do trait `Add` significa que você não precisa especificar o parâmetro extra na maioria das vezes. Em outras palavras, um pouco de código boilerplate de implementação deixa de ser necessário, tornando o uso do trait mais fácil.

O primeiro propósito é semelhante ao segundo, mas ao contrário: se você quiser adicionar um parâmetro de tipo a um trait existente, poderá dar a ele um valor padrão para permitir a extensão da funcionalidade do trait sem quebrar o código de implementação existente.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### Eliminando Ambiguidade Entre Métodos com Nomes Idênticos

Nada no Rust impede que um trait tenha um método com o mesmo nome que o método de outro trait, nem o Rust impede que você implemente ambos os traits em um único tipo. Também é possível implementar um método diretamente no tipo com o mesmo nome que os métodos dos traits.

Ao chamar métodos com o mesmo nome, você precisará dizer ao Rust qual deles deseja usar. Considere o código na Listagem 20-17, onde definimos dois traits, `Pilot` e `Wizard`, que possuem um método chamado `fly`. Em seguida, implementamos ambos os traits em um tipo `Human` que já possui um método chamado `fly` implementado diretamente nele. Cada método `fly` faz algo diferente.

<Listing number="20-17" file-name="src/main.rs" caption="Dois traits são definidos para ter um método `fly` e são implementados no tipo `Human`, e um método `fly` é implementado diretamente em `Human`.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Quando chamamos `fly` em uma instância de `Human`, o compilador chama por padrão o método que está diretamente implementado no tipo, conforme mostrado na Listagem 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Chamando `fly` em uma instância de `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

A execução deste código imprimirá `*waving arms furiously*`, mostrando que o Rust chamou diretamente o método `fly` implementado em `Human`.

Para chamar os métodos `fly` do trait `Pilot` ou do trait `Wizard`, precisamos usar uma sintaxe mais explícita para especificar qual método `fly` queremos dizer. A Listagem 20-19 demonstra esta sintaxe.

<Listing number="20-19" file-name="src/main.rs" caption="Especificando o método `fly` de qual trait queremos chamar">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Especificar o nome do trait antes do nome do método esclarece ao Rust qual implementação de `fly` queremos chamar. Também poderíamos escrever `Human::fly(&person)`, o que é equivalente ao `person.fly()` que usamos na Listagem 20-19, mas isso é um pouco mais longo de escrever se não precisarmos eliminar ambiguidades.

A execução deste código imprime o seguinte:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Como o método `fly` aceita um parâmetro `self`, se tivéssemos dois _tipos_ que implementassem um único _trait_, o Rust conseguiria descobrir qual implementação de trait usar com base no tipo de `self`.

No entanto, funções associadas que não são métodos não possuem um parâmetro `self`. Quando há vários tipos ou traits que definem funções que não são métodos com o mesmo nome de função, o Rust nem sempre sabe a qual tipo você se refere, a menos que você use a sintaxe totalmente qualificada. Por exemplo, na Listagem 20-20, criamos um trait para um abrigo de animais que quer nomear todos os filhotes de cães como Spot. Criamos um trait `Animal` com uma função associada que não é método chamada `baby_name`. O trait `Animal` é implementado para a struct `Dog`, na qual também fornecemos uma função associada que não é método chamada `baby_name` diretamente.

<Listing number="20-20" file-name="src/main.rs" caption="Um trait com uma função associada e um tipo com uma função associada de mesmo nome que também implementa o trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Implementamos o código para nomear todos os filhotes como Spot na função associada `baby_name` que é definida em `Dog`. O tipo `Dog` também implementa o trait `Animal`, que descreve características que todos os animais possuem. Filhotes de cães são chamados de puppies (filhotes), e isso é expresso na implementação do trait `Animal` em `Dog` na função `baby_name` associada ao trait `Animal`.

Em `main`, chamamos a função `Dog::baby_name`, que chama diretamente a função associada definida em `Dog`. Este código imprime o seguinte:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Esta saída não é o que queríamos. Queremos chamar a função `baby_name` que faz parte do trait `Animal` que implementamos em `Dog` para que o código imprima `A baby dog is called a puppy`. A técnica de especificar o nome do trait que usamos na Listagem 20-19 não ajuda aqui; se alterarmos `main` para o código na Listagem 20-21, teremos um erro de compilação.

<Listing number="20-21" file-name="src/main.rs" caption="Tentando chamar a função `baby_name` do trait `Animal`, mas o Rust não sabe qual implementação usar">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Como `Animal::baby_name` não tem um parâmetro `self`, e pode haver outros tipos que implementam o trait `Animal`, o Rust não consegue descobrir qual implementação de `Animal::baby_name` nós queremos. Receberemos este erro do compilador:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Para eliminar a ambiguidade e dizer ao Rust que queremos usar a implementação de `Animal` para `Dog` em vez da implementação de `Animal` para algum outro tipo, precisamos usar a sintaxe totalmente qualificada. A Listagem 20-22 demonstra como usar a sintaxe totalmente qualificada.

<Listing number="20-22" file-name="src/main.rs" caption="Usando a sintaxe totalmente qualificada para especificar que queremos chamar a função `baby_name` do trait `Animal` conforme implementada em `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Estamos fornecendo ao Rust uma anotação de tipo dentro dos colchetes angulares, o que indica que queremos chamar o método `baby_name` do trait `Animal` implementado em `Dog`, dizendo que queremos tratar o tipo `Dog` como um `Animal` para esta chamada de função. Este código agora imprimirá o que queremos:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Em geral, a sintaxe totalmente qualificada é definida da seguinte forma:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Para funções associadas que não são métodos, não haverá um `receiver`: haverá apenas a lista de outros argumentos. Você pode usar a sintaxe totalmente qualificada em todos os lugares onde chama funções ou métodos. No entanto, você tem permissão para omitir qualquer parte dessa sintaxe que o Rust possa deduzir a partir de outras informações no programa. Você só precisa usar esta sintaxe mais detalhada nos casos em que há múltiplas implementações que usam o mesmo nome e o Rust precisa de ajuda para identificar qual implementação você deseja chamar.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Usando Supertraits

Às vezes, você pode escrever a definição de um trait que depende de outro trait: para que um tipo implemente o primeiro trait, você deseja exigir que esse tipo também implemente o segundo trait. Você faria isso para que a definição do seu trait possa fazer uso dos itens associados do segundo trait. O trait do qual a definição do seu trait depende é chamado de _supertrait_ do seu trait.

Por exemplo, digamos que queremos criar um trait `OutlinePrint` com um método `outline_print` que imprimirá um determinado valor formatado de modo que seja emoldurado por asteriscos. Ou seja, dada uma struct `Point` que implementa o trait da biblioteca padrão `Display` para resultar em `(x, y)`, quando chamamos `outline_print` em uma instância de `Point` que tem `1` para `x` e `3` para `y`, ela deve imprimir o seguinte:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Na implementação do método `outline_print`, queremos usar a funcionalidade do trait `Display`. Portanto, precisamos especificar que o trait `OutlinePrint` funcionará apenas para tipos que também implementem `Display` e forneçam a funcionalidade que o `OutlinePrint` precisa. Podemos fazer isso na definição do trait especificando `OutlinePrint: Display`. Essa técnica é semelhante a adicionar um limite de trait (trait bound) ao trait. A Listagem 20-23 mostra uma implementação do trait `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Implementando o trait `OutlinePrint` que requer a funcionalidade de `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Como especificamos que `OutlinePrint` requer o trait `Display`, podemos usar a função `to_string` que é implementada automaticamente para qualquer tipo que implemente `Display`. Se tentássemos usar `to_string` sem adicionar dois pontos e especificar o trait `Display` após o nome do trait, receberíamos um erro dizendo que nenhum método chamado `to_string` foi encontrado para o tipo `&Self` no escopo atual.

Vamos ver o que acontece quando tentamos implementar `OutlinePrint` em um tipo que não implementa `Display`, como a struct `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Recebemos um erro dizendo que `Display` é obrigatório, mas não foi implementado:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Para corrigir isso, implementamos `Display` em `Point` e satisfazemos a restrição que o `OutlinePrint` exige, assim:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Então, implementar o trait `OutlinePrint` em `Point` compilará com sucesso, e podemos chamar `outline_print` em uma instância de `Point` para exibi-la dentro de um contorno de asteriscos.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### Implementando Traits Externos com o Padrão Newtype

Na seção [“Implementando um Trait em um Tipo”][implementing-a-trait-on-a-type]<!-- ignore --> no Capítulo 10, mencionamos a regra órfã (orphan rule), que afirma que só temos permissão para implementar um trait em um tipo se o trait, o tipo ou ambos forem locais ao nosso crate. É possível contornar essa restrição usando o padrão newtype, que envolve a criação de um novo tipo em uma struct de tupla. (Cobrimos structs de tupla na seção [“Criando Diferentes Tipos com Structs de Tupla”][tuple-structs]<!-- ignore --> no Capítulo 5.) A struct de tupla terá um campo e será um wrapper leve em torno do tipo para o qual queremos implementar um trait. Assim, o tipo wrapper é local ao nosso crate, e podemos implementar o trait no wrapper. _Newtype_ é um termo originado da linguagem de programação Haskell. Não há penalidade de desempenho em tempo de execução ao usar esse padrão, e o tipo wrapper é eliminado em tempo de compilação.

Como exemplo, digamos que queremos implementar `Display` em `Vec<T>`, o que a regra órfã nos impede de fazer diretamente porque o trait `Display` e o tipo `Vec<T>` são definidos fora do nosso crate. Podemos criar uma struct `Wrapper` que armazena uma instância de `Vec<T>`; então, podemos implementar `Display` em `Wrapper` e usar o valor de `Vec<T>`, conforme mostrado na Listagem 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Criando um tipo `Wrapper` em torno de `Vec<String>` para implementar `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

A implementação de `Display` usa `self.0` para acessar o `Vec<T>` interno porque `Wrapper` é uma struct de tupla e `Vec<T>` é o item no índice 0 da tupla. Em seguida, podemos usar a funcionalidade do trait `Display` em `Wrapper`.

O lado negativo de usar essa técnica é que `Wrapper` é um novo tipo, portanto ele não possui os métodos do valor que está armazenando. Teríamos que implementar todos os métodos de `Vec<T>` diretamente em `Wrapper` de forma que os métodos deleguem para `self.0`, o que nos permitiria tratar `Wrapper` exatamente como um `Vec<T>`. Se quiséssemos que o novo tipo tivesse todos os métodos que o tipo interno possui, implementar o trait `Deref` no `Wrapper` para retornar o tipo interno seria uma solução (discutimos a implementação do trait `Deref` na seção [“Tratando Ponteiros Inteligentes como Referências Comuns”][smart-pointer-deref]<!-- ignore --> no Capítulo 15). Se não quiséssemos que o tipo `Wrapper` tivesse todos os métodos do tipo interno — por exemplo, para restringir o comportamento do tipo `Wrapper` —, teríamos que implementar manualmente apenas os métodos que desejamos.

Esse padrão newtype também é útil mesmo quando traits não estão envolvidos. Vamos mudar o foco e analisar algumas maneiras avançadas de interagir com o sistema de tipos do Rust.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
