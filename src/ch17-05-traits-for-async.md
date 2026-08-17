<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Uma Olhada Mais Próxima nas Traits para Assincronicidade

Ao longo deste capítulo, usamos as traits `Future`, `Stream` e `StreamExt`
de várias maneiras. No entanto, até agora, evitamos entrar em muitos detalhes
sobre como elas funcionam ou como se encaixam, o que é perfeitamente adequado na
maioria das vezes para o seu trabalho diário em Rust. Às vezes, porém, você se
deparará com situações em que precisará entender um pouco mais sobre os detalhes
dessas traits, junto com o tipo `Pin` e a trait `Unpin`. Nesta seção, vamos nos
aprofundar o suficiente para ajudar nesses cenários, deixando o mergulho
_realmente_ profundo para outra documentação.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### A Trait `Future`

Vamos começar examinando mais de perto como a trait `Future` funciona. É assim
que o Rust a define:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Essa definição de trait inclui vários novos tipos e também alguma sintaxe que
não tínhamos visto antes, então vamos analisar a definição parte por parte.

Primeiro, o tipo associado `Output` de `Future` diz o que o futuro resolve.
Isso é análogo ao tipo associado `Item` da trait `Iterator`.
Segundo, `Future` tem o método `poll`, que recebe uma referência especial `Pin`
para seu parâmetro `self` e uma referência mutável para um tipo `Context`, e
retorna um `Poll<Self::Output>`. Falaremos mais sobre `Pin` e `Context` em
breve. Por enquanto, vamos nos concentrar no que o método retorna, o tipo `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Esse tipo `Poll` é semelhante a um `Option`. Ele possui uma variante que contém
um valor, `Ready(T)`, e uma que não contém, `Pending`. No entanto, `Poll` significa
algo bem diferente de `Option`! A variante `Pending` indica que o futuro ainda
tem trabalho a fazer, portanto, quem chamou precisará verificar novamente mais tarde.
A variante `Ready` indica que o `Future` concluiu seu trabalho e o valor `T`
está disponível.

> Nota: É raro precisar chamar `poll` diretamente, mas se precisar, tenha em
> mente que, na maioria dos futuros, quem chama não deve chamar `poll` novamente após
> o futuro ter retornado `Ready`. Muitos futuros entrarão em pânico se forem consultados
> (polled) novamente após ficarem prontos. Futuros que são seguros para serem consultados
> novamente dirão isso explicitamente em sua documentação. Isso é semelhante a como
> `Iterator::next` se comporta.

Quando você vê código que usa `await`, o Rust o compila internamente para código
que chama `poll`. Se você olhar para a Listagem 17-4, onde imprimimos o título da
página para uma única URL assim que ela foi resolvida, o Rust a compila em algo
parecido (embora não exatamente igual) com isto:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("O título para {url} era {title}"),
        None => println!("{url} não tinha título"),
    }
    Pending => {
        // Mas o que vai aqui?
    }
}
```

O que devemos fazer quando o futuro ainda está `Pending`? Precisamos de alguma forma
de tentar novamente, e novamente, e novamente, até que o futuro finalmente esteja pronto.
Em outras palavras, precisamos de um loop:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("O título para {url} era {title}"),
            None => println!("{url} não tinha título"),
        }
        Pending => {
            // continue
        }
    }
}
```

Se o Rust compilasse exatamente para esse código, no entanto, cada `await` seria
bloqueante — exatamente o oposto do que pretendíamos! Em vez disso, o Rust garante
que o loop possa ceder o controle para algo que possa pausar o trabalho neste
futuro para trabalhar em outros futuros e, em seguida, verificar este novamente mais tarde.
Como vimos, esse algo é um runtime assíncrono, e esse trabalho de agendamento e
coordenação é uma de suas principais tarefas.

Na seção [“Enviando Dados Entre Duas Tarefas Usando Mensagens”][message-passing]<!-- ignore -->,
descrevemos a espera em `rx.recv`. A chamada `recv` retorna um futuro, e o uso de `await`
no futuro faz o *poll* dele. Observamos que um runtime pausará o futuro até que ele esteja
pronto com `Some(message)` ou `None` quando o canal for fechado. Com nossa compreensão
mais profunda da trait `Future`, e especificamente de `Future::poll`, podemos ver
como isso funciona. O runtime sabe que o futuro não está pronto quando ele retorna
`Poll::Pending`. Por outro lado, o runtime sabe que o futuro _está_ pronto e o avança
quando `poll` retorna `Poll::Ready(Some(message))` ou `Poll::Ready(None)`.

Os detalhes exatos de como um runtime faz isso estão além do escopo deste livro,
mas a chave é entender a mecânica básica dos futuros: um runtime faz o *poll*
(_polls_) de cada futuro pelo qual é responsável, colocando o futuro de volta para
dormir quando ele ainda não está pronto.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### O Tipo `Pin` e a Trait `Unpin`

Na Listagem 17-13, usamos a macro `trpl::join!` para aguardar três
futuros. No entanto, é comum termos uma coleção, como um vetor, contendo
um número de futuros que não será conhecido até o tempo de execução. Vamos alterar a
Listagem 17-13 para o código da Listagem 17-23, que coloca os três futuros em um vetor
e chama a função `trpl::join_all`, que ainda não vai compilar.

<Listing number="17-23" caption="Aguardando futuros em uma coleção"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

Colocamos cada futuro dentro de um `Box` para transformá-los em _objetos de trait_
(_trait objects_), assim como fizemos na seção "Retornando Erros de `run`" no Capítulo 12.
(Abordaremos objetos de trait em detalhes no Capítulo 18.) O uso de objetos de trait
nos permite tratar cada um dos futuros anônimos produzidos por esses tipos como o mesmo
tipo, porque todos eles implementam a trait `Future`.

Isso pode ser surpreendente. Afinal, nenhum dos blocos assíncronos retorna nada,
então cada um produz um `Future<Output = ()>`. Lembre-se de que `Future` é uma
trait, no entanto, e que o compilador cria um enum exclusivo para cada bloco
assíncrono, mesmo quando eles têm tipos de saída idênticos. Assim como você não pode
colocar duas structs escritas à mão diferentes em um `Vec`, você não pode misturar
enums gerados pelo compilador.

Em seguida, passamos a coleção de futuros para a função `trpl::join_all` e
aguardamos o resultado. No entanto, isso não compila; aqui está a parte relevante
das mensagens de erro.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ a trait `Unpin` não está implementada para `dyn Future<Output = ()>`
   |
   = note: considere usar a macro `pin!`
           considere usar `Box::pin` se você precisar acessar o valor fixado (pinned) fora do escopo atual
   = note: necessário para que `Box<dyn Future<Output = ()>>` implemente `Future`
note: exigido por uma restrição (bound) em `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- exigido por uma restrição nesta struct
28 | where
29 |     F: Future,
   |        ^^^^^^ exigido por esta restrição em `JoinAll`
```

A nota nesta mensagem de erro nos diz que devemos usar a macro `pin!` para
_fixar_ (`pin`) os valores, o que significa colocá-los dentro do tipo `Pin` que
garante que os valores não serão movidos na memória. A mensagem de erro diz que
a fixação é necessária porque `dyn Future<Output = ()>` precisa implementar a trait
`Unpin` e atualmente não o faz.

A função `trpl::join_all` retorna uma struct chamada `JoinAll`. Essa struct é
genérica sobre um tipo `F`, que é restrito a implementar a trait `Future`.
Aguardar diretamente um futuro com `await` fixa o futuro implicitamente. É por isso
que não precisamos usar `pin!` em todos os lugares onde queremos aguardar futuros.

No entanto, não estamos aguardando diretamente um futuro aqui. Em vez disso, construímos um novo
futuro, JoinAll, passando uma coleção de futuros para a função `join_all`.
A assinatura de `join_all` exige que os tipos dos itens na
coleção implementem a trait `Future`, e `Box<T>` implementa `Future`
apenas se o `T` que ele envolve for um futuro que implementa a trait `Unpin`.

Isso é muita informação para absorver! Para realmente entender, vamos mergulhar um pouco mais
em como a trait `Future` realmente funciona, em particular em relação à fixação (*pinning*).
Olhe novamente para a definição da trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Método obrigatório
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

O parâmetro `cx` e seu tipo `Context` são a chave para como um runtime realmente
sabe quando verificar um determinado futuro, mantendo-se preguiçoso (*lazy*). Novamente,
os detalhes de como isso funciona estão além do escopo deste capítulo, e você geralmente
só precisa pensar nisso ao escrever uma implementação personalizada de `Future`.
Em vez disso, vamos nos concentrar no tipo para `self`, pois esta é a primeira vez
que vemos um método onde `self` tem uma anotação de tipo. Uma anotação de tipo para
`self` funciona como anotações de tipo para outros parâmetros de função, mas com duas
diferenças principais:

- Ela diz ao Rust qual deve ser o tipo de `self` para que o método seja chamado.
- Não pode ser qualquer tipo. É restrito ao tipo no qual o método é
  implementado, uma referência ou ponteiro inteligente para esse tipo, ou um `Pin` envolvendo
  uma referência a esse tipo.

Veremos mais sobre essa sintaxe no [Capítulo 18][ch-18]<!-- ignore -->. Por enquanto,
é o bastante saber que, se quisermos fazer o *poll* de um futuro para verificar se ele está
`Pending` ou `Ready(Output)`, precisamos de uma referência mutável envolvida por `Pin` para
o tipo.

`Pin` é um wrapper para tipos semelhantes a ponteiros, como `&`, `&mut`, `Box` e `Rc`.
(Tecnicamente, `Pin` funciona com tipos que implementam as traits `Deref` ou `DerefMut`,
mas isso é efetivamente equivalente a trabalhar apenas com referências e ponteiros inteligentes.)
`Pin` não é um ponteiro em si e não tem nenhum comportamento próprio como `Rc` e `Arc`
têm com contagem de referência; é puramente uma ferramenta que o compilador pode usar para impor
restrições ao uso de ponteiros.

Lembrar que `await` é implementado em termos de chamadas para `poll` começa a
explicar a mensagem de erro que vimos antes, mas isso foi em termos de `Unpin`, e não
`Pin`. Então, como exatamente `Pin` se relaciona com `Unpin`, e por que `Future` precisa
que `self` esteja em um tipo `Pin` para chamar `poll`?

Lembre-se do início deste capítulo de que uma série de pontos de espera (*await points*) em um futuro
são compilados em uma máquina de estados, e o compilador garante que essa máquina de estados
siga todas as regras normais de segurança do Rust, incluindo empréstimo (*borrowing*) e propriedade (*ownership*).
Para que isso funcione, o Rust analisa quais dados são necessários entre um
ponto de espera e o próximo ponto de espera ou o final do bloco assíncrono.
Ele então cria uma variante correspondente na máquina de estados compilada. Cada
variante obtém o acesso necessário aos dados que serão usados nessa seção
do código-fonte, seja tomando a propriedade desses dados ou obtendo uma
referência mutável ou imutável para eles.

Até aqui, tudo bem: se errarmos algo sobre a propriedade ou referências em
um determinado bloco assíncrono, o verificador de empréstimos nos avisará. Quando queremos mover
o futuro que corresponde a esse bloco — como movê-lo para um `Vec` para
passar para `join_all` — as coisas ficam mais complicadas.

Quando movemos um futuro — seja empurrando-o para uma estrutura de dados para usar como um
iterador com `join_all` ou retornando-o de uma função —, isso realmente significa
mover a máquina de estados que o Rust cria para nós. E, ao contrário da maioria dos outros tipos em
Rust, os futuros que o Rust cria para blocos assíncronos podem acabar com referências a
si mesmos nos campos de qualquer variante específica, conforme mostrado na ilustração simplificada na Figura 17-4.

<figure>

<img alt="Uma tabela de coluna única e três linhas representando um futuro, fut1, que tem valores de dados 0 e 1 nas duas primeiras linhas e uma seta apontando da terceira linha de volta para a segunda linha, representando uma referência interna dentro do futuro." src="img/trpl17-04.svg" class="center" />

<figcaption>Figura 17-4: Um tipo de dados auto-referencial</figcaption>

</figure>

Por padrão, no entanto, qualquer objeto que tenha uma referência a si mesmo é inseguro de mover (`unsafe to move`),
porque as referências sempre apontam para o endereço de memória real do que quer que
estejam se referindo (consulte a Figura 17-5). Se você mover a estrutura de dados em si, essas
referências internas continuarão apontando para o local antigo. No entanto, esse
local de memória agora é inválido. Por um lado, seu valor não será atualizado
quando você fizer alterações na estrutura de dados. Por outro — e mais importante —,
o computador agora está livre para reutilizar essa memória para outros fins! Você pode acabar
lendo dados completamente não relacionados mais tarde.

<figure>

<img alt="Duas tabelas, representando dois futuros, fut1 e fut2, cada um dos quais tem uma coluna e três linhas, representando o resultado de ter movido um futuro de fut1 para fut2. O primeiro, fut1, está esmaecido, com um ponto de interrogação em cada índice, representando memória desconhecida. O segundo, fut2, tem 0 e 1 na primeira e segunda linhas e uma seta apontando de sua terceira linha de volta para a segunda linha de fut1, representando um ponteiro que está referenciando o local antigo na memória do futuro antes de ele ser movido." src="img/trpl17-05.svg" class="center" />

<figcaption>Figura 17-5: O resultado inseguro de mover um tipo de dados auto-referencial</figcaption>

</figure>

Teoricamente, o compilador Rust poderia tentar atualizar cada referência a um
objeto sempre que ele for movido, mas isso pode adicionar muita sobrecarga de desempenho (*performance overhead*),
especialmente se toda uma rede de referências precisar ser atualizada. Se pudéssemos, em vez disso, ter certeza
de que a estrutura de dados em questão _não se move na memória_, não precisaríamos
atualizar nenhuma referência. É exatamente para isso que serve o verificador de empréstimos do Rust:
em código seguro, ele impede que você mova qualquer item com uma referência ativa a
ele.

`Pin` se baseia nisso para nos dar a garantia exata de que precisamos. Quando _fixamos_ (`pin`) um
valor envolvendo um ponteiro para esse valor em `Pin`, ele não pode mais se mover. Assim,
se você tiver `Pin<Box<SomeType>>`, você realmente fixa o valor `SomeType`, e _não_
o ponteiro `Box`. A Figura 17-6 ilustra esse processo.

<figure>

<img alt="Três caixas dispostas lado a lado. A primeira está rotulada como 'Pin', a segunda 'b1' e a terceira 'pinned'. Dentro de 'pinned' há uma tabela rotulada 'fut', com uma única coluna; ela representa um futuro com células para cada parte da estrutura de dados. Sua primeira célula tem o valor '0', sua segunda célula tem uma seta saindo dela e apontando para a quarta e última célula, que tem o valor '1' nela, e a terceira célula tem linhas tracejadas e reticências para indicar que pode haver outras partes na estrutura de dados. Juntas, a tabela 'fut' representa um futuro que é auto-referencial. Uma seta sai da caixa rotulada 'Pin', passa pela caixa rotulada 'b1' e termina dentro da caixa 'pinned' na tabela 'fut'." src="img/trpl17-06.svg" class="center" />

<figcaption>Figura 17-6: Fixando um `Box` que aponta para um tipo de futuro auto-referencial</figcaption>

</figure>

De fato, o ponteiro `Box` ainda pode se mover livremente. Lembre-se: nós nos importamos em
garantir que os dados sendo referenciados permaneçam no lugar. Se um ponteiro
se move, _mas os dados para os quais ele aponta_ estão no mesmo lugar, como na Figura
17-7, não há problema potencial. (Como exercício independente, olhe a documentação
para os tipos, bem como o módulo `std::pin` e tente descobrir como você faria
isso com um `Pin` envolvendo um `Box`.) A chave é que o próprio tipo auto-referencial
não pode se mover, porque ele ainda está fixado.

<figure>

<img alt="Quatro caixas dispostas em três colunas aproximadas, idênticas ao diagrama anterior com uma alteração na segunda coluna. Agora há duas caixas na segunda coluna, rotuladas 'b1' e 'b2', 'b1' está esmaecida, e a seta de 'Pin' passa por 'b2' em vez de 'b1', indicando que o ponteiro se moveu de 'b1' para 'b2', mas os dados em 'pinned' não se moveram." src="img/trpl17-07.svg" class="center" />

<figcaption>Figura 17-7: Movendo um `Box` que aponta para um tipo de futuro auto-referencial</figcaption>

</figure>

No entanto, a maioria dos tipos é perfeitamente segura de mover, mesmo que estejam
atrás de um ponteiro `Pin`. Só precisamos pensar em fixação (*pinning*) quando os itens têm
referências internas. Valores primitivos como números e bouleanoss são seguros
porque obviamente não têm nenhuma referência interna.
O mesmo vale para a maioria dos tipos com os quais você normalmente trabalha em Rust. Você pode mover
um `Vec`, por exemplo, sem se preocupar. Dado o que vimos até agora, se
você tiver um `Pin<Vec<String>>`, você teria que fazer tudo através das APIs seguras, mas
restritivas, fornecidas por `Pin`, mesmo que um `Vec<String>` seja sempre seguro
de mover se não houver outras referências a ele. Precisamos de uma maneira de dizer ao
compilador que não há problema em mover itens em casos como este — e é aí
que o `Unpin` entra em jogo.

`Unpin` é uma trait de marcação (*marker trait*), semelhante às traits `Send` e `Sync` que vimos no
Capítulo 16, e, portanto, não possui funcionalidade própria. Traits de marcação existem apenas
para dizer ao compilador que é seguro usar o tipo que implementa uma determinada trait em um
contexto específico. `Unpin` informa ao compilador que um determinado tipo _não_
precisa cumprir nenhuma garantia sobre se o valor em questão pode ser movido com segurança.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

Assim como com `Send` e `Sync`, o compilador implementa `Unpin` automaticamente
para todos os tipos onde ele pode provar que é seguro. Um caso especial, novamente semelhante a
`Send` e `Sync`, é quando `Unpin` _não_ é implementado para um tipo. A
notação para isso é <code>impl !Unpin for <em>SomeType</em></code>, onde
<code><em>SomeType</em></code> é o nome de um tipo que _precisa_ cumprir
essas garantias para ser seguro sempre que um ponteiro para esse tipo for usado em um `Pin`.

Em outras palavras, há duas coisas a ter em mente sobre a relação
entre `Pin` e `Unpin`. Primeiro, `Unpin` é o caso "normal", e `!Unpin` é o
caso especial. Segundo, se um tipo implementa `Unpin` ou `!Unpin` _só_
importa quando você está usando um ponteiro fixado para esse tipo, como <code>Pin<&mut
<em>SomeType</em>></code>.

Para tornar isso concreto, pense em uma `String`: ela tem um comprimento e os caracteres
Unicode que a compõem. Podemos envolver uma `String` em `Pin`, como visto na Figura
17-8. No entanto, `String` implementa automaticamente `Unpin`, assim como a maioria dos outros tipos
em Rust.

<figure>

<img alt="Uma caixa rotulada 'Pin' à esquerda com uma seta indo dela para uma caixa rotulada 'String' à direita. A caixa 'String' contém os dados 5usize, representando o comprimento da string, e as letras 'h', 'e', 'l', 'l', e 'o' representando os caracteres da string 'hello' armazenados nesta instância de String. Um retângulo pontilhado envolve a caixa 'String' e seu rótulo, mas não a caixa 'Pin'." src="img/trpl17-08.svg" class="center" />

<figcaption>Figura 17-8: Fixando uma `String`; a linha pontilhada indica que a `String` implementa a trait `Unpin` e, portanto, não está fixada</figcaption>

</figure>

Como resultado, podemos fazer coisas que seriam ilegais se `String` implementasse
`!Unpin` em vez disso, como substituir uma string por outra no exato mesmo
local na memória, como na Figura 17-9. Isso não viola o contrato de `Pin`,
porque `String` não tem referências internas que tornem inseguro movê-la.
Essa é precisamente a razão pela qual ela implementa `Unpin` em vez de `!Unpin`.

<figure>

<img alt="Os mesmos dados da string 'hello' do exemplo anterior, agora rotulados como 's1' e esmaecidos. A caixa 'Pin' do exemplo anterior agora aponta para uma instância de String diferente, uma que está rotulada 's2', é válida, tem um comprimento de 7usize e contém os caracteres da string 'goodbye'. s2 é cercada por um retângulo pontilhado porque ela também implementa a trait Unpin." src="img/trpl17-09.svg" class="center" />

<figcaption>Figura 17-9: Substituindo a `String` por uma `String` totalmente diferente na memória</figcaption>

</figure>

Agora sabemos o suficiente para entender os erros relatados para aquela chamada de `join_all`
da Listagem 17-23. Originalmente, tentamos mover os futuros produzidos por
blocos assíncronos para um `Vec<Box<dyn Future<Output = ()>>>`, mas como vimos,
esses futuros podem ter referências internas, então eles não implementam automaticamente
`Unpin`. Uma vez que os fixamos, podemos passar o tipo `Pin` resultante para
o `Vec`, confiantes de que os dados subjacentes nos futuros _não_ serão
movidos. A Listagem 17-24 mostra como corrigir o código chamando a macro `pin!`
onde cada um dos três futuros é definido e ajustando o tipo de objeto de trait.

<Listing number="17-24" caption="Fixando os futuros para permitir movê-los para o vetor">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Este exemplo agora compila e é executado, e poderíamos adicionar ou remover futuros do
vetor em tempo de execução e agrupá-los todos (`join`).

`Pin` e `Unpin` são importantes principalmente para a construção de bibliotecas de nível inferior, ou
quando você está construindo um runtime em si, em vez do código Rust do dia a dia.
Quando você vir essas traits em mensagens de erro, no entanto, agora você terá uma ideia
melhor de como corrigir seu código!

> Nota: Esta combinação de `Pin` e `Unpin` torna possível implementar com segurança
> toda uma classe de tipos complexos em Rust que de outra forma seriam
> desafiadores porque são auto-referenciais. Tipos que exigem `Pin` aparecem
> mais comumente no Rust assíncrono hoje, mas de vez em quando, você pode vê-los
> em outros contextos também.
>
> As especificidades de como `Pin` e `Unpin` funcionam e as regras que eles precisam
> cumprir são amplamente abordadas na documentação da API para `std::pin`, então
> se você tiver interesse em aprender mais, esse é um ótimo ponto de partida.
>
> Se você quiser entender como as coisas funcionam por baixo dos panos com ainda mais detalhes,
> consulte os Capítulos [2][under-the-hood]<!-- ignore --> e
> [4][pinning]<!-- ignore --> de
> [_Asynchronous Programming in Rust_][async-book].

### A Trait `Stream`

Agora que você tem uma compreensão mais profunda das traits `Future`, `Pin` e `Unpin`, podemos
voltar nossa atenção para a trait `Stream`. Como você aprendeu anteriormente no
capítulo, streams são semelhantes a iteradores assíncronos. Ao contrário de `Iterator` e
`Future`, no entanto, `Stream` não tem definição na biblioteca padrão até
o momento desta escrita, mas _existe_ uma definição muito comum da crate `futures`
usada em todo o ecossistema.

Vamos revisar as definições das traits `Iterator` e `Future` antes
de analisar como uma trait `Stream` pode uní-las. De `Iterator`, temos
a ideia de uma sequência: seu método `next` fornece um
`Option<Self::Item>`. De `Future`, temos a ideia de prontidão ao longo do tempo:
seu método `poll` fornece um `Poll<Self::Output>`. Para representar uma sequência de
itens que se tornam prontos ao longo do tempo, definimos uma trait `Stream` que junta esses
recursos:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

A trait `Stream` define um tipo associado chamado `Item` para o tipo dos
itens produzidos pelo stream. Isso é semelhante a `Iterator`, onde pode haver
de zero a muitos itens, e diferente de `Future`, onde há sempre um único
`Output`, mesmo que seja o tipo unitário `()`.

`Stream` também define um método para obter esses itens. Nós o chamamos de `poll_next`, para
deixar claro que ele faz o *poll* da mesma forma que `Future::poll` faz e produz uma
sequência de itens da mesma forma que `Iterator::next` faz. Seu tipo de retorno combina
`Poll` com `Option`. O tipo externo é `Poll`, porque ele precisa ser verificado
quanto à prontidão, assim como um futuro. O tipo interno é `Option`,
porque ele precisa sinalizar se há mais mensagens, assim como um iterador
faz.

Algo muito semelhante a esta definição provavelmente acabará fazendo parte da biblioteca padrão do Rust.
Enquanto isso, faz parte do kit de ferramentas da maioria dos runtimes,
então você pode confiar nisso, e tudo o que cobriremos a seguir deve se aplicar de forma geral!

Nos exemplos que vimos na seção [“Streams: Futuros em Sequência”][streams]<!--
ignore -->, no entanto, não usamos `poll_next` _nem_ `Stream`, mas
usamos `next` e `StreamExt`. Nós _poderíamos_ trabalhar diretamente em termos da
API `poll_next` escrevendo manualmente nossas próprias máquinas de estados `Stream`, é claro,
assim como _poderíamos_ trabalhar com futuros diretamente através de seu método `poll`.
O uso de `await` é muito mais agradável, no entanto, e a trait `StreamExt` fornece o método
`next` para que possamos fazer exatamente isso:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Nota: A definição real que usamos anteriormente no capítulo parece ligeramente
> diferente desta, porque ela suporta versões do Rust que ainda não
> suportavam o uso de funções assíncronas em traits. Como resultado, ela se parece com isto:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Esse tipo `Next` é uma `struct` que implementa `Future` e nos permite nomear
> o tempo de vida (*lifetime*) da referência a `self` com `Next<'_, Self>`, para que o `await`
> possa funcionar com este método.

A trait `StreamExt` também é o lar de todos os métodos interessantes disponíveis
para uso com streams. `StreamExt` é implementada automaticamente para cada tipo
que implementa `Stream`, mas essas traits são definidas separadamente para permitir que
a comunidade crie iterações em APIs de conveniência sem afetar a trait fundamental.

Na versão de `StreamExt` usada na crate `trpl`, a trait não apenas
define o método `next`, mas também fornece uma implementação padrão de `next`
que lida corretamente com os detalhes de chamar `Stream::poll_next`. Isso significa
que mesmo quando você precisa escrever seu próprio tipo de dados de stream, você _só_
precisa implementar `Stream`, e então qualquer pessoa que use seu tipo de dados pode usar
`StreamExt` e seus métodos com ele automaticamente.

Isso é tudo o que vamos cobrir sobre os detalhes de nível inferior dessas traits. Para
concluir, vamos considerar como os futuros (incluindo streams), tarefas e threads se
encaixam!

[message-passing]: ch17-02-concurrency-with-async.md#enviando-dados-entre-duas-tarefas-usando-mensagens
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#nosso-primeiro-programa-assíncrono
[any-number-futures]: ch17-03-more-futures.html#trabalhando-com-qualquer-número-de-futuros
[streams]: ch17-04-streams.html
