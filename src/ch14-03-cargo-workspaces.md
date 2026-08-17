## Workspaces do Cargo

No Capítulo 12, construímos um pacote que incluía uma crate binária e uma crate
de biblioteca. À medida que seu projeto se desenvolve, você pode notar que a
crate de biblioteca continua crescendo e você deseja dividir seu pacote ainda
mais em múltiplas crates de biblioteca. O Cargo oferece um recurso chamado
_workspaces_ (espaços de trabalho) que pode ajudar a gerenciar vários pacotes
relacionados que são desenvolvidos em conjunto.

### Criando um Workspace

Um _workspace_ é um conjunto de pacotes que compartilham o mesmo _Cargo.lock_
e diretório de saída. Vamos criar um projeto usando um workspace — usaremos um
código trivial para que possamos nos concentrar na estrutura do workspace. Há
várias maneiras de estruturar um workspace, então mostraremos apenas uma forma
comum. Teremos um workspace contendo um binário e duas bibliotecas. O binário,
que fornecerá a funcionalidade principal, dependerá das duas bibliotecas. Uma
biblioteca fornecerá uma função `add_one` e a outra biblioteca uma função
`add_two`. Essas três crates farão parte do mesmo workspace. Começaremos criando
um novo diretório para o workspace:

```console
$ mkdir add
$ cd add
```

Em seguida, no diretório _add_, criamos o arquivo _Cargo.toml_ que configurará
todo o workspace. Este arquivo não terá uma seção `[package]`. Em vez disso, ele
começará com uma seção `[workspace]` que nos permitirá adicionar membros ao
workspace. Também fazemos questão de usar a versão mais recente e avançada do
algoritmo de resolução do Cargo em nosso workspace, definindo o valor de
`resolver` como `"3"`:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Em seguida, criaremos a crate binária `adder` executando `cargo new` dentro do
diretório _add_:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Executar `cargo new` dentro de um workspace também adiciona automaticamente o
pacote recém-criado à chave `members` na definição `[workspace]` no _Cargo.toml_
do workspace, assim:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Neste ponto, podemos compilar o workspace executando `cargo build`. Os arquivos
em seu diretório _add_ devem se parecer com isto:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

O workspace possui um único diretório _target_ no nível superior, no qual os
artefatos compilados serão colocados; o pacote `adder` não tem seu próprio
diretório _target_. Mesmo se executássemos `cargo build` de dentro do diretório
_adder_, os artefatos compilados ainda iriam parar em _add/target_ em vez de
_add/adder/target_. O Cargo estrutura o diretório _target_ em um workspace desta
forma porque as crates em um workspace devem depender umas das outras. Se cada
crate tivesse seu próprio diretório _target_, cada crate teria que recompilar
cada uma das outras crates do workspace para colocar os artefatos em seu próprio
diretório _target_. Ao compartilhar um único diretório _target_, as crates
evitam a recompilação desnecessária.

### Criando o Segundo Pacote no Workspace

Em seguida, vamos criar outro pacote membro no workspace e chamá-lo de
`add_one`. Gere uma nova crate de biblioteca chamada `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

O _Cargo.toml_ de nível superior agora incluirá o caminho _add_one_ na lista de
`members`:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Seu diretório _add_ agora deve ter estes diretórios e arquivos:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

No arquivo _add_one/src/lib.rs_, vamos adicionar uma função `add_one`:

<span class="filename">Nome do arquivo: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Agora podemos fazer com que o pacote `adder` com nosso binário dependa do pacote
`add_one` que possui nossa biblioteca. Primeiro, precisaremos adicionar uma
dependência de caminho (_path dependency_) para `add_one` no arquivo
_adder/Cargo.toml_.

<span class="filename">Nome do arquivo: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

O Cargo não assume que crates em um workspace dependerão umas das outras, então
precisamos ser explícitos sobre as relações de dependência.

Em seguida, vamos usar a função `add_one` (da crate `add_one`) na crate `adder`.
Abra o arquivo _adder/src/main.rs_ e altere a função `main` para chamar a função
`add_one`, como na Listagem 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Usando a crate de biblioteca `add_one` a partir da crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Vamos compilar o workspace executando `cargo build` no diretório de nível
superior _add_!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Para executar a crate binária a partir do diretório _add_, podemos especificar
qual pacote no workspace queremos executar usando o argumento `-p` e o nome do
pacote com `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Isso executa o código em _adder/src/main.rs_, que depende da crate `add_one`.

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### Dependendo de um Pacote Externo

Observe que o workspace possui apenas um arquivo _Cargo.lock_ no nível
superior, em vez de ter um _Cargo.lock_ no diretório de cada crate. Isso garante
que todas as crates estejam usando a mesma versão de todas as dependências. Se
adicionarmos o pacote `rand` aos arquivos _adder/Cargo.toml_ e
_add_one/Cargo.toml_, o Cargo resolverá ambos para uma única versão de `rand` e
a registrará no único _Cargo.lock_. Fazer com que todas as crates do workspace
usem as mesmas dependências significa que as crates serão sempre compatíveis
entre si. Vamos adicionar a crate `rand` à seção `[dependencies]` no arquivo
_add_one/Cargo.toml_ para podermos usar a crate `rand` na crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Nome do arquivo: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Agora podemos adicionar `use rand;` ao arquivo _add_one/src/lib.rs_, e compilar
todo o workspace executando `cargo build` no diretório _add_ trará e compilará a
crate `rand`. Receberemos um aviso (_warning_) porque não estamos nos referindo
ao `rand` que trouxemos para o escopo:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.10.1
   --snip--
   Compiling rand v0.10.1
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` (part of `#[warn(unused)]`) on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

O _Cargo.lock_ de nível superior agora contém informações sobre a dependência de
`add_one` em relação ao `rand`. No entanto, embora o `rand` seja usado em algum
lugar do workspace, não podemos usá-lo em outras crates do workspace a menos que
adicionemos o `rand` aos arquivos _Cargo.toml_ delas também. Por exemplo, se
adicionarmos `use rand;` ao arquivo _adder/src/main.rs_ para o pacote `adder`,
obteremos um erro:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Para corrigir isso, edite o arquivo _Cargo.toml_ do pacote `adder` e indique que
`rand` também é uma dependência dele. Compilar o pacote `adder` adicionará o
`rand` à lista de dependências de `adder` no _Cargo.lock_, mas nenhuma cópia
adicional do `rand` será baixada. O Cargo garantirá que cada crate em cada
pacote do workspace que usa o pacote `rand` use a mesma versão, desde que eles
especifiquem versões compatíveis de `rand`, economizando espaço e garantindo
que as crates do workspace sejam compatíveis entre si.

Se as crates no workspace especificarem versões incompatíveis da mesma
dependência, o Cargo resolverá cada uma delas, mas ainda tentará resolver o
menor número possível de versões.

### Adicionando um Teste a um Workspace

Para outro aprimoramento, vamos adicionar um teste para a função
`add_one::add_one` dentro da crate `add_one`:

<span class="filename">Nome do arquivo: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Agora execute `cargo test` no diretório de nível superior _add_. Executar
`cargo test` em um workspace estruturado como este executará os testes para
todas as crates do workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

A primeira seção da saída mostra que o teste `it_works` na crate `add_one`
passou. A próxima seção mostra que zero testes foram encontrados na crate
`adder`, e então a última seção mostra que zero testes de documentação foram
encontrados na crate `add_one`.

Também podemos executar testes para uma crate específica em um workspace a
partir do diretório de nível superior usando a flag `-p` e especificando o nome
da crate que queremos testar:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Esta saída mostra que `cargo test` executou apenas os testes para a crate
`add_one` e não executou os testes da crate `adder`.

Se você publicar as crates do workspace no [crates.io](https://crates.io/)<!--
ignore -->, cada crate do workspace precisará ser publicada separadamente. Assim
como no `cargo test`, podemos publicar uma crate específica em nosso workspace
usando a flag `-p` e especificando o nome da crate que queremos publicar.

Para prática adicional, adicione uma crate `add_two` a este workspace da mesma
forma que a crate `add_one`!

À medida que seu projeto cresce, considere usar um workspace: ele permite que
você trabalhe com componentes menores e mais fáceis de entender do que um único
bloco enorme de código. Além disso, manter as crates em um workspace pode
tornar a coordenação entre as crates mais fácil se elas forem frequentemente
alteradas ao mesmo tempo.