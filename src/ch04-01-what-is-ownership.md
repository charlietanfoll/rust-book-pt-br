## O que é Ownership?

_Ownership_ (posse) é um conjunto de regras que governam como um programa Rust gerencia a memória. Todos os programas precisam gerenciar a forma como usam a memória de um computador enquanto estão em execução. Algumas linguagens possuem coleta de lixo (garbage collection) que procura regularmente por memória que não é mais usada à medida que o programa é executado; em outras linguagens, o programador deve alocar e liberar explicitamente a memória. O Rust usa uma terceira abordagem: a memória é gerenciada por meio de um sistema de ownership com um conjunto de regras que o compilador verifica. Se alguma das regras for violada, o programa não será compilado. Nenhum dos recursos de ownership deixará seu programa mais lento enquanto ele estiver rodando.

Como o ownership é um conceito novo para muitos programadores, leva um tempo para se acostumar. A boa notícia é que quanto mais experiente você se torna com o Rust e com as regras do sistema de ownership, mais fácil se torna desenvolver naturalmente código que seja seguro e eficiente. Continue praticando!

Quando você entender o ownership, terá uma base sólida para compreender os recursos que tornam o Rust único. Neste capítulo, você aprenderá sobre ownership trabalhando em alguns exemplos focados em uma estrutura de dados muito comum: strings.

> ### A Pilha (Stack) e a Fila/Monte (Heap)
>
> Muitas linguagens de programação não exigem que você pense sobre a stack e a heap com muita frequência. Mas em uma linguagem de programação de sistemas como o Rust, saber se um valor está na stack ou na heap afeta o comportamento da linguagem e o motivo pelo qual você precisa tomar certas decisões. Partes do ownership serão descritas em relação à stack e à heap mais adiante neste capítulo, então aqui está uma breve explicação em preparação.
>
> Tanto a stack quanto a heap são partes de memória disponíveis para o seu código usar em tempo de execução, mas são estruturadas de maneiras diferentes. A stack armazena valores na ordem em que os recebe e remove os valores na ordem inversa. Isso é conhecido como _último a entrar, primeiro a sair (LIFO - last in, first out)_. Pense em uma pilha de pratos: quando você adiciona mais pratos, você os coloca no topo da pilha, e quando precisa de um prato, você retira um do topo. Adicionar ou remover pratos do meio ou da base não funcionaria muito bem! Adicionar dados é chamado de _empilhar (pushing onto the stack)_, e remover dados é chamado de _desempilhar (popping off the stack)_. Todos os dados armazenados na stack devem ter um tamanho conhecido e fixo. Dados com tamanho desconhecido em tempo de compilação ou com tamanho que possa mudar devem ser armazenados na heap.
>
> A heap é menos organizada: quando você coloca dados na heap, você solicita uma certa quantidade de espaço. O alocador de memória encontra um espaço vazio na heap que seja grande o suficiente, o marca como em uso e retorna um _ponteiro_, que é o endereço desse local. Esse processo é chamado de _alocar na heap_ e às vezes é abreviado apenas como _alocar_ (empilhar valores na stack não é considerado alocação). Como o ponteiro para a heap tem um tamanho conhecido e fixo, você pode armazenar o ponteiro na stack, mas quando quiser os dados reais, precisará seguir o ponteiro. Pense em estar sentado em um restaurante. Quando você entra, você informa o número de pessoas no seu grupo, e o anfitrião encontra uma mesa vazia que acomode todos e o conduz até lá. Se alguém do seu grupo chegar atrasado, essa pessoa pode perguntar onde vocês foram sentados para encontrá-los.
>
> Empilhar na stack é mais rápido do que alocar na heap porque o alocador nunca precisa procurar um lugar para armazenar novos dados; esse local está sempre no topo da stack. Comparativamente, alocar espaço na heap exige mais trabalho porque o alocador deve primeiro encontrar um espaço grande o suficiente para conter os dados e, em seguida, realizar o registro contábil para se preparar para a próxima alocação.
>
> Acessar dados na heap geralmente é mais lento do que acessar dados na stack porque você precisa seguir um ponteiro para chegar lá. Os processadores contemporâneos são mais rápidos se saltarem menos na memória. Continuando a analogia, considere um garçom em um restaurante anotando pedidos de várias mesas. É mais eficiente anotar todos os pedidos de uma mesa antes de passar para a próxima. Pegar um pedido da mesa A, depois um pedido da mesa B, depois um da mesa A novamente e depois um da mesa B novamente seria um processo muito mais lento. Pela mesma lógica, um processador geralmente pode fazer seu trabalho melhor se trabalhar com dados próximos a outros dados (como estão na stack) em vez de mais distantes (como podem estar na heap).
>
> Quando seu código chama uma função, os valores passados para a função (incluindo, potencialmente, ponteiros para dados na heap) e as variáveis locais da função são empilhados na stack. Quando a função termina, esses valores são desempilhados da stack.
>
> Acompanhar quais partes do código estão usando quais dados na heap, minimizar a quantidade de dados duplicados na heap e limpar dados não utilizados na heap para que você não fique sem espaço são todos problemas que o ownership resolve. Uma vez que você entenda o ownership, não precisará pensar sobre a stack e a heap com tanta frequência. Mas saber que o principal propósito do ownership é gerenciar dados da heap pode ajudar a explicar por que ele funciona da maneira que funciona.

### Regras de Ownership

Primeiro, vamos dar uma olhada nas regras de ownership. Tenha essas regras em mente enquanto percorremos os exemplos que as ilustram:

- Cada valor no Rust tem um _proprietário_ (owner).
- Só pode haver um proprietário de cada vez.
- Quando o proprietário sai de escopo, o valor será descartado.

### Escopo de Variáveis

Agora que já passamos pela sintaxe básica do Rust, não incluiremos todo o código `fn main() {` nos exemplos, então, se você estiver acompanhando, certifique-se de colocar os seguintes exemplos dentro de uma função `main` manualmente. Como resultado, nossos exemplos serão um pouco mais concisos, permitindo-nos focar nos detalhes reais em vez de código corriqueiro (*boilerplate*).

Como primeiro exemplo de ownership, vamos analisar o escopo de algumas variáveis. Um _escopo_ é o intervalo dentro de um programa para o qual um item é válido. Considere a seguinte variável:

```rust
let s = "hello";
```

A variável `s` refere-se a um literal de string, onde o valor da string está embutido (*hardcoded*) no texto do nosso programa. A variável é válida desde o ponto em que é declarada até o final do escopo atual. A Listagem 4-1 mostra um programa com comentários anotando onde a variável `s` seria válida.

<Listing number="4-1" caption="Uma variável e o escopo em que ela é válida">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

Em outras palavras, há dois pontos importantes no tempo aqui:

- Quando `s` entra _no_ escopo, ela é válida.
- Ela permanece válida até que _saia_ de escopo.

Neste ponto, a relação entre escopos e quando as variáveis são válidas é semelhante à de outras linguagens de programação. Agora vamos construir sobre essa compreensão introduzindo o tipo `String`.

### O Tipo `String`

Para ilustrar as regras de ownership, precisamos de um tipo de dado que seja mais complexo do que aqueles que cobrimos na seção [“Tipos de Dados”][data-types]<!-- ignore --> do Capítulo 3. Os tipos cobertos anteriormente têm um tamanho conhecido, podem ser armazenados na stack e desempilhados quando seu escopo termina, e podem ser copiados de forma rápida e trivial para criar uma nova instância independente se outra parte do código precisar usar o mesmo valor em um escopo diferente. Mas queremos analisar dados armazenados na heap e explorar como o Rust sabe quando limpar esses dados, e o tipo `String` é um ótimo exemplo.

Vamos nos concentrar nas partes da `String` que se relacionam com o ownership. Esses aspectos também se aplicam a outros tipos de dados complexos, sejam eles fornecidos pela biblioteca padrão ou criados por você. Discutiremos aspectos de `String` que não são de ownership no [Capítulo 8][ch8]<!-- ignore -->.

Já vimos literais de string, onde um valor de string é embutido em nosso programa. Literais de string são convenientes, mas não são adequados para todas as situações em que podemos querer usar texto. Uma razão é que eles são imutáveis. Outra é que nem todo valor de string pode ser conhecido quando escrevemos nosso código: por exemplo, e se quisermos pegar a entrada do usuário e armazená-la? É para essas situações que o Rust possui o tipo `String`. Esse tipo gerencia dados alocados na heap e, como tal, é capaz de armazenar uma quantidade de texto que nos é desconhecida em tempo de compilação. Você pode criar uma `String` a partir de um literal de string usando a função `from`, assim:

```rust
let s = String::from("hello");
```

O operador de dois pontos duplos `::` nos permite colocar esta função `from` específica sob o namespace do tipo `String`, em vez de usar algum tipo de nome como `string_from`. Discutiremos mais essa sintaxe na seção [“Métodos”][methods]<!-- ignore --> do Capítulo 5 e quando falarmos sobre namespaces com módulos em [“Caminhos para Referenciar um Item na Árvore de Módulos”][paths-module-tree]<!-- ignore --> no Capítulo 7.

Esse tipo de string _pode_ ser mutado:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Então, qual é a diferença aqui? Por que `String` pode ser mutada, mas literais não? A diferença está em como esses dois tipos lidam com a memória.

### Memória e Alocação

No caso de um literal de string, conhecemos o conteúdo em tempo de compilação, portanto, o texto é embutido diretamente no executável final. É por isso que literais de string são rápidos e eficientes. Mas essas propriedades vêm apenas da imutabilidade do literal de string. Infelizmente, não podemos colocar um bloco de memória no binário para cada pedaço de texto cujo tamanho seja desconhecido em tempo de compilação e cujo tamanho possa mudar durante a execução do programa.

Com o tipo `String`, para dar suporte a um pedaço de texto mutável e redimensionável, precisamos alocar uma quantidade de memória na heap, desconhecida em tempo de compilação, para armazenar o conteúdo. Isso significa que:

- A memória deve ser solicitada ao alocador de memória em tempo de execução.
- Precisamos de uma maneira de devolver essa memória ao alocador quando terminarmos com nossa `String`.

A primeira parte é feita por nós: quando chamamos `String::from`, sua implementação solicita a memória de que precisa. Isso é praticamente universal nas linguagens de programação.

No entanto, a segunda parte é diferente. Em linguagens com um _coletor de lixo (GC)_, o GC rastreia e limpa a memória que não está mais sendo usada, e não precisamos pensar sobre isso. Na maioria das linguagens sem um GC, é nossa responsabilidade identificar quando a memória não está mais sendo usada e chamar o código para liberá-la explicitamente, assim como fizemos para solicitá-la. Fazer isso corretamente tem sido historicamente um problema de programação difícil. Se esquecermos, desperdiçaremos memória. Se fizermos muito cedo, teremos uma variável inválida. Se fizermos duas vezes, isso também é um bug. Precisamos parear exatamente um `allocate` com exatamente um `free`.

O Rust segue um caminho diferente: a memória é devolvida automaticamente assim que a variável que a possui sai de escopo. Aqui está uma versão do nosso exemplo de escopo da Listagem 4-1 usando uma `String` em vez de um literal de string:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Há um ponto natural em que podemos devolver a memória que nossa `String` precisa para o alocador: quando `s` sai de escopo. Quando uma variável sai de escopo, o Rust chama uma função especial para nós. Essa função se chama `drop`, e é nela que o autor de `String` pode colocar o código para retornar a memória. O Rust chama `drop` automaticamente na chave fechada.

> Nota: Em C++, esse padrão de desalocação de recursos no final do tempo de vida de um item às vezes é chamado de _Resource Acquisition Is Initialization (RAII)_ (Aquisição de Recurso é Inicialização). A função `drop` em Rust será familiar para você se você já usou padrões RAII.

Esse padrão tem um impacto profundo na forma como o código Rust é escrito. Pode parecer simples agora, mas o comportamento do código pode ser inesperado em situações mais complexas quando queremos que várias variáveis usem os dados que alocamos na heap. Vamos explorar algumas dessas situações agora.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

#### Variáveis e Dados Interagindo com Move

Várias variáveis podem interagir com os mesmos dados de maneiras diferentes no Rust. A Listagem 4-2 mostra um exemplo usando um inteiro.

<Listing number="4-2" caption="Atribuindo o valor inteiro da variável `x` a `y`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

Provavelmente podemos adivinhar o que isso está fazendo: “Vincule o valor `5` a `x`; então, faça uma cópia do valor em `x` e vincule-a a `y`.” Agora temos duas variáveis, `x` e `y`, e ambas são iguais a `5`. Isso é de fato o que está acontecendo, porque inteiros são valores simples com um tamanho conhecido e fixo, e esses dois valores `5` são empilhados na stack.

Agora vamos olhar para a versão com `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Isso parece muito semelhante, então podemos assumir que a maneira como funciona seria a mesma: ou seja, a segunda linha faria uma cópia do valor em `s1` e o vincularia a `s2`. Mas não é exatamente isso que acontece.

Dê uma olhada na Figura 4-1 para ver o que está acontecendo com a `String` nos bastidores. Uma `String` é composta de três partes, mostradas à esquerda: um ponteiro para a memória que contém o conteúdo da string, um comprimento e uma capacidade. Esse grupo de dados é armazenado na stack. À direita está a memória na heap que contém o conteúdo.

<img alt="Duas tabelas: a primeira tabela contém a representação de s1 na stack, consistindo em seu comprimento (5), capacidade (5) e um ponteiro para o primeiro valor na segunda tabela. A segunda tabela contém a representação dos dados da string na heap, byte a byte." src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-1: A representação na memória de uma `String` contendo o valor `"hello"` vinculado a `s1`</span>

O comprimento é quanta memória, em bytes, o conteúdo da `String` está usando atualmente. A capacidade é a quantidade total de memória, em bytes, que a `String` recebeu do alocador. A diferença entre comprimento e capacidade importa, mas não neste contexto, então, por enquanto, não há problema em ignorar a capacidade.

Quando atribuímos `s1` a `s2`, os dados da `String` são copiados, o que significa que copiamos o ponteiro, o comprimento e a capacidade que estão na stack. Nós não copiamos os dados na heap para os quais o ponteiro aponta. Em outras palavras, a representação de dados na memória se parece com a Figura 4-2.

<img alt="Três tabelas: tabelas s1 e s2 representando essas strings na stack, respectivamente, e ambas apontando para os mesmos dados de string na heap." src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-2: A representação na memória da variável `s2` que possui uma cópia do ponteiro, comprimento e capacidade de `s1`</span>

A representação _não_ se parece com a Figura 4-3, que é como a memória se pareceria se o Rust copiasse os dados da heap também. Se o Rust fizesse isso, a operação `s2 = s1` poderia ser muito custosa em termos de desempenho de tempo de execução se os dados na heap fossem grandes.

<img alt="Quatro tabelas: duas tabelas representando os dados da stack para s1 e s2, e cada uma aponta para sua própria cópia dos dados da string na heap." src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-3: Outra possibilidade para o que `s2 = s1` poderia fazer se o Rust também copiasse os dados da heap</span>

Anteriormente, dissemos que quando uma variável sai de escopo, o Rust chama automaticamente a função `drop` e limpa a memória da heap para essa variável. Mas a Figura 4-2 mostra ambos os ponteiros de dados apontando para o mesmo local. Isso é um problema: quando `s2` e `s1` saírem de escopo, ambos tentarão liberar a mesma memória. Isso é conhecido como um erro de _dupla liberação (double free)_ e é um dos bugs de segurança de memória que mencionamos anteriormente. Liberar memória duas vezes pode levar à corrupção de memória, o que pode potencialmente levar a vulnerabilidades de segurança.

Para garantir a segurança de memória, após a linha `let s2 = s1;`, o Rust considera `s1` como não sendo mais válido. Portanto, o Rust não precisa liberar nada quando `s1` sai de escopo. Veja o que acontece quando você tenta usar `s1` após `s2` ser criado; não vai funcionar:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Você receberá um erro como este porque o Rust impede que você use a referência invalidada:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Se você já ouviu os termos _cópia superficial (shallow copy)_ e _cópia profunda (deep copy)_ ao trabalhar com outras linguagens, o conceito de copiar o ponteiro, comprimento e capacidade sem copiar os dados provavelmente soa como fazer uma cópia superficial. Mas como o Rust também invalida a primeira variável, em vez de ser chamada de cópia superficial, é conhecida como um _move_ (movimentação). Neste exemplo, diríamos que `s1` foi _movido_ para `s2`. Então, o que realmente acontece é mostrado na Figura 4-4.

<img alt="Três tabelas: tabelas s1 e s2 representando essas strings na stack, respectivamente, e ambas apontando para os mesmos dados de string na heap. A tabela s1 está esmaecida porque s1 não é mais válida; apenas s2 pode ser usada para acessar os dados da heap." src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-4: A representação na memória após `s1` ter sido invalidada</span>

Isso resolve nosso problema! Com apenas `s2` válido, quando ele sair de escopo, ele sozinho liberará a memória, e pronto.

Além disso, há uma escolha de design implícita nisso: o Rust nunca criará automaticamente cópias “profundas” dos seus dados. Portanto, pode-se assumir que qualquer cópia _automática_ é barata em termos de desempenho de tempo de execução.

#### Escopo e Atribuição

O inverso disso é verdadeiro para a relação entre escopo, ownership e a memória sendo liberada através da função `drop` também. Quando você atribui um valor completamente novo a uma variável existente, o Rust chamará `drop` e liberará a memória do valor original imediatamente. Considere este código, por exemplo:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Inicialmente declaramos uma variável `s` e a vinculamos a uma `String` com o valor `"hello"`. Em seguida, criamos imediatamente uma nova `String` com o valor `"ahoy"` e a atribuímos a `s`. Neste ponto, nada está se referindo ao valor original na heap. A Figura 4-5 ilustra os dados da stack e da heap agora:

<img alt="Uma tabela representando o valor da string na stack, apontando para o segundo pedaço de dados da string (ahoy) na heap, com os dados da string originais (hello) esmaecidos porque não podem mais ser acessados." src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-5: A representação na memória após o valor inicial ter sido substituído por completo</span>

A string original, portanto, sai de escopo imediatamente. O Rust executará a função `drop` nela e sua memória será liberada imediatamente. Quando imprimirmos o valor no final, ele será `"ahoy, world!"`.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

#### Variáveis e Dados Interagindo com Clone

Se _quisermos_ copiar profundamente os dados da heap da `String`, e não apenas os dados da stack, podemos usar um método comum chamado `clone`. Discutiremos a sintaxe de métodos no Capítulo 5, mas como métodos são um recurso comum em muitas linguagens de programação, você provavelmente já os viu antes.

Aqui está um exemplo do método `clone` em ação:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Isso funciona muito bem e produz explicitamente o comportamento mostrado na Figura 4-3, onde os dados da heap _são_ copiados.

Quando você vê uma chamada para `clone`, você sabe que algum código arbitrário está sendo executado e que esse código pode ser custoso. É um indicador visual de que algo diferente está acontecendo.

#### Dados Exclusivos da Stack: Copy

Há outro detalhe de que ainda não falamos. Este código usando inteiros — parte do qual foi mostrado na Listagem 4-2 — funciona e é válido:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Mas este código parece contradizer o acabamos de aprender: não temos uma chamada para `clone`, mas `x` ainda é válido e não foi movido para `y`.

A razão é que tipos como inteiros, que têm um tamanho conhecido em tempo de compilação, são armazenados inteiramente na stack, então cópias dos valores reais são rápidas de fazer. Isso significa que não há razão para querermos impedir que `x` seja válido após criarmos a variável `y`. Em outras palavras, não há diferença entre cópia profunda e superficial aqui, então chamar `clone` não faria nada diferente da cópia superficial usual, e podemos deixá-lo de fora.

O Rust tem uma anotação especial chamada trait `Copy` que podemos colocar em tipos que são armazenados na stack, como os inteiros (falaremos mais sobre traits no [Capítulo 10][traits]<!-- ignore -->). Se um tipo implementa o trait `Copy`, as variáveis que o usam não são movidas, mas sim copiadas de forma trivial, tornando-as ainda válidas após a atribuição a outra variável.

O Rust não nos deixará anotar um tipo com `Copy` se o tipo, ou qualquer uma de suas partes, tiver implementado o trait `Drop`. Se o tipo precisar de algo especial para acontecer quando o valor sair de escopo e adicionarmos a anotação `Copy` a esse tipo, teremos um erro em tempo de compilação. Para saber como adicionar a anotação `Copy` ao seu tipo para implementar o trait, consulte [“Traits Deriváveis”][derivable-traits]<!-- ignore --> no Apêndice C.

Então, quais tipos implementam o trait `Copy`? Você pode verificar a documentação do determinado tipo para ter certeza, mas como regra geral, qualquer grupo de valores escalares simples pode implementar `Copy`, e nada que exija alocação ou seja alguma forma de recurso pode implementar `Copy`. Aqui estão alguns dos tipos que implementam `Copy`:

- Todos os tipos inteiros, como `u32`.
- O tipo booleano, `bool`, com valores `true` e `false`.
- Todos os tipos de ponto flutuante, como `f64`.
- O tipo de caractere, `char`.
- Tuplas, se contiverem apenas tipos que também implementam `Copy`. Por exemplo, `(i32, i32)` implementa `Copy`, mas `(i32, String)` não.

### Ownership e Funções

A mecânica de passar um valor para uma função é semelhante à de atribuir um valor a uma variável. Passar uma variável para uma função fará um movimento (*move*) ou cópia, assim como a atribuição faz. A Listagem 4-3 tem um exemplo com algumas anotações mostrando onde as variáveis entram e saem de escopo.

<Listing number="4-3" file-name="src/main.rs" caption="Funções com ownership e escopo anotados">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

Se tentássemos usar `s` após a chamada para `takes_ownership`, o Rust emitiria um erro em tempo de compilação. Essas verificações estáticas nos protegem de erros. Tente adicionar código a `main` que usa `s` e `x` para ver onde você pode usá-los e onde as regras de ownership o impedem de fazer isso.

### Valores de Retorno e Escopo

Retornar valores também pode transferir o ownership. A Listagem 4-4 mostra um exemplo de uma função que retorna algum valor, com anotações semelhantes às da Listagem 4-3.

<Listing number="4-4" file-name="src/main.rs" caption="Transferindo o ownership de valores de retorno">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

O ownership de uma variável segue o mesmo padrão todas as vezes: atribuir um valor a outra variável o move. Quando uma variável que inclui dados na heap sai de escopo, o valor será limpo por `drop`, a menos que o ownership dos dados tenha sido movido para outra variável.

Embora isso funcione, assumir o ownership e depois devolvê-lo com cada função é um pouco tedioso. E se quisermos permitir que uma função use um valor, mas não assuma o ownership? É bastante irritante que tudo o que passamos também precise ser devolvido se quisermos usá-lo novamente, além de quaisquer dados resultantes do corpo da função que possamos querer retornar também.

O Rust nos permite retornar múltiplos valores usando uma tupla, como mostrado na Listagem 4-5.

<Listing number="4-5" file-name="src/main.rs" caption="Retornando o ownership de parâmetros">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

Mas isso é cerimônia demais e muito trabalho para um conceito que deveria ser comum. Para nossa sorte, o Rust tem um recurso para usar um valor sem transferir o ownership: referências.

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[methods]: ch05-03-method-syntax.html#methods
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
