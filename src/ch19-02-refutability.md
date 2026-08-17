## Refutabilidade: Se um Padrão Pode Falhar ao Coincidir

Os padrões vêm em duas formas: refutáveis e irrefutáveis. Padrões que coincidem
para qualquer valor possível passado são _irrefutáveis_. Um exemplo seria `x` na
instrução `let x = 5;` porque `x` coincide com qualquer coisa e, portanto, não pode
falhar ao coincidir. Padrões que podem falhar ao coincidir para algum valor possível são
_refutáveis_. Um exemplo seria `Some(x)` na expressão `if let Some(x) =
a_value` porque se o valor na variável `a_value` for `None` em vez de
`Some`, o padrão `Some(x)` não coincidirá.

Parâmetros de função, instruções `let` e loops `for` só podem aceitar
padrões irrefutáveis porque o programa não pode fazer nada útil quando os
valores não coincidem. As expressões `if let` e `while let` e a
instrução `let...else` aceitam padrões refutáveis e irrefutáveis, mas o
compilador emite um aviso contra padrões irrefutáveis porque, por definição, eles são
destinados a lidar com falhas possíveis: A funcionalidade de um condicional está em
sua capacidade de agir de maneira diferente dependendo do sucesso ou da falha.

Em geral, você não deve se preocupar com a distinção entre padrões refutáveis
e irrefutáveis; no entanto, você precisa estar familiarizado com o conceito
de refutabilidade para poder responder quando o ver em uma mensagem de erro. Nesses
casos, você precisará alterar o padrão ou a construção com a qual está
usando o padrão, dependendo do comportamento pretendido do código.

Vamos ver um exemplo do que acontece quando tentamos usar um padrão refutável
onde o Rust exige um padrão irrefutável e vice-versa. A Listagem 19-8 mostra uma
instrução `let`, mas para o padrão, especificamos `Some(x)`, um padrão refutável.
Como você pode esperar, este código não compilará.

<Listing number="19-8" caption="Tentando usar um padrão refutável com `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Se `some_option_value` fosse um valor `None`, ele falharia ao coincidir com o padrão
`Some(x)`, o que significa que o padrão é refutável. No entanto, a instrução `let` pode
aceitar apenas um padrão irrefutável porque não há nada válido que o código possa
fazer com um valor `None`. Em tempo de compilação, o Rust reclamará que tentamos
usar um padrão refutável onde um padrão irrefutável é necessário:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Como não cobrimos (e não poderíamos cobrir!) todos os valores válidos com o
padrão `Some(x)`, o Rust corretamente gera um erro de compilação.

Se tivermos um padrão refutável onde um padrão irrefutável é necessário, podemos
consertá-lo alterando o código que usa o padrão: Em vez de usar `let`,
podemos usar `let...else`. Então, se o padrão não coincidir, o código nas chaves
lidará com o valor. A Listagem 19-9 mostra como consertar o código da
Listagem 19-8.

<Listing number="19-9" caption="Usando `let...else` e um bloco com padrões refutáveis em vez de `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Demos ao código uma saída! Este código é perfeitamente válido, embora signifique que
não podemos usar um padrão irrefutável sem receber um aviso. Se dermos a
`let...else` um padrão que sempre coincidirá, como `x`, conforme mostrado na Listagem
19-10, o compilador emitirá um aviso.

<Listing number="19-10" caption="Tentando usar um padrão irrefutável com `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

O Rust reclama que não faz sentido usar `let...else` com um
padrão irrefutável porque o `else` nunca será alcançado:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Por esse motivo, os braços de `match` devem usar padrões refutáveis, exceto pelo último
braço, que deve coincidir com quaisquer valores restantes usando um padrão irrefutável. O Rust
nos permite usar um padrão irrefutável em um `match` com apenas um braço, mas
essa sintaxe não é particularmente útil e pode ser substituída por uma instrução `let`
mais simples.

Agora que você sabe onde usar padrões e a diferença entre padrões refutáveis
e irrefutáveis, vamos abordar toda a sintaxe que podemos usar para criar
padrões.