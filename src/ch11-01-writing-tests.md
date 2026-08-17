## Como Escrever Testes

_Testes_ são funções em Rust que verificam se o código que não é de teste está
funcionando da maneira esperada. Os corpos das funções de teste normalmente
executam estas três ações:

- Configuram qualquer dado ou estado necessário.
- Executam o código que você deseja testar.
- Asseveram (fazem asserções) que os resultados são os esperados.

Vamos analisar os recursos que o Rust fornece especificamente para escrever
testes que realizam essas ações, que incluem o atributo `test`, alguns macros e
o atributo `should_panic`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### Estruturando Funções de Teste

Em sua forma mais simples, um teste em Rust é uma função anotada com o
atributo `test`. Atributos são metadados sobre partes do código Rust; um exemplo
é o atributo `derive` que usamos com structs no Capítulo 5. Para transformar uma
função em uma função de teste, adicione `#[test]` na linha anterior a `fn`.
Quando você executa seus testes com o comando `cargo test`, o Rust cria um
binário executor de testes que roda as funções anotadas e relata se cada função
de teste passou ou falhou.

Sempre que criamos um novo projeto de biblioteca com o Cargo, um módulo de teste
contendo uma função de teste é gerado automaticamente para nós. Este módulo
fornece um modelo para escrever seus testes, para que você não precise procurar
a estrutura e a sintaxe exatas toda vez que iniciar um novo projeto. Você pode
adicionar quantas funções de teste adicionais e quantos módulos de teste quiser!

Exploraremos alguns aspectos de como os testes funcionam experimentando com o
modelo de teste antes de realmente testarmos qualquer código. Depois,
escreveremos alguns testes do mundo real que chamam algum código que escrevemos
e verificam se seu comportamento está correto.

Vamos criar um novo projeto de biblioteca chamado `adder` que somará dois números:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

O conteúdo do arquivo _src/lib.rs_ na sua biblioteca `adder` deve se parecer com a
Listagem 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="O código gerado automaticamente por `cargo new`">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

O arquivo começa com um exemplo de função `add` para que tenhamos algo para testar.

Por enquanto, vamos nos concentrar exclusivamente na função `it_works`. Note a
anotação `#[test]`: este atributo indica que esta é uma função de teste, para que o
executor de testes saiba tratar esta função como um teste. Também podemos ter
funções que não são de teste no módulo `tests` para ajudar a configurar cenários
comuns ou realizar operações comuns, então sempre precisamos indicar quais
funções são testes.

O corpo da função de exemplo usa o macro `assert_eq!` para afirmar que `result`,
que contém o resultado da chamada de `add` com 2 e 2, é igual a 4. Essa
asserção serve como um exemplo do formato de um teste típico. Vamos executá-lo
para ver que este teste passa.

O comando `cargo test` executa todos os testes em nosso projeto, como mostrado na
Listagem 11-2.

<Listing number="11-2" caption="A saída da execução do teste gerado automaticamente">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

O Cargo compilou e executou o teste. Vemos a linha `running 1 test`. A linha
seguinte mostra o nome da função de teste gerada, chamada `tests::it_works`,
e que o resultado da execução desse teste é `ok`. O resumo geral `test
result: ok.` significa que todos os testes passaram, e a parte que diz `1
passed; 0 failed` contabiliza o número total de testes que passaram ou falharam.

É possível marcar um teste como ignorado para que ele não seja executado em uma
instância específica; abordaremos isso na seção [“Ignorando Testes, a Menos que
Seja Especificamente Solicitado”][ignoring]<!-- ignore --> mais adiante neste
capítulo. Como não fizemos isso aqui, o resumo mostra `0 ignored`. Também
podemos passar um argumento para o comando `cargo test` para executar apenas
testes cujos nomes correspondam a uma string; isso é chamado de _filtragem_, e
o abordaremos na seção [“Executando um Subconjunto de Testes por
Nome”][subset]<!-- ignore -->. Aqui, não filtramos os testes que estão sendo
executados, então o final do resumo mostra `0 filtered out`.

A estatística `0 measured` é para testes de benchmark que medem o desempenho.
Testes de benchmark estão, até o momento da escrita deste livro, disponíveis
apenas no Rust nightly. Veja [a documentação sobre testes de benchmark][bench]
para saber mais.

A próxima parte da saída do teste, começando em `Doc-tests adder`, é para os
resultados de quaisquer testes de documentação. Ainda não temos nenhum teste de
documentação, mas o Rust pode compilar quaisquer exemplos de código que apareçam
na documentação da nossa API. Esse recurso ajuda a manter sua documentação e
seu código sincronizados! Discutiremos como escrever testes de documentação na
seção [“Comentários de Documentação como Testes”][doc-comments]<!-- ignore --> do
Capítulo 14. Por enquanto, ignoraremos a saída de `Doc-tests`.

Vamos começar a personalizar o teste para as nossas próprias necessidades. Primeiro,
mude o nome da função `it_works` para um nome diferente, como `exploration`, assim:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Em seguida, execute `cargo test` novamente. A saída agora mostra `exploration` em
vez de `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Agora adicionaremos outro teste, mas desta vez criaremos um teste que falha! Os
testes falham quando algo na função de teste entra em pânico (_panics_). Cada
teste é executado em uma nova thread, e quando a thread principal vê que uma
thread de teste morreu, o teste é marcado como falho. No Capítulo 9, falamos
sobre como a maneira mais simples de causar um pânico é chamar o macro
`panic!`. Insira o novo teste como uma função chamada `another`, de modo que o
seu arquivo _src/lib.rs_ fique parecido com a Listagem 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="Adicionando um segundo teste que falhará porque chamamos o macro `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Execute os testes novamente usando `cargo test`. A saída deve se parecer com a
Listagem 11-4, que mostra que nosso teste `exploration` passou e `another` falhou.

<Listing number="11-4" caption="Resultados dos testes quando um teste passa e um teste falha">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

Em vez de `ok`, a linha `test tests::another` mostra `FAILED`. Duas novas
seções aparecem entre os resultados individuais e o resumo: A primeira exibe
o motivo detalhado de cada falha de teste. Neste caso, obtemos os detalhes de
que `tests::another` falhou porque entrou em pânico com a mensagem `Make
this test fail` na linha 17 do arquivo _src/lib.rs_. A próxima seção lista
apenas os nomes de todos os testes que falharam, o que é útil quando há muitos
testes e muita saída detalhada de testes com falha. Podemos usar o nome de um
teste com falha para executar apenas ele e depurá-lo mais facilmente; falaremos
mais sobre maneiras de executar testes na seção [“Controlando Como os Testes São
Executados”][controlling-how-tests-are-run]<!-- ignore -->.

A linha de resumo é exibida no final: No geral, o resultado do nosso teste é
`FAILED`. Tivemos um teste que passou e um teste que falhou.

Agora que você viu como os resultados dos testes se parecem em diferentes
cenários, vamos analisar alguns macros além de `panic!` que são úteis em testes.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### Verificando Resultados com o Macro `assert!`

O macro `assert!`, fornecido pela biblioteca padrão, é útil quando você deseja
garantir que alguma condição em um teste seja avaliada como `true`. Nós damos
ao macro `assert!` um argumento que é avaliado como um booleano. Se o valor for
`true`, nada acontece e o teste passa. Se o valor for `false`, o macro
`assert!` chama `panic!` para fazer o teste falhar. O uso do macro `assert!`
nos ajuda a verificar se nosso código está funcionando da maneira que pretendemos.

No Capítulo 5, na Listagem 5-15, usamos uma struct `Rectangle` e um método
`can_hold`, que são repetidos aqui na Listagem 11-5. Vamos colocar este código
no arquivo _src/lib.rs_ e, em seguida, escrever alguns testes para ele usando o
macro `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="A struct `Rectangle` e seu método `can_hold` do Capítulo 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

O método `can_hold` retorna um booleano, o que significa que é um caso de uso
perfeito para o macro `assert!`. Na Listagem 11-6, escrevemos um teste que
exercita o método `can_hold` criando uma instância de `Rectangle` que tem largura
8 e altura 7, e afirmando que ela pode conter outra instância de `Rectangle` que
tem largura 5 e altura 1.

<Listing number="11-6" file-name="src/lib.rs" caption="Um teste para `can_hold` que verifica se um retângulo maior pode de fato conter um retângulo menor">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Observe a linha `use super::*;` dentro do módulo `tests`. O módulo `tests` é um
módulo normal que segue as regras usuais de visibilidade que cobrimos no
Capítulo 7 na seção [“Caminhos para Referenciar um Item na Árvore de
Módulos”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->. Como
o módulo `tests` é um módulo interno, precisamos trazer o código sob teste no
módulo externo para o escopo do módulo interno. Usamos um asterisco glob aqui,
então tudo o que definimos no módulo externo fica disponível para este módulo
`tests`.

Nomeamos nosso teste de `larger_can_hold_smaller` e criamos as duas instâncias
de `Rectangle` de que precisamos. Em seguida, chamamos o macro `assert!` e
passamos a ele o resultado da chamada de `larger.can_hold(&smaller)`. Essa
expressão deve retornar `true`, então nosso teste deve passar. Vamos descobrir!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Ele passa! Vamos adicionar outro teste, desta vez afirmando que um retângulo
menor não pode conter um retângulo maior:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Como o resultado correto da função `can_hold` neste caso é `false`, precisamos
negar esse resultado antes de passá-lo para o macro `assert!`. Como resultado,
nosso teste passará se `can_hold` retornar `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Dois testes que passam! Agora vamos ver o que acontece com os resultados dos
nossos testes quando introduzimos um bug em nosso código. Vamos alterar a
implementação do método `can_hold` substituindo o sinal de maior que (`>`) por
um sinal de menor que (`<`) ao comparar as larguras:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

A execução dos testes agora produz o seguinte:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Nossos testes pegaram o bug! Como `larger.width` é `8` e `smaller.width` é
`5`, a comparação das larguras em `can_hold` agora retorna `false`: 8 não é
menor que 5.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### Testando a Igualdade com os Macros `assert_eq!` e `assert_ne!`

Uma maneira comum de verificar a funcionalidade é testar a igualdade entre o
resultado do código sob teste e o valor que você espera que o código retorne.
Você poderia fazer isso usando o macro `assert!` e passando a ele uma expressão
usando o operador `==`. No entanto, este é um teste tão comum que a biblioteca
padrão fornece um par de macros — `assert_eq!` e `assert_ne!` — para realizar
esse teste de forma mais conveniente. Esses macros comparam dois argumentos em
busca de igualdade ou desigualdade, respectivamente. Eles também imprimirão os
dois valores se a asserção falhar, o que facilita ver _por que_ o teste
falhou; por outro lado, o macro `assert!` apenas indica que obteve um valor
`false` para a expressão `==`, sem imprimir os valores que levaram ao valor `false`.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao
seu parâmetro e, em seguida, testamos essa função usando o macro `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="Testando a função `add_two` usando o macro `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Vamos verificar se ele passa!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Criamos uma variável chamada `result` que armazena o resultado da chamada de
`add_two(2)`. Em seguida, passamos `result` e `4` como argumentos para o macro
`assert_eq!`. A linha de saída para este teste é `test tests::it_adds_two ...
ok`, e o texto `ok` indica que nosso teste passou!

Vamos introduzir um bug em nosso código para ver como o `assert_eq!` se parece
quando falha. Altere a implementação da função `add_two` para adicionar `3`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Execute os testes novamente:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Nosso teste pegou o bug! O teste `tests::it_adds_two` falhou, e a mensagem nos
diz que a asserção que falhou foi `left == right` e quais são os valores da
esquerda (`left`) e da direita (`right`). Esta mensagem nos ajuda a começar a
depuração: O argumento `left`, onde tínhamos o resultado da chamada de
`add_two(2)`, era `5`, mas o argumento `right` era `4`. Você pode imaginar que
isso seria especialmente útil quando temos muitos testes rodando.

Note que em algumas linguagens e frameworks de teste, os parâmetros para as
funções de asserção de igualdade são chamados de `expected` (esperado) e
`actual` (atual), e a ordem em que especificamos os argumentos importa. No
entanto, em Rust, eles são chamados de `left` e `right`, e a ordem em que
especificamos o valor que esperamos e o valor que o código produz não importa.
Poderíamos escrever a asserção neste teste como `assert_eq!(4, result)`, o que
resultaria na mesma mensagem de falha exibindo `` assertion `left == right` failed ``.

O macro `assert_ne!` passará se os dois valores que fornecemos a ele não forem
iguais e falhará se forem iguais. Este macro é mais útil para casos em que não
temos certeza de qual _será_ um valor, mas sabemos qual o valor que ele
definitivamente _não deveria_ ser. Por exemplo, se estamos testando uma função
que tem garantia de alterar sua entrada de alguma forma, mas a forma como a
entrada é alterada depende do dia da semana em que executamos nossos testes, a
melhor coisa a afirmar pode ser que a saída da função não é igual à entrada.

Por baixo dos panos, os macros `assert_eq!` e `assert_ne!` usam os operadores `==`
e `!=`, respectivamente. Quando as asserções falham, esses macros imprimem seus
argumentos usando a formatação de depuração (`debug`), o que significa que os
valores que estão sendo comparados devem implementar os traits `PartialEq` e
`Debug`. Todos os tipos primitivos e a maioria dos tipos da biblioteca padrão
implementam esses traits. Para structs e enums que você mesmo define, você
precisará implementar `PartialEq` para afirmar a igualdade desses tipos. Você
também precisará implementar `Debug` para imprimir os valores quando a asserção
falhar. Como ambos os traits são deriváveis, conforme mencionado na Listagem
5-12 no Capítulo 5, isso geralmente é tão simples quanto adicionar a anotação
`#[derive(PartialEq, Debug)]` à definição da sua struct ou enum. Veja o
Apêndice C, [“Traits Deriváveis,”][derivable-traits]<!-- ignore --> para mais
detalhes sobre esses e outros traits deriváveis.

### Adicionando Mensagens de Falha Personalizadas

Você também pode adicionar uma mensagem personalizada para ser impressa junto
com a mensagem de falha como argumentos opcionais para os macros `assert!`,
`assert_eq!` e `assert_ne!`. Quaisquer argumentos especificados após os
argumentos obrigatórios são repassados para o macro `format!` (discutido em
[“Concatenando com `+` ou `format!`”][concatenating]<!-- ignore --> no Capítulo
8), para que você possa passar uma string de formatação que contenha marcadores
`{}` e valores para irem nesses marcadores. Mensagens personalizadas são úteis
para documentar o que uma asserção significa; quando um teste falha, você terá
uma ideia melhor de qual é o problema com o código.

Por exemplo, digamos que temos uma função que cumprimenta as pessoas pelo nome e
queremos testar se o nome que passamos para a função aparece na saída:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Os requisitos para este programa ainda não foram acordados, e temos quase certeza
de que o texto `Hello` no início da saudação vai mudar. Decidimos que não queremos
ter que atualizar o teste quando os requisitos mudarem, então, em vez de
verificar a igualdade exata com o valor retornado pela função `greeting`, vamos
apenas afirmar que a saída contém o texto do parâmetro de entrada.

Agora vamos introduzir um bug neste código alterando `greeting` para excluir
`name` para ver como a falha padrão do teste se parece:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

A execução deste teste produz o seguinte:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Este resultado apenas indica que a asserção falhou e em qual linha a asserção
está. Uma mensagem de falha mais útil imprimiria o valor da função `greeting`.
Vamos adicionar uma mensagem de falha personalizada composta por uma string de
formatação com um marcador preenchido com o valor real que obtivemos da função
`greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Agora, quando executarmos o teste, obteremos uma mensagem de erro mais informativa:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Podemos ver o valor que realmente obtivemos na saída do teste, o que nos ajudaria
a depurar o que aconteceu em vez do que esperávamos que acontecesse.

### Verificando Pânicos com `should_panic`

Além de verificar os valores de retorno, é importante verificar se o nosso código
lida com condições de erro conforme o esperado. Por exemplo, considere o tipo
`Guess` que criamos no Capítulo 9, na Listagem 9-13. Outro código que usa
`Guess` depende da garantia de que instâncias de `Guess` conterão apenas valores
entre 1 e 100. Podemos escrever um teste que garanta que tentar criar uma
instância de `Guess` com um valor fora desse intervalo cause um pânico.

Fazemos isso adicionando o atributo `should_panic` à nossa função de teste. O
teste passa se o código dentro da função entrar em pânico; o teste falha se o
código dentro da função não entrar em pânico.

A Listagem 11-8 mostra um teste que verifica se as condições de erro de
`Guess::new` acontecem quando esperamos que aconteçam.

<Listing number="11-8" file-name="src/lib.rs" caption="Testando que uma condição causará um `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Colocamos o atributo `#[should_panic]` após o atributo `#[test]` e antes da
função de teste à qual ele se aplica. Vamos ver o resultado quando este teste
passa:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Parece bom! Agora vamos introduzir um bug em nosso código removendo a condição
de que a função `new` entrará em pânico se o valor for maior que 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Quando executamos o teste na Listagem 11-8, ele falhará:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Não obtemos uma mensagem muito útil neste caso, mas quando olhamos para a função
de teste, vemos que ela está anotada com `#[should_panic]`. A falha que obtivemos
significa que o código na função de teste não causou um pânico.

Testes que usam `should_panic` podem ser imprecisos. Um teste `should_panic`
passaria mesmo se o teste entrasse em pânico por um motivo diferente daquele que
esperávamos. Para tornar os testes `should_panic` mais precisos, podemos
adicionar um parâmetro opcional `expected` ao atributo `should_panic`. O
executor de testes garantirá que a mensagem de falha contenha o texto fornecido.
Por exemplo, considere o código modificado para `Guess` na Listagem 11-9, onde a
função `new` entra em pânico com mensagens diferentes, dependendo de o valor ser
muito pequeno ou muito grande.

<Listing number="11-9" file-name="src/lib.rs" caption="Testando um `panic!` com uma mensagem de pânico contendo uma substring especificada">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Este teste passará porque o valor que colocamos no parâmetro `expected` do
atributo `should_panic` é uma substring da mensagem com a qual a função
`Guess::new` entra em pânico. Poderíamos ter especificado a mensagem de pânico
inteira que esperamos, que neste caso seria `Guess value must be less than or
equal to 100, got 200`. O que você escolhe especificar depende de quanto da
mensagem de pânico é única ou dinâmica e de quão preciso você deseja que seu
teste seja. Nesse caso, uma substring da mensagem de pânico é suficiente para
garantir que o código na função de teste execute o caso `else if value > 100`.

Para ver o que acontece quando um teste `should_panic` com uma mensagem
`expected` falha, vamos novamente introduzir um bug em nosso código trocando os
corpos dos blocos `if value < 1` e `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Desta vez, quando executarmos o teste `should_panic`, ele falhará:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

A mensagem de falha indica que este teste de fato entrou em pânico como
esperávamos, mas a mensagem de pânico não incluiu a string esperada `less than
or equal to 100`. A mensagem de pânico que obtivemos neste caso foi `Guess value
must be greater than or equal to 1, got 200`. Agora podemos começar a descobrir
onde está o nosso bug!

### Usando `Result<T, E>` em Testes

Todos os nossos testes até agora entram em pânico quando falham. Também podemos
escrever testes que usam `Result<T, E>`! Aqui está o teste da Listagem 11-1,
reescrito para usar `Result<T, E>` e retornar um `Err` em vez de entrar em pânico:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

A função `it_works` agora tem o tipo de retorno `Result<(), String>`. No corpo
da função, em vez de chamar o macro `assert_eq!`, retornamos `Ok(())` quando o
teste passa e um `Err` com uma `String` dentro quando o teste falha.

Escrever testes para que eles retornem um `Result<T, E>` permite que você use o
operador de interrogação (`?`) no corpo dos testes, o que pode ser uma maneira
conveniente de escrever testes que devem falhar se qualquer operação dentro deles
retornar uma variante `Err`.

Você não pode usar a anotação `#[should_panic]` em testes que usam `Result<T,
E>`. Para afirmar que uma operação retorna uma variante `Err`, _não_ use o
operador de interrogação no valor `Result<T, E>`. Em vez disso, use
`assert!(value.is_err())`.

Agora que você conhece várias maneiras de escrever testes, vamos ver o que está
acontecendo quando executamos nossos testes e explorar as diferentes opções que
podemos usar com o `cargo test`.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
