<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## Adicionando Funcionalidades com Desenvolvimento Orientado a Testes (TDD)

Agora que temos a lógica de busca em _src/lib.rs_ separada da função `main`,
é muito mais fácil escrever testes para a funcionalidade principal do nosso
código. Podemos chamar funções diretamente com vários argumentos e verificar
os valores de retorno sem precisar chamar nosso binário a partir da linha de
comando.

Nesta seção, adicionaremos a lógica de busca ao programa `minigrep` usando
o processo de desenvolvimento orientado a testes (TDD) com os seguintes passos:

1. Escreva um teste que falha e execute-o para garantir que ele falhe pelo
   motivo esperado.
2. Escreva ou modifique apenas o código suficiente para fazer o novo teste passar.
3. Refatore o código que você acabou de adicionar ou alterar e certifique-se
   de que os testes continuam passando.
4. Repita a partir do passo 1!

Embora seja apenas uma das muitas maneiras de escrever software, o TDD pode
ajudar a direcionar o design do código. Escrever o teste antes de escrever o
código que faz o teste passar ajuda a manter uma alta cobertura de testes em
todo o processo.

Vamos testar a implementação da funcionalidade que fará a busca real pela string
de consulta no conteúdo do arquivo e produzirá uma lista de linhas que
correspondem à consulta. Adicionaremos essa funcionalidade em uma função chamada
`search`.

### Escrevendo um Teste que Falha

Em _src/lib.rs_, adicionaremos um módulo `tests` com uma função de teste,
como fizemos no [Capítulo 11][ch11-anatomy]<!-- ignore -->. A função de teste
especifica o comportamento que queremos que a função `search` tenha: ela
receberá uma consulta e o texto a ser pesquisado, e retornará apenas as linhas
do texto que contêm a consulta. A Listagem 12-15 mostra este teste.

<Listing number="12-15" file-name="src/lib.rs" caption="Criando um teste que falha para a função `search` com a funcionalidade que gostaríamos de ter">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Este teste busca pela string `"duct"`. O texto que estamos pesquisando tem três
linhas, sendo que apenas uma delas contém `"duct"` (observe que a barra invertida
após as aspas duplas de abertura diz ao Rust para não colocar um caractere de nova
linha no início do conteúdo deste literal de string). Afirmamos que o valor
retornado pela função `search` contém apenas a linha que esperamos.

Se executarmos este teste, ele atualmente falhará porque a macro `unimplemented!`
entra em pânico com a mensagem "not implemented" (não implementado). De acordo
com os princípios do TDD, daremos o pequeno passo de adicionar apenas o código
suficiente para fazer o teste não entrar em pânico ao chamar a função, definindo
a função `search` para sempre retornar um vetor vazio, conforme mostrado na
Listagem 12-16. Então, o teste deve compilar e falhar porque um vetor vazio não
corresponde a um vetor contendo a linha `"safe, fast, productive."`.

<Listing number="12-16" file-name="src/lib.rs" caption="Definindo o mínimo necessário da função `search` para que chamá-la não cause pânico">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Agora vamos discutir por que precisamos definir um tempo de vida explícito `'a`
na assinatura de `search` e usar esse tempo de vida com o argumento `contents`
e o valor de retorno. Lembre-se do [Capítulo 10][ch10-lifetimes]<!-- ignore -->
que os parâmetros de tempo de vida especificam qual tempo de vida do argumento
está conectado ao tempo de vida do valor de retorno. Neste caso, indicamos que
o vetor retornado deve conter fatias de string (string slices) que fazem referência
a fatias do argumento `contents` (em vez do argumento `query`).

Em outras palavras, dizemos ao Rust que os dados retornados pela função
`search` viverão tanto quanto os dados passados para a função `search` no
argumento `contents`. Isso é importante! Os dados referenciados _por_ uma fatia
precisam ser válidos para que a referência seja válida; se o compilador assumir
que estamos criando fatias de string de `query` em vez de `contents`, ele fará
sua verificação de segurança incorretamente.

Se esquecermos as anotações de tempo de vida e tentarmos compilar esta função,
obteremos este erro:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

O Rust não pode saber qual dos dois parâmetros precisamos para a saída, então
precisamos informá-lo explicitamente. Note que o texto de ajuda sugere especificar
o mesmo parâmetro de tempo de vida para todos os parâmetros e para o tipo de
saída, o que está incorreto! Como `contents` é o parâmetro que contém todo o
nosso texto e queremos retornar as partes desse texto que correspondem, sabemos
que `contents` é o único parâmetro que deve ser conectado ao valor de retorno
usando a sintaxe de tempo de vida.

Outras linguagens de programação não exigem que você conecte argumentos a
valores de retorno na assinatura, mas essa prática se tornará mais fácil com o
tempo. Você pode querer comparar este exemplo com os exemplos na seção
[“Validando Referências com Tempos de Vida”][validating-references-with-lifetimes]<!-- ignore -->
no Capítulo 10.

### Escrevendo Código para Passar no Teste

Atualmente, nosso teste está falhando porque sempre retornamos um vetor vazio.
Para corrigir isso e implementar `search`, nosso programa precisa seguir estes
passos:

1. Iterar através de cada linha do conteúdo.
2. Verificar se a linha contém nossa string de consulta.
3. Se contiver, adicioná-la à lista de valores que estamos retornando.
4. Se não contiver, não fazer nada.
5. Retornar a lista de resultados correspondentes.

Vamos trabalhar em cada passo, começando pela iteração através das linhas.

#### Iterando Através de Linhas com o Método `lines`

O Rust tem um método útil para lidar com a iteração linha por linha de strings,
convenientemente chamado de `lines`, que funciona como mostrado na
Listagem 12-17. Note que isso ainda não vai compilar.

<Listing number="12-17" file-name="src/lib.rs" caption="Iterando através de cada linha em `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

O método `lines` retorna um iterador. Falaremos sobre iteradores em profundidade
no [Capítulo 13][ch13-iterators]<!-- ignore -->. Mas lembre-se de que você viu
essa forma de usar um iterador na [Listagem 3-5][ch3-iter]<!-- ignore -->, onde
usamos um loop `for` com um iterador para executar algum código em cada item
de uma coleção.

#### Pesquisando a Consulta em Cada Linha

Em seguida, verificaremos se a linha atual contém nossa string de consulta.
Felizmente, as strings têm um método útil chamado `contains` que faz isso por
nós! Adicione uma chamada ao método `contains` na função `search`, conforme
mostrado na Listagem 12-18. Note que isso ainda não vai compilar.

<Listing number="12-18" file-name="src/lib.rs" caption="Adicionando funcionalidade para ver se a linha contém a string em `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

No momento, estamos construindo a funcionalidade. Para fazer o código compilar,
precisamos retornar um valor do corpo, conforme indicamos que faríamos na
assinatura da função.

#### Armazenando Linhas Correspondentes

Para terminar esta função, precisamos de uma maneira de armazenar as linhas
correspondentes que queremos retornar. Para isso, podemos criar um vetor mutável
antes do loop `for` e chamar o método `push` para armazenar uma `line` no vetor.
Após o loop `for`, retornamos o vetor, conforme mostrado na Listagem 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Armazenando as linhas correspondentes para que possamos retorná-las">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Agora a função `search` deve retornar apenas as linhas que contêm `query`, e
nosso teste deve passar. Vamos executar o teste:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Nosso teste passou, então sabemos que funciona!

Neste ponto, poderíamos considerar oportunidades para refatorar a implementação
da função de busca, mantendo os testes passando para preservar a mesma
funcionalidade. O código na função de busca não é tão ruim, mas não aproveita
alguns recursos úteis de iteradores. Retornaremos a este exemplo no
[Capítulo 13][ch13-iterators]<!-- ignore -->, onde exploraremos iteradores em
detalhes e veremos como melhorá-lo.

Agora o programa inteiro deve funcionar! Vamos testá-lo, primeiro com uma palavra
que deve retornar exatamente uma linha do poema de Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Legal! Agora vamos tentar uma palavra que corresponderá a várias linhas, como _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

E finalmente, vamos garantir que não obtenhamos nenhuma linha quando pesquisarmos
por uma palavra que não está em nenhum lugar do poema, como _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Excelente! Construímos nossa própria mini versão de uma ferramenta clássica e
aprendemos muito sobre como estruturar aplicações. Também aprendemos um pouco
sobre entrada e saída de arquivos, tempos de vida, testes e análise de linha de
comando.

Para concluir este projeto, demonstraremos brevemente como trabalhar com variáveis
de ambiente e como imprimir para o erro padrão, ambos úteis quando você está
escrevendo programas de linha de comando.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
