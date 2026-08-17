## Trabalhando com Variáveis de Ambiente

Vamos melhorar o binário `minigrep` adicionando um recurso extra: uma opção para busca que não diferencia maiúsculas de minúsculas (case-insensitive) que o usuário pode ativar por meio de uma variável de ambiente. Poderíamos fazer desse recurso uma opção de linha de comando e exigir que os usuários adigitem toda vez que quiserem aplicá-la, mas ao transformá-la em uma variável de ambiente, permitimos que nossos usuários configurem a variável de ambiente uma vez e façam com que todas as suas buscas sejam insensíveis a maiúsculas/minúsculas nessa sessão do terminal.

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Escrevendo um Teste que Falha para a Busca Insensível a Maiúsculas e Minúsculas

Primeiro, adicionamos uma nova função `search_case_insensitive` à biblioteca `minigrep` que será chamada quando a variável de ambiente tiver um valor. Continuaremos seguindo o processo de TDD, então o primeiro passo é novamente escrever um teste que falha. Adicionaremos um novo teste para a nova função `search_case_insensitive` e renomearemos nosso antigo teste de `one_result` para `case_sensitive` para esclarecer as diferenças entre os dois testes, conforme mostrado na Listagem 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Adicionando um novo teste que falha para a função case-insensitive que estamos prestes a adicionar">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Note que também editamos o `contents` do teste antigo. Adicionamos uma nova linha com o texto `"Duct tape."` usando um _D_ maiúsculo que não deve corresponder à consulta `"duct"` quando estamos pesquisando de forma sensível a maiúsculas e minúsculas. Alterar o teste antigo dessa forma ajuda a garantir que não quebremos acidentalmente a funcionalidade de busca sensível a maiúsculas e minúsculas que já implementamos. Este teste deve passar agora e deve continuar passando enquanto trabalhamos na busca insensível a maiúsculas e minúsculas.

O novo teste para a busca _insensível_ a maiúsculas e minúsculas usa `"rUsT"` como sua consulta. Na função `search_case_insensitive` que estamos prestes a adicionar, a consulta `"rUsT"` deve corresponder à linha contendo `"Rust:"` com um _R_ maiúsculo e à linha `"Trust me."` mesmo que ambas tenham maiúsculas/minúsculas diferentes da consulta. Este é o nosso teste que falha, e ele falhará ao compilar porque ainda não definimos a função `search_case_insensitive`. Sinta-se à vontade para adicionar uma implementação esquelética que sempre retorne um vetor vazio, de maneira semelhante ao que fizemos para a função `search` na Listagem 12-16 para ver o teste compilar e falhar.

### Implementando a Função `search_case_insensitive`

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase igual à função `search`. A única diferença é que vamos converter a `query` e cada `line` para letras minúsculas (`lowercase`), para que, independentemente da capitalização dos argumentos de entrada, eles fiquem com as mesmas letras minúsculas quando verificarmos se a linha contém a consulta.

<Listing number="12-21" file-name="src/lib.rs" caption="Definindo a função `search_case_insensitive` para converter a consulta e a linha para minúsculas antes de compará-las">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Primeiro, convertemos a string `query` para minúsculas e a armazenamos em uma nova variável com o mesmo nome, fazendo o _shadowing_ (sombreamento) da `query` original. Chamar `to_lowercase` na consulta é necessário para que, não importa se a consulta do usuário seja `"rust"`, `"RUST"`, `"Rust"` ou `"rUsT"`, tratemos a consulta como se fosse `"rust"` e ignoremos maiúsculas e minúsculas. Embora o `to_lowercase` lide com Unicode básico, ele não será 100% preciso. Se estivéssemos escrevendo um aplicativo real, gostaríamos de fazer um pouco mais de trabalho aqui, mas esta seção é sobre variáveis de ambiente, não Unicode, então vamos nos ater a isso por enquanto.

Note que `query` agora é uma `String` em vez de um _string slice_ (fatia de string), porque chamar `to_lowercase` cria novos dados em vez de referenciar dados existentes. Digamos que a consulta seja `"rUsT"`, por exemplo: esse _string slice_ não contém um `u` ou `t` minúsculo para usarmos, então temos que alocar uma nova `String` contendo `"rust"`. Quando passamos `query` como argumento para o método `contains` agora, precisamos adicionar um e-comercial (`&`) porque a assinatura de `contains` é definida para aceitar um _string slice_.

Em seguida, adicionamos uma chamada a `to_lowercase` em cada `line` para colocar todos os caracteres em minúsculas. Agora que convertemos `line` e `query` para minúsculas, encontraremos correspondências não importa a capitalização da consulta.

Vamos ver se esta implementação passa nos testes:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Ótimo! Eles passaram. Agora vamos chamar a nova função `search_case_insensitive` a partir da função `run`. Primeiro, adicionaremos uma opção de configuração à struct `Config` para alternar entre busca sensível e insensível a maiúsculas e minúsculas. Adicionar este campo causará erros de compilação porque ainda não estamos inicializando este campo em lugar nenhum:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Adicionamos o campo `ignore_case` que armazena um booleano. Em seguida, precisamos que a função `run` verifique o valor do campo `ignore_case` e o utilize para decidir se chama a função `search` ou a função `search_case_insensitive`, conforme mostrado na Listagem 12-22. Isso ainda não compila.

<Listing number="12-22" file-name="src/main.rs" caption="Chamando `search` ou `search_case_insensitive` com base no valor em `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Finalmente, precisamos verificar a variável de ambiente. As funções para trabalhar com variáveis de ambiente estão no módulo `env` na biblioteca padrão, que já está no escopo no topo de _src/main.rs_. Usaremos a função `var` do módulo `env` para verificar se algum valor foi definido para uma variável de ambiente chamada `IGNORE_CASE`, conforme mostrado na Listagem 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Verificando qualquer valor em uma variável de ambiente chamada `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Aqui, criamos uma nova variável, `ignore_case`. Para definir seu valor, chamamos a função `env::var` e passamos o nome da variável de ambiente `IGNORE_CASE`. A função `env::var` retorna um `Result` que será a variante de sucesso `Ok` contendo o valor da variável de ambiente se a variável de ambiente estiver definida para qualquer valor. Ela retornará a variante `Err` se a variável de ambiente não estiver definida.

Estamos usando o método `is_ok` no `Result` para verificar se a variável de ambiente está definida, o que significa que o programa deve fazer uma busca insensível a maiúsculas e minúsculas. Se a variável de ambiente `IGNORE_CASE` não estiver definida com nenhum valor, `is_ok` retornará `false` e o programa realizará uma busca sensível a maiúsculas e minúsculas. Nós não nos importamos com o _valor_ da variável de ambiente, apenas se ela está definida ou não, então estamos verificando `is_ok` em vez de usar `unwrap`, `expect` ou qualquer um dos outros métodos que vimos em `Result`.

Passamos o valor na variável `ignore_case` para a instância de `Config` para que a função `run` possa ler esse valor e decidir se chama `search_case_insensitive` ou `search`, como implementamos na Listagem 12-22.

Vamos testar! Primeiro, executaremos nosso programa sem a variável de ambiente definida e com a consulta `to`, que deve corresponder a qualquer linha que contenha a palavra _to_ em letras minúsculas:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Parece que ainda funciona! Agora vamos executar o programa com `IGNORE_CASE` definido como `1`, mas com a mesma consulta `to`:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Se você estiver usando o PowerShell, precisará definir a variável de ambiente e executar o programa como comandos separados:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Isso fará com que `IGNORE_CASE` persista pelo resto da sua sessão de shell. Ela pode ser removida com o cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Devemos obter linhas que contêm _to_ que podem ter letras maiúsculas:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excelente, também obtivemos linhas contendo _To_! Nosso programa `minigrep` agora pode fazer buscas sem distinção entre maiúsculas e minúsculas controladas por uma variável de ambiente. Agora você sabe como gerenciar opções definidas usando argumentos de linha de comando ou variáveis de ambiente.

Alguns programas permitem argumentos _e_ variáveis de ambiente para a mesma configuração. Nesses casos, os programas decidem que um ou outro tem precedência. Para um outro exercício por sua conta, tente controlar a sensibilidade a maiúsculas e minúsculas por meio de um argumento de linha de comando ou de uma variável de ambiente. Decida se o argumento de linha de comando ou a variável de ambiente deve ter precedência se o programa for executado com um definido como sensível a maiúsculas e minúsculas e outro definido para ignorar maiúsculas e minúsculas.

O módulo `std::env` contém muitos outros recursos úteis para lidar com variáveis de ambiente: confira sua documentação para ver o que está disponível.