## Implementando um Padrão de Design Orientado a Objetos

O _padrão state_ (estado) é um padrão de design orientado a objetos. O cerne do
padrão é definirmos um conjunto de estados que um valor pode ter internamente.
Os estados são representados por um conjunto de _objetos de estado_ (state
objects), e o comportamento do valor muda com base no seu estado. Vamos
trabalhar em um exemplo de uma struct de post de blog que possui um campo para
reter seu estado, que será um objeto de estado do conjunto “rascunho” (draft),
“revisão” (review) ou “publicado” (published).

Os objetos de estado compartilham funcionalidades: em Rust, é claro, usamos
structs e traits em vez de objetos e herança. Cada objeto de estado é
responsável por seu próprio comportamento e por governar quando ele deve mudar
para outro estado. O valor que armazena um objeto de estado não sabe nada sobre
o comportamento diferente dos estados ou sobre quando fazer a transição entre os
estados.

A vantagem de usar o padrão state é que, quando os requisitos de negócio do
programa mudam, não precisaremos alterar o código do valor que detém o estado
ou o código que usa o valor. Precisaremos apenas atualizar o código dentro de
um dos objetos de estado para alterar suas regras ou talvez adicionar mais
objetos de estado.

Primeiro, vamos implementar o padrão state de uma maneira mais tradicional e
orientada a objetos. Depois, usaremos uma abordagem que é um pouco mais natural
em Rust. Vamos nos aprofundar na implementação incremental de um fluxo de
trabalho de post de blog usando o padrão state.

A funcionalidade final será parecida com isto:

1. Um post de blog começa como um rascunho vazio.
1. Quando o rascunho é concluído, uma revisão do post é solicitada.
1. Quando o post é aprovado, ele é publicado.
1. Apenas posts de blog publicados retornam conteúdo para impressão, para que
   posts não aprovados não possam ser publicados acidentalmente.

Quaisquer outras alterações tentadas em um post não devem ter efeito. Por
exemplo, se tentarmos aprovar um post de blog em rascunho antes de solicitarmos
uma revisão, o post deve continuar sendo um rascunho não publicado.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### Tentando o Estilo Orientado a Objetos Tradicional

Há infinitas maneiras de estruturar código para resolver o mesmo problema, cada
uma com diferentes compensações (_trade-offs_). A implementação desta seção é
mais próxima de um estilo tradicional orientado a objetos, que é possível de ser
escrito em Rust, mas não tira proveito de alguns dos pontos fortes de Rust.
Mais tarde, demonstraremos uma solução diferente que ainda usa o padrão de
design orientado a objetos, mas está estruturada de uma forma que pode parecer
menos familiar para programadores com experiência em orientação a objetos.
Vamos comparar as duas soluções para vivenciar as compensações de projetar
código em Rust de maneira diferente do código em outras linguagens.

A Listagem 18-11 mostra este fluxo de trabalho em forma de código: Este é um
exemplo de uso da API que implementaremos em uma biblioteca crate chamada
`blog`. Isso ainda não vai compilar porque ainda não implementamos a crate
`blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Código que demonstra o comportamento desejado que queremos que nossa crate `blog` tenha">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Queremos permitir que o usuário crie um novo post de blog em rascunho com
`Post::new`. Queremos permitir que texto seja adicionado ao post do blog. Se
tentarmos obter o conteúdo do post imediatamente, antes da aprovação, não
devemos receber nenhum texto porque o post ainda é um rascunho. Adicionamos
`assert_eq!` no código para fins de demonstração. Um excelente teste unitário
para isso seria afirmar que um post de blog em rascunho retorna uma string vazia
a partir do método `content`, mas não vamos escrever testes para este exemplo.

Em seguida, queremos permitir uma solicitação de revisão do post, e queremos
que `content` retorne uma string vazia enquanto aguarda a revisão. Quando o
post receber aprovação, ele deve ser publicado, o que significa que o texto do
post será retornado quando `content` for chamado.

Note que o único tipo com o qual estamos interagindo na crate é o tipo `Post`.
Este tipo usará o padrão state e conterá um valor que será um entre três
objetos de estado representando os vários estados em que um post pode estar —
rascunho (`draft`), revisão (`review`) ou publicado (`published`). A mudança de
um estado para outro será gerenciada internamente dentro do tipo `Post`. Os
estados mudam em resposta aos métodos chamados pelos usuários da nossa
biblioteca na instância de `Post`, mas eles não precisam gerenciar as mudanças
de estado diretamente. Além disso, os usuários não podem cometer erros com os
estados, como publicar um post antes que ele seja revisado.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### Definindo `Post` e Criando uma Nova Instância

Vamos começar a implementação da biblioteca! Sabemos que precisamos de uma
struct pública `Post` que armazene algum conteúdo, então começaremos com a
definição da struct e uma função pública associada `new` para criar uma
instância de `Post`, como mostrado na Listagem 18-12. Também criaremos uma trait
privada `State` que definirá o comportamento que todos os objetos de estado para
um `Post` devem ter.

Então, `Post` conterá um trait object do tipo `Box<dyn State>` dentro de um
`Option<T>` em um campo privado chamado `state` para armazenar o objeto de
estado. Você verá por que o `Option<T>` é necessário daqui a pouco.

<Listing number="18-12" file-name="src/lib.rs" caption="Definição de uma struct `Post` e uma função `new` que cria uma nova instância de `Post`, de uma trait `State` e de uma struct `Draft`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

A trait `State` define o comportamento compartilhado por diferentes estados de
post. Os objetos de estado são `Draft`, `PendingReview` e `Published`, e todos
eles implementarão a trait `State`. Por enquanto, a trait não possui nenhum
método, e começaremos definindo apenas o estado `Draft`, porque esse é o estado
em que queremos que um post comece.

Quando criamos um novo `Post`, definimos seu campo `state` como um valor `Some`
que contém um `Box`. Este `Box` aponta para uma nova instância da struct
`Draft`. Isso garante que, sempre que criarmos uma nova instância de `Post`,
ela começará como um rascunho. Como o campo `state` de `Post` é privado, não há
maneira de criar um `Post` em nenhum outro estado! Na função `Post::new`,
definimos o campo `content` como uma nova `String` vazia.

#### Armazenando o Texto do Conteúdo do Post

Vimos na Listagem 18-11 que queremos poder chamar um método chamado `add_text` e
passar a ele um `&str` que é então adicionado como o conteúdo de texto do post
do blog. Implementamos isso como um método, em vez de expor o campo `content`
como `pub`, para que depois possamos implementar um método que controlará como
os dados do campo `content` são lidos. O método `add_text` é bastante direto,
então vamos adicionar a implementação da Listagem 18-13 ao bloco `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Implementando o método `add_text` para adicionar texto ao `content` de um post">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

O método `add_text` aceita uma referência mutável para `self` porque estamos
alterando a instância de `Post` na qual estamos chamando `add_text`. Em
seguida, chamamos `push_str` na `String` em `content` e passamos o argumento
`text` para adicionar ao `content` salvo. Esse comportamento não depende do
estado em que o post se encontra, portanto, não faz parte do padrão state. O
método `add_text` não interage com o campo `state` de forma alguma, mas faz
parte do comportamento que queremos suportar.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### Garantindo que o Conteúdo de um Post em Rascunho Esteja Vazio

Mesmo depois de termos chamado `add_text` e adicionado algum conteúdo ao nosso
post, ainda queremos que o método `content` retorne uma fatia de string vazia
porque o post ainda está no estado de rascunho, como demonstrado pelo primeiro
`assert_eq!` na Listagem 18-11. Por enquanto, vamos implementar o método
`content` com a coisa mais simples que atenderá a esse requisito: sempre
retornando uma fatia de string vazia. Mudaremos isso mais tarde, assim que
implementarmos a capacidade de alterar o estado de um post para que ele possa
ser publicado. Até agora, os posts só podem estar no estado de rascunho,
portanto, o conteúdo do post deve estar sempre vazio. A Listagem 18-14 mostra
esta implementação de espaço reservado (_placeholder_).

<Listing number="18-14" file-name="src/lib.rs" caption="Adicionando uma implementação de espaço reservado para o método `content` em `Post` que sempre retorna uma fatia de string vazia">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Com este método `content` adicionado, tudo na Listagem 18-11 até o primeiro
`assert_eq!` funciona como pretendido.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### Solicitando uma Revisão, o que Altera o Estado do Post

Em seguida, precisamos adicionar funcionalidade para solicitar uma revisão de um
post, o que deve alterar seu estado de `Draft` para `PendingReview`. A Listagem
18-15 mostra este código.

<Listing number="18-15" file-name="src/lib.rs" caption="Implementando os métodos `request_review` em `Post` e na trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Damos ao `Post` um método público chamado `request_review` que aceitará uma
referência mutável para `self`. Em seguida, chamamos um método interno
`request_review` no estado atual de `Post`, e este segundo `request_review`
consome o estado atual e retorna um novo estado.

Adicionamos o método `request_review` à trait `State`; todos os tipos que
implementam a trait agora precisarão implementar o método `request_review`.
Note que, em vez de ter `self`, `&self` ou `&mut self` como o primeiro parâmetro
do método, temos `self: Box<Self>`. Esta sintaxe significa que o método só é
válido quando chamado em um `Box` que contém o tipo. Esta sintaxe assume a
propriedade (`ownership`) de `Box<Self>`, invalidando o estado antigo para que
o valor de estado do `Post` possa se transformar em um novo estado.

Para consumir o estado antigo, o método `request_review` precisa assumir a
propriedade do valor de estado. É aqui que o `Option` no campo `state` de `Post`
entra em ação: Chamamos o método `take` para retirar o valor `Some` do campo
`state` e deixar um `None` em seu lugar, porque Rust não nos permite ter campos
não preenchidos em structs. Isso nos permite mover o valor `state` para fora de
`Post` em vez de emprestá-lo. Em seguida, definiremos o valor `state` do post
como o resultado dessa operação.

Precisamos definir `state` como `None` temporariamente, em vez de defini-lo
diretamente com um código como `self.state = self.state.request_review();`,
para obter a propriedade do valor de `state`. Isso garante que o `Post` não
possa usar o valor antigo de `state` depois que o transformamos em um novo
estado.

O método `request_review` em `Draft` retorna uma nova instância empacotada
(_boxed_) de uma nova struct `PendingReview`, que representa o estado quando um
post está aguardando uma revisão. A struct `PendingReview` também implementa o
método `request_review`, mas não faz nenhuma transformação. Em vez disso, ela
retorna a si mesma, pois quando solicitamos uma revisão em um post que já está
no estado `PendingReview`, ele deve permanecer no estado `PendingReview`.

Agora podemos começar a ver as vantagens do padrão state: O método
`request_review` em `Post` é o mesmo, não importa o valor do seu `state`. Cada
estado é responsável pelas suas próprias regras.

Deixaremos o método `content` em `Post` como está, retornando uma fatia de
string vazia. Agora podemos ter um `Post` no estado `PendingReview` bem como no
estado `Draft`, mas queremos o mesmo comportamento no estado `PendingReview`. A
Listagem 18-11 agora funciona até a segunda chamada de `assert_eq!`!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### Adicionando `approve` para Alterar o Comportamento de `content`

O método `approve` será semelhante ao método `request_review`: Ele definirá
`state` para o valor que o estado atual diz que ele deve ter quando esse
estado for aprovado, conforme mostrado na Listagem 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="Implementando o método `approve` em `Post` e na trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Adicionamos o método `approve` à trait `State` e adicionamos uma nova struct que
implementa `State`, o estado `Published`.

De maneira semelhante à forma como `request_review` em `PendingReview`
funciona, se chamarmos o método `approve` em um `Draft`, ele não terá efeito
porque `approve` retornará `self`. Quando chamamos `approve` em
`PendingReview`, ele retorna uma nova instância empacotada da struct
`Published`. A struct `Published` implementa a trait `State`, e tanto para o
método `request_review` quanto para o método `approve`, ela retorna a si mesma,
porque o post deve permanecer no estado `Published` nesses casos.

Agora precisamos atualizar o método `content` em `Post`. Queremos que o valor
retornado de `content` dependa do estado atual do `Post`, então faremos com que o
`Post` delegue para um método `content` definido em seu `state`, como mostrado
na Listagem 18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="Atualizando o método `content` em `Post` para delegar para um método `content` em `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Como o objetivo é manter todas essas regras dentro das structs que implementam
`State`, chamamos um método `content` no valor em `state` e passamos a instância
do post (isto é, `self`) como um argumento. Em seguida, retornamos o valor que é
retornado ao usar o método `content` no valor de `state`.

Chamamos o método `as_ref` no `Option` porque queremos uma referência para o
valor dentro do `Option` em vez da propriedade do valor. Como `state` é um
`Option<Box<dyn State>>`, quando chamamos `as_ref`, um `Option<&Box<dyn
State>>` é retornado. Se não chamássemos `as_ref`, obteríamos um erro porque não
podemos mover `state` para fora do empréstimo `&self` do parâmetro da função.

Em seguida, chamamos o método `unwrap`, que sabemos que nunca vai entrar em
pânico (_panic_) porque sabemos que os métodos em `Post` garantem que `state`
sempre conterá um valor `Some` quando esses métodos terminarem. Este é um dos
casos de que falamos na seção [“Quando Você Tem Mais Informações do que o
Compilador”][more-info-than-rustc]<!-- ignore --> do Capítulo 9, quando sabemos
que um valor `None` nunca é possível, mesmo que o compilador não seja capaz de
entender isso.

Neste ponto, quando chamamos `content` no `&Box<dyn State>`, a coerção de
desreferenciamento (_deref coercion_) entrará em vigor no `&` e no `Box` para
que o método `content` seja finalmente chamado no tipo que implementa a trait
`State`. Isso significa que precisamos adicionar `content` à definição da trait
`State`, e é aí que colocaremos a lógica de qual conteúdo retornar dependendo
de qual estado temos, como mostrado na Listagem 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="Adicionando o método `content` à trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Adicionamos uma implementação padrão para o método `content` que retorna uma
fatia de string vazia. Isso significa que não precisamos implementar `content`
nas structs `Draft` e `PendingReview`. A struct `Published` substituirá
(_override_) o método `content` e retornará o valor em `post.content`. Embora
seja conveniente, fazer com que o método `content` em `State` determine o
conteúdo do `Post` borra as linhas entre a responsabilidade de `State` e a
responsabilidade de `Post`.

Note que precisamos de anotações de tempo de vida (_lifetime annotations_)
neste método, conforme discutimos no Capítulo 10. Estamos aceitando uma
referência a um `post` como argumento e retornando uma referência a uma parte
desse `post`, de modo que o tempo de vida da referência retornada está
relacionado ao tempo de vida do argumento `post`.

E terminamos — tudo na Listagem 18-11 agora funciona! Implementamos o padrão
state com as regras do fluxo de trabalho do post de blog. A lógica relacionada
às regras reside nos objetos de estado, em vez de estar espalhada por todo o
`Post`.

> ### Por Que Não um Enum?
>
> Você pode estar se perguntando por que não usamos um enum com os diferentes
> estados possíveis de post como variantes. Essa é certamente uma solução
> possível; experimente e compare os resultados finais para ver qual você
> prefere! Uma desvantagem de usar um enum é que cada lugar que verifica o
> valor do enum precisará de uma expressão `match` ou similar para lidar com
> cada variante possível. Isso pode se tornar mais repetitivo do que esta
> solução com trait object.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### Avaliando o Padrão State

Mostramos que Rust é capaz de implementar o padrão state orientado a objetos
para encapsular os diferentes tipos de comportamento que um post deve ter em
cada estado. Os métodos em `Post` não sabem nada sobre os vários
comportamentos. Por causa da maneira como organizamos o código, precisamos
olhar em apenas um lugar para conhecer as diferentes maneiras pelas quais um
post publicado pode se comportar: a implementação da trait `State` na struct
`Published`.

Se fôssemos criar uma implementação alternativa que não usasse o padrão state,
poderíamos usar em vez disso expressões `match` nos métodos em `Post` ou mesmo
no código do `main` que verifica o estado do post e altera o comportamento nesses
locais. Isso significaria que teríamos que procurar em vários lugares para
entender todas as implicações de um post estar no estado publicado.

Com o padrão state, os métodos de `Post` e os locais onde usamos `Post` não
precisam de expressões `match`, e para adicionar um novo estado, precisaríamos
apenas adicionar uma nova struct e implementar os métodos da trait nessa única
struct em um único local.

A implementação usando o padrão state é fácil de estender para adicionar mais
funcionalidades. Para ver a simplicidade de manter código que usa o padrão
state, tente algumas destas sugestões:

- Adicione um método `reject` que altera o estado do post de `PendingReview`
  de volta para `Draft`.
- Exija duas chamadas para `approve` antes que o estado possa ser alterado para
  `Published`.
- Permita que os usuários adicionem conteúdo de texto apenas quando um post
  estiver no estado `Draft`. Dica: faça com que o objeto de estado seja
  responsável pelo que pode mudar no conteúdo, mas não responsável por
  modificar o `Post`.

Uma desvantagem do padrão state é que, como os estados implementam as
transições entre os estados, alguns dos estados são acoplados uns aos outros. Se
adicionarmos outro estado entre `PendingReview` e `Published`, como `Scheduled`,
teríamos que alterar o código em `PendingReview` para fazer a transição para
`Scheduled` em vez disso. Seria menos trabalho se `PendingReview` não precisasse
mudar com a adição de um novo estado, mas isso significaria mudar para outro
padrão de design.

Outra desvantagem é que duplicamos parte da lógica. Para eliminar parte da
duplicação, poderíamos tentar criar implementações padrão para os métodos
`request_review` e `approve` na trait `State` que retornam `self`. No entanto,
isso não funcionaria: ao usar `State` como um trait object, a trait não sabe
exatamente qual será o `self` concreto, portanto o tipo de retorno não é conhecido
em tempo de compilação. (Esta é uma das regras de compatibilidade com dyn
mencionadas anteriormente.)

Outra duplicação inclui as implementações semelhantes dos métodos
`request_review` e `approve` em `Post`. Ambos os métodos usam `Option::take` com
o campo `state` de `Post`, e se `state` for `Some`, eles delegam para a
implementação do mesmo método no valor empacotado e definem o novo valor do
campo `state` para o resultado. Se tivéssemos muitos métodos em `Post` que
seguissem esse padrão, poderíamos considerar definir uma macro para eliminar a
repetição (veja a seção [“Macros”][macros]<!-- ignore --> no Capítulo 20).

Ao implementar o padrão state exatamente como ele é definido para linguagens
orientadas a objetos, não estamos tirando o máximo proveito dos pontos fortes de
Rust quanto poderíamos. Vamos analisar algumas alterações que podemos fazer na
crate `blog` que podem tornar estados e transições inválidos em erros de tempo
de compilação.

### Codificando Estados e Comportamento como Tipos

Mostraremos como repensar o padrão state para obter um conjunto diferente de
compensações. Em vez de encapsular os estados e as transições completamente para
que o código externo não tenha conhecimento deles, codificaremos os estados em
diferentes tipos. Consequentemente, o sistema de verificação de tipos de Rust
impedirá tentativas de usar posts em rascunho onde apenas posts publicados são
permitidos, emitindo um erro de compilação.

Vamos considerar a primeira parte do `main` na Listagem 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Ainda permitimos a criação de novos posts no estado de rascunho usando
`Post::new` e a capacidade de adicionar texto ao conteúdo do post. Mas, em vez de
ter um método `content` em um post em rascunho que retorna uma string vazia,
faremos com que os posts em rascunho não tenham o método `content` de todo. Dessa
forma, se tentarmos obter o conteúdo de um post em rascunho, receberemos um erro
de compilação informando que o método não existe. Como resultado, será
impossível para nós exibir acidentalmente o conteúdo de um post em rascunho em
produção, porque esse código nem sequer vai compilar. A Listagem 18-19 mostra a
definição de uma struct `Post` e de uma struct `DraftPost`, bem como métodos em
cada uma.

<Listing number="18-19" file-name="src/lib.rs" caption="Um `Post` com um método `content` e um `DraftPost` sem um método `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Tanto a struct `Post` quanto a `DraftPost` têm um campo privado `content` que
armazena o texto do post do blog. As structs não têm mais o campo `state` porque
estamos movendo a codificação do estado para os tipos das structs. A struct
`Post` representará um post publicado, e ela tem um método `content` que
retorna o `content`.

Ainda temos a função `Post::new`, mas em vez de retornar uma instância de
`Post`, ela retorna uma instância de `DraftPost`. Como `content` é privado e
não há funções que retornem `Post`, não é possível criar uma instância de `Post`
no momento.

A struct `DraftPost` tem um método `add_text`, para que possamos adicionar texto
ao `content` como antes, mas note que `DraftPost` não tem um método `content`
definido! Portanto, agora o programa garante que todos os posts comecem como
posts em rascunho, e os posts em rascunho não têm seu conteúdo disponível para
exibição. Qualquer tentativa de contornar essas restrições resultará em um erro
de compilação.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

Então, como obtemos um post publicado? Queremos impor a regra de que um post em
rascunho deve ser revisado e aprovado antes de poder ser publicado. Um post no
estado de revisão pendente ainda não deve exibir nenhum conteúdo. Vamos
implementar essas restrições adicionando outra struct, `PendingReviewPost`,
definindo o método `request_review` em `DraftPost` para retornar um
`PendingReviewPost` e definindo um método `approve` em `PendingReviewPost` para
retornar um `Post`, conforme mostrado na Listagem 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Um `PendingReviewPost` que é criado chamando `request_review` em `DraftPost` e um método `approve` que transforma um `PendingReviewPost` em um `Post` publicado">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Os métodos `request_review` e `approve` assumem a propriedade de `self`,
consumindo assim as instâncias de `DraftPost` e `PendingReviewPost` e
transformando-as em um `PendingReviewPost` e um `Post` publicado,
respectivamente. Dessa forma, não teremos nenhuma instância persistente de
`DraftPost` depois de chamarmos `request_review` nelas, e assim por diante. A
struct `PendingReviewPost` não tem um método `content` definido nela, portanto,
tentar ler seu conteúdo resulta em um erro de compilação, assim como com
`DraftPost`. Como a única maneira de obter uma instância de `Post` publicada
que tenha um método `content` definido é chamar o método `approve` em um
`PendingReviewPost`, e a única maneira de obter um `PendingReviewPost` é chamar
o método `request_review` em um `DraftPost`, agora codificamos o fluxo de
trabalho do post de blog no sistema de tipos.

Mas também precisamos fazer algumas pequenas alterações em `main`. Os métodos
`request_review` e `approve` retornam novas instâncias em vez de modificar a
struct na qual são chamados, então precisamos adicionar mais atribuições de
sombreado (_shadowing_) `let post =` para salvar as instâncias retornadas.
Também não podemos ter as asserções sobre os conteúdos dos posts em rascunho e
em revisão pendente como strings vazias, nem precisamos delas: não podemos
compilar código que tenta usar o conteúdo de posts nesses estados nunca mais. O
código atualizado em `main` é mostrado na Listagem 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Modificações em `main` para usar a nova implementação do fluxo de trabalho do post de blog">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

As alterações que precisamos fazer em `main` para reatribuir `post` significam
que esta implementação não segue mais exatamente o padrão state orientado a
objetos: as transformações entre os estados não estão mais totalmente
encapsuladas dentro da implementação de `Post`. No entanto, nosso ganho é que
estados inválidos agora são impossíveis devido ao sistema de tipos e à
verificação de tipos que acontece em tempo de compilação! Isso garante que
certos bugs, como a exibição do conteúdo de um post não publicado, sejam
descobertos antes de chegarem à produção.

Experimente as tarefas sugeridas no início desta seção na crate `blog` tal como
ela está após a Listagem 18-21 para ver o que você acha do design desta versão
do código. Note que algumas das tarefas podem já estar concluídas neste design.

Vimos que, embora Rust seja capaz de implementar padrões de design orientados a
objetos, outros padrões, como a codificação de estado no sistema de tipos,
também estão disponíveis em Rust. Esses padrões têm diferentes compensações.
Embora você possa estar muito familiarizado com padrões orientados a objetos,
repensar o problema para tirar proveito dos recursos de Rust pode trazer
benefícios, como a prevenção de alguns bugs em tempo de compilação. Padrões
orientados a objetos nem sempre serão a melhor solução em Rust devido a certos
recursos, como a propriedade, que as linguagens orientadas a objetos não têm.

## Resumo

Independentemente de você achar que Rust é uma linguagem orientada a objetos
após ler este capítulo, agora você sabe que pode usar trait objects para obter
alguns recursos orientados a objetos em Rust. O despacho dinâmico (_dynamic
dispatch_) pode dar ao seu código alguma flexibilidade em troca de um pouco de
desempenho em tempo de execução. Você pode usar essa flexibilidade para
implementar padrões orientados a objetos que podem ajudar na manutenibilidade
do seu código. Rust também possui outros recursos, como propriedade, que as
linguagens orientadas a objetos não possuem. Um padrão orientado a objetos nem
sempre será a melhor maneira de aproveitar os pontos fortes de Rust, mas é uma
opção disponível.

Em seguida, veremos padrões, que são outro recurso de Rust que permite muita
flexibilidade. Nós os analisamos brevemente ao longo do livro, mas ainda não
vimos toda a sua capacidade. Vamos lá!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
