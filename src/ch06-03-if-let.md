## Fluxo de Controle Conciso com `if let` e `let...else`

A sintaxe `if let` permite combinar `if` e `let` em uma forma menos verbosa para
lidar com valores que correspondem a um padrão enquanto ignoram o restante. Considere o
programa na Listagem 6-6 que faz correspondência em um valor `Option<u8>` na
variável `config_max`, mas quer apenas executar código se o valor for a variante
`Some`.

<Listing number="6-6" caption="Um `match` que se importa apenas em executar código quando o valor é `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Se o valor for `Some`, nós imprimimos o valor na variante `Some` fazendo a vinculação
do valor à variável `max` no padrão. Nós não queremos fazer nada
com o valor `None`. Para satisfazer a expressão `match`, temos que adicionar `_ =>
()` após processar apenas uma variante, o que é um código boilerplate chato de
adicionar.

Em vez disso, poderíamos escrever isso de uma forma mais curta usando `if let`. O código
a seguir se comporta da mesma forma que o `match` na Listagem 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

A sintaxe `if let` recebe um padrão e uma expressão separados por um sinal de
igual. Ela funciona da mesma maneira que um `match`, onde a expressão é dada ao
`match` e o padrão é o seu primeiro braço. Neste caso, o padrão é
`Some(max)`, e o `max` se vincula ao valor dentro do `Some`. Nós podemos então
usar `max` no corpo do bloco `if let` da mesma forma que usamos `max` no
braço do `match` correspondente. O código no bloco `if let` só é executado se o
valor corresponder ao padrão.

Usar `if let` significa menos digitação, menos indentação e menos código boilerplate.
No entanto, você perde a verificação exaustiva que o `match` impõe, o que garante
que você não está esquecendo de tratar nenhum caso. Escolher entre `match` e `if
let` depende do que você está fazendo na sua situação específica e se
ganhar concisão é uma compensação apropriada para a perda da verificação exaustiva.

Em outras palavras, você pode pensar no `if let` como um açúcar sintático para um `match` que
executa código quando o valor corresponde a um padrão e então ignora todos os outros valores.

Podemos incluir um `else` com um `if let`. O bloco de código que acompanha o
`else` é o mesmo bloco de código que acompanharia o caso `_` na
expressão `match` que é equivalente ao `if let` e `else`. Lembre-se da
definição do enum `Coin` na Listagem 6-4, onde a variante `Quarter` também continha um
valor `UsState`. Se quiséssemos contar todas as moedas que não são de 25 centavos que vemos ao mesmo
tempo em que anunciamos o estado das moedas de 25 centavos, poderíamos fazer isso com uma expressão
`match`, assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Ou poderíamos usar uma expressão `if let` e `else`, assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Permanecendo no "Caminho Feliz" com `let...else`

O padrão comum é realizar algum cálculo quando um valor está presente e
retornar um valor padrão caso contrário. Continuando com o nosso exemplo de moedas com um
valor `UsState`, se quiséssemos dizer algo engraçado dependendo de quão antigo era
o estado na moeda de 25 centavos, poderíamos introduzir um método em `UsState` para verificar a
idade de um estado, deste modo:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Então, poderíamos usar `if let` para fazer a correspondência no tipo de moeda, introduzindo uma
variável `state` dentro do corpo da condição, como na Listagem 6-7.

<Listing number="6-7" caption="Verificando se um estado existia em 1900 usando condicionais aninhadas dentro de um `if let`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Isso resolve o problema, mas empurrou o trabalho para dentro do corpo da instrução
`if let`, e se o trabalho a ser feito for mais complicado, pode ser
difícil acompanhar exatamente como os ramos de nível superior se relacionam. Também poderíamos
aproveitar o fato de que expressões produzem um valor para produzir o
`state` a partir do `if let` ou para retornar antecipadamente, como na Listagem 6-8. (Você poderia fazer
algo semelhante com um `match` também.)

<Listing number="6-8" caption="Usando `if let` para produzir um valor ou retornar antecipadamente">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

No entanto, isso é um pouco chato de acompanhar à sua própria maneira! Um ramo do `if
let` produz um valor, e o outro retorna da função inteiramente.

Para tornar este padrão comum mais agradável de expressar, o Rust possui o `let...else`. A
sintaxe `let...else` recebe um padrão no lado esquerdo e uma expressão no direito,
muito semelhante ao `if let`, mas ela não tem um ramo `if`, apenas um
ramo `else`. Se o padrão corresponder, ele fará a vinculação do valor do padrão
no escopo externo. Se o padrão _não_ corresponder, o programa fluirá para
o braço `else`, que deve retornar da função.

Na Listagem 6-9, você pode ver como a Listagem 6-8 fica ao usar `let...else` no
lugar de `if let`.

<Listing number="6-9" caption="Usando `let...else` para esclarecer o fluxo através da função">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Note que dessa forma ele permanece no "caminho feliz" no corpo principal da função,
sem ter um fluxo de controle significativamente diferente para dois ramos
da maneira que o `if let` fazia.

Se você tiver uma situação em que seu programa possui uma lógica muito verbosa para
ser expressa usando um `match`, lembre-se de que `if let` e `let...else` também
estão na sua caixa de ferramentas do Rust.

## Resumo

Agora cobrimos como usar enums para criar tipos personalizados que podem ser um de um
conjunto de valores enumerados. Mostramos como o tipo `Option<T>` da biblioteca padrão
ajuda você a usar o sistema de tipos para prevenir erros. Quando os valores de enum possuem
dados dentro deles, você pode usar `match` ou `if let` para extrair e usar esses
valores, dependendo de quantos casos você precisa tratar.

Seus programas em Rust agora podem expressar conceitos em seu domínio usando structs e
enums. Criar tipos personalizados para usar em sua API garante a segurança de tipos: O
compilador garantirá que suas funções recebam apenas valores do tipo que cada
função espera.

Para fornecer uma API bem organizada aos seus usuários que seja direta
de usar e exponha exatamente o que seus usuários precisarão, vamos agora nos voltar
para os módulos do Rust.
