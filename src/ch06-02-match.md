<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## O Construto de Controle de Fluxo `match`

O Rust possui um construto de controle de fluxo extremamente poderoso chamado
`match` que permite comparar um valor a uma série de padrões e então executar
código com base em qual padrão correspondeu. Os padrões podem ser compostos de
valores literais, nomes de variáveis, curingas e muitas outras coisas; o
[Capítulo 19][ch19-00-patterns]<!-- ignore --> cobre todos os diferentes tipos de
padrões e o que eles fazem. O poder do `match` vem da expressividade dos
padrões e do fato de que o compilador confirma que todos os casos possíveis são
tratados.

Pense em uma expressão `match` como sendo como uma máquina de separar moedas:
As moedas deslizam por uma trilha com buracos de vários tamanhos ao longo dela,
e cada moeda cai pelo primeiro buraco que encontra no qual ela se encaixa. Da
mesma forma, os valores passam por cada padrão em um `match`, e no primeiro
padrão em que o valor "se encaixa", o valor cai no bloco de código associado para
ser usado durante a execução.

Por falar em moedas, vamos usá-las como exemplo usando o `match`! Podemos escrever
uma função que recebe uma moeda desconhecida dos EUA e, de maneira semelhante à
máquina de contagem, determina qual moeda ela é e retorna seu valor em centavos,
conforme mostrado na Listagem 6-3.

<Listing number="6-3" caption="Um enum e uma expressão `match` que tem as variantes do enum como seus padrões">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Vamos analisar o `match` na função `value_in_cents`. Primeiro, listamos a
palavra-chave `match` seguida por uma expressão, que neste caso é o valor
`coin`. Isso parece muito com uma expressão condicional usada com `if`, mas há
uma grande diferença: com o `if`, a condição precisa resultar em um valor
booleano, mas aqui pode ser qualquer tipo. O tipo de `coin` neste exemplo é o
enum `Coin` que definimos na primeira linha.

Em seguida estão os braços (*arms*) do `match`. Um braço tem duas partes: um padrão
e algum código. O primeiro braço aqui tem um padrão que é o valor `Coin::Penny` e
então o operador `=>` que separa o padrão e o código a ser executado. O código
neste caso é apenas o valor `1`. Cada braço é separado do seguinte por uma vírgula.

Quando a expressão `match` é executada, ela compara o valor resultante com o
padrão de cada braço, em ordem. Se um padrão corresponder ao valor, o código
associado a esse padrão será executado. Se esse padrão não corresponder ao
valor, a execução continuará para o próximo braço, muito parecido com uma
máquina de separar moedas. Podemos ter quantos braços precisarmos: na Listagem
6-3, nosso `match` tem quatro braços.

O código associado a cada braço é uma expressão, e o valor resultante da
expressão no braço correspondente é o valor que é retornado para toda a
expressão `match`.

Normalmente não usamos chaves se o código do braço do match for curto, como na
Listagem 6-3, onde cada braço apenas retorna um valor. Se você quiser executar
várias linhas de código em um braço do match, você deve usar chaves, e a vírgula
seguindo o braço torna-se opcional. Por exemplo, o seguinte código imprime
"Lucky penny!" toda vez que o método é chamado com um `Coin::Penny`, mas ainda
assim retorna o último valor do bloco, `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Padrões que se Vinculam a Valores

Outro recurso útil dos braços de match é que eles podem se vincular (*bind*) às
partes dos valores que correspondem ao padrão. É assim que podemos extrair valores
de variantes de enum.

Como exemplo, vamos alterar uma de nossas variantes de enum para conter dados
dentro dela. De 1999 a 2008, os Estados Unidos cunharam moedas de 25 centavos
(*quarters*) com designs diferentes para cada um dos 50 estados em um lado.
Nenhuma outra moeda recebeu designs de estado, então apenas as quarters têm
esse valor extra. Podemos adicionar essas informações ao nosso `enum` alterando
a variante `Quarter` para incluir um valor `UsState` armazenado dentro dela, o
que fizemos na Listagem 6-4.

<Listing number="6-4" caption="Um enum `Coin` no qual a variante `Quarter` também armazena um valor `UsState`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Imaginemos que um amigo esteja tentando coletar todas as 25 quarters de estados.
Enquanto separamos nosso troco por tipo de moeda, também diremos em voz alta o
nome do estado associado a cada quarter para que, se for uma que nosso amigo
não tem, ele possa adicioná-la à sua coleção.

Na expressão match para este código, adicionamos uma variável chamada `state`
ao padrão que corresponde aos valores da variante `Coin::Quarter`. Quando um
`Coin::Quarter` corresponder, a variável `state` se vinculará ao valor do
estado daquela quarter. Então, podemos usar `state` no código para esse braço,
assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Se chamarmos `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` será
`Coin::Quarter(UsState::Alaska)`. Quando comparamos esse valor com cada um dos
braços do match, nenhum deles corresponde até chegarmos a `Coin::Quarter(state)`.
Nesse ponto, a vinculação (*binding*) para `state` será o valor `UsState::Alaska`.
Podemos então usar essa vinculação na expressão `println!`, obtendo assim o valor
interno do estado a partir da variante do enum `Coin` para `Quarter`.

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### O Padrão `match` do `Option<T>`

Na seção anterior, queríamos extrair o valor interno `T` do caso `Some` ao usar
`Option<T>`; também podemos lidar com `Option<T>` usando `match`, como fizemos
com o enum `Coin`! Em vez de comparar moedas, compararemos as variantes de
`Option<T>`, mas a maneira como a expressão `match` funciona permanece a mesma.

Digamos que queremos escrever uma função que aceite um `Option<i32>` e, se
houver um valor dentro, adicione 1 a esse valor. Se não houver um valor dentro,
a função deve retornar o valor `None` e não tentar realizar nenhuma operação.

Esta função é muito fácil de escrever, graças ao `match`, e será parecida com a
Listagem 6-5.

<Listing number="6-5" caption="Uma função que usa uma expressão `match` em um `Option<i32>`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Vamos examinar a primeira execução de `plus_one` mais detalhadamente. Quando
chamamos `plus_one(five)`, a variável `x` no corpo de `plus_one` terá o valor
`Some(5)`. Em seguida, comparamos isso com cada braço do match:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

O valor `Some(5)` não corresponde ao padrão `None`, então continuamos para o
próximo braço:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` corresponde a `Some(i)`? Corresponde! Temos a mesma variante. O `i` se
vincula ao valor contido em `Some`, então `i` assume o valor `5`. O código no
braço do match é então executado, adicionando 1 ao valor de `i` e criando um
novo valor `Some` com nosso total de `6` dentro.

Agora vamos considerar a segunda chamada de `plus_one` na Listagem 6-5, onde `x`
é `None`. Entramos no `match` e comparamos com o primeiro braço:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Ele corresponde! Não há nenhum valor para adicionar, então o programa para e
retorna o valor `None` no lado direito de `=>`. Como o primeiro braço correspondeu,
nenhum outro braço é comparado.

Combinar `match` e enums é útil em muitas situações. Você verá esse padrão
muito frequentemente em código Rust: fazer `match` em um enum, vincular uma
variável aos dados internos e, em seguida, executar código com base nisso. É um
pouco complicado no início, mas assim que você se acostumar, desejará tê-lo em
todas as linguagens. É consistentemente o favorito dos usuários.

### Correspondências são Exaustivas (*Exhaustive*)

Há outro aspecto do `match` que precisamos discutir: os padrões dos braços devem
cobrir todas as possibilidades. Considere esta versão da nossa função
`plus_one`, que tem um bug e não compila:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Não tratamos o caso `None`, então este código causará um erro. Por sorte, é um
erro que o Rust sabe como pegar. Se tentarmos compilar este código, receberemos
este erro:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

O Rust sabe que não cobrimos todos os casos possíveis e até sabe qual padrão
esquecemos! As correspondências no Rust são _exaustivas_: devemos esgotar até a
última possibilidade para que o código seja válido. Especialmente no caso de
`Option<T>`, quando o Rust nos impede de esquecer de tratar explicitamente o
caso `None`, ele nos protege de assumir que temos um valor quando podemos ter um
nulo, tornando assim o erro de um bilhão de dólares discutido anteriormente
impossível.

### Padrões Coringa (*Catch-All*) e o Marcador de Posição `_`

Usando enums, também podemos realizar ações especiais para alguns valores
particulares, mas executar uma ação padrão para todos os outros valores.
Imagine que estamos implementando um jogo onde, se você tirar um 3 em uma
jogada de dados, seu jogador não se move, mas ganha um chapéu novo e chique. Se
você tirar um 7, seu jogador perde um chapéu chique. Para todos os outros
valores, seu jogador se move esse número de casas no tabuleiro do jogo. Aqui
está um `match` que implementa essa lógica, com o resultado da rolagem de dados
fixado em código (*hardcoded*) em vez de ser um valor aleatório, e toda a outra
lógica representada por funções sem corpo, pois implementá-las de fato está fora
do escopo deste exemplo:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Para os dois primeiros braços, os padrões são os valores literais `3` e `7`.
Para o último braço que cobre todos os outros valores possíveis, o padrão é a
variável que escolhemos chamar de `other`. O código que é executado para o braço
`other` usa a variável passando-a para a função `move_player`.

Este código compila, mesmo não tendo listado todos os valores possíveis que um
`u8` pode ter, porque o último padrão corresponderá a todos os valores que não
foram especificamente listados. Este padrão curinga (*catch-all*) atende ao
requisito de que o `match` deve ser exaustivo. Note que temos que colocar o
braço curinga por último porque os padrões são avaliados em ordem. Se tivéssemos
colocado o braço curinga antes, os outros braços nunca seriam executados, então
o Rust nos avisará se adicionarmos braços após um curinga!

O Rust também tem um padrão que podemos usar quando queremos um curinga, mas
não queremos _usar_ o valor no padrão curinga: `_` é um padrão especial que
corresponde a qualquer valor e não se vincula a esse valor. Isso diz ao Rust
que não vamos usar o valor, então o Rust não nos avisará sobre uma variável não
utilizada.

Vamos mudar as regras do jogo: agora, se você tirar qualquer coisa diferente de
3 ou 7, você deve rolar novamente. Não precisamos mais usar o valor do curinga,
então podemos alterar nosso código para usar `_` em vez da variável chamada
`other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Este exemplo também atende ao requisito de exaustividade porque estamos
explicitamente ignorando todos os outros valores no último braço; não esquecemos
de nada.

Finalmente, vamos mudar as regras do jogo mais uma vez para que nada mais
aconteça no seu turno se você tirar qualquer coisa diferente de 3 ou 7. Podemos
expressar isso usando o valor unitário (o tipo de tupla vazia que mencionamos na
seção [“O Tipo Tupla”][tuples]<!-- ignore -->) como o código que acompanha o
braço `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Aqui, estamos dizendo explicitamente ao Rust que não vamos usar nenhum outro
valor que não corresponda a um padrão em um braço anterior, e não queremos
executar nenhum código neste caso.

Há mais sobre padrões e correspondências que abordaremos no [Capítulo
19][ch19-00-patterns]<!-- ignore -->. Por enquanto, vamos passar para a sintaxe
`if let`, que pode ser útil em situações em que a expressão `match` é um pouco
extensa.

[tuples]: ch03-02-data-types.html#o-tipo-tupla
[ch19-00-patterns]: ch19-00-patterns.html
