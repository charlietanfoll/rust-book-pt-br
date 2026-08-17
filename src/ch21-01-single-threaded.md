## Construindo um Servidor Web de uma Única Thread

Começaremos fazendo um servidor web de uma única thread funcionar. Antes de começarmos, vamos dar uma olhada rápida nos protocolos envolvidos na construção de servidores web. Os detalhes desses protocolos estão além do escopo deste livro, mas uma breve visão geral fornecerá as informações necessárias.

Os dois principais protocolos envolvidos em servidores web são o _Hypertext Transfer Protocol_ (Protocolo de Transferência de Hipertexto - _HTTP_) e o _Transmission Control Protocol_ (Protocolo de Controle de Transmissão - _TCP_). Ambos os protocolos são protocolos de _requisição-resposta_, o que significa que um _cliente_ inicia as requisições e um _servidor_ escuta as requisições e fornece uma resposta ao cliente. O conteúdo dessas requisições e respostas é definido pelos protocolos.

O TCP é o protocolo de nível inferior que descreve os detalhes de como a informação vai de um servidor para outro, mas não especifica qual é essa informação. O HTTP se constrói sobre o TCP, definindo o conteúdo das requisições e respostas. É tecnicamente possível usar o HTTP com outros protocolos, mas na grande maioria dos casos, o HTTP envia seus dados sobre o TCP. Trabalharemos com os bytes brutos das requisições e respostas TCP e HTTP.

### Escutando a Conexão TCP

Nosso servidor web precisa escutar uma conexão TCP, então essa é a primeira parte em que vamos trabalhar. A biblioteca padrão oferece um módulo `std::net` que nos permite fazer isso. Vamos criar um novo projeto da maneira habitual:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Agora insira o código da Listagem 21-1 em _src/main.rs_ para começar. Este código vai escutar no endereço local `127.0.0.1:7878` por fluxos TCP recebidos. Quando receber um fluxo de entrada, ele imprimirá `Connection established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Escutando fluxos recebidos e imprimindo uma mensagem quando recebemos um fluxo">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Usando `TcpListener`, podemos escutar conexões TCP no endereço `127.0.0.1:7878`. No endereço, a seção antes dos dois-pontos é um endereço IP representando o seu computador (isso é o mesmo em todos os computadores e não representa especificamente o computador dos autores), e `7878` é a porta. Escolhemos esta porta por dois motivos: o HTTP normalmente não é aceito nesta porta, então é improvável que nosso servidor entre em conflito com qualquer outro servidor web que você possa ter rodando em sua máquina, e 7878 é _rust_ digitado em um telefone (no teclado numérico).

A função `bind` neste cenário funciona como a função `new` no sentido de que ela retornará uma nova instância de `TcpListener`. A função se chama `bind` porque, em redes, conectar-se a uma porta para escutá-la é conhecido como "fazer o *bind* em uma porta" (vincular-se a uma porta).

A função `bind` retorna um `Result<T, E>`, o que indica que é possível que a vinculação falhe — por exemplo, se executássemos duas instâncias do nosso programa e, portanto, tivéssemos dois programas escutando a mesma porta. Como estamos escrevendo um servidor básico apenas para fins de aprendizado, não vamos nos preocupar em lidar com esses tipos de erros; em vez disso, usamos `unwrap` para parar o programa se ocorrerem erros.

O método `incoming` em `TcpListener` retorna um iterador que nos fornece uma sequência de fluxos (mais especificamente, fluxos do tipo `TcpStream`). Um único _fluxo_ (_stream_) representa uma conexão aberta entre o cliente e o servidor. _Conexão_ é o nome para o processo completo de requisição e resposta em que um cliente se conecta ao servidor, o servidor gera uma resposta e o servidor fecha a conexão. Sendo assim, leremos do `TcpStream` para ver o que o cliente enviou e, em seguida, escreveremos nossa resposta no fluxo para enviar dados de volta ao cliente. Em suma, este loop `for` processará cada conexão por vez e produzirá uma série de fluxos para lidarmos.

Por enquanto, nosso tratamento do fluxo consiste em chamar `unwrap` para encerrar nosso programa se o fluxo tiver algum erro; se não houver erros, o programa imprime uma mensagem. Adicionaremos mais funcionalidades para o caso de sucesso na próxima listagem. O motivo pelo qual podemos receber erros do método `incoming` quando um cliente se conecta ao servidor é que não estamos realmente iterando sobre conexões. Em vez disso, estamos iterando sobre _tentativas de conexão_. A conexão pode não ser bem-sucedida por vários motivos, muitos deles específicos do sistema operacional. Por exemplo, muitos sistemas operacionais têm um limite para o número de conexões abertas simultâneas que podem suportar; novas tentativas de conexão além desse número produzirão um erro até que algumas das conexões abertas sejam fechadas.

Vamos tentar executar este código! Invoque `cargo run` no terminal e carregue _127.0.0.1:7878_ em um navegador web. O navegador deve exibir uma mensagem de erro como "Conexão redefinida" (Connection reset) porque o servidor atualmente não está enviando nenhum dado de volta. Mas quando você olhar para o seu terminal, deverá ver várias mensagens impressas quando o navegador se conectou ao servidor!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Às vezes, você verá várias mensagens impressas para uma única requisição do navegador; o motivo pode ser que o navegador está fazendo uma requisição para a página, bem como uma requisição para outros recursos, como o ícone _favicon.ico_ que aparece na aba do navegador.

Também pode ser que o navegador esteja tentando se conectar ao servidor várias vezes porque o servidor não está respondendo com nenhum dado. Quando `stream` sai de escopo e é descartado no final do loop, a conexão é fechada como parte da implementação de `drop`. Os navegadores às vezes lidam com conexões fechadas tentando novamente, porque o problema pode ser temporário.

Os navegadores também às vezes abrem várias conexões com o servidor sem enviar nenhuma requisição para que, se *eles vierem* a enviar requisições posteriormente, essas requisições possam acontecer mais rapidamente. Quando isso ocorre, nosso servidor verá cada conexão, independentemente de haver alguma requisição sobre essa conexão. Muitas versões de navegadores baseados no Chrome fazem isso, por exemplo; você pode desativar essa otimização usando o modo de navegação privada ou usando um navegador diferente.

O fator importante é que conseguimos com sucesso obter o controle de uma conexão TCP!

Lembre-se de parar o programa pressionando <kbd>ctrl</kbd>-<kbd>C</kbd> quando terminar de executar uma versão específica do código. Em seguida, reinicie o programa invocando o comando `cargo run` após fazer cada conjunto de alterações no código para garantir que você está executando a versão mais recente.

### Lendo a Requisição

Vamos implementar a funcionalidade para ler a requisição do navegador! Para separar as preocupações de primeiro obter uma conexão e depois tomar alguma ação com a conexão, iniciaremos uma nova função para processar conexões. Nesta nova função `handle_connection`, leremos os dados do fluxo TCP e os imprimiremos para que possamos ver os dados sendo enviados do navegador. Altere o código para que fique como na Listagem 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Lendo do `TcpStream` e imprimindo os dados">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Trazemos `std::io::BufReader` e `std::io::prelude` para o escopo para obter acesso aos traits e tipos que nos permitem ler e escrever no fluxo. No loop `for` na função `main`, em vez de imprimir uma mensagem dizendo que fizemos uma conexão, agora chamamos a nova função `handle_connection` e passamos o `stream` para ela.

Na função `handle_connection`, criamos uma nova instância de `BufReader` que encapsula uma referência ao `stream`. O `BufReader` adiciona buffering gerenciando chamadas aos métodos do trait `std::io::Read` para nós.

Criamos uma variável chamada `http_request` para coletar as linhas da requisição que o navegador envia para o nosso servidor. Indicamos que queremos coletar essas linhas em um vetor adicionando a anotação de tipo `Vec<_>`.

O `BufReader` implementa o trait `std::io::BufRead`, que fornece o método `lines`. O método `lines` retorna um iterador de `Result<String, std::io::Error>` dividindo o fluxo de dados sempre que ele encontra um byte de nova linha. Para obter cada `String`, usamos `map` e `unwrap` em cada `Result`. O `Result` pode ser um erro se os dados não forem UTF-8 válido ou se houver um problema ao ler do fluxo. Novamente, um programa de produção deve lidar com esses erros de forma mais graciosa, mas optamos por parar o programa no caso de erro por simplicidade.

O navegador sinaliza o fim de uma requisição HTTP enviando dois caracteres de nova linha seguidos, então, para obter uma requisição do fluxo, pegamos linhas até obter uma linha que seja a string vazia. Uma vez que coletamos as linhas no vetor, as imprimimos usando formatação de depuração bonita (*pretty debug formatting*) para que possamos dar uma olhada nas instruções que o navegador web está enviando para o nosso servidor.

Vamos testar este código! Inicie o programa e faça uma requisição em um navegador web novamente. Observe que ainda obteremos uma página de erro no navegador, mas a saída do nosso programa no terminal agora será semelhante a isto:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Dependendo do seu navegador, você pode obter uma saída ligeiramente diferente. Agora que estamos imprimindo os dados da requisição, podemos ver por que obtemos múltiplas conexões de uma requisição de navegador olhando para o caminho após `GET` na primeira linha da requisição. Se as conexões repetidas estão todas solicitando _/_, sabemos que o navegador está tentando buscar _/_ repetidamente porque não está obtendo uma resposta do nosso programa.

Vamos decompor esses dados da requisição para entender o que o navegador está pedindo ao nosso programa.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### Olhando Mais de Certo para uma Requisição HTTP

O HTTP é um protocolo baseado em texto, e uma requisição tem este formato:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

A primeira linha é a _linha de requisição_ (_request line_) que contém informações sobre o que o cliente está solicitando. A primeira parte da linha de requisição indica o método que está sendo usado, como `GET` ou `POST`, que descreve como o cliente está fazendo esta requisição. Nosso cliente usou uma requisição `GET`, o que significa que está pedindo informações.

A próxima parte da linha de requisição é _/_, que indica o _identificador uniforme de recursos_ (_Uniform Resource Identifier_ - _URI_) que o cliente está solicitando: Uma URI é quase, mas não totalmente, a mesma coisa que um _localizador uniforme de recursos_ (_Uniform Resource Locator_ - _URL_). A diferença entre URIs e URLs não é importante para nossos propósitos neste capítulo, mas a especificação HTTP usa o termo _URI_, então podemos simplesmente substituir mentalmente _URL_ por _URI_ aqui.

A última parte é a versão HTTP que o cliente usa, e então a linha de requisição termina com uma sequência CRLF. (_CRLF_ significa _carriage return_ [retorno de carro] e _line feed_ [avanço de linha], que são termos dos tempos das máquinas de escrever!) A sequência CRLF também pode ser escrita como `\r\n`, onde `\r` é o retorno de carro e `\n` é o avanço de linha. A _sequência CRLF_ separa a linha de requisição do resto dos dados da requisição. Observe que, quando o CRLF é impresso, vemos o início de uma nova linha em vez de `\r\n`.

Olhando para os dados da linha de requisição que recebemos ao executar nosso programa até agora, vemos que `GET` é o método, _/_ é a URI da requisição e `HTTP/1.1` é a versão.

Após a linha de requisição, as linhas restantes começando de `Host:` em diante são cabeçalhos (_headers_). As requisições `GET` não têm corpo (_body_).

Tente fazer uma requisição de um navegador diferente ou solicitar um endereço diferente, como _127.0.0.1:7878/test_, para ver como os dados da requisição mudam.

Agora que sabemos o que o navegador está pedindo, vamos enviar alguns dados de volta!

### Escrevendo uma Resposta

Vamos implementar o envio de dados em resposta à requisição de um cliente. As respostas têm o seguinte formato:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

A primeira linha é uma _linha de status_ (_status line_) que contém a versão HTTP usada na resposta, um código de status numérico que resume o resultado da requisição e uma frase de motivo (_reason phrase_) que fornece uma descrição textual do código de status. Após a sequência CRLF, vêm quaisquer cabeçalhos, outra sequência CRLF e o corpo da resposta.

Aqui está um exemplo de resposta que usa a versão HTTP 1.1 e tem um código de status 200, uma frase de motivo OK, nenhum cabeçalho e nenhum corpo:

```text
HTTP/1.1 200 OK\r\n\r\n
```

O código de status 200 é a resposta de sucesso padrão. O texto é uma resposta HTTP bem-sucedida minúscula. Vamos escrever isso no fluxo como nossa resposta a uma requisição bem-sucedida! Na função `handle_connection`, remova o `println!` que estava imprimindo os dados da requisição e substitua-o pelo código da Listagem 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Escrevendo uma resposta HTTP bem-sucedida minúscula para o fluxo">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

A primeira nova linha define a variável `response` que contém os dados da mensagem de sucesso. Em seguida, chamamos `as_bytes` em nossa `response` para converter os dados da string em bytes. O método `write_all` em `stream` aceita um `&[u8]` e envia esses bytes diretamente pela conexão. Como a operação `write_all` pode falhar, usamos `unwrap` em qualquer resultado de erro como antes. Novamente, em uma aplicação real, você adicionaria tratamento de erros aqui.

Com essas alterações, vamos executar nosso código e fazer uma requisição. Não estamos mais imprimindo nenhum dado no terminal, então não veremos nenhuma saída além da saída do Cargo. Quando você carregar _127.0.0.1:7878_ em um navegador web, você deve obter uma página em branco em vez de um erro. Você acabou de codificar manualmente o recebimento de uma requisição HTTP e o envio de uma resposta!

### Retornando HTML Real

Vamos implementar a funcionalidade para retornar mais do que uma página em branco. Crie o novo arquivo _hello.html_ na raiz do diretório do seu projeto, não no diretório _src_. Você pode inserir qualquer HTML que quiser; a Listagem 21-4 mostra uma possibilidade.

<Listing number="21-4" file-name="hello.html" caption="Um arquivo HTML de exemplo para retornar em uma resposta">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Este é um documento HTML5 mínimo com um cabeçalho e algum texto. Para retornar isso do servidor quando uma requisição for recebida, modificaremos `handle_connection` conforme mostrado na Listagem 21-5 para ler o arquivo HTML, adicioná-lo à resposta como um corpo e enviá-lo.

<Listing number="21-5" file-name="src/main.rs" caption="Enviando o conteúdo de *hello.html* como o corpo da resposta">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Adicionamos `fs` à instrução `use` para trazer o módulo de sistema de arquivos da biblioteca padrão para o escopo. O código para ler o conteúdo de um arquivo para uma string deve parecer familiar; nós o usamos quando lemos o conteúdo de um arquivo para o nosso projeto de I/O na Listagem 12-4.

Em seguida, usamos `format!` para adicionar o conteúdo do arquivo como o corpo da resposta de sucesso. Para garantir uma resposta HTTP válida, adicionamos o cabeçalho `Content-Length`, que é definido como o tamanho do corpo da nossa resposta — neste caso, o tamanho de `hello.html`.

Execute este código com `cargo run` e carregue _127.0.0.1:7878_ no seu navegador; você deverá ver seu HTML renderizado!

Atualmente, estamos ignorando os dados da requisição em `http_request` e apenas enviando de volta o conteúdo do arquivo HTML incondicionalmente. Isso significa que, se você tentar solicitar _127.0.0.1:7878/something-else_ no seu navegador, ainda receberá esta mesma resposta HTML. No momento, nosso servidor é muito limitado e não faz o que a maioria dos servidores web faz. Queremos personalizar nossas respostas dependendo da requisição e apenas enviar de volta o arquivo HTML para uma requisição bem formatada para _/_.

### Validando a Requisição e Respondendo Seletivamente

Agora, nosso servidor web retornará o HTML no arquivo, não importa o que o cliente tenha solicitado. Vamos adicionar funcionalidade para verificar se o navegador está solicitando _/_ antes de retornar o arquivo HTML e retornar um erro se o navegador solicitar qualquer outra coisa. Para isso, precisamos modificar `handle_connection`, conforme mostrado na Listagem 21-6. Este novo código verifica o conteúdo da requisição recebida em relação ao que sabemos que se parece uma requisição para _/_ e adiciona blocos `if` e `else` para tratar as requisições de forma diferente.

<Listing number="21-6" file-name="src/main.rs" caption="Tratando requisições para */* de forma diferente de outras requisições">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Vamos olhar apenas para a primeira linha da requisição HTTP, então, em vez de ler a requisição inteira para um vetor, estamos chamando `next` para obter o primeiro item do iterador. O primeiro `unwrap` lida com o `Option` e para o programa se o iterador não tiver itens. O segundo `unwrap` lida com o `Result` e tem o mesmo efeito que o `unwrap` que estava no `map` adicionado na Listagem 21-2.

Em seguida, verificamos o `request_line` para ver se ele é igual à linha de requisição de uma requisição GET para o caminho _/_. Se for, o bloco `if` retorna o conteúdo do nosso arquivo HTML.

Se o `request_line` _não_ for igual à requisição GET para o caminho _/_, significa que recebemos alguma outra requisição. Adicionaremos código ao bloco `else` em um momento para responder a todas as outras requisições.

Execute este código agora e solicite _127.0.0.1:7878_; você deve obter o HTML em _hello.html_. Se você fizer qualquer outra requisição, como _127.0.0.1:7878/something-else_, você receberá um erro de conexão semelhante àqueles que você viu ao executar o código na Listagem 21-1 e Listagem 21-2.

Agora vamos adicionar o código da Listagem 21-7 ao bloco `else` para retornar uma resposta com o código de status 404, o que sinaliza que o conteúdo da requisição não foi encontrado. Também retornaremos algum HTML para uma página a ser renderizada no navegador indicando a resposta ao usuário final.

<Listing number="21-7" file-name="src/main.rs" caption="Respondendo com o código de status 404 e uma página de erro se algo diferente de */* foi solicitado">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Aqui, nossa resposta tem uma linha de status com o código de status 404 e a frase de motivo `NOT FOUND`. O corpo da resposta será o HTML no arquivo _404.html_. Você precisará criar um arquivo _404.html_ ao lado de _hello.html_ para a página de erro; novamente, fique à vontade para usar qualquer HTML que quiser, ou use o exemplo de HTML na Listagem 21-8.

<Listing number="21-8" file-name="404.html" caption="Conteúdo de exemplo para a página a ser enviada de volta com qualquer resposta 404">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Com essas alterações, execute seu servidor novamente. Solicitar _127.0.0.1:7878_ deve retornar o conteúdo de _hello.html_, e qualquer outra requisição, como _127.0.0.1:7878/foo_, deve retornar o HTML de erro de _404.html_.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### Refatoração

No momento, os blocos `if` e `else` têm muita repetição: ambos estão lendo arquivos e escrevendo o conteúdo dos arquivos no fluxo. As únicas diferenças são a linha de status e o nome do arquivo. Vamos tornar o código mais conciso extraindo essas diferenças em linhas `if` e `else` separadas que atribuirão os valores da linha de status e do nome do arquivo a variáveis; podemos então usar essas variáveis incondicionalmente no código para ler o arquivo e escrever a resposta. A Listagem 21-9 mostra o código resultante após substituir os grandes blocos `if` e `else`.

<Listing number="21-9" file-name="src/main.rs" caption="Refatorando os blocos `if` e `else` para conter apenas o código que difere entre os dois casos">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Agora os blocos `if` e `else` retornam apenas os valores apropriados para a linha de status e o nome do arquivo em uma tupla; nós então usamos desestruturação para atribuir esses dois valores a `status_line` e `filename` usando um padrão na instrução `let`, conforme discutido no Capítulo 19.

O código anteriormente duplicado agora está fora dos blocos `if` e `else` e usa as variáveis `status_line` e `filename`. Isso facilita ver a diferença entre os dois casos e significa que temos apenas um lugar para atualizar o código se quisermos alterar como a leitura do arquivo e a escrita da resposta funcionam. O comportamento do código na Listagem 21-9 será o mesmo que o da Listagem 21-7.

Incrível! Agora temos um servidor web simples em aproximadamente 40 linhas de código Rust que responde a uma requisição com uma página de conteúdo e responde a todas as outras requisições com uma resposta 404.

Atualmente, nosso servidor roda em uma única thread, o que significa que ele só pode atender a uma requisição por vez. Vamos examinar como isso pode ser um problema simulando algumas requisições lentas. Em seguida, vamos consertá-lo para que nosso servidor possa lidar com várias requisições de uma só vez.