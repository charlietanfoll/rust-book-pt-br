## O Tipo Slice

_Slices_ (fatias) permitem que você faça referência a uma sequência contígua de elementos em uma [coleção](ch08-00-common-collections.md)<!-- ignore --> em vez de toda a coleção. Uma slice é um tipo de referência, portanto, ela não possui propriedade.

Aqui está um pequeno problema de programação: escreva uma função que receba uma string de palavras separadas por espaços e retorne a primeira palavra que encontrar nessa string. Se a função não encontrar um espaço na string, a string inteira deve ser uma única palavra, portanto, toda a string deve ser retornada.

> Nota: Para fins de introdução às slices, estamos assumindo apenas ASCII nesta seção; uma discussão mais aprofundada sobre o tratamento de UTF-8 está na seção [“Armazenando Texto Codificado em UTF-8 com Strings”][strings]<!-- ignore --> do Capítulo 8.

Vamos trabalhar em como escreveríamos a assinatura dessa função sem usar slices, para entender o problema que as slices vão resolver:

```rust,ignore
fn first_word(s: &String) -> ?
```

A função `first_word` tem um parâmetro do tipo `&String`. Não precisamos de propriedade, então isso está bom. (Em Rust idiomático, as funções não assumem a propriedade de seus argumentos a menos que precisem, e os motivos para isso ficarão claros à medida que continuarmos.) Mas o que devemos retornar? Na verdade, não temos uma maneira de falar sobre *parte* de uma string. No entanto, poderíamos retornar o índice do final da palavra, indicado por um espaço. Vamos tentar isso, como mostrado na Listagem 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="A função `first_word` que retorna um valor de índice de byte no parâmetro `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Como precisamos percorrer a `String` elemento por elemento e verificar se um valor é um espaço, converteremos nossa `String` em uma matriz de bytes usando o método `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Em seguida, criamos um iterador sobre a matriz de bytes usando o método `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Discutiremos iteradores com mais detalhes no [Capítulo 13][ch13]<!-- ignore -->. Por enquanto, saiba que `iter` é um método que retorna cada elemento em uma coleção e que `enumerate` envolve o resultado de `iter` e retorna cada elemento como parte de uma tupla. O primeiro elemento da tupla retornada de `enumerate` é o índice, e o segundo elemento é uma referência ao elemento. Isso é um pouco mais conveniente do que calcular o índice nós mesmos.

Como o método `enumerate` retorna uma tupla, podemos usar padrões para desestruturar essa tupla. Discutiremos mais sobre padrões no [Capítulo 6][ch6]<!-- ignore -->. No loop `for`, especificamos um padrão que possui `i` para o índice na tupla e `&item` para o único byte na tupla. Como obtemos uma referência ao elemento de `.iter().enumerate()`, usamos `&` no padrão.

Dentro do loop `for`, procuramos o byte que representa o espaço usando a sintaxe de literal de byte. Se encontrarmos um espaço, retornamos a posição. Caso contrário, retornamos o comprimento da string usando `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Agora temos uma maneira de descobrir o índice do final da primeira palavra na string, mas há um problema. Estamos retornando um `usize` isolado, mas ele é um número significativo apenas no contexto de `&String`. Em outras palavras, por ser um valor separado da `String`, não há garantia de que ele ainda será válido no futuro. Considere o programa na Listagem 4-8 que usa a função `first_word` da Listagem 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Armazenando o resultado da chamada da função `first_word` e depois alterando o conteúdo da `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Este programa compila sem erros e também o faria se usássemos `word` após chamar `s.clear()`. Como `word` não está conectado ao estado de `s` de forma alguma, `word` ainda contém o valor `5`. Poderíamos usar esse valor `5` com a variável `s` para tentar extrair a primeira palavra, mas isso seria um bug porque o conteúdo de `s` mudou desde que salvamos `5` em `word`.

Ter que se preocupar com o fato de o índice em `word` ficar dessincronizado com os dados em `s` é tedioso e propenso a erros! O gerenciamento desses índices é ainda mais frágil se escrevermos uma função `second_word`. A assinatura dela teria que ser parecida com isto:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Agora estamos rastreando um índice inicial _e_ um final, e temos ainda mais valores que foram calculados a partir de dados em um estado específico, mas que não estão atados a esse estado de forma alguma. Temos três variáveis não relacionadas circulando por aí que precisam ser mantidas em sincronia.

Felizmente, o Rust tem uma solução para esse problema: fatias de string (_string slices_).

### Fatias de String (String Slices)

Uma _fatia de string_ é uma referência a uma sequência contígua de elementos de uma `String`, e ela se parece com isto:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Em vez de uma referência a toda a `String`, `hello` é uma referência a uma porção da `String`, especificada no trecho extra `[0..5]`. Criamos fatias usando um intervalo dentro de colchetes especificando `[starting_index..ending_index]`, onde _`starting_index`_ (índice inicial) é a primeira posição na fatia e _`ending_index`_ (índice final) é um a mais do que a última posição na fatia. Internamente, a estrutura de dados da fatia armazena a posição inicial e o comprimento da fatia, o que corresponde a _`ending_index`_ menos _`starting_index`_. Portanto, no caso de `let world = &s[6..11];`, `world` seria uma fatia que contém um ponteiro para o byte no índice 6 de `s` com um valor de comprimento igual a `5`.

A Figura 4-7 mostra isso em um diagrama.

<img alt="Three tables: a table representing the stack data of s, which points
to the byte at index 0 in a table of the string data &quot;hello world&quot; on
the heap. The third table represents the stack data of the slice world, which
has a length value of 5 and points to byte 6 of the heap data table."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-7: Uma fatia de string fazendo referência a parte de uma `String`</span>

Com a sintaxe de intervalo `..` do Rust, se você quiser começar no índice 0, você pode omitir o valor antes dos dois pontos. Em outras palavras, estas opções são iguais:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Da mesma forma, se a sua fatia incluir o último byte da `String`, você poderá omitir o número final. Isso significa que estas opções são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Você também pode omitir ambos os valores para pegar uma fatia de toda a string. Portanto, estas opções são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Nota: Os índices de intervalo de fatias de string devem ocorrer em limites de caracteres UTF-8 válidos. Se você tentar criar uma fatia de string no meio de um caractere multibyte, seu programa será encerrado com um erro.

Com todas essas informações em mente, vamos reescrever `first_word` para retornar uma fatia. O tipo que significa "fatia de string" é escrito como `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Obtemos o índice para o final da palavra da mesma maneira que fizemos na Listagem 4-7, procurando a primeira ocorrência de um espaço. Quando encontramos um espaço, retornamos uma fatia de string usando o início da string e o índice do espaço como os índices inicial e final.

Agora, quando chamamos `first_word`, recebemos de volta um único valor que está atado aos dados subjacentes. O valor é composto por uma referência ao ponto de partida da fatia e o número de elementos na fatia.

Retornar uma fatia também funcionaria para uma função `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Agora temos uma API direta que é muito mais difícil de errar porque o compilador garantirá que as referências na `String` permaneçam válidas. Lembra do bug no programa na Listagem 4-8, quando obtivemos o índice para o final da primeira palavra, mas depois limpamos (`clear`) a string de modo que nosso índice ficou inválido? Aquele código estava logicamente incorreto, mas não apresentou erros imediatos. Os problemas apareceriam mais tarde se continuássemos tentando usar o índice da primeira palavra com uma string esvaziada. As fatias tornam esse bug impossível e nos avisam muito mais cedo de que temos um problema em nosso código. O uso da versão de fatia de `first_word` gerará um erro em tempo de compilação:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Aqui está o erro do compilador:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Lembre-se das regras de empréstimo de que, se tivermos uma referência imutável a algo, não podemos obter também uma referência mutável. Como `clear` precisa truncar a `String`, ele precisa obter uma referência mutável. O `println!` após a chamada para `clear` usa a referência em `word`, portanto, a referência imutável ainda deve estar ativa naquele ponto. O Rust proíbe que a referência mutável em `clear` e a referência imutável em `word` existam ao mesmo tempo, e a compilação falha. O Rust não apenas tornou nossa API mais fácil de usar, como também eliminou toda uma classe de erros em tempo de compilação!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### Literais de String são Slices

Lembre-se de que falamos sobre literais de string serem armazenados dentro do binário. Agora que sabemos sobre fatias, podemos entender adequadamente os literais de string:

```rust
let s = "Hello, world!";
```

O tipo de `s` aqui é `&str`: é uma fatia apontando para aquele ponto específico do binário. É também por isso que os literais de string são imutáveis; `&str` é uma referência imutável.

#### Fatias de String como Parâmetros

Saber que você pode pegar fatias de literais e valores de `String` nos leva a mais uma melhoria em `first_word`, que é a sua assinatura:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Um Rustacean mais experiente escreveria a assinatura mostrada na Listagem 4-9 em vez disso, porque ela nos permite usar a mesma função tanto em valores `&String` quanto em valores `&str`.

<Listing number="4-9" caption="Melhorando a função `first_word` usando uma fatia de string para o tipo do parâmetro `s`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Se tivermos uma fatia de string, podemos passá-la diretamente. Se tivermos uma `String`, podemos passar uma fatia da `String` ou uma referência à `String`. Essa flexibilidade aproveita as coerções de desreferenciação (_deref coercions_), um recurso que abordaremos na seção [“Usando Coerções de Deref em Funções e Métodos”][deref-coercions]<!-- ignore --> do Capítulo 15.

Definir uma função para receber uma fatia de string em vez de uma referência a uma `String` torna nossa API mais genérica e útil sem perder nenhuma funcionalidade:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Outras Slices

Fatias de string, como você pode imaginar, são específicas para strings. Mas há um tipo de fatia mais geral também. Considere este array:

```rust
let a = [1, 2, 3, 4, 5];
```

Assim como podemos querer nos referir a parte de uma string, podemos querer nos referir a parte de um array. Faríamos isso assim:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Esta fatia tem o tipo `&[i32]`. Ela funciona da mesma forma que as fatias de string, armazenando uma referência ao primeiro elemento e um comprimento. Você usará esse tipo de fatia para todos os tipos de outras coleções. Discutiremos essas coleções em detalhes quando falarmos sobre vetores no Capítulo 8.

## Resumo

Os conceitos de propriedade, empréstimo e fatias garantem a segurança de memória em programas Rust em tempo de compilação. A linguagem Rust dá a você controle sobre o uso de memória da mesma forma que outras linguagens de programação de sistemas. Mas ter o proprietário dos dados limpando automaticamente esses dados quando o proprietário sai do escopo significa que você não precisa escrever e depurar código extra para obter esse controle.

A propriedade afeta como muitas outras partes do Rust funcionam, então discutiremos esses conceitos mais a fundo ao longo do restante do livro. Vamos avançar para o Capítulo 5 e ver o agrupamento de pedaços de dados juntos em uma `struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods