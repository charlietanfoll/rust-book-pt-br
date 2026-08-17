## Apêndice C: Traits Deriváveis

Em vários lugares do livro, discutimos o atributo `derive`, que você pode aplicar
a uma definição de `struct` ou `enum`. O atributo `derive` gera código que
implementará uma trait com sua própria implementação padrão no tipo que você
anotou com a sintaxe `derive`.

Neste apêndice, fornecemos uma referência de todas as traits na biblioteca
padrão que você pode usar com `derive`. Cada seção aborda:

- Quais operadores e métodos a derivação desta trait habilitará
- O que a implementação da trait fornecida por `derive` faz
- O que a implementação da trait significa sobre o tipo
- As condições sob as quais você tem permissão ou não para implementar a trait
- Exemplos de operações que exigem a trait

Se você quiser um comportamento diferente daquele fornecido pelo atributo
`derive`, consulte a [documentação da biblioteca padrão](../std/index.html)<!-- ignore -->
para cada trait para obter detalhes sobre como implementá-las manualmente.

As traits listadas aqui são as únicas definidas pela biblioteca padrão que
podem ser implementadas em seus tipos usando `derive`. Outras traits definidas na
biblioteca padrão não têm um comportamento padrão sensato, então cabe a você
implementá-las da maneira que faça sentido para o que você está tentando
realizar.

Um exemplo de trait que não pode ser derivada é `Display`, que lida com a
formatação para usuários finais. Você deve sempre considerar a maneira
apropriada de exibir um tipo para um usuário final. Quais partes do tipo o
usuário final deve ter permissão para ver? Quais partes ele acharia relevantes?
Qual formato dos dados seria mais relevante para ele? O compilador Rust não tem
essa percepção, portanto, ele não pode fornecer um comportamento padrão
apropriado para você.

A lista de traits deriváveis fornecida neste apêndice não é abrangente: as
bibliotecas podem implementar `derive` para suas próprias traits, tornando a
lista de traits com as quais você pode usar `derive` verdadeiramente aberta. A
implementação de `derive` envolve o uso de uma macro procedural, que é abordada
na seção [“Macros `derive` Personalizadas”][custom-derive-macros]<!-- ignore -->
no Capítulo 20.

### `Debug` para Saída do Programador

A trait `Debug` habilita a formatação de depuração em strings de formato, o que
você indica adicionando `:?` dentro dos marcadores `{}`.

A trait `Debug` permite imprimir instâncias de um tipo para fins de depuração,
para que você e outros programadores que usam seu tipo possam inspecionar uma
instância em um ponto específico da execução de um programa.

A trait `Debug` é necessária, por exemplo, no uso da macro `assert_eq!`. Esta
macro imprime os valores das instâncias fornecidas como argumentos se a
asserção de igualdade falhar, para que os programadores possam ver por que as
duas instâncias não eram iguais.

### `PartialEq` e `Eq` para Comparações de Igualdade

A trait `PartialEq` permite comparar instâncias de um tipo para verificar a
igualdade e habilita o uso dos operadores `==` e `!=`.

Derivar `PartialEq` implementa o método `eq`. Quando `PartialEq` é derivado em
structs, duas instâncias são iguais apenas se *todos* os campos forem iguais, e
as instâncias não são iguais se *qualquer* campo não for igual. Quando derivado
em enums, cada variante é igual a si mesma e não é igual às outras variantes.

A trait `PartialEq` é necessária, por exemplo, com o uso da macro `assert_eq!`,
que precisa ser capaz de comparar duas instâncias de um tipo quanto à
igualdade.

A trait `Eq` não tem métodos. Seu objetivo é sinalizar que para cada valor do
tipo anotado, o valor é igual a si mesmo. A trait `Eq` só pode ser aplicada a
tipos que também implementam `PartialEq`, embora nem todos os tipos que
implementam `PartialEq` possam implementar `Eq`. Um exemplo disso são os tipos
de números de ponto flutuante: a implementação de números de ponto flutuante
afirma que duas instâncias do valor não-é-um-número (`NaN`) não são iguais entre
si.

Um exemplo de quando `Eq` é necessário é para chaves em um `HashMap<K, V>` para
que o `HashMap<K, V>` possa dizer se duas chaves são iguais.

### `PartialOrd` e `Ord` para Comparações de Ordenação

A trait `PartialOrd` permite comparar instâncias de um tipo para fins de
ordenação. Um tipo que implementa `PartialOrd` pode ser usado com os operadores
`<`, `>`, `<=` e `>=`. Você só pode aplicar a trait `PartialOrd` a tipos que
também implementam `PartialEq`.

Derivar `PartialOrd` implementa o método `partial_cmp`, que retorna um
`Option<Ordering>` que será `None` quando os valores fornecidos não produzirem
uma ordenação. Um exemplo de valor que não produz uma ordenação, embora a
maioria dos valores desse tipo possa ser comparada, é o valor de ponto flutuante
`NaN`. Chamar `partial_cmp` com qualquer número de ponto flutuante e o valor de
ponto flutuante `NaN` retornará `None`.

Quando derivado em structs, `PartialOrd` compara duas instâncias comparando o
valor em cada campo na ordem em que os campos aparecem na definição da struct.
Quando derivado em enums, as variantes do enum declaradas anteriormente na
definição do enum são consideradas menores do que as variantes listadas
posteriormente.

A trait `PartialOrd` é necessária, por exemplo, para o método `gen_range` da
crate `rand` que gera um valor aleatório no intervalo especificado por uma
expressão de intervalo.

A trait `Ord` permite saber que para quaisquer dois valores do tipo anotado,
existirá uma ordenação válida. A trait `Ord` implementa o método `cmp`, que
retorna um `Ordering` em vez de um `Option<Ordering>` porque uma ordenação
válida sempre será possível. Você só pode aplicar a trait `Ord` a tipos que
também implementam `PartialOrd` e `Eq` (e `Eq` requer `PartialEq`). Quando
derivado em structs e enums, `cmp` se comporta da mesma maneira que a
implementação derivada para `partial_cmp` faz com `PartialOrd`.

Um exemplo de quando `Ord` é necessário é ao armazenar valores em um
`BTreeSet<T>`, uma estrutura de dados que armazena dados com base na ordem de
classificação dos valores.

### `Clone` e `Copy` para Duplicação de Valores

A trait `Clone` permite criar explicitamente uma cópia profunda de um valor, e
o processo de duplicação pode envolver a execução de código arbitrário e a
cópia de dados do heap. Consulte a seção [“Variáveis e Dados Interagindo com
Clone”][variables-and-data-interacting-with-clone]<!-- ignore --> no Capítulo 4
para obter mais informações sobre `Clone`.

Derivar `Clone` implementa o método `clone`, que, quando implementado para todo o
tipo, chama `clone` em cada uma das partes do tipo. Isso significa que todos os
campos ou valores no tipo também devem implementar `Clone` para derivar `Clone`.

Um exemplo de quando `Clone` é necessário é ao chamar o método `to_vec` em uma
fatia (*slice*). A fatia não é proprietária das instâncias de tipo que contém,
mas o vetor retornado de `to_vec` precisará ser proprietário de suas
instâncias, então `to_vec` chama `clone` em cada item. Portanto, o tipo
armazenado na fatia deve implementar `Clone`.

A trait `Copy` permite duplicar um valor copiando apenas os bits armazenados na
stack; nenhum código arbitrário é necessário. Consulte a seção [“Dados
Apenas na Stack: Copy”][stack-only-data-copy]<!-- ignore --> no Capítulo 4 para
obter mais informações sobre `Copy`.

A trait `Copy` não define nenhum método para evitar que os programadores sobrecarreguem
esses métodos e violem a premissa de que nenhum código arbitrário está sendo
executado. Desse modo, todos os programadores podem assumir que a cópia de um
valor será muito rápida.

Você pode derivar `Copy` em qualquer tipo cujas partes implementem `Copy`. Um
tipo que implementa `Copy` também deve implementar `Clone` porque um tipo que
implementa `Copy` tem uma implementação trivial de `Clone` que executa a mesma
tarefa que `Copy`.

A trait `Copy` raramente é necessária; tipos que implementam `Copy` têm
otimizações disponíveis, o que significa que você não precisa chamar `clone`, o
que torna o código mais conciso.

Tudo o que é possível com `Copy` você também pode realizar com `Clone`, mas o
código pode ser mais lento ou ter que usar `clone` em alguns lugares.

### `Hash` para Mapear um Valor para um Valor de Tamanho Fixo

A trait `Hash` permite pegar uma instância de um tipo de tamanho arbitrário e
mapear essa instância para um valor de tamanho fixo usando uma função de hash.
Derivar `Hash` implementa o método `hash`. A implementação derivada do método
`hash` combina o resultado de chamar `hash` em cada uma das partes do tipo, o
que significa que todos os campos ou valores também devem implementar `Hash`
para derivar `Hash`.

Um exemplo de quando `Hash` é necessário é no armazenamento de chaves em um
`HashMap<K, V>` para armazenar dados eficientemente.

### `Default` para Valores Padrão

A trait `Default` permite criar um valor padrão para um tipo. Derivar `Default`
implementa a função `default`. A implementação derivada da função `default`
chama a função `default` em cada parte do tipo, o que significa que todos os
campos ou valores no tipo também devem implementar `Default` para derivar
`Default`.

A função `Default::default` é comumente usada em combinação com a sintaxe de
atualização de struct discutida na seção [“Criando Instâncias a partir de Outras
Instâncias com a Sintaxe de Atualização de
Struct”][creating-instances-from-other-instances-with-struct-update-syntax]<!--
ignore --> no Capítulo 5. Você pode personalizar alguns campos de uma struct e,
em seguida, definir e usar um valor padrão para o restante dos campos usando
`..Default::default()`.

A trait `Default` é necessária quando você usa o método `unwrap_or_default` em
instâncias de `Option<T>`, por exemplo. Se o `Option<T>` for `None`, o método
`unwrap_or_default` retornará o resultado de `Default::default` para o tipo
`T` armazenado no `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
