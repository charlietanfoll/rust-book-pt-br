## Processamento de uma Série de Itens com Iteradores

O padrão de iterador permite que você execute alguma tarefa em uma sequência de
itens em turnos. Um iterador é responsável pela lógica de iterar sobre cada item
e determinar quando a sequência terminou. Quando você usa iteradores, não
precisa reimplementar essa lógica por conta própria.

Em Rust, os iteradores são _preguiçosos_ (lazy), o que significa que eles não têm
efeito até que você chame métodos que consomem o iterador para utilizá-lo. Por
exemplo, o código na Listagem 13-10 cria um iterador sobre os itens no vetor `v1`
chamando o método `iter` definido em `Vec<T>`. Esse código por si só não faz
nada útil.

<Listing number="13-10" file-name="src/main.rs" caption="Criando um iterador">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

O iterador é armazenado na variável `v1_iter`. Uma vez que criamos um iterador,
podemos usá-lo de várias maneiras. Na Listagem 3-5, iteramos sobre um array
usando um loop `for` para executar algum código em cada um de seus itens. Por
trás dos panos, isso criou implicitamente e depois consumiu um iterador, mas nós
passamos por cima de como exatamente isso funciona até agora.

No exemplo da Listagem 13-11, nós separamos a criação do iterador do uso do
iterador no loop `for`. Quando o loop `for` é chamado usando o iterador em
`v1_iter`, cada elemento no iterador é usado em uma iteração do loop, o que
imprime cada valor.

<Listing number="13-11" file-name="src/main.rs" caption="Usando um iterador em um loop `for`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Em linguagens que não possuem iteradores fornecidos por suas bibliotecas
padrão, você provavelmente escreveria essa mesma funcionalidade iniciando uma
variável no índice 0, usando essa variável para indexar o vetor para obter um
valor, e incrementando o valor da variável em um loop até que ela alcance o
número total de itens no vetor.

Os iteradores lidam com toda essa lógica para você, reduzindo código repetitivo
no qual você poderia potencialmente errar. Os iteradores dão a você mais
flexibilidade para usar a mesma lógica com muitos tipos diferentes de
sequências, não apenas estruturas de dados que você pode indexar, como vetores.
Vamos examinar como os iteradores fazem isso.

### O Trait `Iterator` e o Método `next`

Todos os iteradores implementam um trait chamado `Iterator` que é definido na
biblioteca padrão. A definição do trait se parece com isto:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // métodos com implementações padrão omitidos
}
```

Note que esta definição usa alguma sintaxe nova: `type Item` e `Self::Item`, que
estão definindo um tipo associado a este trait. Falaremos sobre tipos
associados em profundidade no Capítulo 20. Por enquanto, tudo o que você precisa
saber é que este código diz que implementar o trait `Iterator` requer que você
também defina um tipo `Item`, e este tipo `Item` é usado no tipo de retorno do
método `next`. Em outras palavras, o tipo `Item` será o tipo retornado pelo
iterador.

O trait `Iterator` requer apenas que os implementadores definiam um método: o
método `next`, que retorna um item do iterador de cada vez, envolvido em `Some`,
e, quando a iteração termina, retorna `None`.

Podemos chamar o método `next` em iteradores diretamente; a Listagem 13-12
demonstra quais valores são retornados de chamadas repetidas a `next` no
iterador criado a partir do vetor.

<Listing number="13-12" file-name="src/lib.rs" caption="Chamando o método `next` em um iterador">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Note que precisamos tornar `v1_iter` mutável: chamar o método `next` em um
iterador altera o estado interno que o iterador usa para acompanhar onde ele
está na sequência. Em outras palavras, este código _consome_, ou esgota, o
iterador. Cada chamada para `next` consome um item do iterador. Não precisamos
tornar `v1_iter` mutável quando usamos um loop `for`, porque o loop assumiu a
posse de `v1_iter` e o tornou mutável por trás dos panos.

Note também que os valores que obtemos das chamadas para `next` são referências
imutáveis para os valores no vetor. O método `iter` produz um iterador sobre
referências imutáveis. Se quisermos criar um iterador que assuma a posse de `v1`
e retorne valores próprios (owned), podemos chamar `into_iter` em vez de
`iter`. Da mesma forma, se quisermos iterar sobre referências mutáveis, podemos
chamar `iter_mut` em vez de `iter`.

### Métodos que Consomem o Iterador

O trait `Iterator` possui vários métodos diferentes com implementações padrão
fornecidas pela biblioteca padrão; você pode descobrir sobre esses métodos
olhando na documentação da API da biblioteca padrão para o trait `Iterator`.
Alguns desses métodos chamam o método `next` em sua definição, e é por isso que
você é obrigado a implementar o método `next` ao implementar o trait
`Iterator`.

Métodos que chamam `next` são chamados de _adaptadores de consumo_ (consuming
adapters) porque chamá-los consome o iterador. Um exemplo é o método `sum`, que
assume a posse do iterador e itera pelos itens chamando repetidamente `next`,
consumindo assim o iterador. Conforme ele itera, ele adiciona cada item a um
total acumulado e retorna o total quando a iteração é concluída. A Listagem
13-13 tem um teste ilustrando o uso do método `sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Chamando o método `sum` para obter o total de todos os itens no iterador">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Não temos permissão para usar `v1_iter` após a chamada para `sum`, porque `sum`
assume a posse do iterador sobre o qual o chamamos.

### Métodos que Produzem Outros Iteradores

_Adaptadores de iterador_ (Iterator adapters) são métodos definidos no trait
`Iterator` que não consomem o iterador. Em vez disso, eles produzem iteradores
diferentes alterando algum aspecto do iterador original.

A Listagem 13-14 mostra um exemplo de chamada ao método adaptador de iterador
`map`, que recebe uma closure para chamar em cada item à medida que os itens
são iterados. O método `map` retorna um novo iterador que produz os itens
modificados. A closure aqui cria um novo iterador no qual cada item do vetor
será incrementado em 1.

<Listing number="13-14" file-name="src/main.rs" caption="Chamando o adaptador de iterador `map` para criar um novo iterador">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

No entanto, este código produz um aviso:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

O código na Listagem 13-14 não faz nada; a closure que especificamos nunca é
chamada. O aviso nos lembra o porquê: os adaptadores de iterador são
preguiçosos, e precisamos consumir o iterador aqui.

Para corrigir este aviso e consumir o iterador, usaremos o método `collect`, que
usamos com `env::args` na Listagem 12-1. Este método consome o iterador e coleta
os valores resultantes em um tipo de dado de coleção.

Na Listagem 13-15, coletamos os resultados de iterar sobre o iterador que é
retornado da chamada para `map` em um vetor. Este vetor terminará contendo cada
item do vetor original, incrementado em 1.

<Listing number="13-15" file-name="src/main.rs" caption="Chamando o método `map` para criar um novo iterador e, em seguida, chamando o método `collect` para consumir o novo iterador e criar um vetor">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Como `map` recebe uma closure, podemos especificar qualquer operação que
queremos realizar em cada item. Este é um ótimo exemplo de como as closures
permitem que você personalize algum comportamento enquanto reutiliza o
comportamento de iteração que o trait `Iterator` fornece.

Você pode encadear várias chamadas a adaptadores de iterador para realizar
ações complexas de maneira legível. Mas como todos os iteradores são
preguiçosos, você deve chamar um dos métodos adaptadores de consumo para obter
resultados das chamadas aos adaptadores de iterador.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### Closures que Capturam Seu Ambiente

Muitos adaptadores de iterador recebem closures como argumentos, e comumente
as closures que especificaremos como argumentos para adaptadores de iterador
serão closures que capturam seu ambiente.

Para este exemplo, usaremos o método `filter` que recebe uma closure. A closure
obtém um item do iterador e retorna um `bool`. Se a closure retornar `true`, o
valor será incluído no iterador produzido por `filter`. Se a closure retornar
`false`, o valor não será incluído.

Na Listagem 13-16, usamos `filter` com uma closure que captura a variável
`shoe_size` de seu ambiente para iterar sobre uma coleção de instâncias da
struct `Shoe`. Ela retornará apenas os sapatos que são do tamanho especificado.

<Listing number="13-16" file-name="src/lib.rs" caption="Usando o método `filter` com uma closure que captura `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

A função `shoes_in_size` assume a posse de um vetor de sapatos e um tamanho de
sapato como parâmetros. Ela retorna um vetor contendo apenas sapatos do tamanho
especificado.

No corpo de `shoes_in_size`, chamamos `into_iter` para criar um iterador que
assume a posse do vetor. Em seguida, chamamos `filter` para adaptar esse
iterador em um novo iterador que contém apenas elementos para os quais a
closure retorna `true`.

A closure captura o parâmetro `shoe_size` do ambiente e compara o valor com o
tamanho de cada sapato, mantendo apenas os sapatos do tamanho especificado.
Finalmente, chamar `collect` reúne os valores retornados pelo iterador
adaptado em um vetor que é retornado pela função.

O teste mostra que, quando chamamos `shoes_in_size`, obtemos de volta apenas os
sapatos que têm o mesmo tamanho que o valor que especificamos.