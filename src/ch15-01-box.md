## Usando `Box<T>` para Apontar para Dados no Heap

O ponteiro inteligente mais direto é a caixa (`box`), cujo tipo é escrito como
`Box<T>`. As _caixas_ permitem que você armazene dados no heap em vez da stack (pilha).
O que permanece na stack é o ponteiro para os dados no heap. Consulte o Capítulo 4
para revisar a diferença entre stack e heap.

As caixas não possuem sobrecarga de desempenho (performance overhead), exceto por armazenarem seus dados no
heap em vez de na stack. Mas elas também não têm muitas capacidades extras.
Você as usará com mais frequência nas seguintes situações:

- Quando você tem um tipo cujo tamanho não pode ser conhecido em tempo de compilação, e quer
  usar um valor desse tipo em um contexto que exige um tamanho exato
- Quando você tem uma grande quantidade de dados e quer transferir a propriedade, mas
  garantir que os dados não serão copiados ao fazer isso
- Quando você quer possuir um valor (ownership), e se importa apenas que ele seja um tipo que
  implementa uma trait específica em vez de ser de um tipo específico

Demonstraremos a primeira situação em [“Habilitando Tipos Recursivos com
Caixas”](#habilitando-tipos-recursivos-com-caixas)<!-- ignore -->. No segundo
caso, transferir a propriedade de uma grande quantidade de dados pode levar muito tempo
porque os dados são copiados pela stack. Para melhorar o desempenho nessa
situação, podemos armazenar a grande quantidade de dados no heap dentro de uma caixa. Assim,
apenas a pequena quantidade de dados do ponteiro é copiada pela stack, enquanto os
dados aos quais ele faz referência permanecem em um único lugar no heap. O terceiro caso é conhecido como um
_objeto trait_ (trait object), e a seção [“Usando Objetos Trait para Abstrair sobre Comportamento Compartilhado”][trait-objects]<!-- ignore --> no Capítulo 18 é dedicada a esse
tópico. Portanto, o que você aprenderá aqui será aplicado novamente nessa seção!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-store-data-on-the-heap"></a>

### Armazenando Dados no Heap

Antes de discutirmos o caso de uso de armazenamento no heap para `Box<T>`, cobriremos a
sintaxe e como interagir com valores armazenados dentro de um `Box<T>`.

A Listagem 15-1 mostra como usar uma caixa para armazenar um valor `i32` no heap.

<Listing number="15-1" file-name="src/main.rs" caption="Armazenando um valor `i32` no heap usando uma caixa">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Definimos a variável `b` para ter o valor de um `Box` que aponta para o
valor `5`, que é alocado no heap. Este programa imprimirá `b = 5`; neste
caso, podemos acessar os dados na caixa de maneira semelhante a como faríamos se esses
dados estivessem na stack. Assim como qualquer valor de propriedade própria, quando uma caixa sai do
escopo, como `b` faz no final de `main`, ela será desalocada. A
desalocação acontece tanto para a caixa (armazenada na stack) quanto para os dados para os quais ela aponta (armazenados no heap).

Colocar um único valor no heap não é muito útil, então você não usará caixas
sozinhas dessa maneira com muita frequência. Ter valores como um único `i32` na
stack, onde eles são armazenados por padrão, é mais apropriado na maioria das
situações. Vamos analisar um caso em que as caixas nos permitem definir tipos que
não teríamos permissão de definir se não as tivéssemos.

### Habilitando Tipos Recursivos com Caixas

O valor de um _tipo recursivo_ pode conter outro valor do mesmo tipo como parte
de si mesmo. Tipos recursivos representam um problema porque o Rust precisa saber em tempo de compilação
quanto espaço um tipo ocupa. No entanto, o aninhamento de valores de tipos recursivos
poderia teoricamente continuar infinitamente, de modo que o Rust não pode saber quanto espaço
o valor precisa. Como as caixas têm um tamanho conhecido, podemos habilitar tipos
recursivos inserindo uma caixa na definição do tipo recursivo.

Como exemplo de tipo recursivo, vamos explorar a lista cons (cons list). Este é um tipo
de dados comumente encontrado em linguagens de programação funcional. O tipo de lista cons
que definiremos é direto, exceto pela recursão; portanto, os
conceitos no exemplo com o qual trabalharemos serão úteis sempre que você entrar
em situações mais complexas envolvendo tipos recursivos.

<!-- Old headings. Do not remove or links may break. -->

<a id="more-information-about-the-cons-list"></a>

#### Entendendo a Lista Cons

Uma _lista cons_ é uma estrutura de dados originária da linguagem de programação Lisp
e de seus dialetos, é composta por pares aninhados e é a versão Lisp de uma
lista encadeada (linked list). Seu nome vem da função `cons` (abreviação de _construct
function_ — função de construção) em Lisp que constrói um novo par a partir de seus dois argumentos. Ao
chamar `cons` em um par consistindo de um valor e outro par, podemos
construir listas cons compostas por pares recursivos.

Por exemplo, aqui está uma representação em pseudocódigo de uma lista cons contendo a
lista `1, 2, 3` com cada par entre parênteses:

```text
(1, (2, (3, Nil)))
```

Cada item em uma lista cons contém dois elementos: o valor do item atual
e o do próximo item. O último item da lista contém apenas um valor chamado
`Nil` sem um próximo item. Uma lista cons é produzida chamando recursivamente a
função `cons`. O nome canônico para denotar o caso base da recursão é
`Nil`. Note que isso não é o mesmo que o conceito "null" ou "nil" discutido
no Capítulo 6, que é um valor inválido ou ausente.

A lista cons não é uma estrutura de dados comumente usada em Rust. Na maioria das
vezes em que você tem uma lista de itens em Rust, `Vec<T>` é uma escolha melhor para usar.
Outros tipos de dados recursivos mais complexos _são_ úteis em várias situações,
mas começando com a lista cons neste capítulo, podemos explorar como as caixas
nos permitem definir um tipo de dado recursivo sem muita distração.

A Listagem 15-2 contém a definição de um enum para uma lista cons. Note que este código
ainda não compilará, porque o tipo `List` não tem um tamanho conhecido, o
que demonstraremos.

<Listing number="15-2" file-name="src/main.rs" caption="A primeira tentativa de definir um enum para representar uma estrutura de dados de lista cons de valores `i32`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Nota: Estamos implementando uma lista cons que armazena apenas valores `i32` para
> os propósitos deste exemplo. Poderíamos tê-la implementado usando genéricos (generics), conforme
> discutimos no Capítulo 10, para definir um tipo de lista cons que pudesse armazenar valores de
> qualquer tipo.

Usar o tipo `List` para armazenar a lista `1, 2, 3` pareceria com o código na
Listagem 15-3.

<Listing number="15-3" file-name="src/main.rs" caption="Usando o enum `List` para armazenar a lista `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

O primeiro valor `Cons` armazena `1` e outro valor `List`. Este valor `List` é
outro valor `Cons` que armazena `2` e outro valor `List`. Este valor `List`
é mais um valor `Cons` que armazena `3` e um valor `List`, que é finalmente
`Nil`, a variante não recursiva que sinaliza o fim da lista.

Se tentarmos compilar o código na Listagem 15-3, obtemos o erro mostrado na
Listagem 15-4.

<Listing number="15-4" caption="O erro que obtemos ao tentar definir um enum recursivo">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

O erro mostra que este tipo “tem tamanho infinito”. A razão é que definimos
`List` com uma variante que é recursiva: Ela armazena outro valor de si mesma
diretamente. Como resultado, o Rust não consegue descobrir quanto espaço ele precisa para armazenar um
valor `List`. Vamos detalhar o porquê de obtermos esse erro. Primeiro, veremos como
o Rust decide quanto espaço ele precisa para armazenar um valor de um tipo não recursivo.

#### Calculando o Tamanho de um Tipo Não Recursivo

Lembre-se do enum `Message` que definimos na Listagem 6-2 quando discutimos definições de enum no Capítulo 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Para determinar quanto espaço alocar para um valor `Message`, o Rust passa
por cada uma das variantes para ver qual variante precisa de mais espaço. O Rust
vê que `Message::Quit` não precisa de espaço, `Message::Move` precisa de espaço suficiente
para armazenar dois valores `i32`, e assim por diante. Como apenas uma variante será
usada, o espaço máximo que um valor `Message` precisará é o espaço que seria necessário
para armazenar a maior de suas variantes.

Compare isso com o que acontece quando o Rust tenta determinar quanto espaço um
tipo recursivo como o enum `List` na Listagem 15-2 precisa. O compilador começa
olhando para a variante `Cons`, que armazena um valor do tipo `i32` e um valor
do tipo `List`. Portanto, `Cons` precisa de uma quantidade de espaço igual ao tamanho de
um `i32` mais o tamanho de um `List`. Para descobrir quanta memória o tipo `List`
precisa, o compilador olha para as variantes, começando com a variante `Cons`. A
variante `Cons` armazena um valor do tipo `i32` e um valor do tipo
`List`, e esse processo continua infinitamente, como mostrado na Figura 15-1.

<img alt="Uma lista Cons infinita: um retângulo rotulado como 'Cons' dividido em dois retângulos menores. O primeiro retângulo menor contém o rótulo 'i32', e o segundo retângulo menor contém o rótulo 'Cons' e uma versão menor do retângulo 'Cons' externo. Os retângulos 'Cons' continuam a conter versões cada vez menores de si mesmos até que o menor retângulo de tamanho confortável contenha um símbolo de infinito, indicando que essa repetição continua para sempre." src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 15-1: Uma `List` infinita consistindo de variantes
`Cons` infinitas</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-get-a-recursive-type-with-a-known-size"></a>

#### Obtendo um Tipo Recursivo com um Tamanho Conhecido

Como o Rust não consegue descobrir quanto espaço alocar para tipos definidos recursivamente,
o compilador gera um erro com esta sugestão útil:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

N этой sugestão, _indireção_ (indirection) significa que, em vez de armazenar um valor
diretamente, devemos alterar a estrutura de dados para armazenar o valor indiretamente por
meio do armazenamento de um ponteiro para o valor.

Como um `Box<T>` é um ponteiro, o Rust sempre sabe quanto espaço um `Box<T>`
precisa: O tamanho de um ponteiro não muda com base na quantidade de dados para os quais
ele está aponta. Isso significa que podemos colocar um `Box<T>` dentro da variante `Cons`
em vez de outro valor `List` diretamente. O `Box<T>` apontará para o próximo valor
`List` que estará no heap em vez de dentro da variante `Cons`.
Conceitualmente, ainda temos uma lista, criada com listas contendo outras listas, mas
esta implementação agora é mais parecida com colocar os itens um ao lado do outro
em vez de um dentro do outro.

Podemos alterar a definição do enum `List` na Listagem 15-2 e o uso
do `List` na Listagem 15-3 para o código na Listagem 15-5, que irá compilar.

<Listing number="15-5" file-name="src/main.rs" caption="A definição de `List` que usa `Box<T>` para ter um tamanho conhecido">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

A variante `Cons` precisa do tamanho de um `i32` mais o espaço para armazenar os dados do ponteiro da caixa. A variante `Nil` não armazena nenhum valor, então ela precisa de menos espaço na stack do que a variante `Cons`. Agora sabemos que qualquer valor `List` ocupará
o tamanho de um `i32` mais o tamanho dos dados do ponteiro de uma caixa. Ao usar uma caixa,
quebramos a cadeia recursiva infinita, para que o compilador possa descobrir o
tamanho necessário para armazenar um valor `List`. A Figura 15-2 mostra como a variante
`Cons` se parece agora.

<img alt="Um retângulo rotulado como 'Cons' dividido em dois retângulos menores. O primeiro retângulo menor contém o rótulo 'i32', e o segundo retângulo menor contém o rótulo 'Box' com um retângulo interno que contém o rótulo 'usize', representando o tamanho finito do ponteiro da caixa." src="img/trpl15-02.svg" class="center" />

<span class="caption">Figura 15-2: Uma `List` que não tem tamanho infinito,
porque `Cons` contém uma `Box`</span>

As caixas fornecem apenas a indireção e a alocação no heap; elas não têm nenhuma
outra capacidade especial, como aquelas que veremos com os outros tipos de ponteiros inteligentes.
Elas também não têm a sobrecarga de desempenho que essas capacidades especiais
acarretam, de modo que podem ser úteis em casos como a lista cons onde a
indireção é o único recurso de que precisamos. Veremos mais casos de uso para caixas no Capítulo 18.

O tipo `Box<T>` é um ponteiro inteligente porque implementa a trait `Deref`,
o que permite que valores `Box<T>` sejam tratados como referências. Quando um valor
`Box<T>` sai do escopo, os dados no heap para os quais a caixa aponta também são limpos
por causa da implementação da trait `Drop`. Essas duas traits serão
ainda mais importantes para a funcionalidade fornecida pelos outros tipos de ponteiros inteligentes
que discutiremos no resto deste capítulo. Vamos explorar essas duas traits
com mais detalhes.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
