## Erros Irrecuperáveis com `panic!`

Às vezes, coisas ruins acontecem no seu código e não há nada que você possa fazer
a respeito. Nesses casos, o Rust possui a macro `panic!`. Há duas maneiras de causar um
panic na prática: executando uma ação que faz nosso código entrar em panic (como
acessar um array além do seu final) ou chamando explicitamente a macro `panic!`.
Em ambos os casos, nós causamos um panic em nosso programa. Por padrão, esses panics vão
imprimir uma mensagem de falha, realizar o *unwinding* (desenrolamento), limpar a pilha e encerrar a execução. Por meio de uma
variável de ambiente, você também pode fazer com que o Rust exiba a pilha de chamadas (*call stack*) quando um
panic ocorrer, para facilitar a localização da origem do erro.

> ### Desenrolando a Pilha (*Unwinding*) ou Interrompendo (*Aborting*) em Resposta a um Panic
>
> Por padrão, quando ocorre um panic, o programa começa a _desenrolar_ (*unwind*), o que significa
> que o Rust percorre a pilha de volta e limpa os dados de cada função que
> ele encontra. No entanto, percorrer de volta e limpar dá muito trabalho.
> Portanto, o Rust permite que você escolha a alternativa de _interromper_ (*abort*) imediatamente,
> o que encerra o programa sem fazer a limpeza.
>
> A memória que o programa estava usando precisará então ser limpa pelo
> sistema operacional. Se no seu projeto você precisar tornar o binário resultante o
> menor possível, você pode alternar do desenrolamento para a interrupção em caso de panic
> adicionando `panic = 'abort'` às seções `[profile]` apropriadas no seu
> arquivo _Cargo.toml_. Por exemplo, se você quiser interromper em caso de panic no modo release,
> adicione isto:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Vamos tentar chamar `panic!` em um programa simples:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Quando você executa o programa, verá algo assim:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

A chamada para `panic!` causa a mensagem de erro contida nas últimas duas linhas.
A primeira linha mostra nossa mensagem de panic e o local em nosso código-fonte onde
o panic ocorreu: _src/main.rs:2:5_ indica que é a segunda linha,
quinto caractere do nosso arquivo _src/main.rs_.

Nesse caso, a linha indicada faz parte do nosso código e, se formos até essa
linha, veremos a chamada da macro `panic!`. Em outros casos, a chamada de `panic!` pode
estar em um código que o nosso código chama, e o nome do arquivo e o número da linha relatados pela
mensagem de erro serão de outra pessoa onde a macro `panic!` é
chamada, e não a linha do nosso código que eventualmente levou à chamada de `panic!`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Podemos usar o rastreamento de pilha (*backtrace*) das funções de onde a chamada de `panic!` veio para descobrir
a parte do nosso código que está causando o problema. Para entender como usar
um rastreamento de pilha de `panic!`, vamos olhar para outro exemplo e ver como é quando
uma chamada de `panic!` vem de uma biblioteca por causa de um bug no nosso código em vez de
ser do nosso código chamando a macro diretamente. A Listagem 9-1 tem algum código que
tenta acessar um índice em um vetor além do intervalo de índices válidos.

<Listing number="9-1" file-name="src/main.rs" caption="Tentando acessar um elemento além do final de um vetor, o que causará uma chamada a `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Aqui, estamos tentando acessar o 100º elemento do nosso vetor (que está no
índice 99 porque a indexação começa do zero), mas o vetor tem apenas três
elementos. Nessa situação, o Rust entrará em panic. Usar `[]` deveria retornar
um elemento, mas se você passar um índice inválido, não há nenhum elemento que o Rust
possa retornar aqui que seria correto.

Em C, tentar ler além do final de uma estrutura de dados é um comportamento indefinido.
Você pode obter o que quer que esteja na posição de memória que
corresponderia a esse elemento na estrutura de dados, mesmo que a memória
não pertença a essa estrutura. Isso é chamado de _leitura excedente de buffer_ (*buffer overread*) e pode
levar a vulnerabilidades de segurança se um atacante conseguir manipular o índice
de forma a ler dados que não deveria ter permissão para acessar, armazenados após
a estrutura de dados.

Para proteger seu programa contra esse tipo de vulnerabilidade, se você tentar ler um
elemento em um índice que não existe, o Rust interromperá a execução e recusará
continuar. Vamos testar isso e ver:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Este erro aponta para a linha 4 do nosso _main.rs_ onde tentamos acessar o índice
99 do vetor em `v`.

A linha `note:` nos diz que podemos definir a variável de ambiente `RUST_BACKTRACE`
para obter um rastreamento de pilha (*backtrace*) exato do que aconteceu para causar o erro. Um
_backtrace_ é uma lista de todas as funções que foram chamadas para chegar a este
ponto. Os rastreamentos de pilha no Rust funcionam como em outras linguagens: A chave para
ler o backtrace é começar do topo e ler até ver os arquivos que
você escreveu. Esse é o ponto onde o problema se originou. As linhas acima desse ponto
são códigos que o seu código chamou; as linhas abaixo são códigos que chamaram o
seu código. Essas linhas de antes e depois podem incluir código principal do Rust (*core*), código da
biblioteca padrão ou crates que você está usando. Vamos tentar obter um backtrace
definindo a variável de ambiente `RUST_BACKTRACE` para qualquer valor exceto `0`.
A Listagem 9-2 mostra uma saída semelhante à que você verá.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="O backtrace gerado por uma chamada a `panic!` exibido quando a variável de ambiente `RUST_BACKTRACE` está definida">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Isso é bastante saída! A saída exata que você vê pode ser diferente dependendo
do seu sistema operacional e da versão do Rust. Para obter rastreamentos de pilha com essas
informações, os símbolos de depuração (*debug symbols*) devem estar ativados. Os símbolos de depuração são ativados por
padrão ao usar `cargo build` ou `cargo run` sem a flag `--release`,
como fizemos aqui.

Na saída da Listagem 9-2, a linha 6 do backtrace aponta para a linha no nosso
projeto que está causando o problema: a linha 4 de _src/main.rs_. Se não quisermos
que nosso programa entre em panic, devemos começar nossa investigação no local apontado
pela primeira linha que menciona um arquivo que escrevemos. Na Listagem 9-1, onde
escrevemos deliberadamente um código que entraria em panic, a maneira de corrigir o panic é não
solicitar um elemento além do intervalo dos índices do vetor. Quando seu código
entrar em panic no futuro, você precisará descobrir qual ação o código está tomando
com quais valores para causar o panic e o que o código deveria fazer em vez disso.

Voltaremos ao `panic!` e a quando devemos ou não usar `panic!` para
lidar com condições de erro na seção [“Usar ou não `panic!`”][to-panic-or-not-to-panic]<!-- ignore --> mais adiante neste
capítulo. Em seguida, veremos como se recuperar de um erro usando `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
