<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## Controlando o Escopo e a Privacidade com Módulos

Nesta seção, falaremos sobre módulos e outras partes do sistema de módulos,
nomeadamente _caminhos_ (paths), que permitem nomear itens; a palavra-chave `use`
que traz um caminho para o escopo; e a palavra-chave `pub` para tornar itens
públicos. Também discutiremos a palavra-chave `as`, pacotes externos e o operador
glob.

### Cola Rápida de Módulos (Cheat Sheet)

Antes de entrarmos nos detalhes de módulos e caminhos, fornecemos aqui uma
referência rápida de como módulos, caminhos, a palavra-chave `use` e a
palavra-chave `pub` funcionam no compilador, e como a maioria dos
desenvolvedores organiza seu código. Passaremos por exemplos de cada uma dessas
regras ao longo deste capítulo, mas este é um ótimo lugar para consultar como
um lembrete de como os módulos funcionam.

- **Comece a partir da raiz da crate**: Ao compilar uma crate, o compilador
  primeiro procura no arquivo raiz da crate (geralmente _src/lib.rs_ para uma
  crate de biblioteca e _src/main.rs_ para uma crate binária) pelo código a ser
  compilado.
- **Declarando módulos**: No arquivo raiz da crate, você pode declarar novos
  módulos; digamos que você declare um módulo “garden” com `mod garden;`. O
  compilador procurará pelo código do módulo nestes lugares:
  - Embutido (inline), dentro de chaves que substituem o ponto e vírgula após
    `mod garden`
  - No arquivo _src/garden.rs_
  - No arquivo _src/garden/mod.rs_
- **Declarando submódulos**: Em qualquer arquivo que não seja a raiz da crate,
  você pode declarar submódulos. Por exemplo, você pode declarar `mod
  vegetables;` em _src/garden.rs_. O compilador procurará pelo código do
  submódulo dentro do diretório nomeado com o módulo pai nestes lugares:
  - Embutido, logo após `mod vegetables`, dentro de chaves em vez do ponto e
    vírgula
  - No arquivo _src/garden/vegetables.rs_
  - No arquivo _src/garden/vegetables/mod.rs_
- **Caminhos para o código nos módulos**: Uma vez que um módulo faz parte da
  sua crate, você pode se referir ao código nesse módulo de qualquer outro lugar
  na mesma crate, desde que as regras de privacidade permitam, usando o
  caminho para o código. Por exemplo, um tipo `Asparagus` no módulo de vegetais
  do jardim seria encontrado em `crate::garden::vegetables::Asparagus`.
- **Privado vs. público**: O código dentro de um módulo é privado em relação
  aos seus módulos pais por padrão. Para tornar um módulo público, declare-o
  com `pub mod` em vez de `mod`. Para tornar os itens dentro de um módulo
  público públicos também, use `pub` antes de suas declarações.
- **A palavra-chave `use`**: Dentro de um escopo, a palavra-chave `use` cria
  atalhos para itens para reduzir a repetição de caminhos longos. Em qualquer
  escopo que possa se referir a `crate::garden::vegetables::Asparagus`, você
  pode criar um atalho com `use crate::garden::vegetables::Asparagus;`, e a
  partir de então você só precisa escrever `Asparagus` para fazer uso desse
  tipo no escopo.

Aqui, criamos uma crate binária chamada `backyard` que ilustra essas regras. O
diretório da crate, também chamado de _backyard_, contém estes arquivos e
diretórios:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

O arquivo raiz da crate neste caso é _src/main.rs_, e ele contém:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

A linha `pub mod garden;` diz ao compilador para incluir o código que ele
encontra em _src/garden.rs_, que é:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Aqui, `pub mod vegetables;` significa que o código em
_src/garden/vegetables.rs_ também é incluído. Esse código é:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Agora vamos entrar nos detalhes dessas regras e demonstrá-las em ação!

### Agrupando Código Relacionado em Módulos

Os _Módulos_ nos permitem organizar o código dentro de uma crate para facilitar
a legibilidade e a reutilização. Os módulos também nos permitem controlar a
_privacidade_ dos itens porque o código dentro de um módulo é privado por
padrão. Itens privados são detalhes de implementação internos que não estão
disponíveis para uso externo. Podemos optar por tornar os módulos e os itens
dentro deles públicos, o que os expõe para permitir que código externo os use e
dependa deles.

Como exemplo, vamos escrever uma crate de biblioteca que fornece a
funcionalidade de um restaurante. Definiremos as assinaturas de funções, mas
deixaremos seus corpos vazios para nos concentrarmos na organização do código
em vez da implementação de um restaurante.

Na indústria de restaurantes, algumas partes de um restaurante são referidas
como salão (front of house) e outras como cozinha (back of house). O _salão_ é
onde os clientes estão; isso abrange onde os recepcionistas acomodam os
clientes, os garçons tiram pedidos e pagamentos, e os bartenders fazem bebidas.
A _cozinha_ é onde os chefs e cozinheiros trabalham no fogão, os lavadores de
pratos limpam e os gerentes fazem o trabalho administrativo.

Para estruturar nossa crate dessa maneira, podemos organizar suas funções em
módulos aninhados. Crie uma nova biblioteca chamada `restaurant` executando
`cargo new restaurant --lib`. Em seguida, insira o código da Listagem 7-1 em
_src/lib.rs_ para definir alguns módulos e assinaturas de funções; este código
é a seção do salão.

<Listing number="7-1" file-name="src/lib.rs" caption="Um módulo `front_of_house` contendo outros módulos que por sua vez contêm funções">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Definimos um módulo com a palavra-chave `mod` seguida pelo nome do módulo (neste
caso, `front_of_house`). O corpo do módulo fica então dentro de chaves. Dentro
dos módulos, podemos colocar outros módulos, como neste caso com os módulos
`hosting` e `serving`. Os módulos também podem conter definições para outros
itens, como structs, enums, constantes, traits e, como na Listagem 7-1, funções.

Ao usar módulos, podemos agrupar definições relacionadas e nomear o motivo de
serem relacionadas. Os programadores que usam este código podem navegar pelo
código com base nos grupos, em vez de terem que ler todas as definições,
tornando mais fácil encontrar as definições relevantes para eles. Programadores
que adicionam nova funcionalidade a este código saberão onde colocar o código
para manter o programa organizado.

Anteriormente, mencionamos que _src/main.rs_ e _src/lib.rs_ são chamados de
_raízes da crate_ (crate roots). O motivo do nome é que o conteúdo de
qualquer um desses dois arquivos forma um módulo chamado `crate` na raiz da
estrutura de módulos da crate, conhecido como _árvore de módulos_.

A Listagem 7-2 mostra a árvore de módulos para a estrutura da Listagem 7-1.

<Listing number="7-2" caption="A árvore de módulos para o código da Listagem 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Esta árvore mostra como alguns dos módulos se aninham dentro de outros módulos;
por exemplo, `hosting` se aninha dentro de `front_of_house`. A árvore também
mostra que alguns módulos são _irmãos_ (siblings), o que significa que eles são
definidos no mesmo módulo; `hosting` e `serving` são irmãos definidos dentro de
`front_of_house`. Se o módulo A estiver contido dentro do módulo B, dizemos que
o módulo A é o _filho_ (child) do módulo B e que o módulo B é o _pai_ (parent)
do módulo A. Observe que toda a árvore de módulos tem sua raiz sob o módulo
implícito chamado `crate`.

A árvore de módulos pode lembrá-lo da árvore de diretórios do sistema de
arquivos no seu computador; esta é uma comparação muito apropriada! Assim como
os diretórios em um sistema de arquivos, você usa módulos para organizar seu
código. E assim como os arquivos em um diretório, precisamos de uma maneira de
encontrar nossos módulos.
