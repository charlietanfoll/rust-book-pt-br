## Armazenando texto codificado em UTF-8 com Strings

Falamos sobre *strings* no Capítulo 4, mas vamos analisá-las com mais profundidade agora. Novos Rustacean geralmente travam em *strings* por uma combinação de três razões: a propensão do Rust em expor possíveis erros, as *strings* serem uma estrutura de dados mais complicada do que muitos programadores imaginam, e o UTF-8. Esses fatores se combinam de uma maneira que pode parecer difícil quando você vem de outras linguagens de programação.

Discutimos *strings* no contexto de coleções porque as *strings* são implementadas como uma coleção de bytes, juntamente com alguns métodos para fornecer funcionalidade útil quando esses bytes são interpretados como texto. Nesta seção, falaremos sobre as operações em `String` que todo tipo de coleção possui, como criação, atualização e leitura. Também discutiremos as maneiras pelas quais `String` é diferente das outras coleções; especificamente, como a indexação em uma `String` é complicada pelas diferenças entre como as pessoas e os computadores interpretam os dados de uma `String`.

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### Definindo Strings

Primeiro, definiremos o que queremos dizer com o termo *string*. O Rust tem apenas um tipo de *string* na linguagem principal, que é a fatia de *string* (*string slice*) `str`, geralmente vista em sua forma emprestada, `&str`. No Capítulo 4, falamos sobre fatias de *string*, que são referências a alguns dados de *string* codificados em UTF-8 armazenados em outro lugar. *Literais de string*, por exemplo, são armazenados no binário do programa e, portanto, são fatias de *string*.

O tipo `String`, que é fornecido pela biblioteca padrão do Rust em vez de ser embutido na linguagem principal, é um tipo de *string* com codificação UTF-8, proprietário, mutável e redimensionável. Quando os Rustaceans se referem a "*strings*" no Rust, eles podem estar se referindo tanto ao tipo `String` quanto à fatia de *string* `&str`, e não apenas a um deles. Embora esta seção seja em grande parte sobre `String`, ambos os tipos são amplamente utilizados na biblioteca padrão do Rust, e tanto `String` quanto as fatias de *string* são codificadas em UTF-8.

### Criando uma Nova String

Muitas das mesmas operações disponíveis com `Vec<T>` também estão disponíveis com `String`, porque `String` é implementada na verdade como um wrapper em torno de um vetor de bytes com algumas garantias, restrições e recursos extras. Um exemplo de função que funciona da mesma maneira com `Vec<T>` e `String` é a função `new` para criar uma instância, mostrada na Listagem 8-11.

<Listing number="8-11" caption="Criando uma `String` nova e vazia">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Esta linha cria uma *string* nova e vazia chamada `s`, na qual podemos carregar dados. Frequentemente, teremos alguns dados iniciais com os quais queremos começar a *string*. Para isso, usamos o método `to_string`, que está disponível em qualquer tipo que implemente o *trait* `Display`, assim como os literais de *string*. A Listagem 8-12 mostra dois exemplos.

<Listing number="8-12" caption="Usando o método `to_string` para criar uma `String` a partir de um literal de string">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Este código cria uma *string* contendo `initial contents`.

Também podemos usar a função `String::from` para criar uma `String` a partir de um literal de *string*. O código na Listagem 8-13 é equivalente ao código na Listagem 8-12 que usa `to_string`.

<Listing number="8-13" caption="Usando a função `String::from` para criar uma `String` a partir de um literal de string">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Como as *strings* são usadas para tantas coisas, podemos usar várias APIs genéricas diferentes para *strings*, nos dando muitas opções. Algumas delas podem parecer redundantes, mas todas têm seu lugar! Nesse caso, `String::from` e `to_string` fazem a mesma coisa, então qual você escolher é uma questão de estilo e legibilidade.

Lembre-se de que as *strings* são codificadas em UTF-8, portanto, podemos incluir nelas qualquer dado codificado corretamente, conforme mostrado na Listagem 8-14.

<Listing number="8-14" caption="Armazenando saudações em diferentes idiomas em strings">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Todos estes são valores de `String` válidos.

### Atualizando uma String

Uma `String` pode crescer em tamanho e seu conteúdo pode mudar, assim como o conteúdo de um `Vec<T>`, se você adicionar mais dados a ela. Além disso, você pode usar convenientemente o operador `+` ou a macro `format!` para concatenar valores de `String`.

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### Adicionando com `push_str` ou `push`

Podemos expandir uma `String` usando o método `push_str` para anexar uma fatia de *string*, conforme mostrado na Listagem 8-15.

<Listing number="8-15" caption="Anexando uma fatia de string a uma `String` usando o método `push_str`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Após essas duas linhas, `s` conterá `foobar`. O método `push_str` aceita uma fatia de *string* porque não queremos necessariamente assumir a propriedade (*ownership*) do parâmetro. Por exemplo, no código da Listagem 8-16, queremos poder usar `s2` após anexar seu conteúdo a `s1`.

<Listing number="8-16" caption="Usando uma fatia de string após anexar seu conteúdo a uma `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Se o método `push_str` assumisse a propriedade de `s2`, não poderíamos imprimir seu valor na última linha. No entanto, este código funciona como esperaríamos!

O método `push` aceita um único caractere como parâmetro e o adiciona à `String`. A Listagem 8-17 adiciona a letra _l_ a uma `String` usando o método `push`.

<Listing number="8-17" caption="Adicionando um caractere a um valor de `String` usando `push`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Como resultado, `s` conterá `lol`.

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### Concatenando com `+` ou `format!`

Frequentemente, você vai querer combinar duas *strings* existentes. Uma maneira de fazer isso é usar o operador `+`, conforme mostrado na Listagem 8-18.

<Listing number="8-18" caption="Usando o operador `+` para combinar dois valores de `String` em um novo valor de `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

A *string* `s3` conterá `Hello, world!`. O motivo pelo qual `s1` não é mais válido após a adição, e o motivo pelo qual usamos uma referência a `s2`, tem a ver com a assinatura do método que é chamado quando usamos o operador `+`. O operador `+` usa o método `add`, cuja assinatura se parece com isto:

```rust,ignore
fn add(self, s: &str) -> String {
```

Na biblioteca padrão, você verá `add` definido usando genéricos e tipos associados. Aqui, substituímos por tipos concretos, que é o que acontece quando chamamos esse método com valores de `String`. Discutiremos genéricos no Capítulo 10. Esta assinatura nos dá as pistas necessárias para entender as partes complexas do operador `+`.

Primeiro, `s2` tem um `&`, o que significa que estamos adicionando uma referência da segunda *string* à primeira *string*. Isso ocorre devido ao parâmetro `s` na função `add`: só podemos adicionar uma fatia de *string* a uma `String`; não podemos somar dois valores de `String`. Mas espere — o tipo de `&s2` é `&String`, e não `&str`, conforme especificado no segundo parâmetro de `add`. Então, por que a Listagem 8-18 compila?

O motivo pelo qual podemos usar `&s2` na chamada para `add` é que o compilador pode coagir o argumento `&String` em um `&str`. Quando chamamos o método `add`, o Rust usa uma coerção de desreferenciação (*deref coercion*), que aqui transforma `&s2` em `&s2[..]`. Discutiremos a coerção de desreferenciação com mais profundidade no Capítulo 15. Como `add` não assume a propriedade do parâmetro `s`, `s2` ainda será uma `String` válida após esta operação.

Segundo, podemos ver na assinatura que `add` assume a propriedade de `self` porque `self` _não_ tem um `&`. Isso significa que `s1` na Listagem 8-18 será movido para dentro da chamada `add` e não será mais válido depois disso. Portanto, embora `let s3 = s1 + &s2;` pareça que vai copiar ambas as *strings* e criar uma nova, esta instrução na verdade assume a propriedade de `s1`, anexa uma cópia do conteúdo de `s2` e, em seguida, retorna a propriedade do resultado. Em outras palavras, parece que está fazendo muitas cópias, mas não está; a implementação é mais eficiente do que copiar.

Se precisarmos concatenar várias *strings*, o comportamento do operador `+` se torna trabalhoso:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Nesse ponto, `s` será `tic-tac-toe`. Com todos os caracteres `+` e `"`, fica difícil entender o que está acontecendo. Para combinar *strings* de maneiras mais complicadas, podemos usar a macro `format!` em vez disso:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Este código também define `s` como `tic-tac-toe`. A macro `format!` funciona como `println!`, mas em vez de imprimir a saída na tela, ela retorna uma `String` com o conteúdo. A versão do código que usa `format!` é muito mais fácil de ler, e o código gerado pela macro `format!` usa referências para que esta chamada não tome a propriedade de nenhum de seus parâmetros.

### Indexando Strings

Em muitas outras linguagens de programação, acessar caracteres individuais em uma *string* referenciando-os por índice é uma operação válida e comum. No entanto, se você tentar acessar partes de uma `String` usando a sintaxe de indexação no Rust, receberá um erro. Considere o código inválido na Listagem 8-19.

<Listing number="8-19" caption="Tentando usar a sintaxe de indexação com uma `String`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Este código resultará no seguinte erro:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

O erro conta a história: as *strings* do Rust não suportam indexação. Mas por que não? Para responder a essa pergunta, precisamos discutir como o Rust armazena *strings* na memória.

#### Representação Interna

Uma `String` é um wrapper sobre um `Vec<u8>`. Vamos olhar para algumas de nossas *strings* de exemplo codificadas em UTF-8 da Listagem 8-14. Primeiro, esta:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Nesse caso, `len` será `4`, o que significa que o vetor que armazena a *string* `"Hola"` tem 4 bytes de comprimento. Cada uma dessas letras ocupa 1 byte quando codificada em UTF-8. A linha a seguir, no entanto, pode surpreendê-lo (observe que esta *string* começa com a letra maiúscula cirílica *Ze*, e não com o número 3):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Se lhe perguntassem qual o tamanho da *string*, você poderia dizer 12. De fato, a resposta do Rust é 24: esse é o número de bytes necessários para codificar "Здравствуйте" em UTF-8, porque cada valor escalar Unicode nessa *string* ocupa 2 bytes de armazenamento. Portanto, um índice nos bytes da *string* nem sempre corresponderá a um valor escalar Unicode válido. Para demonstrar, considere este código Rust inválido:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Você já sabe que `answer` não será `З`, a primeira letra. Quando codificado em UTF-8, o primeiro byte de `З` é `208` e o segundo é `151`, então pareceria que `answer` deveria de fato ser `208`, mas `208` não é um caractere válido por si só. Retornar `208` provavelmente não é o que um usuário gostaria se pedisse a primeira letra desta *string*; no entanto, esses são os únicos dados que o Rust tem no índice de byte 0. Os usuários geralmente não querem que o valor do byte seja retornado, mesmo se a *string* contiver apenas letras latinas: se `&"hi"[0]` fosse um código válido que retornasse o valor do byte, ele retornaria `104`, e não `h`.

A resposta, então, é que para evitar retornar um valor inesperado e causar bugs que podem não ser descobertos imediatamente, o Rust não compila este código e evita mal-entendidos no início do processo de desenvolvimento.

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### Bytes, Valores Escalares e Aglomerados de Grafemas

Outro ponto sobre o UTF-8 é que existem na verdade três maneiras relevantes de olhar para as *strings* da perspectiva do Rust: como bytes, valores escalares e aglomerados de grafemas (*grapheme clusters* — a coisa mais próxima do que chamaríamos de _letras_).

Se olharmos para a palavra em hindi “नमस्ते” escrita no script Devanagari, ela é armazenada como um vetor de valores `u8` que se parece com isto:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Isso são 18 bytes e é como os computadores armazenam esses dados no final das contas. Se olharmos para eles como valores escalares Unicode, que é o que o tipo `char` do Rust é, esses bytes se parecem com isto:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Existem seis valores `char` aqui, mas o quarto e o sexto não são letras: são diacríticos que não fazem sentido por si só. Finalmente, se olharmos para eles como aglomerados de grafemas, obteremos o que uma pessoa chamaria de quatro letras que compõem a palavra em hindi:

```text
["न", "म", "स्", "ते"]
```

O Rust fornece diferentes maneiras de interpretar os dados brutos de *string* que os computadores armazena para que cada programa possa escolher a interpretação de que precisa, não importa em qual idioma humano os dados estejam.

Uma razão final pela qual o Rust não nos permite indexar uma `String` para obter um caractere é que espera-se que as operações de indexação sempre levem tempo constante (O(1)). Mas não é possível garantir esse desempenho com uma `String`, porque o Rust teria que percorrer o conteúdo desde o início até o índice para determinar quantos caracteres válidos existiam.

### Fatiando Strings

Indexar uma *string* costuma ser uma má ideia porque não está claro qual deve ser o tipo de retorno da operação de indexação de *string*: um valor de byte, um caractere, um aglomerado de grafemas ou uma fatia de *string*. Portanto, se você realmente precisa usar índices para criar fatias de *string*, o Rust pede que você seja mais específico.

Em vez de indexar usando `[]` com um único número, você pode usar `[]` com um intervalo (*range*) para criar uma fatia de *string* contendo bytes específicos:

```text
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aqui, `s` será um `&str` que contém os primeiros 4 bytes da *string*. Anteriormente, mencionamos que cada um desses caracteres tinha 2 bytes, o que significa que `s` será `Зд`.

Se tentássemos fatiar apenas parte dos bytes de um caractere com algo como `&hello[0..1]`, o Rust entraria em pânico (*panic*) em tempo de execução da mesma forma que se um índice inválido fosse acessado em um vetor:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Você deve ter cuidado ao criar fatias de *string* com intervalos, pois fazer isso pode travar seu programa.

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### Iterando sobre Strings

A melhor maneira de operar em pedaços de *strings* é ser explícito sobre se você quer caracteres ou bytes. Para valores escalares Unicode individuais, use o método `chars`. Chamar `chars` em "Зд" separa e retorna dois valores do tipo `char`, e você pode iterar sobre o resultado para acessar cada elemento:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Este código imprimirá o seguinte:

```text
З
д
```

Alternativamente, o método `bytes` retorna cada byte bruto, o que pode ser apropriado para o seu domínio:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Este código imprimirá os 4 bytes que compõem esta *string*:

```text
208
151
208
180
```

Mas lembre-se de que valores escalares Unicode válidos podem ser compostos de mais de 1 byte.

Obter aglomerados de grafemas de *strings*, como no script Devanagari, é complexo, portanto essa funcionalidade não é fornecida pela biblioteca padrão. Criptas/Bibliotecas estão disponíveis em [crates.io](https://crates.io/)<!-- ignore --> se essa for a funcionalidade que você precisa.

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### Lidando com as Complexidades de Strings

Em suma, as *strings* são complicadas. Diferentes linguagens de programação fazem escolhas diferentes sobre como apresentar essa complexidade ao programador. O Rust escolheu tornar o tratamento correto de dados de `String` o comportamento padrão para todos os programas Rust, o que significa que os programadores precisam pensar mais em como lidar com dados UTF-8 antecipadamente. Essa compensação expõe mais da complexidade das *strings* do que é aparente em outras linguagens de programação, mas evita que você tenha que lidar com erros envolvendo caracteres não ASCII mais tarde no ciclo de vida do desenvolvimento.

A boa notícia é que a biblioteca padrão oferece muita funcionalidade construída a partir dos tipos `String` e `&str` para ajudar a lidar com essas situações complexas corretamente. Certifique-se de conferir a documentação para métodos úteis como `contains` para pesquisar em uma *string* e `replace` para substituir partes de uma *string* por outra.

Vamos mudar para algo um pouco menos complexo: tabelas hash (*hash maps*)!