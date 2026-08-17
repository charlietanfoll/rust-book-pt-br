## Organização de Testes

Como mencionado no início do capítulo, testes são uma disciplina complexa, e diferentes pessoas usam terminologias e organizações diferentes. A comunidade Rust pensa nos testes em termos de duas categorias principais: testes unitários e testes de integração. Os _testes unitários_ são pequenos e mais focados, testando um módulo isoladamente por vez, e podem testar interfaces privadas. Os _testes de integração_ são inteiramente externos à sua biblioteca e usam seu código da mesma forma que qualquer outro código externo faria, usando apenas a interface pública e potencialmente exercitando múltiplos módulos por teste.

Escrever ambos os tipos de testes é importante para garantir que as partes da sua biblioteca estejam fazendo o que você espera que façam, separadamente e em conjunto.

### Testes Unitários

O propósito dos testes unitários é testar cada unidade de código isoladamente do restante do código para identificar rapidamente onde o código está ou não funcionando como o esperado. Você colocará testes unitários no diretório _src_ em cada arquivo junto com o código que eles estão testando. A convenção é criar um módulo chamado `tests` em cada arquivo para conter as funções de teste e anotar o módulo com `cfg(test)`.

#### O Módulo `tests` e `#[cfg(test)]`

A anotação `#[cfg(test)]` no módulo `tests` diz ao Rust para compilar e executar o código de teste apenas quando você executa `cargo test`, e não quando você executa `cargo build`. Isso economiza tempo de compilação quando você deseja apenas construir a biblioteca e economiza espaço no artefato compilado resultante, pois os testes não são incluídos. Você verá que, como os testes de integração vão para um diretório diferente, eles não precisam da anotação `#[cfg(test)]`. No entanto, como os testes unitários ficam nos mesmos arquivos que o código, você usará `#[cfg(test)]` para especificar que eles não devem ser incluídos no resultado compilado.

Lembre-se de que, quando geramos o novo projeto `adder` na primeira seção deste capítulo, o Cargo gerou este código para nós:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

No módulo `tests` gerado automaticamente, o atributo `cfg` significa _configuration_ (configuração) e diz ao Rust que o item a seguir só deve ser incluído dada uma determinada opção de configuração. Nesse caso, a opção de configuração é `test`, que é fornecida pelo Rust para compilar e executar testes. Ao usar o atributo `cfg`, o Cargo compila nosso código de teste apenas se executarmos ativamente os testes com `cargo test`. Isso inclui quaisquer funções auxiliares que possam estar dentro deste módulo, além das funções anotadas com `#[test]`.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### Testes de Funções Privadas

Há um debate na comunidade de testes sobre se as funções privadas devem ou não ser testadas diretamente, e outras linguagens tornam difícil ou impossível testar funções privadas. Independentemente da ideologia de testes à qual você aderir, as regras de privacidade do Rust permitem que você teste funções privadas. Considere o código na Listagem 11-12 com a função privada `internal_adder`.

<Listing number="11-12" file-name="src/lib.rs" caption="Testando uma função privada">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Observe que a função `internal_adder` não está marcada como `pub`. Os testes são apenas código Rust, e o módulo `tests` é apenas mais um módulo. Como discutimos em [“Caminhos para Referenciar um Item na Árvore de Módulos”][paths]<!-- ignore -->, os itens em módulos filhos podem usar os itens em seus módulos ancestrais. Neste teste, trazemos todos os itens pertencentes ao pai do módulo `tests` para o escopo com `use super::*`, e então o teste pode chamar `internal_adder`. Se você acha que funções privadas não devem ser testadas, não há nada no Rust que o obrigue a fazê-lo.

### Testes de Integração

Em Rust, os testes de integração são inteiramente externos à sua biblioteca. Eles usam sua biblioteca da mesma forma que qualquer outro código faria, o que significa que eles só podem chamar funções que fazem parte da API pública da sua biblioteca. O propósito deles é testar se muitas partes da sua biblioteca funcionam juntas corretamente. Unidades de código que funcionam corretamente por conta própria podem ter problemas quando integradas, portanto, a cobertura de teste do código integrado também é importante. Para criar testes de integração, você primeiro precisa de um diretório _tests_.

#### O Diretório _tests_

Criamos um diretório _tests_ no nível superior do diretório do nosso projeto, ao lado de _src_. O Cargo sabe que deve procurar arquivos de teste de integração neste diretório. Podemos então criar quantos arquivos de teste quisermos, e o Cargo compilará cada um dos arquivos como um crate individual.

Vamos criar um teste de integração. Com o código da Listagem 11-12 ainda no arquivo _src/lib.rs_, crie um diretório _tests_ e crie um novo arquivo chamado _tests/integration_test.rs_. A estrutura do seu diretório deve se parecer com isto:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Insira o código da Listagem 11-13 no arquivo _tests/integration_test.rs_.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="Um teste de integração de uma função no crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Cada arquivo no diretório _tests_ é um crate separado, então precisamos trazer nossa biblioteca para o escopo de cada crate de teste. Por esse motivo, adicionamos `use adder::add_two;` no topo do código, o que não precisávamos nos testes unitários.

Não precisamos anotar nenhum código em _tests/integration_test.rs_ com `#[cfg(test)]`. O Cargo trata o diretório _tests_ de forma especial e compila os arquivos neste diretório apenas quando executamos `cargo test`. Execute `cargo test` agora:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

As três seções de saída incluem os testes unitários, o teste de integração e os testes de documentação (doc tests). Observe que se algum teste em uma seção falhar, as seções seguintes não serão executadas. Por exemplo, se um teste unitário falhar, não haverá nenhuma saída para os testes de integração e de documentação, porque esses testes só serão executados se todos os testes unitários passarem.

A primeira seção para os testes unitários é a mesma que temos visto: uma linha para cada teste unitário (uma chamada `internal` que adicionamos na Listagem 11-12) e, em seguida, uma linha de resumo para os testes unitários.

A seção de testes de integração começa com a linha `Running tests/integration_test.rs`. Em seguida, há uma linha para cada função de teste nesse teste de integração e uma linha de resumo para os resultados do teste de integração logo antes da seção `Doc-tests adder` começar.

Cada arquivo de teste de integração tem sua própria seção, então se adicionarmos mais arquivos no diretório _tests_, haverá mais seções de teste de integração.

Ainda podemos executar uma função específica de teste de integração especificando o nome da função de teste como um argumento para `cargo test`. Para executar todos os testes em um arquivo de teste de integração específico, use o argumento `--test` do `cargo test` seguido pelo nome do arquivo:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Este comando executa apenas os testes no arquivo _tests/integration_test.rs_.

#### Submódulos em Testes de Integração

À medida que você adiciona mais testes de integração, você pode querer fazer mais arquivos no diretório _tests_ para ajudar a organizá-los; por exemplo, você pode agrupar as funções de teste pela funcionalidade que elas estão testando. Como mencionado anteriormente, cada arquivo no diretório _tests_ é compilado como seu próprio crate separado, o que é útil para criar escopos separados para imitar mais de perto a maneira como os usuários finais usarão seu crate. No entanto, isso significa que os arquivos no diretório _tests_ não compartilham o mesmo comportamento que os arquivos em _src_ fazem, como você aprendeu no Capítulo 7 sobre como separar o código em módulos e arquivos.

O comportamento diferente dos arquivos do diretório _tests_ é mais perceptível quando você tem um conjunto de funções auxiliares para usar em vários arquivos de teste de integração e tenta seguir os passos na seção [“Separando Módulos em Diferentes Arquivos”][separating-modules-into-files]<!-- ignore --> do Capítulo 7 para extraí-las para um módulo comum. Por exemplo, se criarmos _tests/common.rs_ e colocarmos uma função chamada `setup` nele, podemos adicionar algum código a `setup` que queremos chamar de várias funções de teste em vários arquivos de teste:

<span class="filename">Nome do arquivo: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Quando executamos os testes novamente, veremos uma nova seção na saída do teste para o arquivo _common.rs_, mesmo que este arquivo não contenha nenhuma função de teste nem tenhamos chamado a função `setup` de lugar nenhum:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Ter `common` aparecendo nos resultados dos testes com `running 0 tests` exibido para ele não é o que queríamos. Nós só queríamos compartilhar algum código com os outros arquivos de teste de integração. Para evitar que `common` apareça na saída do teste, em vez de criar _tests/common.rs_, criaremos _tests/common/mod.rs_. O diretório do projeto agora se parece com isto:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Esta é a convenção de nomenclatura mais antiga que o Rust também entende, que mencionamos em [“Caminhos de Arquivo Alternativos”][alt-paths]<!-- ignore --> no Capítulo 7. Nomear o arquivo dessa maneira diz ao Rust para não tratar o módulo `common` como um arquivo de teste de integração. Quando movemos o código da função `setup` para _tests/common/mod.rs_ e excluímos o arquivo _tests/common.rs_, a seção na saída do teste não aparecerá mais. Os arquivos em subdiretórios do diretório _tests_ não são compilados como crates separados nem possuem seções na saída do teste.

Depois de criarmos _tests/common/mod.rs_, podemos usá-lo a partir de qualquer um dos arquivos de teste de integração como um módulo. Aqui está um exemplo de chamada da função `setup` a partir do teste `it_adds_two` em _tests/integration_test.rs_:

<span class="filename">Nome do arquivo: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Observe que a declaração `mod common;` é a mesma que a declaração de módulo que demonstramos na Listagem 7-21. Em seguida, na função de teste, podemos chamar a função `common::setup()`.

#### Testes de Integração para Crates Binários

Se o nosso projeto for um crate binário que contém apenas um arquivo _src/main.rs_ e não possui um arquivo _src/lib.rs_, não podemos criar testes de integração no diretório _tests_ e trazer funções definidas no arquivo _src/main.rs_ para o escopo com uma instrução `use`. Apenas crates de biblioteca expõem funções que outros crates podem usar; crates binários são feitos para serem executados por conta própria.

Esta é uma das razões pelas quais os projetos Rust que fornecem um binário têm um arquivo _src/main.rs_ simples que chama a lógica que reside no arquivo _src/lib.rs_. Usando essa estrutura, os testes de integração _podem_ testar o crate de biblioteca com `use` para disponibilizar a principal funcionalidade. Se a principal funcionalidade funcionar, a pequena quantidade de código no arquivo _src/main.rs_ funcionará também, e essa pequena quantidade de código não precisa ser testada.

## Resumo

Os recursos de teste do Rust fornecem uma maneira de especificar como o código deve funcionar para garantir que ele continue funcionando como você espera, mesmo à medida que você faz alterações. Os testes unitários exercitam diferentes partes de uma biblioteca separadamente e podem testar detalhes de implementação privada. Os testes de integração verificam se muitas partes da biblioteca funcionam juntas corretamente e usam a API pública da biblioteca para testar o código da mesma forma que o código externo o usará. Mesmo que o sistema de tipos e as regras de propriedade do Rust ajudem a prevenir alguns tipos de bugs, os testes ainda são importantes para reduzir bugs lógicos relacionados a como o seu código deve se comportar.

Vamos combinar o conhecimento que você aprendeu neste capítulo e em capítulos anteriores para trabalhar em um projeto!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
