## Fluxo de Controle

A capacidade de executar código dependendo se uma condição é verdadeira (`true`)
e a capacidade de executar código repetidamente enquanto uma condição é
verdadeira (`true`) são blocos fundamentais de construção na maioria das
linguagens de programação. As construções mais comuns que permitem controlar o
fluxo de execução do código Rust são expressões `if` e loops.

### Expressões `if`

Uma expressão `if` permite ramificar seu código dependendo de condições. Você
fornece uma condição e declara: "Se esta condição for atendida, execute este
bloco de código. Se a condição não for atendida, não execute este bloco de
código."

Crie um novo projeto chamado _branches_ no seu diretório _projects_ para explorar
a expressão `if`. No arquivo _src/main.rs_, insira o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Todas as expressões `if` começam com a palavra-chave `if`, seguida por uma
condição. Neste caso, a condição verifica se a variável `number` tem um valor
menor que 5. Colocamos o bloco de código a ser executado se a condição for
verdadeira (`true`) imediatamente após a condição, dentro de chaves. Os blocos de
código associados às condições nas expressões `if` são às vezes chamados de
_braços_ (_arms_), assim como os braços nas expressões `match` que discutimos na
seção [“Comparando o palpite com o número secreto”][comparing-the-guess-to-the-secret-number]<!--
ignore --> do Capítulo 2.

Opcionalmente, também podemos incluir uma expressão `else`, o que escolhemos fazer
aqui, para dar ao programa um bloco alternativo de código a ser executado caso a
condição resulte em falso (`false`). Se você não fornecer uma expressão `else` e
a condição for falsa (`false`), o programa simplesmente pulará o bloco `if` e
seguirá para o próximo trecho de código.

Tente executar este código; você deverá ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Vamos tentar alterar o valor de `number` para um valor que torne a condição falsa
(`false`) para ver o que acontece:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Execute o programa novamente e observe a saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Vale a pena notar também que a condição neste código _deve_ ser um `bool`. Se a
condição não for um `bool`, teremos um erro. Por exemplo, tente executar o
seguinte código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

A condição do `if` é avaliada para um valor `3` desta vez, e o Rust gera um
erro:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

O erro indica que o Rust esperava um `bool`, mas obteve um inteiro. Ao contrário
de linguagens como Ruby e JavaScript, o Rust não tentará converter
automaticamente tipos não-booleanos em um booleano. Você deve ser explícito e
sempre fornecer um booleano como condição do `if`. Se quisermos que o bloco de
código do `if` seja executado apenas quando um número for diferente de `0`, por
exemplo, podemos alterar a expressão `if` para a seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

A execução deste código imprimirá `number was something other than zero` (o número era diferente de zero).

#### Tratando Múltiplas Condições com `else if`

Você pode usar múltiplas condições combinando `if` e `else` em uma expressão
`else if`. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Este programa tem quatro caminhos possíveis que pode seguir. Após executá-lo,
você deverá ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Quando este programa é executado, ele verifica cada expressão `if` por vez e
executa o primeiro corpo para o qual a condição é avaliada como verdadeira
(`true`). Note que, embora 6 seja divisível por 2, não vemos a saída `number is
divisible by 2`, nem vemos o texto `number is not divisible by 4, 3, or 2` do
bloco `else`. Isso ocorre porque o Rust executa apenas o bloco para a primeira
condição verdadeira (`true`) e, assim que encontra uma, nem verifica o restante.

Usar muitas expressões `else if` pode poluir o seu código, portanto, se você tiver
mais do que uma, talvez queira refatorar seu código. O Capítulo 6 descreve uma
poderosa construção de ramificação do Rust chamada `match` para esses casos.

#### Usando `if` em uma Instrução `let`

Como o `if` é uma expressão, podemos usá-lo no lado direito de uma instrução
`let` para atribuir o resultado a uma variável, como na Listagem 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Atribuindo o resultado de uma expressão `if` a uma variável">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

A variável `number` será vinculada a um valor com base no resultado da
expressão `if`. Execute este código para ver o que acontece:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Lembre-se de que os blocos de código são avaliados como a última expressão neles,
e os números por si só também são expressões. Nesse caso, o valor de toda a
expressão `if` depende de qual bloco de código é executado. Isso significa que os
valores que têm o potencial de ser resultados de cada braço do `if` devem ser do
mesmo tipo; na Listagem 3-2, os resultados do braço do `if` e do braço do `else`
foram inteiros `i32`. Se os tipos não corresponderem, como no exemplo a seguir,
teremos um erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Quando tentamos compilar este código, recebemos um erro. Os braços `if` e `else`
têm tipos de valores incompatíveis, e o Rust indica exatamente onde encontrar o
problema no programa:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

A expressão no bloco `if` é avaliada como um inteiro, e a expressão no bloco
`else` é avaliada como uma string. Isso não vai funcionar, porque as variáveis
devem ter um único tipo, e o Rust precisa saber definitivamente em tempo de
compilação qual é o tipo da variável `number`. Conhecer o tipo de `number`
permite que o compilador verifique se o tipo é válido em todos os lugares onde
usamos `number`. O Rust não seria capaz de fazer isso se o tipo de `number` fosse
determinado apenas em tempo de execução; o compilador seria mais complexo e faria
menos garantias sobre o código se precisasse acompanhar vários tipos hipotéticos
para qualquer variável.

### Repetição com Loops

Muitas vezes é útil executar um bloco de código mais de uma vez. Para esta
tarefa, o Rust fornece vários _loops_, que executam o código dentro do corpo do
loop até o final e, em seguida, recomeçam imediatamente no início. Para
experimentar loops, vamos criar um novo projeto chamado _loops_.

O Rust tem três tipos de loops: `loop`, `while` e `for`. Vamos testar cada um deles.

#### Repetindo Código com `loop`

A palavra-chave `loop` diz ao Rust para executar um bloco de código repetidamente,
seja para sempre ou até que você diga explicitamente para parar.

Como exemplo, altere o arquivo _src/main.rs_ no seu diretório _loops_ para ficar
assim:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Quando executamos este programa, veremos `again!` impresso repetidamente e de
forma contínua até que paremos o programa manualmente. A maioria dos terminais
suporta o atalho de teclado <kbd>ctrl</kbd>-<kbd>C</kbd> para interromper um
programa preso em um loop contínuo. Experimente:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

O símbolo `^C` representa onde você pressionou <kbd>ctrl</kbd>-<kbd>C</kbd>.

Você pode ou não ver a palavra `again!` impressada após o `^C`, dependendo de
onde o código estava no loop quando recebeu o sinal de interrupção.

Felizmente, o Rust também fornece uma maneira de sair de um loop usando código.
Você pode colocar a palavra-chave `break` dentro do loop para dizer ao programa
quando parar de executar o loop. Lembre-se de que fizemos isso no jogo de adivinhação
na seção [“Saindo Após um Palpite Correto”][quitting-after-a-correct-guess]<!-- ignore
--> do Capítulo 2 para sair do programa quando o usuário ganhou o jogo adivinhando o número correto.

Também usamos `continue` no jogo de adivinhação, que em um loop diz ao programa
para pular qualquer código restante nesta iteração do loop e ir para a próxima iteração.

#### Retornando Valores de Loops

Um dos usos de um `loop` é repetir uma operação que você sabe que pode falhar,
como verificar se uma thread concluiu seu trabalho. Você também pode precisar
passar o resultado dessa operação para fora do loop para o resto do seu código.
Para fazer isso, você pode adicionar o valor que deseja retornar após a
expressão `break` que você usa para parar o loop; esse valor será retornado para
fora do loop para que você possa usá-lo, como mostrado aqui:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Antes do loop, declaramos uma variável chamada `counter` e a inicializamos com
`0`. Em seguida, declaramos uma variável chamada `result` para armazenar o valor
retornado do loop. Em cada iteração do loop, adicionamos `1` à variável
`counter` e, em seguida, verificamos se o `counter` é igual a `10`. Quando for,
usamos a palavra-chave `break` com o valor `counter * 2`. Após o loop, usamos um
ponto e vírgula para encerrar a instrução que atribui o valor a `result`. Por
fim, imprimimos o valor em `result`, que neste caso é `20`.

Você também pode usar `return` de dentro de um loop. Enquanto o `break` sai
apenas do loop atual, o `return` sempre sai da função atual.

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### Eliminando Ambiguidade com Rótulos de Loop

Se você tiver loops dentro de loops, `break` e `continue` se aplicam ao loop mais
interno naquele ponto. Opcionalmente, você pode especificar um _rótulo de loop_
(_loop label_) em um loop que pode ser usado com `break` ou `continue` para
especificar que essas palavras-chave se aplicam ao loop rotulado em vez do loop
mais interno. Os rótulos de loop devem começar com uma aspa simples. Aqui está um
exemplo com dois loops aninhados:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

O loop externo tem o rótulo `'counting_up`, e ele fará a contagem de 0 a 2. O
loop interno sem rótulo faz a contagem regressiva de 10 a 9. O primeiro `break`
que não especifica um rótulo sairá apenas do loop interno. A instrução `break
'counting_up;` sairá do loop externo. Este código imprime:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### Simplificando Loops Condicionais com while

Um programa frequentemente precisará avaliar uma condição dentro de um loop.
Enquanto a condição for verdadeira (`true`), o loop será executado. Quando a
condição deixar de ser verdadeira (`true`), o programa chama `break`, parando o
loop. É possível implementar um comportamento como este usando uma combinação de
`loop`, `if`, `else` e `break`; você poderia tentar isso agora em um programa, se
quiser. No entanto, esse padrão é tão comum que o Rust tem uma construção de
linguagem integrada para ele, chamada de loop `while`. Na Listagem 3-3, usamos
`while` para fazer o programa executar o loop três vezes, contando para trás a
cada vez e, depois do loop, imprimindo uma mensagem e saindo.

<Listing number="3-3" file-name="src/main.rs" caption="Usando um loop `while` para executar código enquanto uma condição é avaliada como verdadeira (`true`)">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Essa construção elimina muito do aninhamento que seria necessário se você usasse
`loop`, `if`, `else` e `break`, e é mais clara. Enquanto uma condição for
avaliada como verdadeira (`true`), o código é executado; caso contrário, ele sai
do loop.

#### Percorrendo uma Coleção com `for`

Você pode optar por usar a construção `while` para percorrer os elementos de
uma coleção, como uma matriz (array). Por exemplo, o loop na Listagem 3-4 imprime
cada elemento na matriz `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Percorrendo cada elemento de uma coleção usando um loop `while`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Aqui, o código conta através dos elementos na matriz. Ele começa no índice `0` e
faz um loop até atingir o índice final na matriz (ou seja, quando `index < 5`
deixa de ser verdadeiro (`true`)). A execução deste código imprimirá cada
elemento na matriz:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Todos os cinco valores da matriz aparecem no terminal, conforme o esperado.
Mesmo que `index` atinja o valor `5` em algum momento, o loop para de ser
executado antes de tentar buscar um sexto valor da matriz.

No entanto, essa abordagem é propensa a erros; poderíamos fazer o programa entrar
em pânico (*panic*) se o valor do índice ou a condição de teste estiver incorreta. Por
exemplo, se você alterasse a definição da matriz `a` para ter quatro elementos,
mas esquecesse de atualizar a condição para `while index < 4`, o código entraria
em pânico. Também é lento, porque o compilador adiciona código em tempo de
execução para realizar a verificação condicional de se o índice está dentro dos
limites da matriz em cada iteração do loop.

Como uma alternativa mais concisa, você pode usar um loop `for` e executar algum
código para cada item em uma coleção. Um loop `for` se parece com o código da
Listagem 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Percorrendo cada elemento de uma coleção usando um loop `for`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Quando executamos este código, veremos a mesma saída da Listagem 3-4. Mais
importantemente, agora aumentamos a segurança do código e eliminamos a chance de
bugs que poderiam resultar de ir além do final da matriz ou não ir longe o
suficiente e perder alguns itens. O código de máquina gerado a partir de loops
`for` também pode ser mais eficiente porque o índice não precisa ser comparado
com o comprimento da matriz a cada iteração.

Usando o loop `for`, você não precisaria se lembrar de alterar nenhum outro
código se mudasse o número de valores na matriz, como faria com o método usado na
Listagem 3-4.

A segurança e a concisão dos loops `for` os tornam a construção de loop mais
comumente usada em Rust. Mesmo em situações em que você deseja executar algum
código um determinado número de vezes, como no exemplo de contagem regressiva
que usou um loop `while` na Listagem 3-3, a maioria dos *Rustacean*s usaria um
loop `for`. A maneira de fazer isso seria usar um `Range` (intervalo),
fornecido pela biblioteca padrão, que gera todos os números em sequência
começando de um número e terminando antes de outro número.

Aqui está como a contagem regressiva seria usando um loop `for` e outro método
sobre o qual ainda não falamos, `rev`, para inverter o intervalo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Este código é um pouco mais agradável, não é?

## Resumo

Você conseguiu! Este foi um capítulo considerável: você aprendeu sobre variáveis,
tipos de dados escalares e compostos, funções, comentários, expressões `if` e
loops! Para praticar com os conceitos discutidos neste capítulo, tente criar
programas para fazer o seguinte:

- Converter temperaturas entre Fahrenheit e Celsius.
- Gerar o *n*-ésimo número de Fibonacci.
- Imprimir a letra da música natalina “The Twelve Days of Christmas” (*Os Doze Dias de Natal*), aproveitando a repetição na música.

Quando estiver pronto para avançar, falaremos sobre um conceito em Rust que
_não_ existe comumente em outras linguagens de programação: a propriedade (*ownership*).

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
