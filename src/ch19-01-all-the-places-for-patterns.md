## Todos os Lugares onde os Padrões Podem Ser Usados

Os padrões aparecem em vários lugares no Rust, e você os tem usado muito sem
perceber! Esta seção aborda todos os lugares onde os padrões são válidos.

### Braços de `match`

Como discutido no Capítulo 6, usamos padrões nos braços de expressões `match`.
Formalmente, as expressões `match` são definidas pela palavra-chave `match`, um
valor a ser comparado e um ou mais braços de correspondência que consistem em um
padrão e uma expressão a ser executada se o valor corresponder ao padrão daquele
braço, assim:

<!--
  Formatado manualmente em vez de usar Markdown intencionalmente: O Markdown não
  suporta colocar código em itálico no corpo de um bloco como este!
-->

<pre><code>match <em>VALOR</em> {
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
}</code></pre>

Por exemplo, aqui está a expressão `match` da Listagem 6-5 que corresponde a um
valor `Option<i32>` na variável `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Os padrões nesta expressão `match` são o `None` e o `Some(i)` à esquerda de cada
seta.

Um requisito para as expressões `match` é que elas precisam ser exaustivas, no
sentido de que todas as possibilidades para o valor na expressão `match` devem
ser contempladas. Uma maneira de garantir que você cobriu todas as
possibilidades é ter um padrão coringa ("catch-all") para o último braço: por
exemplo, um nome de variável que corresponda a qualquer valor nunca pode falhar
e, portanto, cobre todos os casos restantes.

O padrão específico `_` corresponderá a qualquer coisa, mas nunca se vincula a
uma variável, por isso é frequentemente usado no último braço de
correspondência. O padrão `_` pode ser útil quando você deseja ignorar qualquer
valor não especificado, por exemplo. Abordaremos o padrão `_` com mais detalhes
em [“Ignorando Valores em um Padrão”][ignoring-values-in-a-pattern]<!-- ignore
--> mais adiante neste capítulo.

### Instruções `let`

Antes deste capítulo, havíamos discutido explicitamente apenas o uso de padrões
com `match` e `if let`, mas, de fato, usamos padrões em outros lugares também,
inclusive em instruções `let`. Por exemplo, considere esta atribuição de variável
simples com `let`:

```rust
let x = 5;
```

Toda vez que você usou uma instrução `let` como essa, você estava usando
padrões, embora possa não ter percebido! Mais formalmente, uma instrução `let` é
assim:

<!--
  Formatado manualmente em vez de usar Markdown intencionalmente: O Markdown não
  suporta colocar código em itálico no corpo de um bloco como este!
-->

<pre>
<code>let <em>PADRÃO</em> = <em>EXPRESSÃO</em>;</code>
</pre>

Em instruções como `let x = 5;` com um nome de variável no espaço do PADRÃO, o
nome da variável é apenas uma forma particularmente simples de padrão. O Rust
compara a expressão com o padrão e atribui quaisquer nomes que encontrar.
Portanto, no exemplo `let x = 5;`, `x` é um padrão que significa "vincule o que
corresponder aqui à variável `x`". Como o nome `x` é o padrão inteiro, esse
padrão significa efetivamente "vincule tudo à variável `x`, qualquer que seja o
valor".

Para ver o aspecto de correspondência de padrões do `let` com mais clareza,
considere a Listagem 19-1, que usa um padrão com `let` para desestruturar uma
tupla.


<Listing number="19-1" caption="Usando um padrão para desestruturar uma tupla e criar três variáveis de uma vez">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Aqui, comparamos uma tupla com um padrão. O Rust compara o valor `(1, 2, 3)` ao
padrão `(x, y, z)` e vê que o valor corresponde ao padrão — ou seja, ele vê que o
número de elementos é o mesmo em ambos — então o Rust vincula `1` a `x`, `2` a
`y` e `3` a `z`. Você pode pensar nesse padrão de tupla como aninhando três
padrões de variáveis individuais dentro dele.

Se o número de elementos no padrão não corresponder ao número de elementos na
tupla, o tipo geral não corresponderá e obteremos um erro de compilação. Por
exemplo, a Listagem 19-2 mostra uma tentativa de desestruturar uma tupla com
três elementos em duas variáveis, o que não funcionará.

<Listing number="19-2" caption="Construindo incorretamente um padrão cujas variáveis não correspondem ao número de elementos na tupla">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Tentar compilar este código resulta neste erro de tipo:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Para corrigir o erro, poderíamos ignorar um ou mais valores na tupla usando `_`
ou `..`, como você verá na seção [“Ignorando Valores em um
Padrão”][ignoring-values-in-a-pattern]<!-- ignore -->. Se o problema for que temos
muitas variáveis no padrão, a solução é fazer os tipos coincidirem removendo
variáveis para que o número de variáveis seja igual ao número de elementos na
tupla.

### Expressões Condicionais `if let`

No Capítulo 6, discutimos como usar expressões `if let` principalmente como uma
maneira mais curta de escrever o equivalente a um `match` que corresponde apenas a
um caso. Opcionalmente, o `if let` pode ter um `else` correspondente contendo
código para executar se o padrão no `if let` não corresponder.

A Listagem 19-3 mostra que também é possível misturar e combinar expressões `if
let`, `else if` e `else if let`. Fazer isso nos dá mais flexibilidade do que uma
expressão `match` na qual podemos expressar apenas um valor para comparar com os
padrões. Além disso, o Rust não exige que as condições em uma série de braços
`if let`, `else if` e `else if let` estejam relacionadas entre si.

O código na Listagem 19-3 determina qual cor dar ao seu fundo com base em uma
série de verificações para várias condições. Para este exemplo, criamos
variáveis com valores codificados (`hardcoded`) que um programa real pode
receber da entrada do usuário.

<Listing number="19-3" file-name="src/main.rs" caption="Misturando `if let`, `else if`, `else if let` e `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Se o usuário especificar uma cor favorita, essa cor será usada como fundo. Se
nenhuma cor favorita for especificada e hoje for terça-feira, a cor de fundo será
verde. Caso contrário, se o usuário especificar sua idade como uma string e
pudermos analisá-la com sucesso como um número, a cor será roxa ou laranja,
dependendo do valor do número. Se nenhuma dessas condições se aplicar, a cor de
fundo será azul.

Essa estrutura condicional nos permite suportar requisitos complexos. Com os
valores codificados que temos aqui, este exemplo imprimirá `Using purple as the
background color` (Usando roxo como a cor de fundo).

Você pode ver que o `if let` também pode introduzir novas variáveis que ocultam
variáveis existentes da mesma forma que os braços de `match` podem: a linha `if
let Ok(age) = age` introduz uma nova variável `age` que contém o valor dentro da
variante `Ok`, ocultando a variável `age` existente. Isso significa que precisamos
colocar a condição `if age > 30` dentro desse bloco: não podemos combinar essas
duas condições em `if let Ok(age) = age && age > 30`. A nova `age` que queremos
comparar com 30 não é válida até que o novo escopo comece com a chave.

A desvantagem de usar expressões `if let` é que o compilar não verifica a
exaustividade, ao contrário das expressões `match`. Se omitíssemos o último bloco
`else` e, portanto, deixássemos de tratar alguns casos, o compilador não nos
alertaria sobre o possível bug de lógica.

### Loops Condicionais `while let`

Semelhante em construção ao `if let`, o loop condicional `while let` permite que
um loop `while` seja executado enquanto um padrão continuar a corresponder. Na
Listagem 19-4, mostramos um loop `while let` que aguarda mensagens enviadas entre
threads, mas neste caso verificando um `Result` em vez de um `Option`.

<Listing number="19-4" caption="Usando um loop `while let` para imprimir valores enquanto `rx.recv()` retornar `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Este exemplo imprime `1`, `2` e depois `3`. O método `recv` pega a primeira
mensagem do lado receptor do canal e retorna um `Ok(value)`. Quando vimos `recv`
pela primeira vez no Capítulo 16, tratamos o erro diretamente (`unwrapped`) ou
interagimos com ele como um iterador usando um loop `for`. Como a Listagem 19-4
mostra, no entanto, também podemos usar `while let`, porque o método `recv`
retorna um `Ok` cada vez que uma mensagem chega, desde que o remetente exista, e
então produz um `Err` assim que o lado remetente é desconectado.

### Loops `for`

Em um loop `for`, o valor que segue imediatamente a palavra-chave `for` é um
padrão. Por exemplo, em `for x in y`, o `x` é o padrão. A Listagem 19-5 demonstra
como usar um padrão em um loop `for` para desestruturar, ou separar, uma tupla
como parte do loop `for`.


<Listing number="19-5" caption="Usando um padrão em um loop `for` para desestruturar uma tupla">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

O código na Listagem 19-5 imprimirá o seguinte:


```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Adaptamos um iterador usando o método `enumerate` para que ele produza um valor e
o índice para esse valor, colocados em uma tupla. O primeiro valor produzido é a
tupla `(0, 'a')`. Quando esse valor é comparado ao padrão `(index, value)`,
index será `0` e value será `'a'`, imprimindo a primeira linha da saída.


### Parâmetros de Função

Os parâmetros de função também podem ser padrões. O código na Listagem 19-6, que
declara uma função chamada `foo` que aceita um parâmetro chamado `x` do tipo
`i32`, já deve parecer familiar.

<Listing number="19-6" caption="Uma assinatura de função usando padrões nos parâmetros">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

A parte `x` é um padrão! Como fizemos com `let`, poderíamos corresponder a uma
tupla nos argumentos de uma função com o padrão. A Listagem 19-7 divide os valores
em uma tupla à medida que a passamos para uma função.

<Listing number="19-7" file-name="src/main.rs" caption="Uma função com parâmetros que desestruturam uma tupla">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Este código imprime `Current location: (3, 5)`. Os valores `&(3, 5)` correspondem
ao padrão `&(x, y)`, então `x` é o valor `3` e `y` é o valor `5`.

Também podemos usar padrões em listas de parâmetros de closures da mesma maneira
que em listas de parâmetros de funções, porque closures são semelhantes a
funções, conforme discutido no Capítulo 13.

Neste ponto, você viu várias maneiras de usar padrões, mas os padrões não
funcionam da mesma forma em todos os lugares onde podemos usá-los. Em alguns
lugares, os padrões devem ser irrefutáveis; em outras circunstâncias, eles podem
ser refutáveis. Discutiremos esses dois conceitos a seguir.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
