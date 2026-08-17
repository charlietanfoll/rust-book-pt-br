<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## Closures (Closures)

As closures do Rust são funções anônimas que você pode armazenar em uma variável ou
passar como argumentos para outras funções. Você pode criar a closure em um lugar e
depois chamá-la em outro lugar para avaliá-la em um contexto diferente. Ao contrário
das funções, as closures podem capturar valores do escopo em que são definidas.
Vamos demonstrar como esses recursos de closure permitem a reutilização de código e a
personalização de comportamento.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### Capturando o Ambiente

Primeiro, examinaremos como podemos usar closures para capturar valores do
ambiente em que foram definidas para uso posterior. Eis o cenário: de vez em
quando, nossa empresa de camisetas oferece uma camisa exclusiva e de edição
limitada para alguém em nossa lista de e-mails como promoção. As pessoas na lista
de e-mails podem opcionalmente adicionar sua cor favorita ao seu perfil. Se a
pessoa escolhida para uma camisa gratuita tiver sua cor favorita definida, ela
recebe uma camisa dessa cor. Se a pessoa não especificou uma cor favorita, ela
recebe qualquer cor que a empresa tenha em maior quantidade no momento.

Há muitas maneiras de implementar isso. Para este exemplo, vamos usar um
enum chamado `ShirtColor` que possui as variantes `Red` e `Blue` (limitando o
número de cores disponíveis por simplicidade). Representamos o estoque da empresa
com uma struct `Inventory` que possui um campo chamado `shirts` que contém
um `Vec<ShirtColor>` representando as cores de camisetas atualmente em estoque.
O método `giveaway` definido em `Inventory` obtém a preferência opcional de cor de
camisa do vencedor da camisa gratuita e retorna a cor da camisa que a pessoa
receberá. Essa configuração é mostrada na Listagem 13-1.

<Listing number="13-1" file-name="src/main.rs" caption="Situação de brinde da empresa de camisetas">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

A `store` definida em `main` tem duas camisetas azuis e uma vermelha restantes
para distribuir nesta promoção de edição limitada. Chamamos o método `giveaway`
para um usuário com preferência por uma camisa vermelha e para um usuário sem
preferência.

Novamente, este código poderia ser implementado de várias maneiras e aqui, para
nos focarmos em closures, nos mantivemos em conceitos que você já aprendeu, exceto
pelo corpo do método `giveaway`, que usa uma closure. No método `giveaway`,
obtemos a preferência do usuário como um parâmetro do tipo `Option<ShirtColor>` e
chamamos o método `unwrap_or_else` em `user_preference`. O [método
`unwrap_or_else` em `Option<T>`][unwrap-or-else]<!-- ignore --> é definido pela
biblioteca padrão. Ele aceita um argumento: uma closure sem argumentos que retorna
um valor `T` (o mesmo tipo armazenado na variante `Some` do `Option<T>`, neste
caso `ShirtColor`). Se o `Option<T>` for a variante `Some`, `unwrap_or_else`
retorna o valor de dentro do `Some`. Se o `Option<T>` for a variante `None`,
`unwrap_or_else` chama a closure e retorna o valor retornado por ela.

Especificamos a expressão de closure `|| self.most_stocked()` como o argumento
para `unwrap_or_else`. Esta é uma closure que não aceita parâmetros (se a
closure tivesse parâmetros, eles apareceriam entre as duas barras verticais). O
corpo da closure chama `self.most_stocked()`. Estamos definindo a closure aqui,
e a implementação de `unwrap_or_else` avaliará a closure mais tarde se o
resultado for necessário.

A execução deste código imprime o seguinte:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Um aspecto interessante aqui é que passamos uma closure que chama
`self.most_stocked()` na instância atual de `Inventory`. A biblioteca padrão
não precisava saber nada sobre os tipos `Inventory` ou `ShirtColor` que
definimos, nem sobre a lógica que queremos usar neste cenário. A closure captura
uma referência imutável para a instância `self` de `Inventory` e a passa com o
código que especificamos para o método `unwrap_or_else`. Funções, por outro lado,
não são capazes de capturar seu ambiente dessa maneira.

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### Inferência e Anotação de Tipos de Closures

Há mais diferenças entre funções e closures. As closures geralmente não exigem
que você anote os tipos dos parâmetros ou do valor de retorno como as funções `fn`
fazem. As anotações de tipo são obrigatórias em funções porque os tipos fazem parte
de uma interface explícita exposta aos seus usuários. Definir essa interface de
forma rígida é importante para garantir que todos concordem sobre quais tipos de
valores uma função usa e retorna. As closures, por outro lado, não são usadas em
uma interface exposta como essa: elas são armazenadas em variáveis e usadas sem
serem nomeadas e expostas aos usuários da nossa biblioteca.

As closures são tipicamente curtas e relevantes apenas dentro de um contexto
restrito, em vez de em qualquer cenário arbitrário. Dentro desses contextos
limitados, o compilador pode inferir os tipos dos parâmetros e o tipo de retorno,
assim como ele é capaz de inferir os tipos da maioria das variáveis (há raros casos
em que o compilador precisa de anotações de tipo em closures também).

Assim como com variáveis, podemos adicionar anotações de tipo se quisermos
aumentar a explicitude e a clareza ao custo de ser mais detalhado do que o
estritamente necessário. Anotar os tipos para uma closure seria parecido com a
definição mostrada na Listagem 13-2. Neste exemplo, estamos definindo uma closure
e armazenando-a em uma variável, em vez de definir a closure no local onde a
passamos como argumento, como fizemos na Listagem 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Adicionando anotações de tipo opcionais dos tipos de parâmetro e valor de retorno na closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Com as anotações de tipo adicionadas, a sintaxe das closures fica mais parecida
com a sintaxe das funções. Aqui, definimos uma função que adiciona 1 ao seu
parâmetro e uma closure que tem o mesmo comportamento, para comparação.
Adicionamos alguns espaços para alinhar as partes relevantes. Isso ilustra como a
sintaxe de closure é semelhante à sintaxe de função, exceto pelo uso de barras
verticais e pela quantidade de sintaxe que é opcional:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

A primeira linha mostra a definição de uma função e a segunda linha mostra a
definição de uma closure totalmente anotada. Na terceira linha, removemos as
anotações de tipo da definição da closure. Na quarta linha, removemos as chaves,
que são opcionais porque o corpo da closure tem apenas uma expressão. Todas estas
são definições válidas que produzirão o mesmo comportamento quando chamadas. As
linhas `add_one_v3` e `add_one_v4` exigem que as closures sejam avaliadas para
poderem compilar, porque os tipos serão inferidos a partir de seu uso. Isso é
semelhante a `let v = Vec::new();` precisar de anotações de tipo ou de valores de
algum tipo serem inseridos no `Vec` para que o Rust possa inferir o tipo.

Para definições de closure, o compilador inferirá um tipo concreto para cada um
de seus parâmetros e para o seu valor de retorno. Por exemplo, a Listagem 13-3
mostra a definição de uma closure curta que apenas retorna o valor que recebe
como parâmetro. Esta closure não é muito útil, exceto para os propósitos deste
exemplo. Note que não adicionamos nenhuma anotação de tipo à definição. Como
não há anotações de tipo, podemos chamar a closure com qualquer tipo, o que
fizemos aqui com `String` na primeira vez. Se tentarmos chamar
`example_closure` com um inteiro em seguida, receberemos um erro.

<Listing number="13-3" file-name="src/main.rs" caption="Tentativa de chamar uma closure cujos tipos são inferidos com dois tipos diferentes">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

O compilador nos dá este erro:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Na primeira vez que chamamos `example_closure` com o valor `String`, o compilador
infere o tipo de `x` e o tipo de retorno da closure como `String`. Esses tipos
são então travados na closure em `example_closure`, e recebemos um erro de tipo
quando tentamos usar um tipo diferente com a mesma closure em seguida.

### Capturando Referências ou Movendo a Posse

As closures podem capturar valores de seu ambiente de três maneiras, que
correspondem diretamente às três maneiras pelas quais uma função pode receber um
parâmetro: emprestando de forma imutável, emprestando de forma mutável e tomando
a posse. A closure decidirá qual delas usar com base no que o corpo da função
faz com os valores capturados.

Na Listagem 13-4, definimos uma closure que captura uma referência imutável para
o vetor chamado `list` porque ela precisa apenas de uma referência imutável para
imprimir o valor.

<Listing number="13-4" file-name="src/main.rs" caption="Definindo e chamando uma closure que captura uma referência imutável">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Este exemplo também ilustra que uma variável pode se vincular a uma definição de
closure, e podemos chamar a closure posteriormente usando o nome da variável e
parênteses, como se o nome da variável fosse um nome de função.

Como podemos ter múltiplas referências imutáveis para `list` ao mesmo tempo,
`list` ainda está acessível a partir do código antes da definição da closure,
após a definição da closure, mas antes que a closure seja chamada, e após a
closure ser chamada. Este código compila, executa e imprime:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Em seguida, na Listagem 13-5, alteramos o corpo da closure para que ela adicione
um elemento ao vetor `list`. A closure agora captura uma referência mutável.

<Listing number="13-5" file-name="src/main.rs" caption="Definindo e chamando uma closure que captura uma referência mutável">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Este código compila, executa e imprime:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Note que não há mais um `println!` entre a definição e a chamada da closure
`borrows_mutably`: Quando `borrows_mutably` é definida, ela captura uma
referência mutável para `list`. Não usamos a closure novamente após ela ser
chamada, então o empréstimo mutável termina. Entre a definição da closure e a
chamada da closure, um empréstimo imutável para imprimir não é permitido, porque
nenhum outro empréstimo é permitido quando há um empréstimo mutável. Tente
adicionar um `println!` ali para ver qual mensagem de erro você obtém!

Se você quiser forçar a closure a tomar a posse (ownership) dos valores que ela
usa no ambiente, mesmo que o corpo da closure não precise estritamente de
posse, você pode usar a palavra-chave `move` antes da lista de parâmetros.

Esta técnica é mais útil ao passar uma closure para uma nova thread para mover os
dados de modo que eles sejam de propriedade da nova thread. Discutiremos threads
e por que você as usaria em detalhes no Capítulo 16, quando falarmos sobre
concorrência, mas por ora, vamos explorar brevemente a criação de uma nova thread
usando uma closure que precisa da palavra-chave `move`. A Listagem 13-6 mostra a
Listagem 13-4 modificada para imprimir o vetor em uma nova thread, em vez da
thread principal.

<Listing number="13-6" file-name="src/main.rs" caption="Usando `move` para forçar a closure da thread a tomar posse de `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Criamos uma nova thread, passando para ela uma closure para executar como
argumento. O corpo da closure imprime a lista. Na Listagem 13-4, a closure
capturou apenas `list` usando uma referência imutável porque essa é a menor
quantidade de acesso a `list` necessária para imprimi-lo. Neste exemplo, mesmo
que o corpo da closure ainda precise apenas de uma referência imutável, precisamos
especificar que `list` deve ser movido para dentro da closure colocando a
palavra-chave `move` no início da definição da closure. Se a thread principal
realizasse mais operações antes de chamar `join` na nova thread, a nova thread
poderia terminar antes do resto da thread principal, ou a thread principal
poderia terminar primeiro. Se a thread principal mantivesse a posse de `list`,
mas terminasse antes da nova thread e descartasse (`drop`) `list`, a referência
imutável na thread seria inválida. Portanto, o compilador exige que `list` seja
movido para dentro da closure fornecida à nova thread para que a referência seja
válida. Tente remover a palavra-chave `move` ou usar `list` na thread principal
após a definição da closure para ver quais erros de compilação você obtém!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### Movendo Valores Capturados para Fora de Closures

Uma vez que uma closure tenha capturado uma referência ou a posse de um valor do
ambiente onde a closure é definida (afetando assim o que, se algo, é movido
_para dentro_ da closure), o código no corpo da closure define o que acontece com
as referências ou valores quando a closure é avaliada posteriormente (afetando
assim o que, se algo, é movido _para fora_ da closure).

O corpo de uma closure pode fazer qualquer uma das seguintes ações: Mover um
valor capturado para fora da closure, mutar o valor capturado, nem mover nem mutar
o valor, ou não capturar nada do ambiente para começar.

A maneira como uma closure captura e lida com valores do ambiente afeta quais
traits a closure implementa, e traits são a forma como funções e structs podem
especificar quais tipos de closures elas podem usar. As closures implementarão
automaticamente uma, duas ou todas as três destas traits `Fn`, de forma aditiva,
dependendo de como o corpo da closure lida com os valores:

* `FnOnce` aplica-se a closures que podem ser chamadas uma vez. Todas as closures
  implementam pelo menos esta trait porque todas as closures podem ser chamadas.
  Uma closure que move valores capturados para fora de seu corpo implementará
  apenas `FnOnce` e nenhuma das outras traits `Fn`, porque ela só pode ser
  chamada uma vez.
* `FnMut` aplica-se a closures que não movem valores capturados para fora de seu
  corpo, mas podem mutar os valores capturados. Essas closures podem ser chamadas
  mais de uma vez.
* `Fn` aplica-se a closures que não movem valores capturados para fora de seu
  corpo e não mutam os valores capturados, bem como closures que não capturam
  nada de seu ambiente. Essas closures podem ser chamadas mais de uma vez sem
  mutar seu ambiente, o que é importante em casos como chamar uma closure várias
  vezes concorrentemente.

Vamos olhar para a definição do método `unwrap_or_else` em `Option<T>` que
usamos na Listagem 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Lembre-se de que `T` é o tipo genérico que representa o tipo do valor na
variante `Some` de um `Option`. Esse tipo `T` é também o tipo de retorno da
função `unwrap_or_else`: o código que chama `unwrap_or_else` em um
`Option<String>`, por exemplo, receberá uma `String`.

Em seguida, note que a função `unwrap_or_else` tem o parâmetro de tipo genérico
adicional `F`. O tipo `F` é o tipo do parâmetro chamado `f`, que é a closure que
fornecemos ao chamar `unwrap_or_else`.

O limite de trait (trait bound) especificado no tipo genérico `F` é
`FnOnce() -> T`, o que significa que `F` deve ser capaz de ser chamado uma vez,
não aceitar argumentos e retornar um `T`. O uso de `FnOnce` no limite de trait
expressa a restrição de que `unwrap_or_else` não chamará `f` mais de uma vez. No
corpo de `unwrap_or_else`, podemos ver que se o `Option` for `Some`, `f` não
será chamado. Se o `Option` for `None`, `f` será chamado uma vez. Como todas as
closures implementam `FnOnce`, `unwrap_or_else` aceita todos os três tipos de
closures e é o mais flexível possível.

> Nota: Se o que queremos fazer não requer capturar um valor do ambiente,
> podemos usar o nome de uma função em vez de uma closure onde precisamos de
> algo que implemente uma das traits `Fn`. Por exemplo, em um valor
> `Option<Vec<T>>`, poderíamos chamar `unwrap_or_else(Vec::new)` para obter um
> novo vetor vazio se o valor for `None`. O compilador implementa
> automaticamente qual quer que seja a trait `Fn` aplicável para uma definição
> de função.

Agora vamos analisar o método da biblioteca padrão `sort_by_key`, definido em
fatias (slices), para ver como ele difere de `unwrap_or_else` e por que
`sort_by_key` usa `FnMut` em vez de `FnOnce` para o limite de trait. A closure
recebe um argumento na forma de uma referência ao item atual na fatia sendo
considerada, e retorna um valor do tipo `K` que pode ser ordenado. Esta função é
útil quando você deseja ordenar uma fatia por um atributo específico de cada
item. Na Listagem 13-7, temos uma lista de instâncias de `Rectangle`, e usamos
`sort_by_key` para ordená-las por seu atributo `width` do menor para o maior.

<Listing number="13-7" file-name="src/main.rs" caption="Usando `sort_by_key` para ordenar retângulos por largura">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Este código imprime:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

A razão pela qual `sort_by_key` é definido para aceitar uma closure `FnMut` é
que ele chama a closure várias vezes: uma vez para cada item na fatia. A
closure `|r| r.width` não captura, muta ou move nada para fora de seu ambiente,
então ela atende aos requisitos do limite de trait.

Em contraste, a Listagem 13-8 mostra um exemplo de uma closure que implementa
apenas a trait `FnOnce`, porque ela move um valor para fora do ambiente. O
compilador não nos deixará usar esta closure com `sort_by_key`.

<Listing number="13-8" file-name="src/main.rs" caption="Tentativa de usar uma closure `FnOnce` com `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Esta é uma maneira forçada e convoluta (que não funciona) de tentar contar o
número de vezes que `sort_by_key` chama a closure ao ordenar `list`. Este código
tenta fazer essa contagem inserindo (`push`) `value`—uma `String` do ambiente da
closure—no vetor `sort_operations`. A closure captura `value` e então move
`value` para fora da closure transferindo a posse de `value` para o vetor
`sort_operations`. Esta closure pode ser chamada uma vez; tentar chamá-la uma
segunda vez não funcionaria, porque `value` não estaria mais no ambiente para
ser inserido em `sort_operations` novamente! Portanto, esta closure implementa
apenas `FnOnce`. Quando tentamos compilar este código, recebemos este erro de que
`value` não pode ser movido para fora da closure porque a closure deve
implementar `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

O erro aponta para a linha no corpo da closure que move `value` para fora do
ambiente. Para corrigir isso, precisamos alterar o corpo da closure para que ela
não mova valores para fora do ambiente. Manter um contador no ambiente e
incrementar seu valor no corpo da closure é uma maneira mais direta de contar o
número de vezes que a closure é chamada. A closure na Listagem 13-9 funciona com
`sort_by_key` porque ela está apenas capturando uma referência mutável para o
contador `num_sort_operations` e, portanto, pode ser chamada mais de uma vez.

<Listing number="13-9" file-name="src/main.rs" caption="O uso de uma closure `FnMut` com `sort_by_key` é permitido.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

As traits `Fn` são importantes ao definir ou usar funções ou tipos que fazem uso
de closures. Na próxima seção, discutiremos iteradores. Muitos métodos de
iteradores aceitam argumentos de closure, então mantenha esses detalhes de
closure em mente enquanto continuamos!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
