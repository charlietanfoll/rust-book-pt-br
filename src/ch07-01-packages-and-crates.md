## Pacotes e Crates (Crates)

As primeiras partes do sistema de módulos que vamos cobrir são pacotes e crates.

Uma _crate_ é a menor quantidade de código que o compilador Rust considera de
uma só vez. Mesmo que você execute `rustc` em vez de `cargo` e passe um único
arquivo de código-fonte (como fizemos lá atrás em [“Noções Básicas de Programas
Rust”][basics]<!-- ignore --> no Capítulo 1), o compilador considera esse arquivo
como uma crate. Crates podem conter módulos, e os módulos podem ser definidos em
outros arquivos que são compilados com a crate, como veremos nas próximas seções.

Uma crate pode vir em uma de duas formas: uma crate binária ou uma crate de
biblioteca. _Crates binárias_ são programas que você pode compilar para um
executável que pode ser executado, como um programa de linha de comando ou um
servidor. Cada uma deve ter uma função chamada `main` que define o que acontece
quando o executável é rodado. Todas as crates que criamos até agora têm sido
crates binárias.

_Crates de biblioteca_ não têm uma função `main` e não são compiladas para um
executável. Em vez disso, elas definem funcionalidades destinadas a serem
compartilhadas com vários projetos. Por exemplo, a crate `rand` que usamos no
[Capítulo 2][rand]<!-- ignore --> fornece funcionalidade que gera números
aleatórios. Na maior parte do tempo, quando os "Rustaceans" dizem "crate", eles
querem dizer crate de biblioteca, e usam "crate" de forma intercambiável com o
conceito geral de programação de "biblioteca".

A _raiz da crate_ (_crate root_) é um arquivo de fonte a partir do qual o
compilador Rust começa e que compõe o módulo raiz da sua crate (vamos explicar
os módulos em profundidade em [“Controlando o Escopo e a Privacidade com
Módulos”][modules]<!-- ignore -->).

Um _pacote_ (_package_) é um conjunto de uma ou mais crates que fornece um
conjunto de funcionalidades. Um pacote contém um arquivo _Cargo.toml_ que descreve
como construir essas crates. O Cargo é, na verdade, um pacote que contém a crate
binária para a ferramenta de linha de comando que você tem usado para construir
seu código. O pacote Cargo também contém uma crate de biblioteca da qual a crate
binária depende. Outros projetos podem depender da crate de biblioteca do Cargo
para usar a mesma lógica que a ferramenta de linha de comando do Cargo usa.

Um pacote pode conter quantas crates binárias você quiser, mas no máximo apenas
uma crate de biblioteca. Um pacote deve conter pelo menos uma crate, seja ela uma
crate de biblioteca ou binária.

Vamos analisar o que acontece quando criamos um pacote. Primeiro, digitamos o
comando `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Depois de executarmos `cargo new my-project`, usamos `ls` para ver o que o Cargo
cria. No diretório _my-project_, há um arquivo _Cargo.toml_, nos dando um
pacote. Há também um diretório _src_ que contém _main.rs_. Abra o _Cargo.toml_
no seu editor de texto e note que não há menção a _src/main.rs_. O Cargo segue
uma convenção de que _src/main.rs_ é a raiz da crate de uma crate binária com o
mesmo nome do pacote. Da mesma forma, o Cargo sabe que se o diretório do pacote
contém _src/lib.rs_, o pacote contém uma crate de biblioteca com o mesmo nome do
pacote, e _src/lib.rs_ é a sua raiz. O Cargo passa os arquivos raiz da crate para
o `rustc` para construir a biblioteca ou o binário.

Aqui, temos um pacote que contém apenas _src/main.rs_, o que significa que ele
contém apenas uma crate binária chamada `my-project`. Se um pacote contém
_src/main.rs_ e _src/lib.rs_, ele tem duas crates: uma binária e uma biblioteca,
ambas com o mesmo nome do pacote. Um pacote pode ter várias crates binárias
colocando arquivos no diretório _src/bin_: Cada arquivo será uma crate binária
separada.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
