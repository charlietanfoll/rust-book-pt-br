## `RefCell<T>` e o Padrão de Mutabilidade Interior

_Mutabilidade interior_ (interior mutability) é um padrão de design em Rust que
permite mutar dados mesmo quando existem referências imutáveis a esses dados;
normalmente, essa ação é proibida pelas regras de empréstimo (borrowing). Para
mutar dados, o padrão usa código `unsafe` dentro de uma estrutura de dados para
contornar as regras usuais do Rust que governam a mutação e o empréstimo. O
código unsafe indica ao compilador que estamos verificando as regras
manualmente em vez de confiar no compilador para verificá-las por nós; nós
discutiremos código unsafe mais a fundo no Capítulo 20.

Podemos usar tipos que utilizam o padrão de mutabilidade interior apenas quando
podemos garantir que as regras de empréstimo serão seguidas em tempo de
execução, mesmo que o compilador não possa garantir isso. O código `unsafe`
envolvido é então encapsulado em uma API segura, e o tipo externo continua
sendo imutável.

Vamos explorar esse conceito olhando para o tipo `RefCell<T>`, que segue o
padrão de mutabilidade interior.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### Aplicando Regras de Empréstimo em Tempo de Execução

Diferente do `Rc<T>`, o tipo `RefCell<T>` representa propriedade única sobre os
dados que ele armazena. Então, o que torna o `RefCell<T>` diferente de um tipo
como `Box<T>`? Lembre-se das regras de empréstimo que você aprendeu no Capítulo
4:

- Em qualquer momento dado, você pode ter _ou_ uma referência mutável, _ou_
  qualquer quantidade de referências imutáveis (mas não ambos).
- As referências devem ser sempre válidas.

Com referências e `Box<T>`, os invariantes das regras de empréstimo são
aplicados em tempo de compilação. Com `RefCell<T>`, esses invariantes são
aplicados _em tempo de execução_. Com referências, se você quebrar essas
regras, você receberá um erro de compilação. Com `RefCell<T>`, se você quebrar
essas regras, seu programa entrará em pânico (_panic_) e será encerrado.

As vantagens de verificar as regras de empréstimo em tempo de compilação são
que os erros são detectados mais cedo no processo de desenvolvimento, e não há
impacto no desempenho em tempo de execução porque toda a análise é concluída
antecipadamente. Por esses motivos, verificar as regras de empréstimo em tempo
de compilação é a melhor escolha na maioria dos casos, razão pela qual esta é
a escolha padrão do Rust.

A vantagem de verificar as regras de execução em tempo de execução, por outro
lado, é que determinados cenários seguros para a memória passam a ser
permitidos, onde seriam proibidos pelas verificações em tempo de compilação. A
análise estática, como o compilador do Rust, é inerentemente conservadora.
Algumas propriedades de código são impossíveis de detectar analisando o código:
o exemplo mais famoso é o Problema da Parada (_Halting Problem_), que está fora
do escopo deste livro, mas é um tópico interessante para pesquisar.

Como alguma análise é impossível, se o compilador Rust não tiver certeza de que
o código cumpre as regras de propriedade, ele pode rejeitar um programa correto;
dessa forma, ele é conservador. Se o Rust aceitasse um programa incorreto, os
usuários não poderiam confiar nas garantias que o Rust faz. No entanto, se o
Rust rejeita um programa correto, o programador sofrerá um inconveniente, mas
nada catastrófico pode ocorrer. O tipo `RefCell<T>` é útil quando você tem
certeza de que seu código segue as regras de empréstimo, mas o compilador é
incapaz de entender e garantir isso.

Semelhante ao `Rc<T>`, o `RefCell<T>` é apenas para uso em cenários de thread
única e resultará em um erro de compilação se você tentar usá-lo em um contexto
multithread. Falaremos sobre como obter a funcionalidade de um `RefCell<T>` em
um programa multithread no Capítulo 16.

Aqui está um resumo dos motivos para escolher `Box<T>`, `Rc<T>` ou `RefCell<T>`:

- `Rc<T>` permite múltiplos proprietários dos mesmos dados; `Box<T>` e
  `RefCell<T>` possuem proprietários únicos.
- `Box<T>` permite empréstimos imutáveis ou mutáveis verificados em tempo de
  compilação; `Rc<T>` permite apenas empréstimos imutáveis verificados em tempo
  de compilação; `RefCell<T>` permite empréstimos imutáveis ou mutáveis
  verificados em tempo de execução.
- Como `RefCell<T>` permite empréstimos mutáveis verificados em tempo de
  execução, você pode mutar o valor dentro do `RefCell<T>` mesmo quando o
  `RefCell<T>` for imutável.

Mutar o valor dentro de um valor imutável é o padrão de mutabilidade interior.
Vamos olhar para uma situação em que a mutabilidade interior é útil e examinar
como isso é possível.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### Usando Mutabilidade Interior

Uma consequência das regras de empréstimo é que, quando você tem um valor
imutável, você não pode emprestá-lo como mutável. Por exemplo, este código não
vai compilar:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Se você tentar compilar este código, você receberá o seguinte erro:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

No entanto, existem situações em que seria útil para um valor mutar a si mesmo
em seus métodos, mas parecer imutável para o restante do código. O código fora
dos métodos do valor não seria capaz de mutar o valor. Usar `RefCell<T>` é uma
maneira de obter a capacidade de ter mutabilidade interior, mas o
`RefCell<T>` não contorna completamente as regras de empréstimo: o verificador
de empréstimos (_borrow checker_) no compilador permite essa mutabilidade
interior, e as regras de empréstimo são verificadas em tempo de execução em seu
lugar. Se você violar as regras, você receberá um `panic!` em vez de um erro de
compilação.

Vamos trabalhar em um exemplo prático onde podemos usar `RefCell<T>` para mutar
um valor imutável e ver por que isso é útil.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### Testando com Objetos Falsos (Mock Objects)

Às vezes, durante os testes, um programador usará um tipo no lugar de outro
tipo para observar um comportamento específico e afirmar que ele foi
implementado corretamente. Esse tipo de espaço reservado é chamado de _duplo de
teste_ (_test double_). Pense nisso no sentido de um dublê de corpo no cinema,
onde uma pessoa entra e substitui um ator para fazer uma cena particularmente
difícil. Duplos de teste substituem outros tipos quando estamos executando
testes. _Objetos simulados_ (_Mock objects_) são tipos específicos de duplos de
teste que registram o que acontece durante um teste para que você possa afirmar
que as ações corretas ocorreram.

O Rust não tem objetos no mesmo sentido que outras linguagens têm objetos, e o
Rust não tem funcionalidade de objeto simulado embutida na biblioteca padrão
como algumas outras linguagens têm. No entanto, você definitivamente pode criar
uma struct que servirá aos mesmos propósitos de um objeto simulado.

Aqui está o cenário que testaremos: criaremos uma biblioteca que rastreia um
valor em relação a um valor máximo e envia mensagens com base no quão perto do
valor máximo o valor atual está. Essa biblioteca pode ser usada para
acompanhar a cota de um usuário para o número de chamadas de API que ele tem
permissão para fazer, por exemplo.

Nossa biblioteca fornecerá apenas a funcionalidade de rastrear quão perto do
máximo um valor está e quais devem ser as mensagens em quais momentos. Espera-se
que as aplicações que usam nossa biblioteca forneçam o mecanismo para enviar as
mensagens: a aplicação pode mostrar a mensagem diretamente ao usuário, enviar
um e-mail, enviar uma mensagem de texto ou fazer outra coisa. A biblioteca não
precisa saber desse detalhe. Tudo o que ela precisa é de algo que implemente
uma trait que forneceremos, chamada `Messenger`. A Listagem 15-20 mostra o
código da biblioteca.

<Listing number="15-20" file-name="src/lib.rs" caption="Uma biblioteca para acompanhar quão perto um valor está de um valor máximo e avisar quando o valor estiver em determinados níveis">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Uma parte importante desse código é que a trait `Messenger` tem um método
chamado `send` que recebe uma referência imutável para `self` e o texto da
mensagem. Esta trait é a interface que nosso objeto simulado precisa
implementar para que o mock possa ser usado da mesma forma que um objeto real.
A outra parte importante é que queremos testar o comportamento do método
`set_value` no `LimitTracker`. Podemos alterar o que passamos para o parâmetro
`value`, mas `set_value` não retorna nada para fazermos asserções. Queremos ser
capazes de dizer que, se criarmos um `LimitTracker` com algo que implementa a
trait `Messenger` e um valor específico para `max`, o mensageiro será instruído
a enviar as mensagens apropriadas quando passarmos números diferentes para
`value`.

Precisamos de um objeto simulado que, em vez de enviar um e-mail ou mensagem de
texto quando chamamos `send`, apenas acompanhe as mensagens que foi instruído a
enviar. Podemos criar uma nova instância do objeto simulado, criar um
`LimitTracker` que usa o objeto simulado, chamar o método `set_value` no
`LimitTracker` e, em seguida, verificar se o objeto simulado tem as mensagens
que esperamos. A Listagem 15-21 mostra uma tentativa de implementar um objeto
simulado para fazer exatamente isso, mas o verificador de empréstimos não
permitirá.

<Listing number="15-21" file-name="src/lib.rs" caption="Uma tentativa de implementar um `MockMessenger` que não é permitida pelo verificador de empréstimos">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Este código de teste define uma struct `MockMessenger` que possui um campo
`sent_messages` com um `Vec` de valores `String` para acompanhar as mensagens
que foi instruída a enviar. Também definimos uma função associada `new` para
tornar conveniente a criação de novos valores `MockMessenger` que começam com
uma lista vazia de mensagens. Em seguida, implementamos a trait `Messenger`
para `MockMessenger` para que possamos fornecer um `MockMessenger` a um
`LimitTracker`. Na definição do método `send`, pegamos a mensagem passada como
parâmetro e a armazenamos na lista `sent_messages` do `MockMessenger`.

No teste, estamos testando o que acontece quando o `LimitTracker` é instruído a
definir `value` para algo que é mais do que 75 por cento do valor `max`.
Primeiro, criamos um novo `MockMessenger`, que começará com uma lista vazia de
mensagens. Em seguida, criamos um novo `LimitTracker` e damos a ele uma
referência para o novo `MockMessenger` e um valor `max` de `100`. Chamamos o
método `set_value` no `LimitTracker` com um valor de `80`, que é mais de 75 por
cento de 100. Então, afirmamos que a lista de mensagens que o `MockMessenger`
está acompanhando agora deve ter uma mensagem nela.

No entanto, há um problema com este teste, conforme mostrado aqui:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Não podemos modificar o `MockMessenger` para acompanhar as mensagens, porque o
método `send` aceita uma referência imutável para `self`. Também não podemos
aceitar a sugestão do texto do erro de usar `&mut self` tanto no método `impl`
quanto na definição da trait. Nós não queremos alterar a trait `Messenger`
unicamente por causa dos testes. Em vez disso, precisamos encontrar uma maneira
de fazer nosso código de teste funcionar corretamente com nosso design
existente.

Esta é uma situação em que a mutabilidade interior pode ajudar! Armazenaremos
as `sent_messages` dentro de um `RefCell<T>`, e então o método `send` será capaz
de modificar `sent_messages` para armazenar as mensagens que vimos. A Listagem
15-22 mostra como isso se parece.

<Listing number="15-22" file-name="src/lib.rs" caption="Usando `RefCell<T>` para mutar um valor interno enquanto o valor externo é considerado imutável">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

O campo `sent_messages` agora é do tipo `RefCell<Vec<String>>` em vez de
`Vec<String>`. Na função `new`, criamos uma nova instância de
`RefCell<Vec<String>>` em torno do vetor vazio.

Para a implementação do método `send`, o primeiro parâmetro ainda é um
empréstimo imutável de `self`, que corresponde à definição da trait. Chamamos
`borrow_mut` no `RefCell<Vec<String>>` em `self.sent_messages` para obter uma
referência mutável para o valor dentro do `RefCell<Vec<String>>`, que é o
vetor. Em seguida, podemos chamar `push` na referência mutável para o vetor
para acompanhar as mensagens enviadas durante o teste.

A última alteração que temos que fazer é na asserção: para ver quantos itens
estão no vetor interno, chamamos `borrow` no `RefCell<Vec<String>>` para obter
uma referência imutável para o vetor.

Agora que você viu como usar `RefCell<T>`, vamos nos aprofundar em como ele
funciona!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### Rastreando Empréstimos em Tempo de Execução

Ao criar referências imutáveis e mutáveis, usamos a sintaxe `&` e `&mut`,
respectivamente. Com `RefCell<T>`, usamos os métodos `borrow` e `borrow_mut`,
que fazem parte da API segura que pertence ao `RefCell<T>`. O método `borrow`
retorna o tipo de ponteiro inteligente `Ref<T>`, e `borrow_mut` retorna o tipo
de ponteiro inteligente `RefMut<T>`. Ambos os tipos implementam `Deref`, então
podemos tratá-los como referências normais.

O `RefCell<T>` acompanha quantos ponteiros inteligentes `Ref<T>` e `RefMut<T>`
estão ativos no momento. Cada vez que chamamos `borrow`, o `RefCell<T>` aumenta
sua contagem de quantos empréstimos imutáveis estão ativos. Quando um valor
`Ref<T>` sai de escopo, a contagem de empréstimos imutáveis diminui em 1. Assim
como as regras de empréstimo em tempo de compilação, o `RefCell<T>` nos permite
ter vários empréstimos imutáveis ou um empréstimo mutável em qualquer ponto no
tempo.

Se tentarmos violar essas regras, em vez de obter um erro de compilação como
faríamos com referências, a implementação do `RefCell<T>` entrará em pânico em
tempo de execução. A Listagem 15-23 mostra uma modificação da implementação de
`send` na Listagem 15-22. Estamos tentando deliberadamente criar dois
empréstimos mutáveis ativos para o mesmo escopo para ilustrar que o
`RefCell<T>` nos impede de fazer isso em tempo de execução.

<Listing number="15-23" file-name="src/lib.rs" caption="Criando duas referências mutáveis no mesmo escopo para ver que o `RefCell<T>` entrará em pânico">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Criamos uma variável `one_borrow` para o ponteiro inteligente `RefMut<T>`
retornado de `borrow_mut`. Em seguida, criamos outro empréstimo mutável da
mesma forma na variável `two_borrow`. Isso cria duas referências mutáveis no
mesmo escopo, o que não é permitido. Quando executamos os testes para nossa
biblioteca, o código na Listagem 15-23 compilará sem erros, mas o teste falhará:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Note que o código entrou em pânico com a mensagem `already borrowed:
BorrowMutError`. É assim que o `RefCell<T>` lida com violações das regras de
empréstimo em tempo de execução.

Escolher capturar erros de empréstimo em tempo de execução em vez de em tempo
de compilação, como fizemos aqui, significa que você potencialmente encontrará
erros em seu código mais tarde no processo de desenvolvimento: possivelmente não
até que seu código seja implantado em produção. Além disso, seu código incorrerá
em uma pequena penalidade de desempenho em tempo de execução como resultado de
acompanhar os empréstimos em tempo de execução em vez de em tempo de
compilação. No entanto, o uso de `RefCell<T>` torna possível escrever um objeto
simulado que pode modificar a si mesmo para acompanhar as mensagens que viu
enquanto você o está usando em um contexto onde apenas valores imutáveis são
permitidos. Você pode usar `RefCell<T>` apesar de suas compensações (_trade-offs_)
para obter mais funcionalidade do que as referências comuns fornecem.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### Permitindo Múltiplos Proprietários de Dados Mutáveis

Uma maneira comum de usar `RefCell<T>` é em combinação com `Rc<T>`. Lembre-se
de que o `Rc<T>` permite que você tenha vários proprietários de alguns dados,
mas ele fornece apenas acesso imutável a esses dados. Se você tiver um
`Rc<T>` que contém um `RefCell<T>`, você pode obter um valor que pode ter
vários proprietários _e_ que você pode mutar!

Por exemplo, lembre-se do exemplo da lista ligada (`cons list`) na Listagem
15-18, onde usamos `Rc<T>` para permitir que várias listas compartilhem a
propriedade de outra lista. Como o `Rc<T>` contém apenas valores imutáveis, não
podemos alterar nenhum dos valores da lista depois de criados. Vamos adicionar
`RefCell<T>` por sua capacidade de alterar os valores nas listas. A Listagem
15-24 mostra que, usando um `RefCell<T>` na definição de `Cons`, podemos
modificar o valor armazenado em todas as listas.

<Listing number="15-24" file-name="src/main.rs" caption="Usando `Rc<RefCell<i32>>` para criar uma `List` que podemos mutar">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Criamos um valor que é uma instância de `Rc<RefCell<i32>>` e o armazenamos em
uma variável chamada `value` para que possamos acessá-lo diretamente mais
tarde. Em seguida, criamos uma `List` em `a` com uma variante `Cons` que contém
`value`. Precisamos clonar `value` para que tanto `a` quanto `value` tenham a
propriedade do valor interno `5`, em vez de transferir a propriedade de `value`
para `a` ou fazer com que `a` pegue emprestado de `value`.

Envolvemos a lista `a` em um `Rc<T>` para que, quando criarmos as listas `b` e
`c`, ambas possam se referir a `a`, que foi o que fizemos na Listagem 15-18.

Depois de criarmos as listas em `a`, `b` e `c`, queremos adicionar 10 ao valor
em `value`. Fazemos isso chamando `borrow_mut` em `value`, que usa o recurso
de desreferenciamento automático que discutimos em [“Onde está o operador
`->`?”][wheres-the---operator]<!-- ignore --> no Capítulo 5 para desreferenciar
o `Rc<T>` para o valor interno `RefCell<T>`. O método `borrow_mut` retorna um
ponteiro inteligente `RefMut<T>`, e usamos o operador de desreferenciamento nele
e alteramos o valor interno.

Quando imprimimos `a`, `b` e `c`, podemos ver que todos eles têm o valor
modificado de `15` em vez de `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Esta técnica é bem legal! Usando `RefCell<T>`, temos um valor `List`
exteriormente imutável. Mas podemos usar os métodos no `RefCell<T>` que fornecem
acesso à sua mutabilidade interior para que possamos modificar nossos dados
quando precisarmos. As verificações em tempo de execução das regras de
empréstimo nos protegem contra corridas de dados (_data races_), e às vezes vale
a pena trocar um pouco de velocidade por essa flexibilidade em nossas
estruturas de dados. Note que `RefCell<T>` não funciona para código
multithread! `Mutex<T>` é a versão segura para threads de `RefCell<T>`, e nós
discutiremos o `Mutex<T>` no Capítulo 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
