## Armazenando Chaves com Valores Associados em Hash Maps

A última das nossas coleções comuns é o hash map. O tipo `HashMap<K, V>`
armazena um mapeamento de chaves do tipo `K` para valores do tipo `V` usando uma
_função de hash_ (_hashing function_), que determina como essas chaves e valores
são colocados na memória. Muitas linguagens de programação suportam este tipo
de estrutura de dados, mas frequentemente usam um nome diferente, como
_hash_, _map_ (mapa), _object_ (objeto), _hash table_ (tabela hash),
_dictionary_ (dicionário) ou _associative array_ (array associativo), apenas
para citar alguns.

Hash maps são úteis quando você quer buscar dados não usando um índice, como
você pode fazer com vetores, mas usando uma chave que pode ser de qualquer tipo.
Por exemplo, em um jogo, você pode controlar a pontuação de cada time em um hash
map no qual cada chave é o nome de um time e os valores são a pontuação de cada
um. Dado o nome de um time, você pode recuperar a pontuação dele.

Nesta seção, vamos ver a API básica de hash maps, mas há muitas outras
funcionalidades úteis escondidas nas funções definidas em `HashMap<K, V>` pela
biblioteca padrão. Como sempre, consulte a documentação da biblioteca padrão
para obter mais informações.

### Criando um Novo Hash Map

Uma maneira de criar um hash map vazio é usar `new` e adicionar elementos com
`insert`. Na Listagem 8-20, estamos controlando a pontuação de dois times cujos
nomes são _Blue_ (Azul) e _Yellow_ (Amarelo). O time Azul começa com 10 pontos,
e o time Amarelo começa com 50.

<Listing number="8-20" caption="Criando um novo hash map e inserindo algumas chaves e valores">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Note que primeiro precisamos trazer (`use`) o `HashMap` da parte de coleções da
biblioteca padrão. Das nossas três coleções comuns, esta é a menos usada, então
ela não é incluída automaticamente no escopo pelo _prelude_. Os hash maps
também têm menos suporte da biblioteca padrão; por exemplo, não há nenhuma macro
embutida para construí-los.

Assim como os vetores, os hash maps armazenam seus dados no _heap_. Este
`HashMap` tem chaves do tipo `String` e valores do tipo `i32`. Assim como os
vetores, os hash maps são homogêneos: todas as chaves devem ter o mesmo tipo e
todos os valores devem ter o mesmo tipo.

### Acessando Valores em um Hash Map

Podemos obter um valor de dentro do hash map fornecendo sua chave para o método
`get`, como mostrado na Listagem 8-21.

<Listing number="8-21" caption="Acessando a pontuação do time Azul armazenada no hash map">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Aqui, `score` terá o valor associado ao time Azul, e o resultado será `10`. O
método `get` retorna um `Option<&V>`; se não houver valor para essa chave no hash
map, `get` retornará `None`. Este programa lida com o `Option` chamando `copied`
para obter um `Option<i32>` em vez de um `Option<&i32>`, e então `unwrap_or` para
definir `score` como zero se `scores` não tiver uma entrada para a chave.

Podemos iterar sobre cada par chave-valor em um hash map de maneira similar à
que fazemos com vetores, usando um loop `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Este código imprimirá cada par em uma ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### Gerenciando a Posse (Ownership) em Hash Maps

Para tipos que implementam o _trait_ `Copy`, como `i32`, os valores são copiados
para dentro do hash map. Para valores com posse explícita como `String`, os
valores serão movidos (moved) e o hash map se tornará o dono desses valores,
conforme demonstrado na Listagem 8-22.

<Listing number="8-22" caption="Mostrando que chaves e valores passam a pertencer ao hash map assim que são inseridos">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Não podemos mais usar as variáveis `field_name` e `field_value` após elas terem
sido movidas para o hash map através da chamada a `insert`.

Se inserirmos referências a valores no hash map, os valores não serão movidos
para dentro dele. Os valores para os quais as referências apontam devem
permanecer válidos por pelo menos tanto tempo quanto o hash map for válido.
Falaremos mais sobre essas questões em [“Validando Referências com
Tempos de Vida (Lifetimes)”][validating-references-with-lifetimes]<!-- ignore -->
no Capítulo 10.

### Atualizando um Hash Map

Embora o número de pares de chaves e valores seja expansível, cada chave única
pode ter apenas um valor associado a ela de cada vez (mas não o contrário: por
exemplo, tanto o time Azul quanto o time Amarelo poderiam ter o valor `10`
armazenado no hash map `scores`).

Quando você quer alterar os dados em um hash map, você tem que decidir como
tratar o caso em que uma chave já tem um valor atribuído. Você pode substituir
o valor antigo pelo novo valor, desconsiderando completamente o valor antigo.
Você pode manter o valor antigo e ignorar o novo valor, adicionando o novo
valor apenas se a chave _ainda não_ tiver um valor. Ou você pode combinar o
valor antigo e o novo valor. Vamos ver como fazer cada uma dessas operações!

#### Sobrescrevendo um Valor

Se inserirmos uma chave e um valor em um hash map e depois inserirmos essa mesma
chave com um valor diferente, o valor associado a essa chave será substituído.
Mesmo que o código na Listagem 8-23 chame `insert` duas vezes, o hash map conterá
apenas um par chave-valor porque estamos inserindo o valor para a chave do time
Azul em ambas as ocasiões.

<Listing number="8-23" caption="Substituindo um valor armazenado com uma chave específica">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Este código imprimirá `{"Blue": 25}`. O valor original de `10` foi sobrescrito.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Adicionando uma Chave e Valor Apenas Se a Chave Não Estiver Presente

É comum verificar se uma chave específica já existe no hash map com um valor e,
em seguida, tomar as seguintes ações: se a chave já existe no hash map, o valor
existente deve permanecer como está; se a chave não existe, insira-a junto com
um valor para ela.

Os hash maps têm uma API especial para isso chamada `entry` que recebe a chave
que você quer verificar como parâmetro. O valor de retorno do método `entry` é
um enum chamado `Entry` que representa um valor que pode ou não existir.
Digamos que queremos verificar se a chave do time Amarelo tem um valor
associado a ela. Se não tiver, queremos inserir o valor `50`, e o mesmo para o
time Azul. Usando a API `entry`, o código fica como na Listagem 8-24.

<Listing number="8-24" caption="Usando o método `entry` para inserir apenas se a chave ainda não tiver um valor">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

O método `or_insert` em `Entry` é definido para retornar uma referência mutável
ao valor da chave correspondente em `Entry` se essa chave existir; caso
contrário, ele insere o parâmetro como o novo valor para esta chave e retorna
uma referência mutável para o novo valor. Essa técnica é muito mais limpa do
que escrever a lógica por conta própria e, além disso, lida muito melhor com o
verificador de empréstimos (_borrow checker_).

Executar o código na Listagem 8-24 imprimirá `{"Yellow": 50, "Blue": 10}`. A
primeira chamada a `entry` inserirá a chave para o time Amarelo com o valor `50`
porque o time Amarelo ainda não tem um valor. A segunda chamada a `entry` não
alterará o hash map, porque o time Azul já tem o valor `10`.

#### Atualizando um Valor Baseado no Valor Antigo

Outro caso de uso comum para hash maps é buscar o valor de uma chave e então
atualizá-lo com base no valor antigo. Por exemplo, a Listagem 8-25 mostra um
código que conta quantas vezes cada palavra aparece em um texto. Usamos um hash
map com as palavras como chaves e incrementamos o valor para controlar quantas
vezes vimos essa palavra. Se for a primeira vez que vemos uma palavra, primeiro
inseriremos o valor `0`.

<Listing number="8-25" caption="Contando ocorrências de palavras usando um hash map que armazena palavras e contagens">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Este código imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. Você pode ver os
mesmos pares chave-valor impressos em uma ordem diferente: Lembre-se de [“Acessando
Valores em um Hash Map”][access]<!-- ignore --> que a iteração sobre um hash map
ocorre em uma ordem arbitrária.

O método `split_whitespace` retorna um iterador sobre subfatias (_subslices_),
separadas por espaços em branco, do valor em `text`. O método `or_insert`
retorna uma referência mutável (`&mut V`) para o valor da chave especificada.
Aqui, armazenamos essa referência mutável na variável `count`, então para poder
atribuir um valor a ela, devemos primeiro desreferenciar `count` usando o
asterisco (`*`). A referência mutável sai de escopo no final do loop `for`,
então todas essas alterações são seguras e permitidas pelas regras de
empréstimo.

### Funções de Hash (_Hashing Functions_)

Por padrão, o `HashMap` usa uma função de hash chamada _SipHash_ que pode
fornecer resistência a ataques de negação de serviço (DoS) envolvendo tabelas
hash[^siphash]<!-- ignore -->. Este não é o algoritmo de hash mais rápido
disponível, mas o compromisso por uma melhor segurança que vem com a queda no
desempenho vale a pena. Se você analisar o desempenho do seu código e achar que
a função de hash padrão está muito lenta para os seus propósitos, você pode
mudar para outra função especificando um _hasher_ diferente. Um _hasher_ é um
tipo que implementa o _trait_ `BuildHasher`. Falaremos sobre _traits_ e como
implementá-los no [Capítulo 10][traits]<!-- ignore -->. Você não precisa
necessariamente implementar seu próprio _hasher_ do zero; o
[crates.io](https://crates.io/)<!-- ignore --> possui bibliotecas
compartilhadas por outros usuários do Rust que fornecem _hashers_ implementando
muitos algoritmos de hash comuns.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Resumo

Vetores, strings e hash maps fornecerão uma grande quantidade de funcionalidade
necessária em programas quando você precisar armazenar, acessar e modificar
dados. Aqui estão alguns exercícios que agora você deve estar apto a resolver:

1. Dada uma lista de inteiros, use um vetor e retorne a mediana (quando
   ordenado, o valor na posição do meio) e a moda (o valor que ocorre com mais
   frequência; um hash map será útil aqui) da lista.
1. Converta strings para Pig Latin (latim dos porcos). A primeira consoante de
   cada palavra é movida para o final da palavra e _ay_ é adicionado, de modo
   que _first_ vira _irst-fay_. Palavras que começam com uma vogal recebem
   _hay_ adicionado ao final em vez disso (_apple_ vira _apple-hay_). Lembre-se
   dos detalhes sobre a codificação UTF-8!
1. Usando um hash map e vetores, crie uma interface de texto para permitir que
   um usuário adicione nomes de funcionários a um departamento em uma empresa;
   por exemplo, “Add Sally to Engineering” (Adicionar Sally à Engenharia) ou
   “Add Amir to Sales” (Adicionar Amir a Vendas). Em seguida, permita que o
   usuário recupere uma lista de todas as pessoas em um departamento ou de todas
   as pessoas na empresa por departamento, ordenadas alfabeticamente.

A documentação da API da biblioteca padrão descreve métodos que vetores,
strings e hash maps possuem e que serão úteis para esses exercícios!

Estamos entrando em programas mais complexos nos quais operações podem falhar,
então é o momento perfeito para discutir tratamento de erros. Faremos isso a
seguir!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
