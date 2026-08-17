## Funções

As funções são predominantes no código Rust. Você já viu uma das funções mais
importantes da linguagem: a função `main`, que é o ponto de entrada de muitos
programas. Você também viu a palavra-chave `fn`, que permite declarar novas
funções.

O código Rust usa o _snake case_ como o estilo convencional para nomes de
funções e variáveis, onde todas as letras são minúsculas e os sublinhados (_)
separam as palavras. Aqui está um programa que contém um exemplo de definição de
função:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Definimos uma função em Rust inserindo `fn` seguido por um nome de função e um
conjunto de parênteses. As chaves dizem ao compilador onde o corpo da função
começa e termina.

Podemos chamar qualquer função que definimos inserindo seu nome seguido por um
conjunto de parênteses. Como `another_function` é definida no programa, ela pode
ser chamada de dentro da função `main`. Note que definimos `another_function`
_após_ a função `main` no código-fonte; poderíamos tê-la definido antes também. O
Rust não se importa onde você define suas funções, apenas que elas estejam
definidas em algum escopo que possa ser visto por quem as chama.

Vamos iniciar um novo projeto binário chamado _functions_ para explorar mais as
funções. Coloque o exemplo `another_function` em _src/main.rs_ e execute-o. Você
deve ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

As linhas são executadas na ordem em que aparecem na função `main`. Primeiro, a
mensagem "Hello, world!" é impressa e, em seguida, `another_function` é chamada
e sua mensagem é impressa.

### Parâmetros

Podemos definir funções para terem _parâmetros_, que são variáveis especiais que
fazem parte da assinatura de uma função. Quando uma função tem parâmetros, você
pode fornecer valores concretos para esses parâmetros. Tecnicamente, os valores
concretos são chamados de _argumentos_, mas em conversas informais, as pessoas
tendem a usar as palavras _parâmetro_ e _argumento_ de forma intercambiável, seja
para as variáveis na definição de uma função ou para os valores concretos
passados quando você chama uma função.

Nesta versão de `another_function`, adicionamos um parâmetro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Tente executar este programa; você deve obter a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

A declaração de `another_function` tem um parâmetro chamado `x`. O tipo de `x`
é especificado como `i32`. Quando passamos `5` para `another_function`, a macro
`println!` coloca `5` onde estava o par de chaves contendo `x` na string de
formatação.

Nas assinaturas de funções, você _deve_ declarar o tipo de cada parâmetro. Esta
é uma decisão deliberada no design do Rust: exigir anotações de tipo nas
definições de funções significa que o compilador quase nunca precisa que você
as use em outros lugares do código para descobrir qual tipo você quis dizer. O
compilador também é capaz de fornecer mensagens de erro mais úteis se souber
quais tipos a função espera.

Ao definir vários parâmetros, separe as declarações de parâmetros com vírgulas,
assim:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Este exemplo cria uma função chamada `print_labeled_measurement` com dois
parâmetros. O primeiro parâmetro se chama `value` e é um `i32`. O segundo se
chama `unit_label` e é do tipo `char`. A função então imprime um texto contendo
tanto o `value` quanto o `unit_label`.

Vamos tentar executar este código. Substitua o programa atualmente no arquivo
_src/main.rs_ do seu projeto _functions_ pelo exemplo anterior e execute-o usando
`cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Como chamamos a função com `5` como o valor para `value` e `'h'` como o valor
para `unit_label`, a saída do programa contém esses valores.

### Instruções e Expressões

Os corpos das funções são compostos por uma série de instruções que opcionalmente
terminam com uma expressão. Até agora, as funções que cobrimos não incluíam uma
expressão final, mas você já viu uma expressão como parte de uma instrução.
Como o Rust é uma linguagem baseada em expressões, esta é uma distinção importante
de entender. Outras linguagens não têm as mesmas distinções, então vamos olhar
o que são instruções e expressões e como suas diferenças afetam os corpos das
funções.

- _Instruções_ (Statements) são instruções que executam alguma ação e não
  retornam um valor.
- _Expressões_ (Expressions) são avaliadas resultando em um valor.

Vamos ver alguns exemplos.

Na verdade, já usamos instruções e expressões. Criar uma variável e atribuir um
valor a ela com a palavra-chave `let` é uma instrução. Na Listagem 3-1,
`let y = 6;` é uma instrução.

<Listing number="3-1" file-name="src/main.rs" caption="Uma declaração de função `main` contendo uma instrução">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

As definições de funções também são instruções; todo o exemplo anterior é uma
instrução em si. (Como veremos em breve, chamar uma função não é uma instrução,
no entanto.)

Instruções não retornam valores. Portanto, você não pode atribuir uma instrução
`let` a outra variável, como o código a seguir tenta fazer; você receberá um
erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Quando você executar este programa, o erro que você receberá será parecido com
isto:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

A instrução `let y = 6` não retorna um valor, então não há nada para `x` se
ligar. Isso é diferente do que acontece em outras linguagens, como C e Ruby,
onde a atribuição retorna o valor da atribuição. Nessas linguagens, você pode
escrever `x = y = 6` e fazer com que tanto `x` quanto `y` tenham o valor `6`;
esse não é o caso em Rust.

Expressões são avaliadas como um valor e compõem a maior parte do restante do
código que você escreverá em Rust. Considere uma operação matemática, como
`5 + 6`, que é uma expressão avaliada com o valor `11`. Expressões podem fazer
parte de instruções: na Listagem 3-1, o `6` na instrução `let y = 6;` é uma
expressão avaliada como o valor `6`. Chamar uma função é uma expressão. Chamar
uma macro é uma expressão. Um novo bloco de escopo criado com chaves é uma
expressão, por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Esta expressão:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

é um bloco que, neste caso, é avaliado como `4`. Esse valor é vinculado a `y`
como parte da instrução `let`. Observe a linha `x + 1` sem um ponto e vírgula no
final, o que é diferente da maioria das linhas que você viu até agora. Expressões
não incluem ponto e vírgula no final. Se você adicionar um ponto e vírgula ao
final de uma expressão, você a transforma em uma instrução, e ela deixará de
retornar um valor. Lembre-se disso ao explorar valores de retorno de funções e
expressões a seguir.

### Funções com Valores de Retorno

Funções podem retornar valores para o código que as chama. Nós não damos nomes
aos valores de retorno, mas devemos declarar o tipo deles após uma seta (`->`).
Em Rust, o valor de retorno da função é sinônimo do valor da expressão final no
bloco do corpo de uma função. Você pode retornar antecipadamente de uma função
usando a palavra-chave `return` e especificando um valor, mas a maioria das
funções retorna a última expressão implicitamente. Aqui está um exemplo de uma
função que retorna um valor:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Não há chamadas de função, macros ou mesmo instruções `let` na função `five` —
apenas o número `5` sozinho. Essa é uma função perfeitamente válida em Rust.
Note que o tipo de retorno da função também é especificado como `-> i32`. Tente
executar este código; a saída deve ser parecida com esta:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

O `5` em `five` é o valor de retorno da função, e é por isso que o tipo de
retorno é `i32`. Vamos examinar isso com mais detalhes. Há dois pontos
importantes: Primeiro, a linha `let x = five();` mostra que estamos usando o
valor de retorno de uma função para inicializar uma variável. Como a função
`five` retorna um `5`, essa linha é equivalente à seguinte:

```rust
let x = 5;
```

Segundo, a função `five` não tem parâmetros e define o tipo do valor de retorno,
mas o corpo da função é um solitário `5` sem ponto e vírgula porque é uma
expressão cujo valor queremos retornar.

Vamos ver outro exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Executar este código imprimirá `The value of x is: 6`. Mas o que acontece se
colocarmos um ponto e vírgula no final da linha que contém `x + 1`, mudando-a de
uma expressão para uma instrução?

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

A compilação deste código produzirá um erro, da seguinte forma:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

A mensagem de erro principal, `mismatched types` (tipos incompatíveis), revela o
problema central com este código. A definição da função `plus_one` diz que ela
retornará um `i32`, mas as instruções não são avaliadas como um valor, o que é
expresso por `()`, o tipo unitário. Portanto, nada é retornado, o que contraria
a definição da função e resulta em um erro. Nesta saída, o Rust fornece uma
mensagem para possivelmente ajudar a corrigir esse problema: ele sugere remover o
ponto e vírgula, o que corrigiria o erro.