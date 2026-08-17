<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Instalando Binários com `cargo install`

O comando `cargo install` permite que você instale e use crates binárias
localmente. Isso não tem a intenção de substituir pacotes do sistema; é um meio
conveniente para desenvolvedores Rust instalarem ferramentas que outros
compartilharam no [crates.io](https://crates.io/)<!-- ignore -->. Note que você só
pode instalar pacotes que possuem alvos binários. Um _alvo binário_ é o programa
executável criado se o crate tiver um arquivo _src/main.rs_ ou outro arquivo
especificado como binário, em oposição a um alvo de biblioteca, que não é
executável por si só, mas é adequado para ser incluído em outros programas.
Normalmente, os crates têm informações no arquivo README sobre se o crate é uma
biblioteca, possui um alvo binário ou ambos.

Todos os binários instalados com `cargo install` são armazenados na pasta _bin_
da raiz de instalação. Se você instalou o Rust usando _rustup.rs_ e não tem
nenhuma configuração personalizada, este diretório será *$HOME/.cargo/bin*.
Certifique-se de que este diretório esteja no seu `$PATH` para poder executar os
programas que você instalou com `cargo install`.

Por exemplo, no Capítulo 12 mencionamos que existe uma implementação em Rust
da ferramenta `grep` chamada `ripgrep` para buscar em arquivos. Para instalar o
`ripgrep`, podemos executar o seguinte:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

A penúltima linha da saída mostra o local e o nome do binário instalado, que no
caso do `ripgrep` é `rg`. Contanto que o diretório de instalação esteja no seu
`$PATH`, como mencionado anteriormente, você poderá executar `rg --help` e
começar a usar uma ferramenta mais rápida e com a cara de Rust para buscar em
arquivos!