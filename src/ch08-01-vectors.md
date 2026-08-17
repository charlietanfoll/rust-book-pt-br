## Armazenando Listas de Valores com Vetores

O primeiro tipo de coleção que vamos ver é o `Vec<T>`, também conhecido como vetor.
Os vetores permitem que você armazene mais de um valor em uma única estrutura de
dados que coloca todos os valores um ao lado do outro na memória. Os vetores só
podem armazenar valores do mesmo tipo. Eles são úteis quando você tem uma lista
de itens, como as linhas de texto em um arquivo ou os preços dos itens em um
carrinho de compras.

### Criando um Novo Vetor

Para criar um vetor novo e vazio, chamamos a função `Vec::new`, como mostrado na
Listagem 8-1.

<Listing number="8-1" caption="Criando um novo vetor vazio para conter valores do tipo `i32`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Note que adicionamos uma anotação de tipo aqui. Como não estamos inserindo
nenhum valor neste vetor, o Rust não sabe que tipo de elementos pretendemos
armazenar. Este é um ponto importante. Os vetores são implementados usando
generics; abordaremos como usar generics com seus próprios tipos no Capítulo 10.
Por enquanto, saiba que o tipo `Vec<T>` fornecido pela biblioteca padrão pode
conter qualquer tipo. Quando criamos um vetor para conter um tipo específico,
podemos especificar o tipo entre colchetes angulares. Na Listagem 8-1, dissemos
ao Rust que o `Vec<T>` em `v` conterá elementos do tipo `i32`.

Mais frequentemente, você criará um `Vec<T>` com valores iniciais, e o Rust
inferirá o tipo de valor que você deseja armazenar, então você raramente
precisará fazer essa anotação de tipo. O Rust fornece convenientemente a macro
`vec!`, que criará um novo vetor contendo os valores fornecidos. A Listagem 8-2
cria um novo `Vec<i32>` contendo os valores `1`, `2` e `3`. O tipo de inteiro é
`i32` porque esse é o tipo de inteiro padrão, como discutimos na seção [“Tipos
de Dados”][data-types]<!-- ignore --> do Capítulo 3.

<Listing number="8-2" caption="Criando um novo vetor contendo valores">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Como fornecemos valores iniciais do tipo `i32`, o Rust pode inferir que o tipo de
`v` é `Vec<i32>`, e a anotação de tipo não é necessária. Em seguida, veremos como
modificar um vetor.

### Atualizando um Vetor

Para criar um vetor e depois adicionar elementos a ele, podemos usar o método
`push`, como mostrado na Listagem 8-3.

<Listing number="8-3" caption="Usando o método `push` para adicionar valores a um vetor">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Como em qualquer variável, se quisermos ser capazes de alterar seu valor,
precisamos torná-lo mutável usando a palavra-chave `mut`, conforme discutido no
Capítulo 3. Os números que colocamos dentro são todos do tipo `i32`, e o Rust
infere isso a partir dos dados, então não precisamos da anotação `Vec<i32>`.

### Lendo Elementos de Vetores

Existem duas maneiras de referenciar um valor armazenado em um vetor: via
indexação ou usando o método `get`. Nos exemplos a seguir, anotamos os tipos dos
valores que são retornado por essas funções para maior clareza.

A Listagem 8-4 mostra ambos os métodos de acesso a um valor em um vetor, com a
sintaxe de indexação e o método `get`.

<Listing number="8-4" caption="Usando a sintaxe de indexação e o método `get` para acessar um item em um vetor">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Note alguns detalhes aqui. Usamos o valor de índice `2` para obter o terceiro
elemento porque os vetores são indexados por número, começando em zero. Usar `&`
e `[]` nos dá uma referência ao elemento no valor do índice. Quando usamos o
método `get` com o índice passado como argumento, obtemos um `Option<&T>` que
podemos usar com `match`.

O Rust fornece essas duas maneiras de referenciar um elemento para que você possa
escolher como o programa se comporta quando você tenta usar um valor de índice
fora do intervalo de elementos existentes. Como exemplo, vamos ver o que
acontece quando temos um vetor de cinco elementos e tentamos acessar um
elemento no índice 100 com cada técnica, conforme mostrado na Listagem 8-5.

<Listing number="8-5" caption="Tentando acessar o elemento no índice 100 em um vetor contendo cinco elementos">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Quando executamos este código, o primeiro método com `[]` fará com que o
programa entre em pânico (*panic*) porque ele referencia um elemento
inexistente. Este método é melhor usado quando você quer que seu programa pare
de funcionar se houver uma tentativa de acessar um elemento além do final do
vetor.

Quando o método `get` recebe um índice que está fora do vetor, ele retorna
`None` sem entrar em pânico. Você usaria este método se o acesso a um elemento
além do intervalo do vetor puder acontecer ocasionalmente em circunstâncias
normais. Seu código terá então uma lógica para lidar com o recebimento de
`Some(&element)` ou `None`, conforme discutido no Capítulo 6. Por exemplo, o
índice pode vir de uma pessoa digitando um número. Se ela acidentalmente digitar
um número muito grande e o programa receber um valor `None`, você poderá dizer
ao usuário quantos itens existem no vetor atual e dar a ele outra chance de
digitar um valor válido. Isso seria mais amigável do que causar um pânico no
programa devido a um erro de digitação!

Quando o programa tem uma referência válida, o verificador de empréstimos
(*borrow checker*) aplica as regras de propriedade e empréstimo (cobertas no
Capítulo 4) para garantir que esta referência e quaisquer outras referências ao
conteúdo do vetor permaneçam válidas. Lembre-se da regra que afirma que você
não pode ter referências mutáveis e imutáveis no mesmo escopo. Essa regra se
aplica na Listagem 8-6, onde mantemos uma referência imutável ao primeiro
elemento em um vetor e tentamos adicionar um elemento ao final. Este programa
não funcionará se tentarmos também nos referir a esse elemento mais tarde na
função.

<Listing number="8-6" caption="Tentando adicionar um elemento a um vetor enquanto mantém uma referência a um item">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

A compilação deste código resultará neste erro:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

O código na Listagem 8-6 pode parecer que deveria funcionar: por que uma
referência ao primeiro elemento deveria se importar com alterações no final do
vetor? Esse erro se deve à forma como os vetores funcionam: como os vetores
colocam os valores um ao lado do outro na memória, adicionar um novo elemento ao
final do vetor pode exigir a alocação de nova memória e a cópia dos elementos
antigos para o novo espaço, caso não haja espaço suficiente para colocar todos
os elementos juntos onde o vetor está armazenado no momento. Nesse caso, a
referência ao primeiro elemento estaria apontando para memória desalocada. As
regras de empréstimo evitam que os programas cheguem a essa situação.

> Nota: Para mais detalhes sobre a implementação do tipo `Vec<T>`, consulte [“O
> Rustonomicon”][nomicon].

### Iterando sobre os Valores em um Vetor

Para acessar cada elemento em um vetor sequencialmente, iteraríamos por todos os
elementos em vez de usar índices para acessar um de cada vez. A Listagem 8-7
mostra como usar um loop `for` para obter referências imutáveis a cada elemento
em um vetor de valores `i32` e imprimi-los.

<Listing number="8-7" caption="Imprimindo cada elemento em um vetor iterando sobre os elementos usando um loop `for`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Também podemos iterar sobre referências mutáveis para cada elemento em um vetor
mutável para fazer alterações em todos os elementos. O loop `for` na Listagem
8-8 adicionará `50` a cada elemento.

<Listing number="8-8" caption="Iterando sobre referências mutáveis para elementos em um vetor">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Para alterar o valor ao qual a referência mutável se refere, temos que usar o
operador de desreferência `*` para acessar o valor em `i` antes de podermos usar o
operador `+=`. Falaremos mais sobre o operador de desreferência na seção
[“Seguindo a Referência até o Valor”][deref]<!-- ignore --> do Capítulo 15.

Iterar sobre um vetor, seja de forma imutável ou mutável, é seguro devido às
regras do verificador de empréstimos. Se tentássemos inserir ou remover itens
nos corpos dos loops `for` nas Listagens 8-7 e 8-8, obteríamos um erro de
compilação semelhante ao que obtivemos com o código na Listagem 8-6. A
referência ao vetor que o loop `for` mantém impede a modificação simultânea de
todo o vetor.

### Usando um Enum para Armazenar Múltiplos Tipos

Os vetores só podem armazenar valores que sejam do mesmo tipo. Isso pode ser
inconveniente; certamente existem casos de uso em que é necessário armazenar
uma lista de itens de tipos diferentes. Felizmente, as variantes de um enum são
definidas sob o mesmo tipo de enum, então quando precisamos de um tipo para
representar elementos de tipos diferentes, podemos definir e usar um enum!

Por exemplo, digamos que queremos obter valores de uma linha em uma planilha na
qual algumas das colunas contêm inteiros, algumas números de ponto flutuante e
algumas strings. Podemos definir um enum cujas variantes conterão os diferentes
tipos de valor, e todas as variantes do enum serão consideradas do mesmo tipo: o
tipo do enum. Então, podemos criar um vetor para conter esse enum e, em última
análise, conter tipos diferentes. Demonstramos isso na Listagem 8-9.

<Listing number="8-9" caption="Definindo um enum para armazenar valores de diferentes tipos em um único vetor">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

O Rust precisa saber quais tipos estarão no vetor em tempo de compilação para
saber exatamente quanta memória no heap será necessária para armazenar cada
elemento. Também devemos ser explícitos sobre quais tipos são permitidos neste
vetor. Se o Rust permitisse que um vetor contivesse qualquer tipo, haveria a
chance de que um ou mais tipos causassem erros com as operações executadas nos
elementos do vetor. Usar um enum mais uma expressão `match` significa que o Rust
garantirá em tempo de compilação que todos os casos possíveis sejam tratados,
conforme discutido no Capítulo 6.

Se você não conhece o conjunto exaustivo de tipos que um programa obterá em
tempo de execução para armazenar em um vetor, a técnica do enum não funcionará.
Em vez disso, você pode usar um objeto trait (*trait object*), que abordaremos no
Capítulo 18.

Agora que discutimos algumas das maneiras mais comuns de usar vetores, não deixe
de revisar a [documentação da API][vec-api]<!-- ignore --> para conhecer os
vários métodos úteis definidos em `Vec<T>` pela biblioteca padrão. Por exemplo,
além do `push`, o método `pop` remove e retorna o último elemento.

### Descartar um Vetor Descartará Seus Elementos

Como qualquer outra `struct`, um vetor é liberado quando sai de escopo, conforme
anotado na Listagem 8-10.

<Listing number="8-10" caption="Mostrando onde o vetor e seus elementos são descartados">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Quando o vetor é descartado, todo o seu conteúdo também é descartado, o que
significa que os inteiros que ele contém serão limpos. O verificador de
empréstimos garante que quaisquer referências ao conteúdo de um vetor sejam
usadas apenas enquanto o próprio vetor for válido.

Vamos passar para o próximo tipo de coleção: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
