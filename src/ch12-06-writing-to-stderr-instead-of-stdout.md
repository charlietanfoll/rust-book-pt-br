<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Redirecionando Erros para a Saída de Erro Padrão

No momento, estamos escrevendo toda a nossa saída para o terminal usando a macro
`println!`. Na maioria dos terminais, existem dois tipos de saída: _saída
padrão_ (`stdout`) para informações gerais e _erro padrão_ (`stderr`) para
mensagens de erro. Essa distinção permite que os usuários escolham direcionar a
saída bem-sucedida de um programa para um arquivo, mas ainda assim imprimir as
mensagens de erro na tela.

A macro `println!` é capaz de imprimir apenas na saída padrão, então precisamos
usar outra coisa para imprimir no erro padrão.

### Verificando Onde os Erros São Escritos

Primeiro, vamos observar como o conteúdo impresso pelo `minigrep` está sendo
escrito atualmente na saída padrão, incluindo quaisquer mensagens de erro que
queremos escrever no erro padrão. Faremos isso redirecionando o fluxo de saída
padrão para um arquivo enquanto causamos um erro intencionalmente. Não
redirecionaremos o fluxo de erro padrão, então qualquer conteúdo enviado para o
erro padrão continuará sendo exibido na tela.

Espera-se que programas de linha de comando enviem mensagens de erro para o
fluxo de erro padrão para que ainda possamos ver as mensagens de erro na tela,
mesmo se redirecionarmos o fluxo de saída padrão para um arquivo. Nosso
programa atualmente não está se comportando bem: estamos prestes a ver que ele
salva a saída da mensagem de erro em um arquivo!

Para demonstrar esse comportamento, executaremos o programa com `>` e o caminho
do arquivo, _output.txt_, para o qual queremos redirecionar o fluxo de saída
padrão. Não passaremos nenhum argumento, o que deve causar um erro:

```console
$ cargo run > output.txt
```

A sintaxe `>` diz ao shell para escrever o conteúdo da saída padrão em
_output.txt_ em vez da tela. Não vimos a mensagem de erro que esperávamos ser
impressa na tela, o que significa que ela deve ter ido parar no arquivo. É isso
que _output.txt_ contém:

```text
Problem parsing arguments: not enough arguments
```

Pois é, nossa mensagem de erro está sendo impressa na saída padrão. É muito mais
útil que mensagens de erro como esta sejam impressas no erro padrão para que
apenas os dados de uma execução bem-sucedida terminem no arquivo. Vamos mudar
isso.

### Imprimindo Erros no Erro Padrão

Usaremos o código na Listagem 12-24 para mudar a forma como as mensagens de erro
são impressas. Por causa da refatoração que fizemos antes neste capítulo, todo
o código que imprime mensagens de erro está em uma única função, `main`. A
biblioteca padrão fornece a macro `eprintln!` que imprime no fluxo de erro
padrão, então vamos alterar os dois lugares onde estávamos chamando `println!`
para imprimir erros e usar `eprintln!` em vez disso.

<Listing number="12-24" file-name="src/main.rs" caption="Escrevendo mensagens de erro no erro padrão em vez da saída padrão usando `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Agora vamos executar o programa novamente da mesma forma, sem nenhum argumento e
redirecionando a saída padrão com `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Agora vemos o erro na tela e _output.txt_ não contém nada, que é o
comportamento que esperamos de programas de linha de comando.

Vamos executar o programa novamente com argumentos que não causam um erro, mas
ainda redirecionam a saída padrão para um arquivo, assim:

```console
$ cargo run -- to poem.txt > output.txt
```

Não veremos nenhuma saída no terminal, e _output.txt_ conterá nossos
resultados:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Isso demonstra que agora estamos usando a saída padrão para saídas
bem-sucedidas e o erro padrão para saídas de erro, conforme apropriado.

## Resumo

Este capítulo recapitulou alguns dos principais conceitos que você aprendeu até
agora e cobriu como realizar operações de E/S comuns em Rust. Ao usar
argumentos de linha de comando, arquivos, variáveis de ambiente e a macro
`eprintln!` para imprimir erros, você agora está preparado para escrever
aplicações de linha de comando. Combinado com os conceitos dos capítulos
anteriores, seu código estará bem organizado, armazenará dados de forma eficaz
nas estruturas de dados apropriadas, lidará com erros adequadamente e será bem
testado.

Em seguida, exploraremos alguns recursos do Rust que foram influenciados por
linguagens funcionais: _closures_ e iteradores.
