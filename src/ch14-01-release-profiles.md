## Personalizando Compilações com Perfis de Lançamento

Em Rust, _perfis de lançamento_ (_release profiles_) são perfis predefinidos e personalizáveis com
diferentes configurações que permitem ao programador ter mais controle sobre
várias opções de compilação de código. Cada perfil é configurado independentemente dos
outros.

O Cargo possui dois perfis principais: o perfil `dev`, que o Cargo usa quando você executa `cargo
build`, e o perfil `release`, que o Cargo usa quando você executa `cargo build
--release`. O perfil `dev` é definido com bons padrões para desenvolvimento,
e o perfil `release` possui bons padrões para compilações de lançamento (release).

Esses nomes de perfil podem ser familiares a partir da saída de suas compilações:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` e `release` são esses diferentes perfis usados pelo compilador.

O Cargo possui configurações padrão para cada um dos perfis que se aplicam quando você não
adicionou explicitamente nenhuma seção `[profile.*]` no arquivo _Cargo.toml_ do projeto.
Ao adicionar seções `[profile.*]` para qualquer perfil que você queira personalizar, você
substitui qualquer subconjunto das configurações padrão. Por exemplo, aqui estão os valores
padrão para a configuração `opt-level` para os perfis `dev` e `release`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

A configuração `opt-level` controla o número de otimizações que o Rust aplicará ao
seu código, com um intervalo de 0 a 3. Aplicar mais otimizações estende o
tempo de compilação, portanto, se você estiver em desenvolvimento e compilando seu código frequentemente,
vai querer menos otimizações para compilar mais rápido, mesmo que o código resultante
execute mais devagar. O `opt-level` padrão para `dev` é, portanto, `0`. Quando você
estiver pronto para lançar seu código, é melhor gastar mais tempo compilando. Você só
vai compilar no modo release uma vez, mas vai executar o programa compilado muitas vezes,
então o modo release troca um tempo de compilação maior por código que é executado mais rápido. É por
isso que o `opt-level` padrão para o perfil `release` é `3`.

Você pode substituir uma configuração padrão adicionando um valor diferente para ela no
_Cargo.toml_. Por exemplo, se quisermos usar o nível de otimização 1 no
perfil de desenvolvimento, podemos adicionar estas duas linhas ao arquivo _Cargo.toml_
do nosso projeto:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Este código substitui a configuração padrão de `0`. Agora, quando executarmos `cargo build`,
o Cargo usará os padrões para o perfil `dev` mais a nossa personalização para
`opt-level`. Como definimos `opt-level` como `1`, o Cargo aplicará mais
otimizações do que o padrão, mas não tantas quanto em uma compilação de lançamento (release).

Para a lista completa de opções de configuração e padrões para cada perfil, consulte a
[documentação do Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).