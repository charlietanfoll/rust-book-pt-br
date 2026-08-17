# Tipos Genéricos, Traits e Tempos de Vida (Lifetimes)

Toda linguagem de programação possui ferramentas para lidar de forma eficaz com
a duplicação de conceitos. Em Rust, uma dessas ferramentas são os _genéricos_
(generics): substitutos abstratos para tipos concretos ou outras propriedades.
Podemos expressar o comportamento de genéricos ou como eles se relacionam com
outros genéricos sem saber o que estará em seu lugar ao compilar e executar o
código.

Funções podem aceitar parâmetros de algum tipo genérico, em vez de um tipo
concreto como `i32` ou `String`, da mesma forma que aceitam parâmetros com
valores desconhecidos para executar o mesmo código em múltiplos valores
concretos. De fato, já usamos genéricos no Capítulo 6 com `Option<T>`, no
Capítulo 8 com `Vec<T>` e `HashMap<K, V>`, e no Capítulo 9 com `Result<T, E>`.
Neste capítulo, você explorará como definir seus próprios tipos, funções e
métodos com genéricos!

Primeiro, revisaremos como extrair uma função para reduzir a duplicação de
código. Em seguida, usaremos a técnica mesma para criar uma função genérica a
partir de duas funções que diferem apenas nos tipos de seus parâmetros. Também
explicaremos como usar tipos genéricos em definições de `struct` e `enum`.

Depois, você aprenderá a usar `traits` para definir comportamento de maneira
genérica. Você pode combinar `traits` com tipos genéricos para restringir um
tipo genérico a aceitar apenas aqueles tipos que possuem um comportamento
específico, em vez de qualquer tipo.

Finalmente, discutiremos _tempos de vida_ (_lifetimes_): uma variedade de
genéricos que fornecem informações ao compilador sobre como as referências se
relacionam entre si. Os tempos de vida nos permitem dar informações suficientes
ao compilador sobre valores emprestados para que ele possa garantir que as
referências serão válidas em mais situações do que seria possível sem a nossa
ajuda.

## Removendo Duplicação Extraindo uma Função

Os genéricos nos permitem substituir tipos específicos por um espaço reservado
(_placeholder_) que representa múltiplos tipos para remover a duplicação de
código. Antes de mergulhar na sintaxe de genéricos, vamos primeiro analisar
como remover a duplicação de uma forma que não envolva tipos genéricos,
extraindo uma função que substitui valores específicos por um espaço reservado
que representa múltiplos valores. Em seguida, aplicaremos a mesma técnica para
extrair uma função genérica! Ao observar como reconhecer código duplicado que
você pode extrair para uma função, você começará a reconhecer código duplicado
que pode usar genéricos.

Começaremos com o pequeno programa na Listagem 10-1 que encontra o maior número
em uma lista.

<Listing number="10-1" file-name="src/main.rs" caption="Encontrando o maior número em uma lista de números">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Armazenamos uma lista de inteiros na variável `number_list` e colocamos uma
referência para o primeiro número da lista em uma variável chamada `largest`. Em
seguida, iteramos por todos os números da lista e, se o número atual for maior
que o número armazenado em `largest`, substituímos a referência nessa variável.
No entanto, se o número atual for menor ou igual ao maior número visto até
agora, a variável não muda e o código avança para o próximo número da lista.
Após considerar todos os números da lista, `largest` deve se referir ao maior
número, que neste caso é 100.

Agora recebemos a tarefa de encontrar o maior número em duas listas diferentes
de números. Para fazer isso, podemos optar por duplicar o código da Listagem
10-1 e usar a mesma lógica em dois lugares diferentes do programa, conforme
mostrado na Listagem 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Código para encontrar o maior número em *duas* listas de números">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Embora este código funcione, duplicar código é entediante e suscetível a erros.
Também precisamos nos lembrar de atualizar o código em vários lugares quando
quisermos alterá-lo.

Para eliminar essa duplicação, criaremos uma abstração definindo uma função que
opera em qualquer lista de inteiros passada como parâmetro. Essa solução torna
nosso código mais claro e nos permite expressar o conceito de encontrar o maior
número em uma lista de forma abstrata.

Na Listagem 10-3, extraímos o código que encontra o maior número para uma função
chamada `largest`. Em seguida, chamamos a função para encontrar o maior número
nas duas listas da Listagem 10-2. Também poderíamos usar a função em qualquer
outra lista de valores `i32` que possamos ter no futuro.

<Listing number="10-3" file-name="src/main.rs" caption="Código abstraído para encontrar o maior número em duas listas">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

A função `largest` possui um parâmetro chamado `list`, que representa qualquer
fatia (_slice_) concreta de valores `i32` que possamos passar para a função.
Como resultado, quando chamamos a função, o código é executado nos valores
específicos que passamos.

Em resumo, aqui estão os passos que seguimos para alterar o código da Listagem
10-2 para a Listagem 10-3:

1. Identificar o código duplicado.
1. Extrair o código duplicado para o corpo da função e especificar as entradas e
   os valores de retorno desse código na assinatura da função.
1. Atualizar as duas instâncias do código duplicado para chamar a função em seu
   lugar.

Em seguida, usaremos esses mesmos passos com genéricos para reduzir a duplicação
de código. Da mesma forma que o corpo da função pode operar em uma `list`
abstrata em vez de valores específicos, os genéricos permitem que o código
opere em tipos abstratos.

Por exemplo, digamos que tivéssemos duas funções: uma que encontra o maior item
em uma fatia de valores `i32` e outra que encontra o maior item em uma fatia de
valores `char`. Como eliminaríamos essa duplicação? Vamos descobrir!