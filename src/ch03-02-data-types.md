## Tipos de Dados

Cada valor em Rust é de um determinado _tipo de dado_, o que diz ao Rust que tipo
de dado está sendo especificado para que ele saiba como trabalhar com esse dado.
Veremos dois subconjuntos de tipos de dados: escalares e compostos.

Lembre-se de que Rust é uma linguagem de _tipagem estática_, o que significa que
ela deve saber os tipos de todas as variáveis em tempo de compilação. O
compilador geralmente pode deduzir qual tipo queremos usar com base no valor e em
como o usamos. Nos casos em que muitos tipos são possíveis, como quando
convertemos uma `String` em um tipo numérico usando `parse` na seção [“Comparando
o palpite com o número secreto”][comparing-the-guess-to-the-secret-number]<!--
ignore --> do Capítulo 2, devemos adicionar uma anotação de tipo, assim:

```rust
let guess: u32 = "42".parse().expect("Não é um número!");
```

Se não adicionarmos a anotação de tipo `: u32` mostrada no código anterior, o
Rust exibirá o seguinte erro, o que significa que o compilador precisa de mais
informações nossas para saber qual tipo queremos usar:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Você verá diferentes anotações de tipo para outros tipos de dados.

### Tipos Escalares

Um tipo _escalar_ representa um único valor. O Rust tem quatro tipos escalares
principais: inteiros, números de ponto flutuante, booleanos e caracteres. Você
pode reconhecê-los de outras linguagens de programação. Vamos ver como eles
funcionam em Rust.

#### Tipos Inteiros

Um _inteiro_ é um número sem componente fracionário. Usamos um tipo inteiro no
Capítulo 2, o tipo `u32`. Esta declaração de tipo indica que o valor a ele
associado deve ser um inteiro sem sinal (tipos inteiros com sinal começam com `i`
em vez de `u`) que ocupa 32 bits de espaço. A Tabela 3-1 mostra os tipos inteiros
embutidos no Rust. Podemos usar qualquer uma dessas variantes para declarar o
tipo de um valor inteiro.

<span class="caption">Tabela 3-1: Tipos inteiros em Rust</span>

| Comprimento | Com sinal | Sem sinal |
| ----------- | --------- | --------- |
| 8-bit       | `i8`      | `u8`      |
| 16-bit      | `i16`     | `u16`     |
| 32-bit      | `i32`     | `u32`     |
| 64-bit      | `i64`     | `u64`     |
| 128-bit     | `i128`    | `u128`    |
| Dependente da arquitetura | `isize` | `usize` |

Cada variante pode ter sinal ou não e tem um tamanho explícito. _Com sinal_ e
_sem sinal_ referem-se à possibilidade de o número ser negativo — em outras
palavras, se o número precisa ter um sinal junto a ele (com sinal) ou se será
sempre positivo e, portanto, pode ser representado sem um sinal (sem sinal). É
como escrever números no papel: quando o sinal importa, um número é mostrado com
um sinal de mais ou um sinal de menos; no entanto, quando é seguro assumir que o
número é positivo, ele é mostrado sem sinal. Os números com sinal são
armazenados usando a representação de [complemento de
dois][twos-complement]<!-- ignore -->.

Cada variante com sinal pode armazenar números de −(2<sup>n − 1</sup>) até
2<sup>n − 1</sup> − 1 inclusives, onde _n_ é o número de bits que essa variante
usa. Portanto, um `i8` pode armazenar números de −(2<sup>7</sup>) a 2<sup>7</sup>
− 1, o que equivale a −128 a 127. Variantes sem sinal podem armazenar números de
0 a 2<sup>n</sup> − 1, portanto, um `u8` pode armazenar números de 0 a 2<sup>8</sup>
− 1, o que equivale a 0 a 255.

Além disso, os tipos `isize` e `usize` dependem da arquitetura do computador em
que seu programa está rodando: 64 bits se você estiver em uma arquitetura de 64
bits e 32 bits se estiver em uma arquitetura de 32 bits.

Você pode escrever literais inteiros em qualquer uma das formas mostradas na
Tabela 3-2. Note que literais numéricos que podem ser de vários tipos numéricos
permitem um sufixo de tipo, como `57u8`, para designar o tipo. Literais
numéricos também podem usar `_` como um separador visual para tornar o número
mais fácil de ler, como `1_000`, que terá o mesmo valor de se você tivesse
especificado `1000`.

<span class="caption">Tabela 3-2: Literais inteiros em Rust</span>

| Literais numéricos | Exemplo       |
| ------------------ | ------------- |
| Decimal            | `98_222`      |
| Hexadecimal        | `0xff`        |
| Octal              | `0o77`        |
| Binário            | `0b1111_0000` |
| Byte (apenas `u8`) | `b'A'`        |

Então, como você sabe qual tipo de inteiro usar? Se você não tiver certeza, os
padrões do Rust geralmente são bons pontos de partida: os tipos inteiros têm
como padrão `i32`. A principal situação em que você usaria `isize` ou `usize` é
ao indexar algum tipo de coleção.

> ##### Estouro de Inteiro (Integer Overflow)
>
> Digamos que você tenha uma variável do tipo `u8` que pode conter valores entre
> 0 e 255. Se você tentar alterar a variável para um valor fora desse intervalo,
> como 256, ocorrerá um _estouro de inteiro_ (integer overflow), o que pode
> resultar em um de dois comportamentos. Quando você está compilando no modo de
> depuração (debug), o Rust inclui verificações de estouro de inteiro que fazem
> seu programa entrar em _pânico_ (panic) em tempo de execução se esse
> comportamento ocorrer. O Rust usa o termo _entrar em pânico_ quando um programa
> é encerrado com um erro; discutiremos pânicos mais a fundo na seção [“Erros
> Irrecuperáveis com `panic!`”][unrecoverable-errors-with-panic]<!-- ignore -->
> no Capítulo 9.
>
> Quando você está compilando no modo de lançamento (release) com a flag
> `--release`, o Rust _não_ inclui verificações de estouro de inteiro que causam
> pânicos. Em vez disso, se ocorrer estouro, o Rust realiza o _empacotamento por
> complemento de dois_ (two’s complement wrapping). Em suma, valores maiores que o
> valor máximo que o tipo pode conter “dão a volta” para o mínimo dos valores
> que o tipo pode conter. No caso de um `u8`, o valor 256 vira 0, o valor 257 vira
> 1, e assim por diante. O programa não entrará em pânico, mas a variável terá um
> valor que provavelmente não é o que você esperava que ela tivesse. Confiar no
> comportamento de empacotamento do estouro de inteiro é considerado um erro.
>
> Para lidar explicitamente com a possibilidade de estouro, você pode usar estas
> famílias de métodos fornecidas pela biblioteca padrão para tipos numéricos
> primitivos:
>
> - Envolver (wrap) em todos os modos de compilação com os métodos
>   `wrapping_*`, como `wrapping_add`.
> - Retornar o valor `None` se houver estouro com os métodos `checked_*`.
> - Retornar o valor e um booleano indicando se houve estouro com os métodos
>   `overflowing_*`.
> - Saturar nos valores mínimo ou máximo do tipo com os métodos `saturating_*`.

#### Tipos de Ponto Flutuante

O Rust também tem dois tipos primitivos para _números de ponto flutuante_, que
são números com ponto decimal. Os tipos de ponto flutuante do Rust são `f32` e
`f64`, que têm 32 bits e 64 bits de tamanho, respectivamente. O tipo padrão é
`f64` porque em CPUs modernas ele tem aproximadamente a mesma velocidade que o
`f32`, mas é capaz de maior precisão. Todos os tipos de ponto flutuante têm sinal.

Aqui está um exemplo que mostra números de ponto flutuante em ação:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Os números de ponto flutuante são representados de acordo com o padrão IEEE-754.

#### Operações Numéricas

O Rust suporta as operações matemáticas básicas que você esperaria para todos os
tipos numéricos: adição, subtração, multiplicação, divisão e resto. A divisão de
inteiros é truncada em direção a zero para o inteiro mais próximo. O código
seguinte mostra como você usaria cada operação numérica em uma instrução `let`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Cada expressão nessas instruções usa um operador matemático e é avaliada como um
único valor, que é então vinculado a uma variável. O [Apêndice
B][appendix_b]<!-- ignore --> contém uma lista de todos os operadores que o
Rust fornece.

#### O Tipo Booleano

Como na maioria das outras linguagens de programação, um tipo booleano em Rust
tem dois valores possíveis: `true` (verdadeiro) e `false` (falso). Os booleanos
têm o tamanho de um byte. O tipo booleano em Rust é especificado usando `bool`.
Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

A principal forma de usar valores booleanos é através de condicionais, como uma
expressão `if`. Abordaremos como as expressões `if` funcionam em Rust na seção
[“Fluxo de Controle”][control-flow]<!-- ignore -->.

#### O Tipo Caractere

O tipo `char` do Rust é o tipo alfabético mais primitivo da linguagem. Aqui
estão alguns exemplos de declaração de valores `char`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Note que especificamos literais `char` com aspas simples, ao contrário de
literais de string, que usam aspas duplas. O tipo `char` do Rust tem 4 bytes de
tamanho e representa um valor escalar Unicode, o que significa que ele pode
representar muito mais do que apenas ASCII. Letras acentuadas; caracteres
chineses, japoneses e coreanos; emojis; e espaços de largura zero são todos
valores `char` válidos em Rust. Os valores escalares Unicode vão de `U+0000` a
`U+D7FF` e de `U+E000` a `U+10FFFF` inclusives. No entanto, um “caractere” não é
realmente um conceito em Unicode, então sua intuição humana sobre o que é um
“caractere” pode não corresponder ao que é um `char` em Rust. Discutiremos este
tópico em detalhes em [“Armazenando texto codificado em UTF-8 com
Strings”][strings]<!-- ignore --> no Capítulo 8.

### Tipos Compostos

Os _tipos compostos_ podem agrupar vários valores em um único tipo. O Rust tem
dois tipos compostos primitivos: tuplas e matrizes (arrays).

#### O Tipo Tupla

Uma _tupla_ é uma maneira geral de agrupar vários valores com uma variedade de
tipos em um único tipo composto. As tuplas têm um comprimento fixo: uma vez
declaradas, elas não podem crescer ou diminuir de tamanho.

Criamos uma tupla escrevendo uma lista de valores separados por vírgula dentro
de parênteses. Cada posição na tupla tem um tipo, e os tipos dos diferentes
valores na tupla não precisam ser os mesmos. Adicionamos anotações de tipo
opcionais neste exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

A variável `tup` está vinculada a toda a tupla porque uma tupla é considerada um
único elemento composto. Para extrair os valores individuais de uma tupla,
podemos usar correspondência de padrões (pattern matching) para desestruturar um
valor de tupla, assim:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Este programa primeiro cria uma tupla e a vincula à variável `tup`. Em seguida,
ele usa um padrão com `let` para pegar `tup` e transformá-lo em três variáveis
separadas, `x`, `y` e `z`. Isso é chamado de _desestruturação_ (destructuring)
porque divide a tupla única em três partes. Por fim, o programa imprime o valor
de `y`, que é `6.4`.

Também podemos acessar um elemento da tupla diretamente usando um ponto (`.`)
seguido pelo índice do valor que queremos acessar. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Este programa cria a tupla `x` e então acessa cada elemento da tupla usando seus
respectivos índices. Como na maioria das linguagens de programação, o primeiro
índice em uma tupla é 0.

A tupla sem nenhum valor tem um nome especial, _unit_ (unidade). Esse valor e
seu tipo correspondente são ambos escritos como `()` e representam um valor vazio
ou um tipo de retorno vazio. As expressões retornam implicitamente o valor unit
se não retornarem nenhum outro valor.

#### O Tipo Matriz (Array)

Outra maneira de ter uma coleção de vários valores é com uma _matriz_ (array).
Ao contrário de uma tupla, cada elemento de uma matriz deve ter o mesmo tipo.
Ao contrário das matrizes em algumas outras linguagens, as matrizes em Rust têm
um comprimento fixo.

Escrevemos os valores em uma matriz como uma lista separada por vírgulas dentro
de colchetes:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

As matrizes são úteis quando você deseja que seus dados sejam alocados na pilha
(stack), da mesma forma que os outros tipos que vimos até agora, em vez de na
memória heap (discutiremos pilha e heap mais a fundo no [Capítulo
4][stack-and-heap]<!-- ignore -->), ou quando você deseja garantir que sempre
terá um número fixo de elementos. Uma matriz não é tão flexível quanto o tipo
vetor (vector), no entanto. Um vetor é um tipo de coleção semelhante fornecido
pela biblioteca padrão que _pode_ crescer ou diminuir de tamanho porque seu
conteúdo vive na heap. Se você não tiver certeza se deve usar uma matriz ou um
vetor, é provável que deva usar um vetor. O [Capítulo 8][vectors]<!-- ignore -->
discute vetores em mais detalhes.

No entanto, as matrizes são mais úteis quando você sabe que o número de
elementos não precisará mudar. Por exemplo, se você estivesse usando os nomes dos
meses em um programa, provavelmente usaria uma matriz em vez de um vetor porque
sabe que ela sempre conterá 12 elementos:

```rust
let months = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho",
              "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];
```

Você escreve o tipo de uma matriz usando colchetes com o tipo de cada elemento,
um ponto e vírgula e, em seguida, o número de elementos na matriz, assim:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Aqui, `i32` é o tipo de cada elemento. Após o ponto e vírgula, o número `5`
indica que a matriz contém cinco elementos.

Você também pode inicializar uma matriz para conter o mesmo valor para cada
elemento especificando o valor inicial, seguido por um ponto e vírgula e, em
seguida, o comprimento da matriz entre colchetes, conforme mostrado aqui:

```rust
let a = [3; 5];
```

A matriz chamada `a` conterá `5` elementos que serão todos definidos com o valor
`3` inicialmente. Isso é o mesmo que escrever `let a = [3, 3, 3, 3, 3];`, mas de
uma forma mais concisa.

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### Acesso a Elementos da Matriz

Uma matriz é um único bloco de memória de um tamanho conhecido e fixo que pode
ser alocado na pilha. Você pode acessar elementos de uma matriz usando
indexação, assim:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Neste exemplo, a variável chamada `first` obterá o valor `1` porque esse é o
valor no índice `[0]` na matriz. A variável chamada `second` obterá o valor `2`
do índice `[1]` na matriz.

#### Acesso Inválido a Elementos da Matriz

Vamos ver o que acontece se você tentar acessar um elemento de uma matriz que
está além do fim da matriz. Digamos que você execute este código, semelhante ao
jogo de adivinhação do Capítulo 2, para obter um índice de matriz do usuário:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Este código compila com sucesso. Se você executar este código usando `cargo run`
e digitar `0`, `1`, `2`, `3` ou `4`, o programa imprimirá o valor correspondente
naquele índice da matriz. Se você digitar um número além do fim da matriz, como
`10`, verá uma saída como esta:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

O programa resultou em um erro em tempo de execução no momento do uso de um
valor inválido na operação de indexação. O programa foi encerrado com uma
mensagem de erro e não executou a instrução `println!` final. Quando você tenta
acessar um elemento usando indexação, o Rust verifica se o índice que você
especificou é menor que o comprimento da matriz. Se o índice for maior ou igual
ao comprimento, o Rust entrará em pânico. Essa verificação precisa acontecer em
tempo de execução, especialmente neste caso, porque o compilador não tem como
saber qual valor um usuário digitará quando executar o código mais tarde.

Este é um exemplo dos princípios de segurança de memória do Rust em ação. Em
muitas linguagens de baixo nível, esse tipo de verificação não é feito e,
quando você fornece um índice incorreto, memória inválida pode ser acessada. O
Rust protege você contra esse tipo de erro saindo imediatamente, em vez de
permitir o acesso à memória e continuar. O Capítulo 9 discute mais sobre o
tratamento de erros do Rust e como você pode escrever código legível e seguro
que nem entra em pânico nem permite acesso inválido à memória.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md