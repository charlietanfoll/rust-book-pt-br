## Refatorando para Melhorar a Modularidade e o Tratamento de Erros

Para melhorar nosso programa, vamos corrigir quatro problemas que têm a ver com a
estrutura do programa e como ele lida com potenciais erros. Primeiro, nossa função
`main` agora executa duas tarefas: ela analisa os argumentos e lê arquivos. Conforme
nosso programa cresce, o número de tarefas separadas que a função `main` gerencia
aumentará. À medida que uma função ganha responsabilidades, torna-se mais difícil
de raciocinar sobre ela, mais difícil de testar e mais difícil de alterar sem quebrar
uma de suas partes. É melhor separar a funcionalidade para que cada função seja
responsável por uma única tarefa.

Esse problema também se conecta ao segundo problema: embora `query` e `file_path`
sejam variáveis de configuração para o nosso programa, variáveis como `contents`
são usadas para executar a lógica do programa. Quanto mais longo o `main` se torna,
mais variáveis precisamos trazer para o escopo; quanto mais variáveis temos no
escopo, mais difícil será acompanhar o propósito de cada uma. É melhor agrupar as
variáveis de configuração em uma única estrutura para tornar seu propósito claro.

O terceiro problema é que usamos `expect` para imprimir uma mensagem de erro quando
a leitura do arquivo falha, mas a mensagem de erro apenas imprime `Should have been
able to read the file` (Deveria ter conseguido ler o arquivo). A leitura de um
arquivo pode falhar de várias maneiras: por exemplo, o arquivo pode estar faltando
ou podemos não ter permissão para abri-lo. No momento, independentemente da
situação, imprimiríamos a mesma mensagem de erro para tudo, o que não daria ao
usuário nenhuma informação!

Quarto, usamos `expect` para lidar com um erro, e se o usuário executar nosso programa
sem especificar argumentos suficientes, ele receberá um erro `index out of bounds`
(índice fora dos limites) do Rust que não explica claramente o problema. Seria
melhor se todo o código de tratamento de erros estivesse em um só lugar, para que
futuros mantenedores tivessem apenas um lugar para consultar o código caso a lógica
de tratamento de erros precisasse ser alterada. Ter todo o código de tratamento de
erros em um só lugar também garantirá que estamos imprimindo mensagens que façam
sentido para nossos usuários finais.

Vamos resolver esses quatro problemas refatorando nosso projeto.

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### Separando Preocupações em Projetos Binários

O problema organizacional de alocar a responsabilidade por várias tarefas à
função `main` é comum a muitos projetos binários. Como resultado, muitos
programadores Rust acham útil separar as diferentes preocupações de um programa
binário quando a função `main` começa a ficar grande. Esse processo tem os
seguintes passos:

- Divida seu programa em um arquivo _main.rs_ e um arquivo _lib.rs_ e mova a
  lógica do seu programa para _lib.rs_.
- Enquanto sua lógica de análise de linha de comando for pequena, ela pode
  permanecer na função `main`.
- Quando a lógica de análise de linha de comando começar a ficar complicada,
  extraia-a da função `main` para outras funções ou tipos.

As responsabilidades que permanecem na função `main` após esse processo devem
ser limitadas ao seguinte:

- Chamar a lógica de análise de linha de comando com os valores dos argumentos
- Configurar qualquer outra configuração
- Chamar uma função `run` em _lib.rs_
- Lidar com o erro caso `run` retorne um erro

Esse padrão trata de separar preocupações: _main.rs_ lida com a execução do
programa e _lib.rs_ lida com toda a lógica da tarefa em questão. Como você
não pode testar a função `main` diretamente, essa estrutura permite testar toda
a lógica do seu programa movendo-a para fora da função `main`. O código que
permanece na função `main` será pequeno o suficiente para verificar sua correção
lendo-o. Vamos reescrever nosso programa seguindo esse processo.

#### Extraindo o Analisador de Argumentos

Vamos extrair a funcionalidade de análise de argumentos para uma função que
`main` chamará. A Listagem 12-5 mostra o novo início da função `main` que
chama uma nova função `parse_config`, que definiremos em _src/main.rs_.

<Listing number="12-5" file-name="src/main.rs" caption="Extraindo uma função `parse_config` de `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Ainda estamos coletando os argumentos da linha de comando em um vetor, mas em vez
de atribuir o valor do argumento no índice 1 à variável `query` e o valor do
argumento no índice 2 à variável `file_path` dentro da função `main`, passamos
o vetor inteiro para a função `parse_config`. A função `parse_config` então
contém a lógica que determina qual argumento vai em qual variável e passa os
valores de volta para `main`. Ainda criamos as variáveis `query` e `file_path`
em `main`, mas `main` não tem mais a responsabilidade de determinar como os
argumentos da linha de comando e as variáveis correspondem.

Essa reformulação pode parecer exagero para o nosso pequeno programa, mas
estamos refatorando em etapas pequenas e incrementais. Após fazer essa alteração,
execute o programa novamente para verificar se a análise de argumentos ainda
funciona. É bom verificar seu progresso com frequência para ajudar a identificar
a causa dos problemas quando eles ocorrem.

#### Agrupando Valores de Configuração

Podemos dar mais um pequeno passo para melhorar ainda função `parse_config`.
No momento, estamos retornando uma tupla, mas logo em seguida quebramos essa
tupla em partes individuais novamente. Isso é um sinal de que talvez ainda
não tenhamos a abstração correta.

Outro indicador de que há margem para melhoria é a parte `config` de
`parse_config`, o que implica que os dois valores que retornamos estão
relacionados e fazem parte de um único valor de configuração. Atualmente, não
estamos transmitindo esse significado na estrutura dos dados, a não ser agrupando
os dois valores em uma tupla; em vez disso, colocaremos os dois valores em uma
única struct e daremos a cada campo da struct um nome significativo. Fazer isso
tornará mais fácil para os futuros mantenedores deste código entenderem como
os diferentes valores se relacionam entre si e qual é o propósito deles.

A Listagem 12-6 mostra as melhorias na função `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Refatorando `parse_config` para retornar uma instância de uma struct `Config`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Adicionamos uma struct chamada `Config` definida com os campos chamados `query`
e `file_path`. A assinatura de `parse_config` agora indica que ela retorna um
valor `Config`. No corpo de `parse_config`, onde costumávamos retornar fatias
de string (_string slices_) que fazem referência a valores `String` em `args`,
agora definimos `Config` para conter valores `String` próprios (ou seja,
possuídos/owned). A variável `args` em `main` é a dona dos valores dos argumentos
e está apenas permitindo que a função `parse_config` os empreste, o que significa
que violaríamos as regras de empréstimo do Rust se `Config` tentasse assumir a
posse dos valores em `args`.

Há várias maneiras de gerenciar os dados de `String`; o caminho mais fácil,
embora um pouco ineficiente, é chamar o método `clone` nos valores. Isso fará
uma cópia completa dos dados para a instância `Config` possuir, o que consome
mais tempo e memória do que armazenar uma referência aos dados da string. No
entanto, clonar os dados também torna nosso código muito direto, porque não
precisamos gerenciar os tempos de vida (_lifetimes_) das referências; nessa
circunstância, abrir mão de um pouco de desempenho para ganhar simplicidade é
uma troca que vale a pena.

> ### As Compensações do Uso de `clone`
>
> Há uma tendência entre muitos entusiastas do Rust de evitar o uso de `clone`
> para resolver problemas de propriedade por causa de seu custo em tempo de
> execução. No [Capítulo 13][ch13]<!-- ignore -->, você aprenderá a usar
> métodos mais eficientes nesse tipo de situação. Mas, por enquanto, tudo bem
> copiar algumas strings para continuar progredindo, porque você fará essas
> cópias apenas uma vez e seu caminho de arquivo e string de busca são muito
> pequenos. É melhor ter um programa funcionando que seja um pouco ineficiente
> do que tentar hiperotimizar o código na primeira passada. À medida que você
> ganhar mais experiência com Rust, será mais fácil começar com a solução mais
> eficiente, mas por enquanto, é perfeitamente aceitável chamar `clone`.

Atualizamos `main` para que ela coloque a instância de `Config` retornada por
`parse_config` em uma variável chamada `config`, e atualizamos o código que
anteriormente usava as variáveis separadas `query` e `file_path` para que agora
ele use os campos da struct `Config`.

Agora nosso código transmite mais claramente que `query` e `file_path` estão
relacionados e que seu propósito é configurar como o programa funcionará.
Qualquer código que use esses valores sabe que deve encontrá-los na instância
`config` nos campos nomeados para o seu propósito.

#### Criando um Construtor para `Config`

Até agora, extraímos a lógica responsável por analisar os argumentos da linha
de comando de `main` e a colocamos na função `parse_config`. Fazer isso nos
ajudou a ver que os valores `query` e `file_path` estavam relacionados, e essa
relação deve ser transmitida em nosso código. Em seguida, adicionamos uma struct
`Config` para nomear o propósito relacionado de `query` e `file_path` e para
poder retornar os nomes dos valores como nomes de campos de struct a partir da
função `parse_config`.

Portanto, agora que o propósito da função `parse_config` é criar uma instância
`Config`, podemos mudar `parse_config` de uma função simples para uma função
chamada `new` que está associada à struct `Config`. Fazer essa alteração tornará
o código mais idiomático. Podemos criar instâncias de tipos na biblioteca padrão,
como `String`, chamando `String::new`. Da mesma forma, ao transformar
`parse_config` em uma função `new` associada a `Config`, poderemos criar
instâncias de `Config` chamando `Config::new`. A Listagem 12-7 mostra as
alterações que precisamos fazer.

<Listing number="12-7" file-name="src/main.rs" caption="Mudando `parse_config` para `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Atualizamos `main` onde estávamos chamando `parse_config` para chamar `Config::new`
em vez disso. Mudamos o nome de `parse_config` para `new` e o movemos para dentro
de um bloco `impl`, que associa a função `new` a `Config`. Tente compilar este
código novamente para ter certeza de que funciona.

### Corrigindo o Tratamento de Erros

Agora vamos trabalhar na correção do nosso tratamento de erros. Lembre-se de que
tentar acessar os valores no vetor `args` no índice 1 ou índice 2 fará com que o
programa entre em pânico (_panic_) se o vetor contiver menos de três itens. Tente
executar o programa sem nenhum argumento; ele se parecerá com isto:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

A linha `index out of bounds: the len is 1 but the index is 1` (índice fora dos
limites: o tamanho é 1 mas o índice é 1) é uma mensagem de erro destinada a
programadores. Ela não ajudará nossos usuários finais a entenderem o que devem
fazer em vez disso. Vamos corrigir isso agora.

#### Melhorando a Mensagem de Erro

Na Listagem 12-8, adicionamos uma verificação na função `new` que verificará se
a fatia é longa o suficiente antes de acessar o índice 1 e o índice 2. Se a
fatia não for longa o suficiente, o programa entra em pânico e exibe uma mensagem
de erro melhor.

<Listing number="12-8" file-name="src/main.rs" caption="Adicionando uma verificação para o número de argumentos">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Este código é semelhante [à função `Guess::new` que escrevemos na Listagem
9-13][ch9-custom-types]<!-- ignore -->, onde chamamos `panic!` quando o argumento
`value` estava fora do intervalo de valores válidos. Em vez de verificar um
intervalo de valores aqui, estamos verificando se o comprimento de `args` é de
pelo menos `3` e o restante da função pode operar sob a premissa de que essa
condição foi atendida. Se `args` tiver menos de três itens, essa condição será
`true` e chamamos a macro `panic!` para encerrar o programa imediatamente.

Com essas poucas linhas extras de código em `new`, vamos executar o programa sem
nenhum argumento novamente para ver como o erro se parece agora:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Essa saída está melhor: agora temos uma mensagem de erro razoável. No entanto,
também temos informações estranhas que não queremos dar aos nossos usuários.
Talvez a técnica que usamos na Listagem 9-13 não seja a melhor para usar aqui:
Uma chamada para `panic!` é mais apropriada para um problema de programação do que
para um problema de uso, [conforme discutido no Capítulo 9][ch9-error-guidelines]<!-- ignore -->.
Em vez disso, usaremos a outra técnica sobre a qual você aprendeu no Capítulo 9—[retornar
um `Result`][ch9-result]<!-- ignore --> que indica sucesso ou erro.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Retornando um `Result` em Vez de Chamar `panic!`

Podemos retornar um valor `Result` que conterá uma instância `Config` no caso de
sucesso e descreverá o problema no caso de erro. Também vamos mudar o nome da
função de `new` para `build`, porque muitos programadores esperam que funções `new`
nunca falhem. Quando `Config::build` estiver se comunicando com `main`, podemos
usar o tipo `Result` para sinalizar que houve um problema. Então, podemos alterar
`main` para converter uma variante `Err` em um erro mais prático para nossos
usuários, sem o texto ao redor sobre `thread 'main'` e `RUST_BACKTRACE` que uma
chamada para `panic!` causa.

A Listagem 12-9 mostra as alterações que precisamos fazer no valor de retorno da
função que agora estamos chamando de `Config::build` e no corpo da função necessário
para retornar um `Result`. Observe que isso não compilará até atualizarmos o `main`
também, o que faremos na próxima listagem.

<Listing number="12-9" file-name="src/main.rs" caption="Retornando um `Result` de `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Nossa função `build` retorna um `Result` com uma instância `Config` no caso de
sucesso e um literal de string no caso de erro. Nossos valores de erro serão
sempre literais de string que possuem o tempo de vida `'static`.

Fizemos duas alterações no corpo da função: em vez de chamar `panic!` quando o
usuário não passa argumentos suficientes, agora retornamos um valor `Err`, e
envolvemos o valor de retorno `Config` em um `Ok`. Essas alterações fazem com
que a função esteja em conformidade com sua nova assinatura de tipo.

Retornar um valor `Err` de `Config::build` permite que a função `main` lide com
o valor `Result` retornado pela função `build` e saia do processo de forma mais
limpa no caso de erro.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Chamando `Config::build` e Lidando com Erros

Para lidar com o caso de erro e imprimir uma mensagem amigável ao usuário,
precisamos atualizar `main` para lidar com o `Result` sendo retornado por
`Config::build`, conforme mostrado na Listagem 12-10. Também assumiremos a
responsabilidade de sair da ferramenta de linha de comando com um código de erro
diferente de zero, retirando-a do `panic!` e implementando-a manualmente. Um status
de saída diferente de zero é uma convenção para sinalizar ao processo que chamou
nosso programa que o programa foi encerrado com um estado de erro.

<Listing number="12-10" file-name="src/main.rs" caption="Saindo com um código de erro se a construção de um `Config` falhar">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Nesta listagem, usamos um método que ainda não abordamos em detalhes:
`unwrap_or_else`, que é definido em `Result<T, E>` pela biblioteca padrão. O uso
de `unwrap_or_else` nos permite definir algum tratamento de erro personalizado e
que não seja um `panic!`. Se o `Result` for um valor `Ok`, o comportamento desse
método é semelhante a `unwrap`: ele retorna o valor interno que `Ok` está
envolvendo. No entanto, se o valor for um valor `Err`, este método chama o código
na closure, que é uma função anônima que definimos e passamos como argumento para
`unwrap_or_else`. Abordaremos closures com mais detalhes no [Capítulo 13][ch13]<!-- ignore -->.
Por enquanto, você só precisa saber que `unwrap_or_else` passará o valor interno
do `Err`, que neste caso é a string estática `"not enough arguments"` que
adicionamos na Listagem 12-9, para nossa closure no argumento `err` que aparece
entre as barras verticais. O código na closure pode então usar o valor `err`
quando for executado.

Adicionamos uma nova linha `use` para trazer `process` da biblioteca padrão para o
escopo. O código na closure que será executado no caso de erro tem apenas duas
linhas: imprimimos o valor `err` e chamamos `process::exit`. A função
`process::exit` interromperá o programa imediatamente e retornará o número que
foi passado como o código de status de saída. Isso é semelhante ao tratamento
baseado em `panic!` que usamos na Listagem 12-8, mas não recebemos mais toda a
saída extra. Vamos tentar:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Ótimo! Esta saída é muito mais amigável para nossos usuários.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### Extraindo Lógica de `main`

Agora que terminamos de refatorar a análise de configuração, vamos nos voltar
para a lógica do programa. Como afirmamos em [“Separando Preocupações em Projetos
Binários”](#separation-of-concerns-for-binary-projects)<!-- ignore -->, extrairemos
uma função chamada `run` que conterá toda a lógica atualmente na função `main`
que não está envolvida com a configuração ou com o tratamento de erros. Quando
terminarmos, a função `main` será concisa e fácil de verificar por inspeção, e
poderemos escrever testes para toda a outra lógica.

A Listagem 12-11 mostra a melhoria pequena e incremental de extrair uma função
`run`.

<Listing number="12-11" file-name="src/main.rs" caption="Extraindo uma função `run` contendo o restante da lógica do programa">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

A função `run` agora contém toda a lógica restante de `main`, começando pela
leitura do arquivo. A função `run` recebe a instância `Config` como argumento.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### Retornando Erros da Função `run`

Com a lógica restante do programa separada na função `run`, podemos melhorar o
tratamento de erros, assim como fizemos com `Config::build` na Listagem 12-9. Em
vez de permitir que o programa entre em pânico chamando `expect`, a função `run`
retornará um `Result<T, E>` quando algo der errado. Isso nos permitirá consolidar
ainda mais a lógica em torno do tratamento de erros em `main` de uma maneira
amigável ao usuário. A Listagem 12-12 mostra as alterações que precisamos fazer
na assinatura e no corpo de `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Alterando a função `run` para retornar `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Fizemos três alterações significativas aqui. Primeiro, mudamos o tipo de retorno da
função `run` para `Result<(), Box<dyn Error>>`. Esta função anteriormente
retornava o tipo unitário, `()`, e mantemos isso como o valor retornado no caso
`Ok`.

Para o tipo de erro, usamos o objeto trait `Box<dyn Error>` (e trouxemos
`std::error::Error` para o escopo com uma declaração `use` no topo). Abordaremos
objetos trait no [Capítulo 18][ch18]<!-- ignore -->. Por enquanto, saiba apenas
que `Box<dyn Error>` significa que a função retornará um tipo que implementa o
trait `Error`, mas não precisamos especificar qual tipo específico será o valor
de retorno. Isso nos dá flexibilidade para retornar valores de erro que podem ser
de tipos diferentes em diferentes casos de erro. A palavra-chave `dyn` é uma
abreviação para _dinâmico_ (_dynamic_).

Segundo, removemos a chamada para `expect` em favor do operador `?`, como
conversamos no [Capítulo 9][ch9-question-mark]<!-- ignore -->. Em vez de entrar
em pânico (`panic!`) em caso de erro, o operador `?` retornará o valor de erro
da função atual para que o chamador possa lidar com ele.

Terceiro, a função `run` agora retorna um valor `Ok` no caso de sucesso.
Declaramos o tipo de sucesso da função `run` como `()` na assinatura, o que
significa que precisamos envolver o valor do tipo unitário no valor `Ok`. Esta
sintaxe `Ok(())` pode parecer um pouco estranha no início. Mas usar `()` assim
é a maneira idiomática de indicar que estamos chamando `run` apenas por seus
efeitos colaterais; ela não retorna um valor de que precisamos.

Quando você executa este código, ele compila, mas exibe um aviso:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

O Rust nos diz que nosso código ignorou o valor `Result` e o valor `Result` pode
indicar que ocorreu um erro. Mas não estamos verificando se houve ou não um
erro, e o compilador nos lembra que provavelmente pretendíamos ter algum código
de tratamento de erro aqui! Vamos remediar esse problema agora.

#### Lidando com Erros Retornados por `run` em `main`

Verificaremos se há erros e lidaremos com eles usando uma técnica semelhante à
que usamos com `Config::build` na Listagem 12-10, mas com uma pequena diferença:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Usamos `if let` em vez de `unwrap_or_else` para verificar se `run` retorna um
valor `Err` e para chamar `process::exit(1)` caso retorne. A função `run` não
retorna um valor que queremos desembrulhar (`unwrap`) da mesma maneira que
`Config::build` retorna a instância `Config`. Como `run` retorna `()` no caso de
sucesso, nós nos importamos apenas em detectar um erro, então não precisamos que
`unwrap_or_else` retorne o valor desembrulhado, que seria apenas `()`.

Os corpos das funções `if let` e `unwrap_or_else` são os mesmos em ambos os casos:
imprimimos o erro e saímos.

### Dividindo o Código em uma Crate de Biblioteca

Nosso projeto `minigrep` está com uma ótima aparência até agora! Agora vamos
dividir o arquivo _src/main.rs_ e colocar parte do código no arquivo _src/lib.rs_.
Dessa forma, podemos testar o código e ter um arquivo _src/main.rs_ com menos
responsabilidades.

Vamos definir o código responsável por pesquisar texto em _src/lib.rs_ em vez
de em _src/main.rs_, o que nos permitirá (ou a qualquer outra pessoa usando nossa
biblioteca `minigrep`) chamar a função de pesquisa a partir de mais contextos do
que apenas o nosso binário `minigrep`.

Primeiro, vamos definir a assinatura da função `search` em _src/lib.rs_ conforme
mostrado na Listagem 12-13, com um corpo que chama a macro `unimplemented!`.
Explicaremos a assinatura com mais detalhes quando preenchermos a implementação.

<Listing number="12-13" file-name="src/lib.rs" caption="Definindo a função `search` em *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

Usamos a palavra-chave `pub` na definição da função para designar `search` como
parte da API pública da nossa crate de biblioteca. Agora temos uma crate de
biblioteca que podemos usar a partir de nossa crate binária e que podemos testar!

Agora precisamos trazer o código definido em _src/lib.rs_ para o escopo da crate
binária em _src/main.rs_ e chamá-lo, conforme mostrado na Listagem 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Usando a função `search` da crate de biblioteca `minigrep` em *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Adicionamos uma linha `use minigrep::search` para trazer a função `search` da
crate de biblioteca para o escopo da crate binária. Em seguida, na função `run`,
em vez de imprimir o conteúdo do arquivo, chamamos a função `search` e passamos
o valor `config.query` e `contents` como argumentos. Então, `run` usará um loop
`for` para imprimir cada linha retornada de `search` que correspondeu à busca.
Este também é um bom momento para remover as chamadas `println!` na função
`main` que exibiam a consulta e o caminho do arquivo, para que nosso programa
imprima apenas os resultados da pesquisa (se nenhum erro ocorrer).

Observe que a função de pesquisa coletará todos os resultados em um vetor que ela
retorna antes que qualquer impressão aconteça. Essa implementação pode ser lenta
para exibir resultados ao pesquisar arquivos grandes, porque os resultados não
são impressos à medida que são encontrados; discutiremos uma maneira possível de
corrigir isso usando iteradores no Capítulo 13.

Ufa! Esse foi um trabalho árduo, mas nos preparamos para o sucesso no futuro.
Agora é muito mais fácil lidar com erros e tornamos o código mais modular. Quase
todo o nosso trabalho será feito em _src/lib.rs_ daqui em diante.

Vamos aproveitar essa nova modularidade fazendo algo que teria sido difícil com
o código antigo, mas é fácil com o novo código: vamos escrever alguns testes!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
