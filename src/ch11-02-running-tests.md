## Controlando Como os Testes São Executados

Assim como o `cargo run` compila o seu código e depois executa o binário resultante,
o `cargo test` compila o seu código em modo de teste e executa o binário de teste
resultante. O comportamento padrão do binário produzido pelo `cargo test` é executar
todos os testes em paralelo e capturar a saída gerada durante a execução dos testes,
impedindo que a saída seja exibida e tornando mais fácil a leitura da saída relacionada
aos resultados dos testes. No entanto, você pode especificar opções de linha de comando
para alterar esse comportamento padrão.

Algumas opções de linha de comando vão para o `cargo test`, e outras vão para o binário
de teste resultante. Para separar esses dois tipos de argumentos, você lista os argumentos que
vão para o `cargo test`, seguidos pelo separador `--` e, em seguida, aqueles que vão para
o binário de teste. Executar `cargo test --help` exibe as opções que você pode usar
com o `cargo test`, e executar `cargo test -- --help` exibe as opções que você pode
usar após o separador. Essas opções também estão documentadas na [seção “Tests”
do _The `rustc` Book_][tests] (em inglês).

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### Executando Testes em Paralelo ou Consecutivamente

Quando você executa vários testes, por padrão eles são executados em paralelo usando threads,
o que significa que eles terminam de rodar mais rapidamente e você obtém feedback mais cedo. Como
os testes estão rodando ao mesmo tempo, você deve garantir que seus testes não dependam
uns dos outros ou de qualquer estado compartilhado, incluindo um ambiente compartilhado,
como o diretório de trabalho atual ou variáveis de ambiente.

Por exemplo, digamos que cada um dos seus testes execute algum código que cria um arquivo no disco
chamado _test-output.txt_ e escreve alguns dados nele. Em seguida, cada teste
lê os dados nesse arquivo e afirma que o arquivo contém um valor específico,
que é diferente em cada teste. Como os testes rodam ao mesmo tempo,
um teste pode sobrescrever o arquivo no intervalo entre o momento em que outro teste está
escrevendo e lendo o arquivo. O segundo teste falhará, não porque o
código esteja incorreto, mas porque os testes interferiram uns nos outros enquanto
rodavam em paralelo. Uma solução é garantir que cada teste escreva em um
arquivo diferente; outra solução é executar os testes um de cada vez.

Se você não quiser executar os testes em paralelo ou se quiser um controle mais refinado
sobre o número de threads usadas, você pode enviar a flag `--test-threads` e o
número de threads que deseja usar para o binário de teste. Dê uma olhada no
exemplo a seguir:

```console
$ cargo test -- --test-threads=1
```

Definimos o número de threads de teste como `1`, dizendo ao programa para não usar nenhum
paralelismo. Executar os testes usando uma única thread levará mais tempo do que executá-los
em paralelo, mas os testes não interferirão uns nos outros caso compartilhem estado.

### Mostrando a Saída de Funções

Por padrão, se um teste passa, a biblioteca de testes do Rust captura tudo o que é impresso na
saída padrão. Por exemplo, se chamarmos `println!` em um teste e o teste
passar, não veremos a saída do `println!` no terminal; veremos apenas a
linha que indica que o teste passou. Se um teste falhar, veremos tudo o que foi
impresso na saída padrão junto com o resto da mensagem de falha.

Como exemplo, a Listagem 11-10 tem uma função simples que imprime o valor de seu
parâmetro e retorna 10, bem como um teste que passa e um teste que falha.

<Listing number="11-10" file-name="src/lib.rs" caption="Testes para uma função que chama `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Quando executamos esses testes com `cargo test`, veremos a seguinte saída:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Note que em nenhum lugar desta saída nós vemos `I got the value 4`, que é
impresso quando o teste que passa é executado. Essa saída foi capturada. A
saída do teste que falhou, `I got the value 8`, aparece na seção
do resumo da saída do teste, que também mostra a causa da falha do teste.

Se quisermos ver os valores impressos para os testes que passam também, podemos dizer ao Rust para
também mostrar a saída dos testes bem-sucedidos com `--show-output`:

```console
$ cargo test -- --show-output
```

Quando executamos os testes da Listagem 11-10 novamente com a flag `--show-output`, nós
vemos a seguinte saída:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Executando um Subconjunto de Testes por Nome

Executar uma suíte de testes completa às vezes pode levar muito tempo. Se você está trabalhando em
código em uma área específica, pode querer executar apenas os testes pertencentes a
esse código. Você pode escolher quais testes executar passando para o `cargo test` o nome
ou os nomes do(s) teste(s) que deseja executar como argumento.

Para demonstrar como executar um subconjunto de testes, primeiro criaremos três testes para
nossa função `add_two`, conforme mostrado na Listagem 11-11, e escolheremos quais executar.

<Listing number="11-11" file-name="src/lib.rs" caption="Três testes com três nomes diferentes">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Se executarmos os testes sem passar nenhum argumento, como vimos anteriormente, todos os
testes serão executados em paralelo:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Executando Testes Individuais

Podemos passar o nome de qualquer função de teste para o `cargo test` para executar apenas esse teste:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Apenas o teste com o nome `one_hundred` foi executado; os outros dois testes não corresponderam
a esse nome. A saída do teste nos avisa que tivemos mais testes que não foram executados
exibindo `2 filtered out` no final.

Não podemos especificar os nomes de vários testes dessa maneira; apenas o primeiro valor
fornecido ao `cargo test` será usado. Mas há uma maneira de executar vários testes.

#### Filtrando para Executar Vários Testes

Podemos especificar parte do nome de um teste, e qualquer teste cujo nome corresponda a esse valor
será executado. Por exemplo, como dois dos nomes dos nossos testes contêm `add`, podemos
executar ambos executando `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Este comando executou todos os testes com `add` no nome e filtrou o teste
chamado `one_hundred`. Note também que o módulo em que um teste aparece se torna
parte do nome do teste, então podemos executar todos os testes em um módulo filtrando
pelo nome do módulo.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### Ignorando Testes, a Menos que Especificamente Solicitado

Às vezes, alguns testes específicos podem ser muito demorados para executar, então você
pode querer excluí-los durante a maioria das execuções do `cargo test`. Em vez de
listar como argumentos todos os testes que você deseja executar, você pode anotar os
testes demorados usando o atributo `ignore` para excluí-los, como mostrado
aqui:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Após `#[test]`, adicionamos a linha `#[ignore]` ao teste que queremos excluir.
Agora, quando executamos nossos testes, `it_works` é executado, mas `expensive_test` não:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

A função `expensive_test` é listada como `ignored` (ignorada). Se quisermos executar apenas
os testes ignorados, podemos usar `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Ao controlar quais testes são executados, você pode garantir que os resultados do seu `cargo test`
sejam retornados rapidamente. Quando você estiver em um ponto em que faça sentido verificar
os resultados dos testes `ignored` e tiver tempo para esperar pelos resultados,
você pode executar `cargo test -- --ignored` em vez disso. Se você quiser executar todos os testes,
independentemente de terem sido ignorados ou não, você pode executar `cargo test -- --include-ignored`.
