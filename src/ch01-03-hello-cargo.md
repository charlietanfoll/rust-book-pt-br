## Olá, Cargo!

O Cargo é o sistema de build e gerenciador de pacotes do Rust. A maioria dos
Rustaceans usa essa ferramenta para gerenciar seus projetos Rust porque o Cargo
lida com várias tarefas para você, como compilar seu código, baixar as
bibliotecas de que seu código depende e compilar essas bibliotecas. (Chamamos as
bibliotecas de que seu código precisa de _dependências_.)

Os programas em Rust mais simples, como o que escrevemos até agora, não têm
nenhuma dependência. Se tivéssemos criado o projeto “Olá, mundo!” com o Cargo,
ele usaria apenas a parte do Cargo que lida com a compilação do seu código. À
medida que você escreve programas em Rust mais complexos, você adiciona
dependências, e se você iniciar um projeto usando o Cargo, adicionar
dependências será muito mais fácil de fazer.

Como a grande maioria dos projetos Rust usa o Cargo, o resto deste livro assume
que você também está usando o Cargo. O Cargo vem instalado com o Rust se você
usou os instaladores oficiais discutidos na seção
[“Instalação”][installation]<!-- ignore -->. Se você instalou o Rust por algum
outro meio, verifique se o Cargo está instalado digitando o seguinte no seu
terminal:

```console
$ cargo --version
```

Se você vir um número de versão, você o tem! Se você ver um erro, como `command
not found` (comando não encontrado), consulte a documentação do seu método de
instalação para determinar como instalar o Cargo separadamente.

### Criando um Projeto com o Cargo

Vamos criar um novo projeto usando o Cargo e ver como ele difere do nosso
projeto original “Olá, mundo!”. Volte para o seu diretório _projects_ (ou onde
quer que você tenha decidido armazenar seu código). Em seguida, em qualquer
sistema operacional, execute o seguinte:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

O primeiro comando cria um novo diretório e projeto chamado _hello_cargo_.
Nomeamos nosso projeto de _hello_cargo_, e o Cargo cria seus arquivos em um
diretório com o mesmo nome.

Entre no diretório _hello_cargo_ e liste os arquivos. Você verá que o Cargo
gerou dois arquivos e um diretório para nós: um arquivo _Cargo.toml_ e um
diretório _src_ com um arquivo _main.rs_ dentro.

Ele também inicializou um novo repositório Git junto com um arquivo
_.gitignore_. Os arquivos do Git não serão gerados se você executar `cargo new`
dentro de um repositório Git existente; você pode substituir esse comportamento
usando `cargo new --vcs=git`.

> Nota: O Git é um sistema de controle de versão comum. Você pode alterar o
> `cargo new` para usar um sistema de controle de versão diferente ou nenhum
> sistema de controle de versão usando a flag `--vcs`. Execute `cargo new --help`
> para ver as opções disponíveis.

Abra o _Cargo.toml_ no seu editor de texto de preferência. Ele deve ser
semelhante ao código da Listagem 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Conteúdo de *Cargo.toml* gerado por `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Este arquivo está no formato [_TOML_][toml]<!-- ignore --> (_Tom’s Obvious,
Minimal Language_ — Linguagem Óbvia e Mínima do Tom), que é o formato de
configuração do Cargo.

A primeira linha, `[package]`, é um cabeçalho de seção que indica que as
instruções a seguir estão configurando um pacote. Conforme adicionarmos mais
informações a este arquivo, adicionaremos outras seções.

As três linhas seguintes definem as informações de configuração que o Cargo
precisa para compilar seu programa: o nome, a versão e a edição do Rust a ser
usada. Falaremos sobre a chave `edition` no [Apêndice E][appendix-e]<!-- ignore
-->.

A última linha, `[dependencies]`, é o início de uma seção para você listar
qualquer uma das dependências do seu projeto. Em Rust, pacotes de código são
chamados de _crates_. Não precisaremos de nenhum outro crate para este projeto,
mas precisaremos no primeiro projeto do Capítulo 2, então usaremos esta seção de
dependências nessa ocasião.

Agora abra _src/main.rs_ e dê uma olhada:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

O Cargo gerou um programa “Olá, mundo!” para você, assim como o que escrevemos
na Listagem 1-1! Até agora, as diferenças entre o nosso projeto e o projeto que
o Cargo gerou são que o Cargo colocou o código no diretório _src_, e nós temos
um arquivo de configuração _Cargo.toml_ no diretório raiz.

O Cargo espera que seus arquivos fonte fiquem dentro do diretório _src_. O
diretório raiz do projeto é apenas para arquivos README, informações de
licença, arquivos de configuração e qualquer outra coisa não relacionada ao
seu código. Usar o Cargo ajuda você a organizar seus projetos. Há um lugar para
cada coisa, e cada coisa está em seu lugar.

Se você iniciou um projeto que não usa o Cargo, como fizemos com o projeto
“Olá, mundo!”, você pode convertê-lo em um projeto que usa o Cargo. Mova o
código do projeto para o diretório _src_ e crie um arquivo _Cargo.toml_
adequado. Uma maneira fácil de obter esse arquivo _Cargo.toml_ é executar
`cargo init`, que o criará para você automaticamente.

### Compilando e Executando um Projeto Cargo

Agora vamos ver o que é diferente quando compilamos e executamos o programa
“Olá, mundo!” com o Cargo! A partir do seu diretório _hello_cargo_, compile seu
projeto digitando o seguinte comando:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Este comando cria um arquivo executável em _target/debug/hello_cargo_ (ou
_target\debug\hello_cargo.exe_ no Windows) em vez do seu diretório atual. Como
o build padrão é um build de depuração (debug), o Cargo coloca o binário em um
diretório chamado _debug_. Você pode executar o executável com este comando:

```console
$ ./target/debug/hello_cargo # ou .\target\debug\hello_cargo.exe no Windows
Hello, world!
```

Se tudo correr bem, `Hello, world!` deve ser impresso no terminal. Executar
`cargo build` pela primeira vez também faz com que o Cargo crie um novo arquivo
no nível raiz: _Cargo.lock_. Este arquivo rastreia as versões exatas das
dependências no seu projeto. Este projeto não tem dependências, então o arquivo
está um pouco vazio. Você nunca precisará alterar este arquivo manualmente; o
Cargo gerencia seu conteúdo para você.

Acabamos de compilar um projeto com `cargo build` e o executamos com
`./target/debug/hello_cargo`, mas também podemos usar `cargo run` para compilar
o código e, em seguida, executar o executável resultante em um único comando:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Usar `cargo run` é mais conveniente do que ter que se lembrar de executar
`cargo build` e depois usar o caminho completo para o binário, então a maioria
dos desenvolvedores usa `cargo run`.

Note que desta vez não vimos nenhuma saída indicando que o Cargo estava
compilando o `hello_cargo`. O Cargo percebeu que os arquivos não haviam
mudado, então ele não recompilou, apenas executou o binário. Se você tivesse
modificado seu código-fonte, o Cargo teria recompilado o projeto antes de
executá-lo, e você teria visto esta saída:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

O Cargo também fornece um comando chamado `cargo check`. Este comando verifica
rapidamente o seu código para garantir que ele compila, mas não produz um
executável:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Por que você não iria querer um executável? Frequentemente, o `cargo check` é
muito mais rápido do que o `cargo build` porque ele pula a etapa de produzir um
executável. Se você estiver verificando continuamente seu trabalho enquanto
escreve o código, usar o `cargo check` acelerará o processo de informá-lo se o
seu projeto ainda está compilando! Como tal, muitos Rustaceans executam o
`cargo check` periodicamente enquanto escrevem seu programa para garantir que
ele compila. Em seguida, eles executam o `cargo build` quando estão prontos para
usar o executável.

Vamos recapitular o que aprendemos até agora sobre o Cargo:

- Podemos criar um projeto usando `cargo new`.
- Podemos compilar um projeto usando `cargo build`.
- Podemos compilar e executar um projeto em uma única etapa usando `cargo run`.
- Podemos compilar um projeto sem produzir um binário para verificar erros
  usando `cargo check`.
- Em vez de salvar o resultado da compilação no mesmo diretório do nosso código,
  o Cargo o armazena no diretório _target/debug_.

Uma vantagem adicional de usar o Cargo é que os comandos são os mesmos, não
importa em qual sistema operacional você esteja trabalhando. Portanto, a partir
deste ponto, não forneceremos mais instruções específicas para Linux e macOS
versus Windows.

### Compilando para Lançamento (Release)

Quando seu projeto estiver finalmente pronto para lançamento, você pode usar
`cargo build --release` para compilá-lo com otimizações. Este comando criará
um executável em _target/release_ em vez de _target/debug_. As otimizações
fazem seu código Rust rodar mais rápido, mas ativá-las aumenta o tempo que seu
programa leva para compilar. É por isso que existem dois perfis diferentes: um
para o desenvolvimento, quando você deseja recompilar de forma rápida e
frequente, e outro para construir o programa final que você dará a um usuário,
o que não será recompilado repetidamente e funcionará o mais rápido possível.
Se você estiver fazendo benchmarks do tempo de execução do seu código, certifique-se
de executar `cargo build --release` e fazer o benchmark com o executável em
_target/release_.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Aproveitando as Convenções do Cargo

Com projetos simples, o Cargo não oferece muito valor em comparação com o uso
simples do `rustc`, mas ele provará seu valor à medida que seus programas se
tornarem mais complexos. Uma vez que os programas crescem para vários arquivos
ou precisam de uma dependência, é muito mais fácil deixar o Cargo coordenar o
build.

Mesmo que o projeto `hello_cargo` seja simples, ele agora usa grande parte das
ferramentas reais que você usará no resto da sua carreira em Rust. De fato, para
trabalhar em qualquer projeto existente, você pode usar os seguintes comandos
para baixar o código usando o Git, mudar para o diretório desse projeto e
compilar:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Para obter mais informações sobre o Cargo, consulte [sua
documentação][cargo].

## Resumo

Você já começou muito bem sua jornada no Rust! Neste capítulo, você aprendeu
como:

- Instalar a versão estável mais recente do Rust usando o `rustup`.
- Atualizar para uma versão mais recente do Rust.
- Abrir a documentação instalada localmente.
- Escrever e executar um programa “Olá, mundo!” usando o `rustc` diretamente.
- Criar e executar um novo projeto usando as convenções do Cargo.

Este é um ótimo momento para construir um programa mais substancial para se
acostumar a ler e escrever código Rust. Então, no Capítulo 2, construiremos um
programa de jogo de adivinhação. Se você preferir começar aprendendo como os
conceitos comuns de programação funcionam em Rust, consulte o Capítulo 3 e
depois retorne ao Capítulo 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
