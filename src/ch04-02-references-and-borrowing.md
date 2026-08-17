## Referências e Empréstimos (Borrowing)

O problema com o código da tupla na Listagem 4-5 é que precisamos retornar a
`String` para a função chamadora para que ainda possamos usar a `String` após a
chamada para `calculate_length`, porque a `String` foi movida para dentro de
`calculate_length`. Em vez disso, podemos fornecer uma referência ao valor da
`String`. Uma referência é como um ponteiro, pois é um endereço que podemos
seguir para acessar os dados armazenados nesse endereço; esses dados são de
propriedade de alguma outra variável. Ao contrário de um ponteiro, é garantido
que uma referência aponte para um valor válido de um tipo específico durante o
tempo de vida dessa referência.

Aqui está como você definiria e usaria uma função `calculate_length` que possui
uma referência a um objeto como parâmetro, em vez de assumir a propriedade do
valor:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Primeiro, note que todo o código da tupla na declaração da variável e no valor de
retorno da função desapareceu. Segundo, note que passamos `&s1` para
`calculate_length` e, em sua definição, aceitamos `&String` em vez de
`String`. Esses e-comerciais (`&`) representam referências, e eles permitem que
você se refira a algum valor sem assumir a propriedade dele. A Figura 4-6 retrata
esse conceito.

<img alt="Three tables: the table for s contains only a pointer to the table
for s1. The table for s1 contains the stack data for s1 and points to the
string data on the heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">Figura 4-6: Um diagrama de `&String` `s` apontando para
`String` `s1`</span>

> Nota: O oposto de referenciar usando `&` é a _desreferenciação_ (dereferencing),
> que é realizada com o operador de desreferência, `*`. Veremos alguns usos do
> operador de desreferência no Capítulo 8 e discutiremos os detalhes da
> desreferenciação no Capítulo 15.

Vamos dar uma olhada mais de perto na chamada de função aqui:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

A sintaxe `&s1` nos permite criar uma referência que _se refere_ ao valor de
`s1`, mas não o possui. Como a referência não o possui, o valor para o qual ela
aponta não será descartado (dropped) quando a referência deixar de ser usada.

Da mesma forma, a assinatura da função usa `&` para indicar que o tipo do
parâmetro `s` é uma referência. Vamos adicionar algumas anotações explicativas:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

O escopo em que a variável `s` é válida é o mesmo que o escopo de qualquer
parâmetro de função, mas o valor apontado pela referência não é descartado quando
`s` para de ser usado, porque `s` não tem a propriedade. Quando as funções têm
referências como parâmetros em vez dos valores reais, não precisaremos retornar
os valores para devolver a propriedade, porque nunca tivemos a propriedade.

Chamamos a ação de criar uma referência de _empréstimo_ (borrowing). Como na vida
real, se uma pessoa possui algo, você pode pegar emprestado dela. Quando terminar,
você tem que devolver. Você não é o dono.

Então, o que acontece se tentarmos modificar algo que estamos emprestando?
Tente o código na Listagem 4-6. Alerta de spoiler: Não funciona!

<Listing number="4-6" file-name="src/main.rs" caption="Tentando modificar um valor emprestado">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Assim como as variáveis são imutáveis por padrão, as referências também são.
Não temos permissão para modificar algo para o qual temos uma referência.

### Referências Mutáveis

Podemos corrigir o código da Listagem 4-6 para nos permitir modificar um valor
emprestado com apenas alguns pequenos ajustes que usam, em vez disso, uma
_referência mutável_:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Primeiro, alteramos `s` para ser `mut`. Em seguida, criamos uma referência mutável
com `&mut s` onde chamamos a função `change` e atualizamos a assinatura da função
para aceitar uma referência mutável com `some_string: &mut String`. Isso deixa
muito claro que a função `change` irá mutar o valor que ela empresta.

Referências mutáveis têm uma grande restrição: Se você tiver uma referência
mutável para um valor, não poderá ter nenhuma outra referência para esse valor.
Este código que tenta criar duas referências mutáveis para `s` vai falhar:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Este erro diz que este código é inválido porque não podemos emprestar `s` como
mutável mais de uma vez por vez. O primeiro empréstimo mutável está em `r1` e
deve durar até ser usado no `println!`, mas entre a criação dessa referência
mutável e seu uso, tentamos criar outra referência mutável em `r2` que empresta
os mesmos dados que `r1`.

A restrição que impede múltiplas referências mutáveis para os mesmos dados ao
mesmo tempo permite a mutação, mas de uma maneira muito controlada. É algo com o
qual os novos "Rustaceans" lutam porque a maioria das linguagens permite que você
mute sempre que quiser. O benefício de ter essa restrição é que o Rust pode
prevenir "data races" (corridas de dados) em tempo de compilação. Uma _corrida de
dados_ é semelhante a uma condição de corrida (race condition) e ocorre quando
estes três comportamentos acontecem:

- Dois ou mais ponteiros acedem aos mesmos dados ao mesmo tempo.
- Pelo menos um dos ponteiros está sendo usado para escrever nos dados.
- Não há mecanismo sendo usado para sincronizar o acesso aos dados.

Corridas de dados causam comportamento indefinido e podem ser difíceis de
diagnosticar e corrigir quando você está tentando rastreá-las em tempo de
execução; o Rust evita esse problema recusando-se a compilar códigos com corridas
de dados!

Como sempre, podemos usar chaves para criar um novo escopo, permitindo múltiplas
referências mutáveis, apenas não _simultâneas_:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

O Rust impõe uma regra semelhante para combinar referências mutáveis e imutáveis.
Este código resulta em um erro:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Here’s the error:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Ufa! Nós _também_ não podemos ter uma referência mutável enquanto tivermos uma
imutável para o mesmo valor.

Os usuários de uma referência imutável não esperam que o valor mude repentinamente
debaixo deles! No entanto, múltiplas referências imutáveis são permitidas porque
ninguém que está apenas lendo os dados tem a capacidade de afetar a leitura dos
dados por outra pessoa.

Note que o escopo de uma referência começa de onde ela é introduzida e continua
até a última vez que essa referência é usada. Por exemplo, este código irá
compilar porque o último uso das referências imutáveis é no `println!`, antes
que a referência mutável seja introduzida:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Os escopos das referências imutáveis `r1` e `r2` terminam após o `println!` onde
são usadas pela última vez, o que ocorre antes da referência mutável `r3` ser
criada. Esses escopos não se sobrepõem, então este código é permitido: O
compilador pode dizer que a referência não está mais sendo usada em um ponto
anterior ao final do escopo.

Mesmo que os erros de empréstimo possam ser frustrantes às vezes, lembre-se de
que é o compilador do Rust apontando um bug potencial cedo (em tempo de
compilação, em vez de em tempo de execução) e mostrando exatamente onde está o
problema. Assim, você não precisa rastrear o porquê de seus dados não serem o
que você pensava que eram.

### Referências Pendentes (Dangling References)

Em linguagens com ponteiros, é fácil criar erroneamente um _ponteiro pendente_
(dangling pointer) — um ponteiro que faz referência a um local na memória que
pode ter sido dado a outra pessoa — liberando alguma memória enquanto preserva
um ponteiro para essa memória. No Rust, por outro lado, o compilador garante que
as referências nunca serão referências pendentes: Se você tiver uma referência a
algum dado, o compilador garantirá que os dados não sairão de escopo antes que a
referência aos dados o faça.

Vamos tentar criar uma referência pendente para ver como o Rust as evita com um
erro em tempo de compilação:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Esta mensagem de erro refere-se a um recurso que ainda não cobrimos: tempos de
vida (lifetimes). Discutiremos os tempos de vida em detalhes no Capítulo 10.
Mas, se você desconsiderar as partes sobre tempos de vida, a mensagem contém a
chave para o motivo pelo qual este código é um problema:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Vamos dar uma olhada mais de perto no que está acontecendo exatamente em cada
estágio do nosso código `dangle`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Como `s` é criado dentro de `dangle`, quando o código de `dangle` terminar, `s`
será desalocado. Mas tentamos retornar uma referência a ele. Isso significa que
essa referência estaria apontando para uma `String` inválida. Isso não é bom!
O Rust não nos deixa fazer isso.

A solução aqui é retornar a `String` diretamente:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Isso funciona sem nenhum problema. A propriedade é movida para fora e nada é
desalocado.

### As Regras das Referências

Vamos recapitular o que discutimos sobre referências:

- A qualquer momento, você pode ter _ou_ uma referência mutável, _ou_ qualquer
  número de referências imutáveis.
- As referências devem ser sempre válidas.

A seguir, veremos um tipo diferente de referência: fatias (slices).