## Separando Módulos em Diferentes Arquivos

Até agora, todos os exemplos neste capítulo definiram múltiplos módulos em um único arquivo. Quando os módulos crescem, você pode querer mover suas definições para um arquivo separado para facilitar a navegação pelo código.

Por exemplo, vamos começar a partir do código na Listagem 7-17 que tinha múltiplos módulos de restaurantes. Vamos extrair os módulos para arquivos em vez de ter todos os módulos definidos no arquivo raiz da crate. Neste caso, o arquivo raiz da crate é _src/lib.rs_, mas este procedimento também funciona com crates binárias cujo arquivo raiz é _src/main.rs_.

Primeiro, vamos extrair o módulo `front_of_house` para seu próprio arquivo. Remova o código dentro das chaves do módulo `front_of_house`, deixando apenas a declaração `mod front_of_house;`, de modo que _src/lib.rs_ contenha o código mostrado na Listagem 7-21. Note que isso não vai compilar até criarmos o arquivo _src/front_of_house.rs_ na Listagem 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="Declarando o módulo `front_of_house` cujo corpo estará em *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Em seguida, coloque o código que estava dentro das chaves em um novo arquivo chamado _src/front_of_house.rs_, conforme mostrado na Listagem 7-22. O compilador sabe onde procurar por este arquivo porque ele encontrou a declaração do módulo na raiz da crate com o nome `front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="Definições dentro do módulo `front_of_house` em *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Note que você só precisa carregar um arquivo usando uma declaração `mod` _uma vez_ na sua árvore de módulos. Uma vez que o compilador sabe que o arquivo faz parte do projeto (e sabe onde na árvore de módulos o código reside por causa de onde você colocou a instrução `mod`), outros arquivos no seu projeto devem se referir ao código do arquivo carregado usando um caminho para onde ele foi declarado, como abordado na seção [“Caminhos para Referenciar um Item na Árvore de Módulos”][paths]<!-- ignore -->. Em outras palavras, `mod` _não_ é uma operação de "inclusão" que você pode ter visto em outras linguagens de programação.

Em seguida, vamos extrair o módulo `hosting` para seu próprio arquivo. O processo é um pouco diferente porque `hosting` é um submódulo de `front_of_house`, e não do módulo raiz. Vamos colocar o arquivo para `hosting` em um novo diretório que será nomeado de acordo com seus ancestrais na árvore de módulos, neste caso _src/front_of_house_.

Para começar a mover `hosting`, alteramos _src/front_of_house.rs_ para conter apenas a declaração do módulo `hosting`:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Então, criamos um diretório _src/front_of_house_ e um arquivo _hosting.rs_ para conter as definições feitas no módulo `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Se em vez disso colocássemos _hosting.rs_ no diretório _src_, o compilador esperaria que o código de _hosting.rs_ estivesse em um módulo `hosting` declarado na raiz da crate e não declarado como um filho do módulo `front_of_house`. As regras do compilador sobre quais arquivos verificar para o código de quais módulos significam que os diretórios e arquivos correspondem mais de perto à árvore de módulos.

> ### Caminhos de Arquivo Alternativos
>
> Até agora cobrimos os caminhos de arquivo mais idiomáticos que o compilador Rust usa, mas o Rust também suporta um estilo mais antigo de caminho de arquivo. Para um módulo chamado `front_of_house` declarado na raiz da crate, o compilador procurará o código do módulo em:
>
> - _src/front_of_house.rs_ (o que cobrimos)
> - _src/front_of_house/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Para um módulo chamado `hosting` que é um submódulo de `front_of_house`, o compilador procurará o código do módulo em:
>
> - _src/front_of_house/hosting.rs_ (o que cobrimos)
> - _src/front_of_house/hosting/mod.rs_ (estilo mais antigo, caminho ainda suportado)
>
> Se você usar ambos os estilos para o mesmo módulo, você receberá um erro do compilador. Usar uma mistura de ambos os estilos para diferentes módulos no mesmo projeto é permitido, mas pode ser confuso para pessoas navegando pelo seu projeto.
>
> A principal desvantagem do estilo que usa arquivos chamados _mod.rs_ é que seu projeto pode acabar com muitos arquivos chamados _mod.rs_, o que pode se tornar confuso quando você os tiver abertos no seu editor ao mesmo tempo.

Nós movemos o código de cada módulo para um arquivo separado, e a árvore de módulos permanece a mesma. As chamadas de função em `eat_at_restaurant` funcionarão sem nenhuma modificação, mesmo que as definições vivam em arquivos diferentes. Esta técnica permite que você mova módulos para novos arquivos conforme eles crescem em tamanho.

Note que a instrução `pub use crate::front_of_house::hosting` em _src/lib.rs_ também não mudou, nem `use` tem qualquer impacto sobre quais arquivos são compilados como parte da crate. A palavra-chave `mod` declara módulos, e o Rust procura em um arquivo com o mesmo nome do módulo pelo código que vai para dentro desse módulo.

## Resumo

O Rust permite que você divida um pacote em múltiplas crates e uma crate em módulos para que você possa se referir a itens definidos em um módulo a partir de outro módulo. Você pode fazer isso especificando caminhos absolutos ou relativos. Esses caminhos podem ser trazidos para o escopo com uma instrução `use` para que você possa usar um caminho mais curto para múltiplos usos do item naquele escopo. O código do módulo é privado por padrão, mas você pode tornar as definições públicas adicionando a palavra-chave `pub`.

No próximo capítulo, veremos algumas estruturas de dados de coleções na biblioteca padrão que você pode usar em seu código perfeitamente organizado.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
