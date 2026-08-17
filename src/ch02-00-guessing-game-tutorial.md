# Programando um Jogo de Adivinhação

Vamos mergulhar no Rust trabalhando juntos em um projeto prático! Este capítulo
apresenta alguns conceitos comuns do Rust, mostrando como usá-los em um programa
real. Você aprenderá sobre `let`, `match`, métodos, funções associadas, crates
externos e muito mais! Nos próximos capítulos, exploraremos essas ideias com mais
detalhes. Neste capítulo, você apenas praticará os fundamentos.

Vamos implementar um problema clássico de programação para iniciantes: um jogo
de adivinhação. Funciona assim: o programa gerará um número inteiro aleatório
entre 1 e 100. Em seguida, ele pedirá ao jogador para inserir um palpite. Depois
que um palpite for inserido, o programa indicará se o palpite está muito baixo
ou muito alto. Se o palpite estiver correto, o jogo exibirá uma mensagem de
parabéns e será encerrado.

## Configurando um Novo Projeto

Para configurar um novo projeto, vá para o diretório _projects_ que você criou no
Capítulo 1 e crie um novo projeto usando o Cargo, da seguinte forma:

```console
$ cargo new guessing_game
$ cd guessing_game
```

O primeiro comando, `cargo new`, recebe o nome do projeto (`guessing_game`)
como o primeiro argumento. O segundo comando muda para o diretório do novo
projeto.

Olhe para o arquivo _Cargo.toml_ gerado:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Como você viu no Capítulo 1, o `cargo new` gera um programa “Hello, world!” para
você. Confira o arquivo _src/main.rs_:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Agora vamos compilar este programa “Hello, world!” e executá-lo na mesma etapa
usando o comando `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

O comando `run` é útil quando você precisa iterar rapidamente em um projeto,
como faremos neste jogo, testando rapidamente cada iteração antes de passar para
a próxima.

Reabra o arquivo _src/main.rs_. Você escreverá todo o código neste arquivo.

## Processando um Palpite

A primeira parte do programa do jogo de adivinhação solicitará a entrada do
usuário, processará essa entrada e verificará se a entrada está no formato
esperado. Para começar, permitiremos que o jogador insira um palpite. Insira o
código da Listagem 2-1 em _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Código que obtém um palpite do usuário e o imprime">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Este código contém muitas informações, então vamos analisá-lo linha por linha. Para
obter a entrada do usuário e, em seguida, imprimir o resultado como saída, precisamos
trazer a biblioteca de entrada/saída `io` para o escopo. A biblioteca `io` vem da
biblioteca padrão, conhecida como `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Por padrão, o Rust tem um conjunto de itens definidos na biblioteca padrão que ele
traz para o escopo de todo programa. Esse conjunto é chamado de _prelude_, e
você pode ver tudo o que está nele [na documentação da biblioteca padrão][prelude].

Se o tipo que você deseja usar não estiver no prelude, você precisará trazer esse
tipo para o escopo explicitamente com uma instrução `use`. O uso da biblioteca
`std::io` fornece vários recursos úteis, incluindo a capacidade de aceitar a
entrada do usuário.

Como você viu no Capítulo 1, a função `main` é o ponto de entrada do programa:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

A sintaxe `fn` declara uma nova função; os parênteses, `()`, indicam que
não há parâmetros; e a chave, `{`, inicia o corpo da função.

Como você também aprendeu no Capítulo 1, `println!` é uma macro que imprime uma string na
tela:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Este código está imprimindo um aviso indicando o que é o jogo e solicitando uma
entrada do usuário.

### Armazenando Valores com Variáveis

Em seguida, criaremos uma _variável_ para armazenar a entrada do usuário, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Agora o programa está ficando interessante! Muita coisa está acontecendo nesta
pequena linha. Usamos a instrução `let` para criar a variável. Aqui está outro exemplo:

```rust,ignore
let apples = 5;
```

Esta linha cria uma nova variável chamada `apples` e a associa ao valor `5`.
No Rust, as variáveis são imutáveis por padrão, o que significa que, uma vez que
damos um valor à variável, o valor não mudará. Discutiremos esse conceito em
detalhes na seção [“Variáveis e Mutabilidade”][variables-and-mutability]<!-- ignore -->
no Capítulo 3. Para tornar uma variável mutável, adicionamos `mut` antes do
nome da variável:

```rust,ignore
let apples = 5; // imutável
let mut bananas = 5; // mutável
```

> Nota: A sintaxe `//` inicia um comentário que continua até o final da
> linha. O Rust ignora tudo dentro dos comentários. Discutiremos os comentários com
> mais detalhes no [Capítulo 3][comments]<!-- ignore -->.

Voltando ao programa do jogo de adivinhação, agora você sabe que `let mut guess`
introduzirá uma variável mutável chamada `guess`. O sinal de igual (`=`) diz ao
Rust que queremos associar algo à variável agora. À direita do sinal de igual está
o valor ao qual `guess` está associado, que é o resultado da chamada de
`String::new`, uma função que retorna uma nova instância de uma `String`.
[`String`][string]<!-- ignore --> é um tipo de string fornecido pela biblioteca
padrão que é um pedaço de texto redimensionável codificado em UTF-8.

A sintaxe `::` na linha `::new` indica que `new` é uma função associada
do tipo `String`. Uma _função associada_ é uma função implementada em um
tipo, neste caso `String`. Esta função `new` cria uma string nova e vazia.
Você encontrará uma função `new` em muitos tipos porque é um nome comum
para uma função que cria um novo valor de algum tipo.

Em suma, a linha `let mut guess = String::new();` criou uma variável mutável
que está atualmente associada a uma nova instância vazia de uma `String`. Ufa!

### Recebendo a Entrada do Usuário

Lembre-se de que incluímos a funcionalidade de entrada/saída da biblioteca
padrão com `use std::io;` na primeira linha do programa. Agora chamaremos
a função `stdin` do módulo `io`, o que nos permitirá lidar com a entrada
do usuário:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Se não tivéssemos importado o módulo `io` com `use std::io;` no início do
programa, ainda poderíamos usar a função escrevendo esta chamada de função como
`std::io::stdin`. A função `stdin` retorna uma instância de
[`std::io::Stdin`][iostdin]<!-- ignore -->, que é um tipo que representa um
manipulador (handle) para a entrada padrão do seu terminal.

Em seguida, a linha `.read_line(&mut guess)` chama o método [`read_line`][read_line]<!--
ignore --> no manipulador de entrada padrão para obter a entrada do usuário.
Também estamos passando `&mut guess` como argumento para `read_line` para dizer a
ela em qual string armazenar a entrada do usuário. O trabalho completo de
`read_line` é pegar o que o usuário digitar na entrada padrão e anexar isso a
uma string (sem sobrescrever seu conteúdo), portanto, passamos essa string como
argumento. O argumento de string precisa ser mutável para que o método possa alterar
o conteúdo da string.

O `&` indica que este argumento é uma _referência_, o que oferece uma maneira de
permitir que várias partes do seu código acessem um único pedaço de dados sem a
necessidade de copiar esses dados para a memória várias vezes. As referências são
um recurso complexo, e uma das principais vantagens do Rust é como é seguro e
fácil usar referências. Você não precisa saber muitos desses detalhes para terminar
este programa. Por enquanto, tudo o que você precisa saber é que, assim como as
variáveis, as referências são imutáveis por padrão. Portanto, você precisa escrever
`&mut guess` em vez de `&guess` para torná-la mutável. (O Capítulo 4 explicará as
referências mais detalhadamente.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Lidando com Possíveis Falhas com o Tipo `Result`

Ainda estamos trabalhando nesta linha de código. Agora estamos discutindo uma terceira
linha de texto, mas note que ela ainda faz parte de uma única linha lógica de
código. A próxima parte é este método:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Poderíamos ter escrito este código como:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

No entanto, uma linha longa é difícil de ler, por isso é melhor dividi-la. Geralmente
é sensato introduzir uma quebra de linha e outros espaços em branco para ajudar a
dividir linhas longas quando você chama um método com a sintaxe `.method_name()`.
Agora vamos discutir o que esta linha faz.

Como mencionado anteriormente, `read_line` coloca o que o usuário digitar na string
que passamos para ele, mas também retorna um valor `Result`. [`Result`][result]<!--
ignore --> é uma [_enumeração_][enums]<!-- ignore -->, frequentemente chamada de
_enum_, que é um tipo que pode estar em um de vários estados possíveis. Chamamos
cada estado possível de _variante_.

O [Capítulo 6][enums]<!-- ignore --> abordará enums com mais detalhes. O propósito
desses tipos `Result` é codificar informações de tratamento de erros.

As variantes de `Result` são `Ok` e `Err`. A variante `Ok` indica que a
operação foi bem-sucedida e contém o valor gerado com sucesso. A variante `Err`
significa que a operação falhou e contém informações sobre como ou por que a
operação falhou.

Os valores do tipo `Result`, assim como os valores de qualquer tipo, possuem métodos
definidos neles. Uma instância de `Result` tem um [`método expect`][expect]<!-- ignore -->
que você pode chamar. Se esta instância de `Result` for um valor `Err`, o `expect`
fará com que o programa feche abruptamente (panic) e exiba a mensagem que você passou
como argumento para o `expect`. Se o método `read_line` retornar um `Err`, provavelmente
será o resultado de um erro proveniente do sistema operacional subjacente. Se esta
instância de `Result` for um valor `Ok`, o `expect` pegará o valor de retorno que
o `Ok` está guardando e retornará apenas esse valor para você, para que você possa
usá-lo. Nesse caso, esse valor é o número de bytes na entrada do usuário.

Se você não chamar `expect`, o programa compilará, mas você receberá um aviso:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

O Rust avisa que você não usou o valor `Result` retornado de `read_line`, indicando
que o programa não tratou um possível erro.

A maneira correta de suprimir o aviso é realmente escrever o código de tratamento
de erros, mas no nosso caso queremos apenas fechar o programa quando ocorrer um
problema, para que possamos usar o `expect`. Você aprenderá sobre como se recuperar
de erros no [Capítulo 9][recover]<!-- ignore -->.

### Imprimindo Valores com Marcadores de Posição do `println!`

Além da chave de fechamento, há apenas mais uma linha para discutir no código até agora:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Esta linha imprime a string que agora contém a entrada do usuário. O conjunto
`{}` de chaves é um marcador de posição (placeholder): Pense em `{}` como pequenas
pinças de caranguejo que seguram um valor no lugar. Ao imprimir o valor de uma
variável, o nome da variável pode ir dentro das chaves. Ao imprimir o resultado
de avaliar uma expressão, coloque chaves vazias na string de formato e, em seguida,
siga a string de formato com uma lista de expressões separadas por vírgulas para
imprimir em cada marcador de posição de chave vazia na mesma ordem. Imprimir uma
variável e o resultado de uma expressão em uma única chamada para `println!` seria assim:

```rust
let x = 5;
let y = 10;

println!("x = {x} e y + 2 = {}", y + 2);
```

Este código imprimiria `x = 5 e y + 2 = 12`.

### Testando a Primeira Parte

Vamos testar a primeira parte do jogo de adivinhação. Execute-o usando `cargo run`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Neste ponto, a primeira parte do jogo está concluída: estamos obtendo a entrada
do teclado e imprimindo-a.

## Gerando um Número Secreto

Em seguida, precisamos gerar um número secreto que o usuário tentará adivinhar. O
número secreto deve ser diferente a cada vez para que o jogo seja divertido de
jogar mais de uma vez. Usaremos um número aleatório entre 1 e 100 para que o
jogo não seja muito difícil. O Rust ainda não inclui funcionalidade de números
aleatórios em sua biblioteca padrão. No entanto, a equipe do Rust fornece um
[`crate rand`][randcrate] com a referida funcionalidade.

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### Aumentando a Funcionalidade com um Crate

Lembre-se de que um crate é uma coleção de arquivos de código-fonte Rust. O
projeto que construímos é um crate binário, que é um executável. O crate `rand`
é um crate de biblioteca, que contém código destinado a ser usado em outros
programas e não pode ser executado por conta própria.

A coordenação de crates externos do Cargo é onde o Cargo realmente brilha. Antes
de podermos escrever código que use `rand`, precisamos modificar o arquivo
_Cargo.toml_ para incluir o crate `rand` como uma dependência. Abra esse arquivo
agora e adicione a seguinte linha na parte inferior, abaixo do cabeçalho da seção
`[dependencies]` que o Cargo criou para você. Certifique-se de especificar `rand`
exatamente como temos aqui, com este número de versão, ou os exemplos de código
neste tutorial podem não funcionar:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

No arquivo _Cargo.toml_, tudo o que segue um cabeçalho faz parte dessa seção
que continua até que outra seção comece. Em `[dependencies]`, você diz ao
Cargo de quais crates externos seu projeto depende e quais versões desses crates
você precisa. Nesse caso, especificamos o crate `rand` com o especificador de
versão semântica `0.10.1`. O Cargo entende a [Versionamento Semântico][semver]<!-- ignore -->
(às vezes chamado de _SemVer_), que é um padrão para escrever números de versão.
O especificador `0.10.1` é na verdade um atalho para `^0.10.1`, o que significa
qualquer versão que seja pelo menos 0.10.1, mas abaixo de 0.11.0.

O Cargo considera que essas versões têm APIs públicas compatíveis com a versão
0.10.1, e essa especificação garante que você obterá a versão de patch mais
recente que ainda compilará com o código neste capítulo. Qualquer versão 0.11.0
ou superior não tem garantia de ter a mesma API que os exemplos a seguir usam.

Agora, sem alterar nenhum código, vamos construir o projeto, conforme mostrado
na Listagem 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="A saída da execução de `cargo build` após adicionar o crate `rand` como dependência">

```console
$ cargo build
    Updating crates.io index
     Locking 8 packages to latest Rust 1.96.0 compatible versions
  Downloaded rand_core v0.10.1
  Downloaded chacha20 v0.10.1
  Downloaded rand v0.10.1
  Downloaded 3 crates (162.9KiB) in 0.59s
   Compiling libc v0.2.186
   Compiling rand_core v0.10.1
   Compiling getrandom v0.4.3
   Compiling cfg-if v1.0.4
   Compiling chacha20 v0.10.1
   Compiling rand v0.10.1
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.03s
```

</Listing>

Você pode ver números de versão diferentes (mas todos serão compatíveis com o
código, graças ao SemVer!) e linhas diferentes (dependendo do sistema operacional),
e as linhas podem estar em uma ordem diferente.

Quando incluímos uma dependência externa, o Cargo busca as versões mais recentes
de tudo o que essa dependência precisa no _registry_, que é uma cópia de dados
do [Crates.io][cratesio]. O Crates.io é onde as pessoas no ecossistema Rust postam
seus projetos Rust de código aberto para que outros os usem.

Após atualizar o registro, o Cargo verifica a seção `[dependencies]` e baixa
quaisquer crates listados que ainda não foram baixados. Nesse caso, embora tenhamos
listado apenas `rand` como dependência, o Cargo também pegou outros crates dos
quais `rand` depende para funcionar. Depois de baixar os crates, o Rust os
compila e, em seguida, compila o projeto com as dependências disponíveis.

Se você executar imediatamente o `cargo build` novamente sem fazer nenhuma alteração,
você não obterá nenhuma saída além da linha `Finished`. O Cargo sabe que já baixou
e compilou as dependências e você não alterou nada sobre elas no seu arquivo
_Cargo.toml_. O Cargo também sabe que você não alterou nada sobre o seu código,
então ele não recompila isso também. Sem nada para fazer, ele simplesmente sai.

Se você abrir o arquivo _src/main.rs_, fizer uma alteração trivial, salvá-lo e
compilar novamente, você verá apenas duas linhas de saída:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Estas linhas mostram que o Cargo atualiza apenas a compilação com a sua pequena
alteração no arquivo _src/main.rs_. Suas dependências não mudaram, então o Cargo
sabe que pode reutilizar o que já baixou e compilou para elas.

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Garantindo Compilações Reprodutíveis com o Arquivo Cargo.lock

O Cargo tem um mecanismo que garante que você possa reconstruir o mesmo artefato
todas as vezes que você ou qualquer outra pessoa construir seu código: o Cargo
usará apenas as versões das dependências que você especificou até que você indique
o contrário. Por exemplo, digamos que na próxima semana seja lançada a versão 0.10.2
do crate `rand` e essa versão contenha uma correção de bug importante, mas também
contenha uma regressão que quebrará seu código. Para lidar com isso, o Rust cria o
arquivo _Cargo.lock_ na primeira vez que você executa `cargo build`, então agora
temos isso no diretório _guessing_game_.

Quando você constrói um projeto pela primeira vez, o Cargo descobre todas as versões
das dependências que se ajustam aos critérios e as grava no arquivo _Cargo.lock_.
Quando você construir seu projeto no futuro, o Cargo verá que o arquivo _Cargo.lock_
existe e usará as versões especificadas lá, em vez de fazer todo o trabalho de
descobrir as versões novamente. Isso permite que você tenha uma compilação reprodutível
automaticamente. Em outras palavras, seu projeto permanecerá na versão 0.10.1 até
que você faça um upgrade explicitamente, graças ao arquivo _Cargo.lock_. Como o
arquivo _Cargo.lock_ é importante para compilações reprodutíveis, ele geralmente
é incluído no controle de versão junto com o resto do código do seu projeto.

#### Atualizando um Crate para Obter uma Nova Versão

Quando você _quiser_ atualizar um crate, o Cargo fornece o comando `update`,
que ignorará o arquivo _Cargo.lock_ e descobrirá todas as versões mais recentes
que se ajustam às suas especificações em _Cargo.toml_. O Cargo então gravará
essas versões no arquivo _Cargo.lock_. Caso contrário, por padrão, o Cargo procurará
apenas por versões maiores que 0.10.1 e menores que 0.11.0. Se o crate `rand` lançou
as duas novas versões 0.10.2 e 0.999.0, você veria o seguinte se executasse `cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.96.0 compatible version
    Updating rand v0.10.1 -> v0.10.2 (available: v0.999.0)
```

O Cargo ignora a versão 0.999.0. Neste ponto, você também notaria uma alteração
no seu arquivo _Cargo.lock_ observando que a versão do crate `rand` que você
está usando agora é 0.10.2. Para usar a versão `rand` 0.999.0 ou qualquer versão
na série 0.999._x_, você teria que atualizar o arquivo _Cargo.toml_ para ficar
assim (não faça essa alteração porque os exemplos a seguir assumem que você está
usando o `rand` 0.10):

```toml
[dependencies]
rand = "0.999.0"
```

Da próxima vez que você executar `cargo build`, o Cargo atualizará o registro de
crates disponíveis e reavaliará seus requisitos de `rand` de acordo com a nova
versão que você especificou.

Há muito mais a dizer sobre o [Cargo][doccargo]<!-- ignore --> e [seu ecossistema][doccratesio]<!-- ignore -->,
o qual discutiremos no Capítulo 14, mas por enquanto, isso é tudo o que você
precisa saber. O Cargo torna muito fácil reutilizar bibliotecas, para que os
Rustacean possam escrever projetos menores que são montados a partir de vários pacotes.

### Gerando um Número Aleatório

Vamos começar a usar o `rand` para gerar um número a ser adivinhado. O próximo
passo é atualizar _src/main.rs_, conforme mostrado na Listagem 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Adicionando código para gerar um número aleatório">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Primeiro, adicionamos a linha `use rand::prelude::*;`. O módulo `prelude` contém
as partes mais comumente usadas do crate `rand`, e `use` torna esses itens
disponíveis no escopo do nosso programa.

Em seguida, estamos adicionando duas linhas no meio. Na primeira linha, chamamos
a função `rand::rng` que nos dá o gerador de números aleatórios específico que
vamos usar: um que é local para a thread atual de execução e é inicializado (seeded)
pelo sistema operacional. Em seguida, chamamos o método `random_range` no gerador
de números aleatórios. Este método é definido pela trait `RngExt` que faz parte
do módulo `rand::prelude` que trouxemos para o escopo com a instrução `use rand::prelude::*;`.
O método `random_range` aceita uma expressão de intervalo (range) como argumento e
gera um número aleatório dentro desse intervalo. O tipo de expressão de intervalo
que estamos usando aqui assume a forma `start..=end` e é inclusivo nos limites
inferior e superior, então precisamos especificar `1..=100` para solicitar um
número entre 1 e 100.

> Nota: Você não saberá de cabeça o que trazer para o escopo e quais métodos e
> funções chamar de um crate, então cada crate tem documentação com instruções
> de uso. Outro recurso interessante do Cargo é que a execução do comando
> `cargo doc --open` compilará a documentação fornecida por todas as suas
> dependências localmente e a abrirá no seu navegador. Se você estiver interessado
> em outras funcionalidades no crate `rand`, por exemplo, execute `cargo doc --open`
> e clique em `rand` na barra lateral à esquerda.

A segunda nova linha imprime o número secreto. Isso é útil enquanto estamos
desenvolvendo o programa para poder testá-lo, mas vamos excluí-lo da versão final.
Não é muito jogo se o programa imprime a resposta assim que começa!

Tente executar o programa algumas vezes:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Você deve obter números aleatórios diferentes, e todos devem ser números entre 1 e
100. Se você receber avisos (warnings), eles são seguros de ignorar. Se você
receber erros, verifique se você tem `rand = "0.10.1"` no seu *Cargo.toml*, pois
versões futuras do `rand` podem ter uma API diferente, mas qualquer versão da
série `0.10` deve funcionar com o código neste capítulo.

## Comparando o Palpite com o Número Secreto

Agora que temos a entrada do usuário e um número aleatório, podemos compará-los.
Essa etapa é mostrada na Listagem 2-4. Observe que este código ainda não compilará,
como explicaremos.

<Listing number="2-4" file-name="src/main.rs" caption="Tratando os possíveis valores de retorno da comparação de dois números">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Primeiro, adicionamos outra instrução `use`, trazendo um tipo chamado `std::cmp::Ordering`
para o escopo da biblioteca padrão. O tipo `Ordering` é outro enum e tem as
variantes `Less`, `Greater` e `Equal`. Estes são os três resultados possíveis
quando você compara dois valores.

Em seguida, adicionamos cinco novas linhas na parte inferior que usam o tipo `Ordering`.
O método `cmp` compara dois valores e pode ser chamado em qualquer coisa que possa
ser comparada. Ele aceita uma referência ao que você deseja comparar: Aqui, ele
está comparando `guess` com `secret_number`. Em seguida, ele retorna uma variante
do enum `Ordering` que trouxemos para o escopo com a instrução `use`. Usamos uma
expressão [`match`][match]<!-- ignore --> para decidir o que fazer a seguir com
base em qual variante de `Ordering` foi retornada da chamada para `cmp` com os
valores em `guess` e `secret_number`.

Uma expressão `match` é composta de _braços_ (arms). Um braço consiste em um
_padrão_ para correspondência e o código que deve ser executado se o valor fornecido
ao `match` se encaixar no padrão daquele braço. O Rust pega o valor fornecido ao
`match` e examina o padrão de cada braço por vez. Padrões e a construção `match`
s são recursos poderosos do Rust: eles permitem expressar uma variedade de situações
que seu código pode encontrar e garantem que você lide com todas elas. Esses recursos
serão abordados em detalhes no Capítulo 6 e no Capítulo 19, respectivamente.

Vamos analisar um exemplo com a expressão `match` que usamos aqui. Digamos que o
usuário adivinhou 50 e o número secreto gerado aleatoriamente desta vez é 38.

Quando o código compara 50 com 38, o método `cmp` retornará `Ordering::Greater`
porque 50 é maior que 38. A expressão `match` obtém o valor `Ordering::Greater` e
começa a verificar o padrão de cada braço. Ele olha para o padrão do primeiro braço,
`Ordering::Less`, e vê que o valor `Ordering::Greater` não corresponde a `Ordering::Less`,
portanto, ele ignora o código nesse braço e passa para o próximo braço. O padrão do
próximo braço é `Ordering::Greater`, que _corresponde_ a `Ordering::Greater`! O
código associado nesse braço será executado e imprimirá `Too big!` na tela. A expressão
`match` termina após a primeira correspondência bem-sucedida, portanto, ela não
olhará para o último braço neste cenário.

No entanto, o código na Listagem 2-4 ainda não compilará. Vamos tentar:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

O núcleo do erro afirma que existem _tipos incompatíveis_ (mismatched types). O
Rust possui um sistema de tipos forte e estático. No entanto, ele também possui
inferência de tipos. Quando escrevemos `let mut guess = String::new()`, o Rust
pôde deduzir que `guess` deveria ser uma `String` e não nos obrigou a escrever
o tipo. O `secret_number`, por outro lado, é um tipo numérico. Vários dos tipos
numéricos do Rust podem ter um valor entre 1 e 100: `i32`, um número de 32 bits;
`u32`, um número sem sinal de 32 bits; `i64`, um número de 64 bits; bem como outros.
A menos que especificado de outra forma, o Rust assume o padrão `i32`, que é o
tipo de `secret_number`, a menos que você adicione informações de tipo em outro
lugar que façam o Rust inferir um tipo numérico diferente. O motivo do erro é que
o Rust não pode comparar uma string e um tipo numérico.

Em última análise, queremos converter a `String` que o programa lê como entrada
em um tipo numérico para que possamos compará-la numericamente com o número secreto.
Fazemos isso adicionando esta linha ao corpo da função `main`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

A linha é:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Criamos uma variável chamada `guess`. Mas espere, o programa já não tem uma
variável chamada `guess`? Tem, mas utilmente o Rust nos permite sombrear (shadow)
o valor anterior de `guess` com um novo. O _Shadowing_ (sombreamento) nos permite
reutilizar o nome da variável `guess` em vez de nos forçar a criar duas variáveis
exclusivas, como `guess_str` e `guess`, por exemplo. Abordaremos isso com mais
detalhes no [Capítulo 3][shadowing]<!-- ignore -->, mas por enquanto, saiba que
esse recurso costuma ser usado quando você deseja converter um valor de um tipo
para outro.

Nós associamos essa nova variável à expressão `guess.trim().parse()`. O `guess`
na expressão refere-se à variável `guess` original que continha a entrada como
uma string. O método `trim` em uma instância de `String` eliminará qualquer espaço
em branco no início e no fim, o que devemos fazer antes de podermos converter a
string para um `u32`, que pode conter apenas dados numéricos. O usuário deve
pressionar <kbd>enter</kbd> para satisfazer `read_line` e inserir seu palpite,
o que adiciona um caractere de nova linha à string. Por exemplo, se o usuário
digitar <kbd>5</kbd> e pressionar <kbd>enter</kbd>, `guess` fica assim: `5\n`.
O `\n` representa “nova linha”. (No Windows, pressionar <kbd>enter</kbd> resulta
em um retorno de carro e uma nova linha, `\r\n`.) O método `trim` elimina `\n` ou
`\r\n`, resultando apenas em `5`.

O [`método parse` em strings][parse]<!-- ignore --> converte uma string em
outro tipo. Aqui, nós o usamos para converter de uma string para um número. Precisamos
dizer ao Rust o tipo numérico exato que queremos usando `let guess: u32`. Os
dois-pontos (`:`) após `guess` dizem ao Rust que anotaremos o tipo da variável.
O Rust possui alguns tipos numéricos integrados; o `u32` visto aqui é um inteiro
sem sinal de 32 bits. É uma boa escolha padrão para um pequeno número positivo.
Você aprenderá sobre outros tipos numéricos no [Capítulo 3][integers]<!-- ignore -->.

Além disso, a anotação `u32` neste programa de exemplo e a comparação com
`secret_number` significam que o Rust deduzirá que `secret_number` também deve
ser um `u32`. Portanto, agora a comparação será entre dois valores do mesmo tipo!

O método `parse` funcionará apenas em caracteres que podem ser logicamente convertidos
em números e, portanto, pode causar erros facilmente. Se, por exemplo, a string
contivesse `A👍%`, não haveria maneira de converter isso em um número. Como pode
falhar, o método `parse` retorna um tipo `Result`, assim como o método `read_line`
faz (discutido anteriormente em [“Lidando com Possíveis Falhas com o Tipo `Result`”](#handling-potential-failure-with-the-result-type)<!-- ignore -->).
Trataremos este `Result` da mesma maneira, usando o método `expect` novamente. Se
`parse` retornar uma variante `Err` de `Result` porque não pôde criar um número
a partir da string, a chamada `expect` travará o jogo e imprimirá a mensagem que
lhe damos. Se `parse` puder converter com sucesso a string em um número, ele
retornará a variante `Ok` do `Result`, e `expect` retornará o número que queremos
do valor `Ok`.

Vamos executar o programa agora:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Legal! Mesmo que espaços tenham sido adicionados antes do palpite, o programa
ainda descobriu que o usuário adivinhou 76. Execute o programa algumas vezes para
verificar o comportamento diferente com diferentes tipos de entrada: Adivinhe o
número corretamente, adivinhe um número muito alto e adivinhe um número muito baixo.

Temos a maior parte do jogo funcionando agora, mas o usuário pode fazer apenas um palpite.
Vamos mudar isso adicionando um loop!

## Permitindo Vúltiplos Palpites com Loop

A palavra-chave `loop` cria um loop infinito. Adicionaremos um loop para dar aos
usuários mais chances de adivinhar o número:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Como você pode ver, movemos tudo a partir do prompt de entrada de palpite para dentro
de um loop. Certifique-se de recuar (indent) as linhas dentro do loop em mais quatro
espaços cada uma e execute o programa novamente. O programa agora pedirá outro
palpite para sempre, o que na verdade introduz um novo problema. Não parece que
o usuário pode sair!

O usuário sempre pode interromper o programa usando o atalho de teclado <kbd>ctrl</kbd>-<kbd>C</kbd>.
Mas há outra maneira de escapar desse monstro insaciável, conforme mencionado na
discussão do `parse` em [“Comparando o Palpite com o Número Secreto”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->:
Se o usuário inserir uma resposta que não seja um número, o programa travará. Podemos
aproveitar isso para permitir que o usuário saia, conforme mostrado aqui:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' (6694925) panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Digitar `quit` encerrará o jogo, mas como você notará, o mesmo acontecerá ao
inserir qualquer outra entrada que não seja um número. Isso é subótimo, para dizer
o mínimo; queremos que o jogo também pare quando o número correto for adivinhado.

### Saindo após um Palpite Correto

Vamos programar o jogo para sair quando o usuário vencer, adicionando uma instrução `break`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Adicionar a linha `break` após `You win!` faz com que o programa saia do loop
quando o usuário adivinhar o número secreto corretamente. Sair do loop também significa
sair do programa, porque o loop é a última parte do `main`.

### Tratando Entrada Inválida

Para refinar ainda mais o comportamento do jogo, em vez de travar o programa quando
o usuário inserir algo que não seja um número, vamos fazer com que o jogo ignore
entradas que não sejam números para que o usuário possa continuar adivinhando.
Podemos fazer isso alterando a linha onde `guess` é convertido de uma `String`
para um `u32`, conforme mostrado na Listagem 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Ignorando um palpite que não é um número e pedindo outro palpite em vez de travar o programa">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Mudamos de uma chamada `expect` para uma expressão `match` para passar de travamento
em um erro para o tratamento do erro. Lembre-se de que `parse` retorna um tipo `Result`
e `Result` é um enum que possui as variantes `Ok` e `Err`. Estamos usando uma
expressão `match` aqui, assim como fizemos com o resultado `Ordering` do método `cmp`.

Se o `parse` conseguir transformar com sucesso a string em um número, ele retornará
um valor `Ok` que contém o número resultante. Esse valor `Ok` corresponderá ao
padrão do primeiro braço, e a expressão `match` simplesmente retornará o valor `num`
que o `parse` produziu e colocou dentro do valor `Ok`. Esse número terminará exatamente
onde o queremos na nova variável `guess` que estamos criando.

Se o `parse` _não_ conseguir transformar a string em um número, ele retornará um
valor `Err` que contém mais informações sobre o erro. O valor `Err` não corresponde
ao padrão `Ok(num)` no primeiro braço do `match`, mas corresponde ao padrão `Err(_)`
no segundo braço. O sublinhado, `_`, é um valor curinga (catch-all); neste exemplo,
estamos dizendo que queremos corresponder a todos os valores `Err`, independentemente
das informações que eles contêm. Portanto, o programa executará o código do segundo
braço, `continue`, que diz ao programa para ir para a próxima iteração do `loop`
e pedir outro palpite. Assim, efetivamente, o programa ignora todos os erros que
o `parse` possa encontrar!

Agora tudo no programa deve funcionar conforme o esperado. Vamos tentar:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Incrível! Com um pequeno ajuste final, terminaremos o jogo de adivinhação. Lembre-se
de que o programa ainda está imprimindo o número secreto. Isso funcionou bem para
testes, mas estraga o jogo. Vamos deletar o `println!` que exibe o número secreto.
A Listagem 2-6 mostra o código final.

<Listing number="2-6" file-name="src/main.rs" caption="Código completo do jogo de adivinhação">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Neste ponto, você construiu com sucesso o jogo de adivinhação. Parabéns!

## Resumo

Este projeto foi uma maneira prática de apresentar muitos conceitos novos do Rust:
`let`, `match`, funções, o uso de crates externos e muito mais. Nos próximos
capítulos, você aprenderá sobre esses conceitos com mais detalhes. O Capítulo 3
aborda conceitos que a maioria das linguagens de programação possui, como variáveis,
tipos de dados e funções, e mostra como usá-los em Rust. O Capítulo 4 explora a
posse (ownership), um recurso que torna o Rust diferente de outras linguagens. O
Capítulo 5 discute structs e sintaxe de métodos, e o Capítulo 6 explica como os enums funcionam.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types