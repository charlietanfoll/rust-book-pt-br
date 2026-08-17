<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## Usando Objetos Trait para Abstrair sobre Comportamento Compartilhado

No Capítulo 8, mencionamos que uma das limitações dos vetores é que eles só
podem armazenar elementos de um único tipo. Criamos uma solução alternativa na Listagem 8-9,
onde definimos um enum `SpreadsheetCell` que tinha variantes para armazenar inteiros,
pontos flutuantes e texto. Isso significava que podíamos armazenar diferentes tipos de
dados em cada célula e ainda ter um vetor que representava uma linha de células. Esta é
uma solução perfeitamente boa quando nossos itens intercambiáveis são um conjunto fixo de
tipos que conhecemos quando nosso código é compilado.

No entanto, às vezes queremos permitir que quem usa nossa biblioteca possa estender o conjunto de
tipos válidos em uma situação específica. Para mostrar como podemos alcançar
isso, criaremos uma ferramenta de interface gráfica de usuário (GUI) de exemplo que iterará
através de uma lista de itens, chamando um método `draw` em cada um para desenhá-lo na
tela — uma técnica comum para ferramentas de GUI. Criaremos uma biblioteca chamada
`gui` que contém a estrutura de uma biblioteca de GUI. Esta crate pode incluir
alguns tipos para as pessoas usarem, como `Button` ou `TextField`. Além disso,
os usuários de `gui` vão querer criar seus próprios tipos que podem ser desenhados: Por
exemplo, um programador pode adicionar um `Image`, e outro pode adicionar um
`SelectBox`.

No momento da escrita da biblioteca, não podemos saber e definir todos os tipos que
outros programadores podem querer criar. Mas sabemos que `gui` precisa acompanhar
muitos valores de tipos diferentes, e precisa chamar um método `draw`
em cada um desses valores de tipos diferentes. Não precisa saber exatamente o que
acontecerá quando chamarmos o método `draw`, apenas que o valor terá esse
método disponível para chamarmos.

Para fazer isso em uma linguagem com herança, poderíamos definir uma classe chamada
`Component` que tem um método chamado `draw` nela. As outras classes, como
`Button`, `Image` e `SelectBox`, herdariam de `Component` e, portanto,
herdariam o método `draw`. Cada uma delas poderia sobrescrever o método `draw` para definir
seu comportamento personalizado, mas o framework poderia tratar todos os tipos como se
fossem instâncias de `Component` e chamar `draw` neles. Mas como o Rust
não tem herança, precisamos de outra maneira de estruturar a biblioteca `gui` para
permitir que os usuários criem novos tipos compatíveis com a biblioteca.

### Definindo um Trait para Comportamento Comum

Para implementar o comportamento que queremos que `gui` tenha, definiremos um trait
chamado `Draw` que terá um método chamado `draw`. Em seguida, podemos definir um
vetor que aceita um objeto trait (_trait object_). Um _objeto trait_ aponta tanto para uma
instância de um tipo que implementa nosso trait especificado quanto para uma tabela usada para
consultar métodos de trait nesse tipo em tempo de execução. Criamos um objeto trait especificando algum
tipo de ponteiro, como uma referência ou um ponteiro inteligente `Box<T>`, seguido pela
palavra-chave `dyn` e, em seguida, especificando o trait relevante. (Falaremos sobre
o motivo pelo qual os objetos trait devem usar um ponteiro em [“Tipos de Tamanho Dinâmico e o
Trait `Sized`”][dynamically-sized]<!-- ignore --> no Capítulo 20.) Podemos usar
objetos trait no lugar de um tipo genérico ou concreto. Onde quer que usemos um objeto
trait, o sistema de tipos do Rust garantirá em tempo de compilação que qualquer valor usado nesse
contexto implementará o trait do objeto trait. Consequentemente, não precisamos
conhecer todos os tipos possíveis em tempo de compilação.

Mencionamos que, em Rust, evitamos chamar structs e enums de
"objetos" para distingui-los dos objetos de outras linguagens. Em uma struct ou
enum, os dados nos campos da struct e o comportamento nos blocos `impl` são
separados, enquanto em outras linguagens, os dados e o comportamento combinados em um único
conceito são frequentemente rotulados como um objeto. Os objetos trait diferem de objetos em outras
linguagens pelo fato de não podermos adicionar dados a um objeto trait. Os objetos trait não são tão
geralmente úteis quanto os objetos em outras linguagens: O propósito específico deles é
permitir abstração sobre comportamento comum.

A Listagem 18-3 mostra como definir um trait chamado `Draw` com um método chamado
`draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Definição do trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Esta sintaxe deve parecer familiar devido às nossas discussões sobre como definir traits
no Capítulo 10. Em seguida vem uma nova sintaxe: a Listagem 18-4 define uma struct chamada
`Screen` que contém um vetor chamado `components`. Este vetor é do tipo
`Box<dyn Draw>`, que é um objeto trait; é um espaço reservado para qualquer tipo dentro de uma
`Box` que implemente o trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Definição da struct `Screen` com um campo `components` contendo um vetor de objetos trait que implementam o trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Na struct `Screen`, definiremos um método chamado `run` que chamará o
método `draw` em cada um de seus `components`, conforme mostrado na Listagem 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Um método `run` em `Screen` que chama o método `draw` em cada componente">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Isso funciona de forma diferente de definir uma struct que usa um parâmetro de tipo genérico
com restrições de trait. Um parâmetro de tipo genérico pode ser substituído por
apenas um tipo concreto por vez, enquanto os objetos trait permitem que vários
tipos concretos substituam o objeto trait em tempo de execução. Por exemplo,
poderíamos ter definido a struct `Screen` usando um tipo genérico e uma restrição de trait,
como na Listagem 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="Uma implementação alternativa da struct `Screen` e seu método `run` usando genéricos e restrições de trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Isso nos restringe a uma instância de `Screen` que possui uma lista de componentes todos do
tipo `Button` ou todos do tipo `TextField`. Se você sempre terá coleções homogêneas,
usar genéricos e restrições de trait é preferível porque as definições serão
monomorfizadas em tempo de compilação para usar os tipos concretos.

Por outro lado, com o método que usa objetos trait, uma única instância de `Screen`
pode conter um `Vec<T>` que contém tanto um `Box<Button>` quanto um
`Box<TextField>`. Vamos ver como isso funciona e, em seguida, falaremos sobre
as implicações de desempenho em tempo de execução.

### Implementando o Trait

Agora adicionaremos alguns tipos que implementam o trait `Draw`. Forneceremos o
tipo `Button`. Novamente, implementar de fato uma biblioteca de GUI está além do escopo
deste livro, então o método `draw` não terá nenhuma implementação útil em seu
corpo. Para imaginar como seria a implementação, uma struct `Button`
poderia ter campos para `width` (largura), `height` (altura) e `label` (rótulo), conforme mostrado na Listagem 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="Uma struct `Button` que implementa o trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Os campos `width`, `height` e `label` em `Button` serão diferentes dos
campos em outros componentes; por exemplo, um tipo `TextField` pode ter esses
mesmos campos mais um campo `placeholder`. Cada um dos tipos que queremos desenhar na
tela implementará o trait `Draw`, mas usará códigos diferentes no
método `draw` para definir como desenhar aquele tipo específico, assim como o `Button` fez aqui
(sem o código real de GUI, conforme mencionado). O tipo `Button`, por exemplo,
pode ter um bloco `impl` adicional contendo métodos relacionados ao que
acontece quando um usuário clica no botão. Esses tipos de métodos não se aplicarão a
tipos como `TextField`.

Se alguém usando nossa biblioteca decidir implementar uma struct `SelectBox` que possui
os campos `width`, `height` e `options`, essa pessoa implementará o trait `Draw`
no tipo `SelectBox` também, conforme mostrado na Listagem 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Outra crate usando `gui` e implementando o trait `Draw` em uma struct `SelectBox`">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

O usuário da nossa biblioteca agora pode escrever sua função `main` para criar uma
instância de `Screen`. Na instância de `Screen`, ele pode adicionar um `SelectBox` e um
`Button` colocando cada um em um `Box<T>` para se tornar um objeto trait. Ele pode então chamar o
método `run` na instância de `Screen`, que chamará `draw` em cada um dos
componentes. A Listagem 18-9 mostra essa implementação.

<Listing number="18-9" file-name="src/main.rs" caption="Usando objetos trait para armazenar valores de diferentes tipos que implementam o mesmo trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Quando escrevemos a biblioteca, não sabíamos que alguém poderia adicionar o
tipo `SelectBox`, mas nossa implementação de `Screen` foi capaz de operar no
novo tipo e desenhá-lo porque `SelectBox` implementa o trait `Draw`, o que
significa que ele implementa o método `draw`.

Esse conceito — de se preocupar apenas com as mensagens às quais um valor responde em
vez do tipo concreto do valor — é semelhante ao conceito de _duck
typing_ (tipagem pato) em linguagens com tipagem dinâmica: Se anda como um pato e grasna como
um pato, então deve ser um pato! Na implementação de `run` em `Screen` na
Listagem 18-5, `run` não precisa saber qual é o tipo concreto de cada
componente. Ele não verifica se um componente é uma instância de um `Button`
ou de um `SelectBox`, ele apenas chama o método `draw` no componente. Ao
especificar `Box<dyn Draw>` como o tipo dos valores no vetor `components`,
definimos que `Screen` precisa de valores nos quais possamos chamar o
método `draw`.

A vantagem de usar objetos trait e o sistema de tipos do Rust para escrever código
semelhante ao código que usa duck typing é que nunca precisamos verificar se um
valor implementa um determinado método em tempo de execução ou nos preocupar em obter erros
se um valor não implementa um método mas o chamamos mesmo assim. O Rust não compilará
nosso código se os valores não implementarem os traits que os objetos trait exigem.

Por exemplo, a Listagem 18-10 mostra o que acontece se tentarmos criar um `Screen`
com uma `String` como componente.

<Listing number="18-10" file-name="src/main.rs" caption="Tentando usar um tipo que não implementa o trait do objeto trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Receberemos este erro porque `String` não implementa o trait `Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Este erro nos avisa que ou estamos passando algo para `Screen` que não
pretendíamos passar e, portanto, devemos passar um tipo diferente, ou devemos implementar
`Draw` em `String` para que `Screen` possa chamar `draw` nela.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### Realizando Despacho Dinâmico

Lembre-se em [“Desempenho de Código Usando
Genéricos”][performance-of-code-using-generics]<!-- ignore --> no Capítulo 10 da nossa
discussão sobre o processo de monomorfização realizado em genéricos pelo
compilador: O compilador gera implementações não genéricas de funções e
métodos para cada tipo concreto que usamos no lugar de um parâmetro de tipo
genérico. O código resultante da monomorfização está fazendo _despacho
estático_ (_static dispatch_), que é quando o compilador sabe qual método você está chamando em
tempo de compilação. Isso se opõe ao _despacho dinâmico_ (_dynamic dispatch_), que é quando o compilador
não consegue dizer em tempo de compilação qual método você está chamando. Nos casos de despacho
dinâmico, o compilador emite código que, em tempo de execução, saberá qual método chamar.

Quando usamos objetos trait, o Rust deve usar o despacho dinâmico. O compilador não
conhece todos os tipos que podem ser usados com o código que está utilizando objetos trait,
portanto, ele não sabe qual método implementado em qual tipo chamar. Em vez disso, em tempo de
execução, o Rust usa os ponteiros dentro do objeto trait para saber qual método chamar.
Essa busca acarreta um custo em tempo de execução que não ocorre com o despacho estático.
O despacho dinâmico também impede que o compilador escolha eminlinear (_inline_) o código de um método,
o que por sua vez impede algumas otimizações, e o Rust tem algumas regras sobre
onde você pode e não pode usar o despacho dinâmico, chamadas de _compatibilidade com dyn_ (_dyn compatibility_).
Essas regras estão além do escopo desta discussão, mas você pode ler mais sobre elas
[na reference (em inglês)][dyn-compatibility]<!-- ignore -->. No entanto, obtivemos flexibilidade
extra no código que escrevemos na Listagem 18-5 e pudemos dar suporte na Listagem 18-9,
portanto, é uma troca a ser considerada.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
