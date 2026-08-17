## Ciclos de Referência Podem Vazamento de Memória (Memory Leaks)

As garantias de segurança de memória do Rust tornam difícil, mas não impossível,
criar acidentalmente memória que nunca é limpa (conhecido como um _vazamento de memória_ ou _memory leak_).
Evitar vazamentos de memória inteiramente não é uma das garantias do Rust, o que significa
que vazamentos de memória são seguros em termos de memória no Rust. Podemos ver que o Rust permite vazamentos de memória
ao usar `Rc<T>` e `RefCell<T>`: É possível criar referências onde
os itens se referem uns aos outros em um ciclo. Isso cria vazamentos de memória porque a
contagem de referências de cada item no ciclo nunca chegará a 0, e os valores
nunca serão descartados (_dropped_).

### Criando um Ciclo de Referência

Vamos ver como um ciclo de referência pode acontecer e como evitá-lo,
começando com a definição do enum `List` e um método `tail` na Listagem
15-25.

<Listing number="15-25" file-name="src/main.rs" caption="Uma definição de cons list que contém um `RefCell<T>` para que possamos modificar o que uma variante `Cons` está referenciando">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Estamos usando outra variação da definição de `List` da Listagem 15-5. O
segundo elemento na variante `Cons` agora é `RefCell<Rc<List>>`, o que significa que
em vez de termos a capacidade de modificar o valor `i32` como fizemos na Listagem
15-24, queremos modificar o valor `List` para o qual uma variante `Cons` está apontando.
Também estamos adicionando um método `tail` para tornar conveniente acessarmos o
segundo item se tivermos uma variante `Cons`.

Na Listagem 15-26, estamos adicionando uma função `main` que usa as definições da
Listagem 15-25. Esse código cria uma lista em `a` e uma lista em `b` que aponta para
a lista em `a`. Em seguida, ele modifica a lista em `a` para apontar para `b`, criando um
ciclo de referência. Há instruções `println!` ao longo do caminho para mostrar quais são as
contagens de referência em vários pontos desse processo.

<Listing number="15-26" file-name="src/main.rs" caption="Criando um ciclo de referência de dois valores `List` apontando um para o outro">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Criamos uma instância `Rc<List>` contendo um valor `List` na variável `a`
com uma lista inicial de `5, Nil`. Em seguida, criamos uma instância `Rc<List>` contendo
outro valor `List` na variável `b` que contém o valor `10` e
aponta para a lista em `a`.

Modificamos `a` para que ela aponte para `b` em vez de `Nil`, criando um ciclo. Nós
fazemos isso usando o método `tail` para obter uma referência ao
`RefCell<Rc<List>>` em `a`, que colocamos na variável `link`. Então, usamos
o método `borrow_mut` no `RefCell<Rc<List>>` para alterar o valor interno
de um `Rc<List>` que contém um valor `Nil` para o `Rc<List>` em `b`.

Quando executamos este código, mantendo o último `println!` comentado por enquanto,
obteremos esta saída:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

A contagem de referência das instâncias `Rc<List>` em `a` e `b` é 2 após
alterarmos a lista em `a` para apontar para `b`. No final de `main`, o Rust descarta a
variável `b`, o que diminui a contagem de referência da instância `Rc<List>` de `b`
de 2 para 1. A memória que o `Rc<List>` possui no _heap_ não será
descartada neste ponto porque sua contagem de referência é 1, não 0. Em seguida, o Rust descarta
`a`, o que diminui a contagem de referência da instância `Rc<List>` de `a`
de 2 para 1 também. A memória desta instância também não pode ser descartada, porque a outra
instância `Rc<List>` ainda se refere a ela. A memória alocada para a lista permanecerá
sem ser coletada para sempre. Para visualizar este ciclo de referência, criamos
o diagrama na Figura 15-4.

<img alt="Um retângulo rotulado como 'a' que aponta para um retângulo contendo o inteiro 5. Um retângulo rotulado como 'b' que aponta para um retângulo contendo o inteiro 10. O retângulo contendo 5 aponta para o retângulo contendo 10, e o retângulo contendo 10 aponta de volta para o retângulo contendo 5, criando um ciclo." src="img/trpl15-04.svg" class="center" />

<span class="caption">Figura 15-4: Um ciclo de referência das listas `a` e `b`
apontando uma para a outra</span>

Se você descomentar o último `println!` e executar o programa, o Rust tentará
imprimir este ciclo com `a` apontando para `b` apontando para `a` e assim por diante até
estourar a _stack_ (pilha).

Comparado a um programa do mundo real, as consequências de criar um ciclo de
referência neste exemplo não são tão graves: logo após criarmos o ciclo de
referência, o programa termina. No entanto, se um programa mais complexo alocasse muita
memória em um ciclo e a mantivesse por um longo tempo, o programa usaria mais
memória do que o necessário e poderia sobrecarregar o sistema, fazendo com que ele ficasse sem
memória disponível.

Criar ciclos de referência não é algo fácil de fazer, mas também não é impossível.
Se você tiver valores `RefCell<T>` que contêm valores `Rc<T>` ou combinações aninhadas semelhantes
de tipos com mutabilidade interior e contagem de referências, você deve
garantir que não crie ciclos; você não pode contar com o Rust para pegá-los.
Criar um ciclo de referência seria um bug lógico em seu programa que você deve
usar testes automatizados, revisões de código e outras práticas de desenvolvimento de software para
minimizar.

Outra solução para evitar ciclos de referência é reorganizar suas estruturas de dados
para que algumas referências expressem propriedade (_ownership_) e outras referências não.
Como resultado, você pode ter ciclos compostos por algumas relações de propriedade e
algumas relações de não-propriedade, e apenas as relações de propriedade afetam
se um valor pode ou não ser descartado. Na Listagem 15-25, nós sempre queremos que as variantes `Cons`
sejam donas de sua lista, então reorganizar a estrutura de dados não é possível.
Vamos ver um exemplo usando grafos compostos por nós pais e nós filhos para ver
quando as relações de não-propriedade são uma maneira apropriada de evitar ciclos de referência.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### Prevendo Ciclos de Referência Usando `Weak<T>`

Até agora, demonstramos que chamar `Rc::clone` aumenta o
`strong_count` de uma instância `Rc<T>`, e uma instância `Rc<T>` só é limpa
se o seu `strong_count` for 0. Você também pode criar uma referência fraca para o
valor dentro de uma instância `Rc<T>` chamando `Rc::downgrade` e passando uma
referência para o `Rc<T>`. *Referências fortes* (*Strong references*) são a forma como você pode compartilhar a propriedade
de uma instância `Rc<T>`. *Referências fracas* (*Weak references*) não expressam uma relação
de propriedade, e sua contagem não afeta quando uma instância `Rc<T>` é
limpa. Elas não causarão um ciclo de referência, porque qualquer ciclo envolvendo
algumas referências fracas será quebrado assim que a contagem de referência forte dos
valores envolvidos for 0.

Quando você chama `Rc::downgrade`, você obtém um ponteiro inteligente do tipo `Weak<T>`.
Em vez de aumentar o `strong_count` na instância `Rc<T>` em 1, chamar
`Rc::downgrade` aumenta o `weak_count` em 1. O tipo `Rc<T>` usa
`weak_count` para acompanhar quantas referências `Weak<T>` existem, de forma semelhante
ao `strong_count`. A diferença é que o `weak_count` não precisa ser 0 para que a
instância `Rc<T>` seja limpa.

Como o valor que `Weak<T>` referencia pode ter sido descartado, para fazer
qualquer coisa com o valor para o qual um `Weak<T>` está apontando, você deve garantir que
o valor ainda exista. Faça isso chamando o método `upgrade` em uma instância
`Weak<T>`, que retornará um `Option<Rc<T>>`. Você obterá um resultado `Some`
se o valor `Rc<T>` ainda não tiver sido descartado e um resultado `None` se o
valor `Rc<T>` tiver sido descartado. Como `upgrade` retorna um `Option<Rc<T>>`,
o Rust garantirá que o caso `Some` e o caso `None` sejam tratados, e
não haverá um ponteiro inválido.

Como exemplo, em vez de usar uma lista cujos itens sabem apenas sobre o próximo
item, criaremos uma árvore cujos itens sabem sobre seus itens filhos _e_ seus
itens pais.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### Criando uma Estrutura de Dados de Árvore

Para começar, construiremos uma árvore com nós que sabem sobre seus nós filhos.
Criaremos uma _struct_ chamada `Node` que armazena seu próprio valor `i32` bem como
referências aos seus valores `Node` filhos:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Queremos que um `Node` seja dono de seus filhos, e queremos compartilhar essa propriedade com
variáveis para podermos acessar cada `Node` na árvore diretamente. Para fazer isso,
definimos os itens do `Vec<T>` para serem valores do tipo `Rc<Node>`. Também queremos
modificar quais nós são filhos de outro nó, então temos um `RefCell<T>` em
`children` envolvendo o `Vec<Rc<Node>>`.

Em seguida, usaremos nossa definição de _struct_ e criaremos uma instância `Node` chamada
`leaf` com o valor `3` e nenhum filho, e outra instância chamada `branch`
com o valor `5` e `leaf` como um de seus filhos, conforme mostrado na Listagem 15-27.

<Listing number="15-27" file-name="src/main.rs" caption="Criando um nó `leaf` sem filhos e um nó `branch` tendo `leaf` como um de seus filhos">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Clonamos o `Rc<Node>` em `leaf` e o armazenamos em `branch`, o que significa que o
`Node` em `leaf` agora tem dois donos: `leaf` e `branch`. Podemos ir de
`branch` para `leaf` através de `branch.children`, mas não há como ir de
`leaf` para `branch`. O motivo é que `leaf` não tem referência para `branch` e
não sabe que eles estão relacionados. Queremos que `leaf` saiba que `branch` é seu
pai. Faremos isso em seguida.

#### Adicionando uma Referência de um Filho para o seu Pai

Para fazer com que o nó filho tenha conhecimento de seu pai, precisamos adicionar um campo `parent` à
definição da nossa _struct_ `Node`. O problema é decidir qual deve ser o tipo de
`parent`. Sabemos que ele não pode conter um `Rc<T>`, porque isso
criaria um ciclo de referência com `leaf.parent` apontando para `branch` e
`branch.children` apontando para `leaf`, o que faria com que seus valores de `strong_count`
nunca fossem 0.

Pensando nas relações de outra forma, um nó pai deve ser dono de seus filhos: Se um nó pai for descartado, seus nós filhos devem ser descartados
também. No entanto, um filho não deve ser dono de seu pai: Se descartarmos um nó filho, o
pai ainda deve existir. Este é um caso para referências fracas!

Portanto, em vez de `Rc<T>`, faremos o tipo de `parent` usar `Weak<T>`,
especificamente um `RefCell<Weak<Node>>`. Agora a definição da nossa _struct_ `Node` se parece
com isto:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Um nó será capaz de se referir ao seu nó pai, mas não é dono de seu pai. Na
Listagem 15-28, atualizamos o `main` para usar esta nova definição para que o nó `leaf`
tenha uma maneira de se referir ao seu pai, `branch`.

<Listing number="15-28" file-name="src/main.rs" caption="Um nó `leaf` com uma referência fraca para o seu nó pai, `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

Criar o nó `leaf` parece semelhante à Listagem 15-27, com exceção do
campo `parent`: `leaf` começa sem um pai, então criamos uma nova
instância de referência `Weak<Node>` vazia.

Neste ponto, quando tentamos obter uma referência para o pai de `leaf` usando
o método `upgrade`, obtemos um valor `None`. Vemos isso na saída do
primeiro comando `println!`:

```text
leaf parent = None
```

Quando criamos o nó `branch`, ele também terá uma nova referência `Weak<Node>`
no campo `parent` porque `branch` não tem um nó pai. Nós
ainda temos `leaf` como um dos filhos de `branch`. Assim que temos a instância `Node`
em `branch`, podemos modificar `leaf` para dar a ele uma referência `Weak<Node>`
para o seu pai. Usamos o método `borrow_mut` no `RefCell<Weak<Node>>` no
campo `parent` de `leaf`, e então usamos a função `Rc::downgrade` para
criar uma referência `Weak<Node>` para `branch` a partir do `Rc<Node>` em `branch`.

Quando imprimirmos o pai de `leaf` novamente, desta vez obteremos uma variante `Some`
contendo `branch`: Agora `leaf` pode acessar seu pai! Quando imprimimos `leaf`, nós
também evitamos o ciclo que eventualmente terminava em um estouro de pilha (*stack overflow*) como tivemos na
Listagem 15-26; as referências `Weak<Node>` são impressas como `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

A falta de saída infinita indica que este código não criou um ciclo de
referência. Também podemos notar isso olhando para os valores que obtemos ao chamar
`Rc::strong_count` e `Rc::weak_count`.

#### Visualizando Mudanças em `strong_count` e `weak_count`

Vamos ver como os valores `strong_count` e `weak_count` das instâncias `Rc<Node>`
mudam criando um novo escopo interno e movendo a criação de
`branch` para esse escopo. Ao fazer isso, podemos ver o que acontece quando `branch` é
criado e depois descartado quando ele sai de escopo. As modificações são mostradas na
Listagem 15-29.

<Listing number="15-29" file-name="src/main.rs" caption="Criando `branch` em um escopo interno e examinando as contagens de referência forte e fraca">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

Depois que `leaf` é criado, seu `Rc<Node>` tem uma contagem forte de 1 e uma contagem fraca
de 0. No escopo interno, criamos `branch` e o associamos a
`leaf`, momento em que, quando imprimimos as contagens, o `Rc<Node>` em `branch`
terá uma contagem forte de 1 e uma contagem fraca de 1 (para `leaf.parent` apontando
para `branch` com um `Weak<Node>`). Quando imprimirmos as contagens em `leaf`, veremos
que ele terá uma contagem forte de 2 porque `branch` agora tem um clone do
`Rc<Node>` de `leaf` armazenado em `branch.children`, mas ainda terá uma contagem fraca
de 0.

Quando o escopo interno termina, `branch` sai de escopo e a contagem forte do
`Rc<Node>` diminui para 0, então seu `Node` é descartado. A contagem fraca de 1
de `leaf.parent` não tem influência sobre se o `Node` é ou não descartado, então
não temos nenhum vazamento de memória!

Se tentarmos acessar o pai de `leaf` após o término do escopo, obteremos
`None` novamente. No final do programa, o `Rc<Node>` em `leaf` tem uma contagem forte
de 1 e uma contagem fraca de 0 porque a variável `leaf` é agora a única
referência ao `Rc<Node>` novamente.

Toda a lógica que gerencia as contagens e o descarte de valores é construída em
`Rc<T>` e `Weak<T>` e suas implementações da _trait_ `Drop`. Ao
especificar que a relação de um filho para seu pai deve ser uma
referência `Weak<T>` na definição de `Node`, você consegue fazer com que nós pais
apontem para nós filhos e vice-versa sem criar um ciclo de referência
e vazamentos de memória.

## Resumo

Este capítulo abordou como usar ponteiros inteligentes para fazer diferentes garantias e
compromissos (_trade-offs_) daqueles que o Rust faz por padrão com referências regulares. O
tipo `Box<T>` tem um tamanho conhecido e aponta para dados alocados no _heap_. O
tipo `Rc<T>` acompanha o número de referências a dados no _heap_ para
que os dados possam ter vários proprietários. O tipo `RefCell<T>` com sua mutabilidade
interior nos dá um tipo que podemos usar quando precisamos de um tipo imutável, mas
precisamos alterar um valor interno desse tipo; ele também impõe as regras de empréstimo
em tempo de execução em vez de em tempo de compilação.

Também foram discutidas as _traits_ `Deref` e `Drop`, que habilitam grande parte da
funcionalidade dos ponteiros inteligentes. Exploramos ciclos de referência que podem causar
vazamentos de memória e como evitá-los usando `Weak<T>`.

Se este capítulo despertou seu interesse e você deseja implementar seus próprios
ponteiros inteligentes, confira [“The Rustonomicon”][nomicon] para obter mais
informações úteis.

Em seguida, falaremos sobre concorrência em Rust. Você até aprenderá sobre alguns novos
ponteiros inteligentes.

[nomicon]: ../nomicon/index.html
