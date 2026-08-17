<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## Tratando Ponteiros Inteligentes como Referências Comuns

Implementar o *trait* `Deref` permite que você personalize o comportamento do
_operador de desreferenciação_ `*` (não deve ser confundido com o operador de
multiplicação ou glob). Ao implementar `Deref` de forma que um ponteiro
inteligente possa ser tratado como uma referência comum, você pode escrever
código que opera em referências e usar esse código com ponteiros inteligentes
também.

Primeiro, vamos ver como o operador de desreferenciação funciona com
referências comuns. Depois, tentaremos definir um tipo personalizado que se
comporte como `Box<T>` e ver por que o operador de desreferenciação não funciona
como uma referência em nosso tipo recém-definido. Vamos explorar como a
implementação do *trait* `Deref` torna possível que ponteiros inteligentes
funcionem de maneiras semelhantes às referências. Em seguida, veremos o recurso
de coerção `deref` (*deref coercion*) do Rust e como ele nos permite trabalhar
com referências ou ponteiros inteligentes.

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### Seguindo a Referência até o Valor

Uma referência comum é um tipo de ponteiro, e uma maneira de pensar sobre um
ponteiro é como uma seta para um valor armazenado em algum lugar. Na Listagem
15-6, criamos uma referência a um valor `i32` e, em seguida, usamos o operador de
desreferenciação para seguir a referência até o valor.

<Listing number="15-6" file-name="src/main.rs" caption="Usando o operador de desreferenciação para seguir uma referência a um valor `i32`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

A variável `x` armazena um valor `i32` `5`. Definimos `y` como igual a uma
referência para `x`. Podemos afirmar que `x` é igual a `5`. No entanto, se
quisermos fazer uma afirmação sobre o valor em `y`, temos que usar `*y` para
seguir a referência até o valor para o qual ela aponta (daí, _desreferenciar_)
para que o compilador possa comparar o valor real. Uma vez que desreferenciamos
`y`, temos acesso ao valor inteiro para o qual `y` aponta e que podemos comparar
com `5`.

Se tentássemos escrever `assert_eq!(5, y);` em vez disso, receberíamos este erro
de compilação:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Comparar um número e uma referência a um número não é permitido porque são tipos
diferentes. Devemos usar o operador de desreferenciação para seguir a referência
até o valor para o qual ela aponta.

### Usando `Box<T>` Como uma Referência

Podemos reescrever o código na Listagem 15-6 para usar um `Box<T>` em vez de uma
referência; o operador de desreferenciação usado no `Box<T>` na Listagem 15-7
funciona da mesma maneira que o operador de desreferenciação usado na
referência na Listagem 15-6.

<Listing number="15-7" file-name="src/main.rs" caption="Usando o operador de desreferenciação em um `Box<i32>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

A principal diferença entre a Listagem 15-7 e a Listagem 15-6 é que aqui nós
definimos `y` como uma instância de uma *box* apontando para uma cópia do valor
de `x`, em vez de uma referência apontando para o valor de `x`. Na última
asserção, podemos usar o operador de desreferenciação para seguir o ponteiro da
*box* da mesma forma que fizemos quando `y` era uma referência. A seguir, vamos
explorar o que há de especial no `Box<T>` que nos permite usar o operador de
desreferenciação definindo nosso próprio tipo de *box*.

### Definindo Nosso Próprio Ponteiro Inteligente

Vamos construir um tipo wrapper semelhante ao tipo `Box<T>` fornecido pela
biblioteca padrão para entender como os tipos de ponteiros inteligentes se
comportam de maneira diferente das referências por padrão. Em seguida, veremos
como adicionar a capacidade de usar o operador de desreferenciação.

> Nota: Há uma grande diferença entre o tipo `MyBox<T>` que estamos prestes a
> construir e o `Box<T>` real: nossa versão não armazenará seus dados no heap.
> Estamos focando este exemplo em `Deref`, então onde os dados são realmente
> armazenados é menos importante do que o comportamento semelhante a um
> ponteiro.

O tipo `Box<T>` é definido em última análise como uma *tuple struct* (estrutura
em tupla) com um elemento, então a Listagem 15-8 define um tipo `MyBox<T>` da
mesma maneira. Também definiremos uma função `new` para corresponder à função
`new` definida em `Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="Definindo um tipo `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Definimos uma estrutura chamada `MyBox` e declaramos um parâmetro genérico `T`
porque queremos que nosso tipo armazene valores de qualquer tipo. O tipo
`MyBox` é uma *tuple struct* com um elemento do tipo `T`. A função
`MyBox::new` aceita um parâmetro do tipo `T` e retorna uma instância de `MyBox`
que armazena o valor passado.

Vamos tentar adicionar a função `main` da Listagem 15-7 à Listagem 15-8 e
alterá-la para usar o tipo `MyBox<T>` que definimos em vez de `Box<T>`. O código
na Listagem 15-9 não compilará, porque o Rust não sabe como desreferenciar
`MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="Tentando usar `MyBox<T>` da mesma forma que usamos referências e `Box<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Aqui está o erro de compilação resultante:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Nosso tipo `MyBox<T>` não pode ser desreferenciado porque não implementamos essa
capacidade em nosso tipo. Para habilitar a desreferenciação com o operador `*`,
implementamos o *trait* `Deref`.

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### Implementando o *Trait* `Deref`

Como discutido em [“Implementando um Trait em um Tipo”][impl-trait]<!-- ignore -->
no Capítulo 10, para implementar um *trait* precisamos fornecer
implementações para os métodos obrigatórios do *trait*. O *trait* `Deref`,
fornecido pela biblioteca padrão, exige que implementemos um método chamado
`deref` que empresta (`borrows`) `self` e retorna uma referência aos dados
internos. A Listagem 15-10 contém uma implementação de `Deref` para adicionar à
definição de `MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="Implementando `Deref` em `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

A sintaxe `type Target = T;` define um tipo associado para o *trait* `Deref`
usar. Tipos associados são uma maneira ligeiramente diferente de declarar um
parâmetro genérico, mas você não precisa se preocupar com eles por enquanto;
vamos cobri-los com mais detalhes no Capítulo 20.

Preenchemos o corpo do método `deref` com `&self.0` para que `deref` retorne uma
referência ao valor que queremos acessar com o operador `*`; lembre-se de
[“Criando Diferentes Tipos com Tuple Structs”][tuple-structs]<!-- ignore --> no
Capítulo 5 que `.0` acessa o primeiro valor em uma estrutura em tupla. A função
`main` na Listagem 15-9 que chama `*` no valor `MyBox<T>` agora compila, e as
asserções passam!

Sem o *trait* `Deref`, o compilador só pode desreferenciar referências `&`. O
método `deref` dá ao compilador a capacidade de pegar um valor de qualquer tipo
que implemente `Deref` e chamar o método `deref` para obter uma referência que
ele sabe como desreferenciar.

Quando digitamos `*y` na Listagem 15-9, por baixo dos panos o Rust realmente
executou este código:

```rust,ignore
*(y.deref())
```

O Rust substitui o operador `*` por uma chamada ao método `deref` e, em seguida,
por uma desreferenciação simples, para que não tenhamos que pensar se precisamos
ou não chamar o método `deref`. Esse recurso do Rust nos permite escrever
código que funciona de forma idêntica, quer tenhamos uma referência comum ou um
tipo que implemente `Deref`.

A razão pela qual o método `deref` retorna uma referência a um valor, e pela
qual a desreferenciação simples fora dos parênteses em `*(y.deref())` ainda é
necessária, tem a ver com o sistema de propriedade (*ownership*). Se o método
`deref` retornasse o valor diretamente em vez de uma referência ao valor, o
valor seria movido (*moved out*) de `self`. Não queremos assumir a propriedade
do valor interno dentro de `MyBox<T>` neste caso ou na maioria dos casos em que
usamos o operador de desreferenciação.

Note que o operador `*` é substituído por uma chamada ao método `deref` e, em
seguida, por uma chamada ao operador `*` apenas uma vez, cada vez que usamos um
`*` em nosso código. Como a substituição do operador `*` não é recursiva de
forma infinita, acabamos com dados do tipo `i32`, o que corresponde ao `5` em
`assert_eq!` na Listagem 15-9.

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### Usando a Coerção Deref em Funções e Métodos

A *Coerção Deref* (*Deref coercion*) converte uma referência a um tipo que
implementa o *trait* `Deref` em uma referência a outro tipo. Por exemplo, a
coerção deref pode converter `&String` em `&str` porque `String` implementa o
*trait* `Deref` de modo que ele retorne `&str`. A coerção deref é uma
comodidade que o Rust realiza em argumentos para funções e métodos, e funciona
apenas em tipos que implementam o *trait* `Deref`. Ela acontece automaticamente
quando passamos uma referência ao valor de um determinado tipo como argumento
para uma função ou método cujo parâmetro possui um tipo diferente na definição.
Uma sequência de chamadas para o método `deref` converte o tipo que fornecemos
no tipo que o parâmetro precisa.

A coerção deref foi adicionada ao Rust para que os programadores que escrevem
chamadas de funções e métodos não precisem adicionar tantas referências e
desreferenciações explícitas com `&` e `*`. O recurso de coerção deref também
nos permite escrever mais código que pode funcionar tanto para referências
quanto para ponteiros inteligentes.

Para ver a coerção deref em ação, vamos usar o tipo `MyBox<T>` que definimos na
Listagem 15-8, bem como a implementação de `Deref` que adicionamos na Listagem
15-10. A Listagem 15-11 mostra a definição de uma função que possui um parâmetro
de fatia de string (*string slice*).

<Listing number="15-11" file-name="src/main.rs" caption="Uma função `hello` que tem o parâmetro `name` do tipo `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Podemos chamar a função `hello` com uma fatia de string como argumento, como
`hello("Rust");`, por exemplo. A coerção deref torna possível chamar `hello`
com uma referência a um valor do tipo `MyBox<String>`, como mostrado na
Listagem 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="Chamando `hello` com uma referência a um valor `MyBox<String>`, o que funciona por causa da coerção deref">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Aqui estamos chamando a função `hello` com o argumento `&m`, que é uma
referência a um valor `MyBox<String>`. Como implementamos o *trait* `Deref` em
`MyBox<T>` na Listagem 15-10, o Rust pode transformar `&MyBox<String>` em
`&String` chamando `deref`. A biblioteca padrão fornece uma implementação de
`Deref` em `String` que retorna uma fatia de string, e isso está na
documentação da API para `Deref`. O Rust chama `deref` novamente para
transformar o `&String` em `&str`, o que corresponde à definição da função
`hello`.

Se o Rust não implementasse a coerção deref, teríamos que escrever o código na
Listagem 15-13 em vez do código na Listagem 15-12 para chamar `hello` com um
valor do tipo `&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="O código que teríamos que escrever se o Rust não tivesse coerção deref">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

O `(*m)` desreferencia o `MyBox<String>` em uma `String`. Em seguida, o `&` e o
`[..]` pegam uma fatia de string da `String` que é igual a toda a string para
corresponder à assinatura de `hello`. Esse código sem coerções deref é mais
difícil de ler, escrever e entender com todos esses símbolos envolvidos. A
coerção deref permite que o Rust lide com essas conversões para nós
automaticamente.

Quando o *trait* `Deref` é definido para os tipos envolvidos, o Rust analisará os
tipos e usará `Deref::deref` quantas vezes forem necessárias para obter uma
referência que corresponda ao tipo do parâmetro. O número de vezes que
`Deref::deref` precisa ser inserido é resolvido em tempo de compilação, então
não há penalidade de tempo de execução (*runtime*) por tirar proveito da coerção
deref!

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### Lidando com a Coerção Deref com Referências Mutáveis

Da mesma forma que você usa o *trait* `Deref` para substituir o operador `*`
em referências imutáveis, você pode usar o *trait* `DerefMut` para substituir o
operador `*` em referências mutáveis.

O Rust faz a coerção deref quando encontra tipos e implementações de *traits*
em três casos:

1. De `&T` para `&U` quando `T: Deref<Target=U>`
2. De `&mut T` para `&mut U` quando `T: DerefMut<Target=U>`
3. De `&mut T` para `&U` quando `T: Deref<Target=U>`

Os dois primeiros casos são iguais, exceto pelo fato de que o segundo implementa
a mutabilidade. O primeiro caso afirma que, se você tem um `&T`, e `T`
implementa `Deref` para algum tipo `U`, você pode obter um `&U` de forma
transparente. O segundo caso afirma que a mesma coerção deref acontece para
referências mutáveis.

O terceiro caso é mais complicado: o Rust também coagirá uma referência mutável
em uma imutável. Mas o inverso *não* é possível: referências imutáveis nunca
serão coagidas em referências mutáveis. Por causa das regras de empréstimo
(*borrowing rules*), se você tem uma referência mutável, essa referência
mutável deve ser a única referência a esses dados (caso contrário, o programa
não compilaria). Converter uma referência mutável em uma referência imutável
nunca quebrará as regras de empréstimo. Converter uma referência imutável em
uma referência mutável exigiria que a referência imutável inicial fosse a única
referência imutável para esses dados, mas as regras de empréstimo não garantem
isso. Portanto, o Rust não pode assumir que converter uma referência imutável
em uma referência mutável é possível.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
