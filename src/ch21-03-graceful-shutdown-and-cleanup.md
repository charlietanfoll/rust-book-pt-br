## Encerramento Gracioso e Limpeza

O código na Listagem 21-20 está respondendo a requisições de forma assídua através do uso de um pool de threads, como pretendíamos. Recebemos alguns avisos sobre os campos `workers`, `id` e `thread` que não estamos usando de forma direta, o que nos lembra que não estamos limpando nada. Quando usamos o método menos elegante <kbd>ctrl</kbd>-<kbd>C</kbd> para interromper a thread principal, todas as outras threads são interrompidas imediatamente também, mesmo se estiverem no meio do atendimento a uma requisição.

A seguir, portanto, implementaremos a trait `Drop` para chamar `join` em cada uma das threads no pool para que elas possam terminar as requisições em que estão trabalhando antes de fechar. Depois, implementaremos uma maneira de dizer às threads que elas devem parar de aceitar novas requisições e desligar. Para ver esse código em ação, modificaremos nosso servidor para aceitar apenas duas requisições antes de desligar graciosamente o seu pool de threads.

Uma coisa a notar à medida que avançamos: nada disso afeta as partes do código que lidam com a execução das closures, então tudo aqui seria igual se estivéssemos usando um pool de threads para um runtime assíncrono.

### Implementando a Trait `Drop` em `ThreadPool`

Vamos começar implementando `Drop` no nosso pool de threads. Quando o pool é descartado (dropped), todas as nossas threads devem se juntar (join) para garantir que terminem seu trabalho. A Listagem 21-22 mostra uma primeira tentativa de implementação de `Drop`; este código ainda não vai funcionar completamente.

<Listing number="21-22" file-name="src/lib.rs" caption="Fazendo join de cada thread quando o pool de threads sai de escopo">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Primeiro, fazemos um loop por cada um dos `workers` do pool de threads. Usamos `&mut` para isso porque `self` é uma referência mutável, e também precisamos ser capazes de mutar `worker`. Para cada `worker`, imprimimos uma mensagem dizendo que esta instância específica de `Worker` está sendo encerrada, e então chamamos `join` na thread daquela instância de `Worker`. Se a chamada para `join` falhar, usamos `unwrap` para fazer o Rust entrar em pânico e ir para um encerramento indelicado (ungraceful shutdown).

Aqui está o erro que recebemos quando compilamos este código:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

O erro nos diz que não podemos chamar `join` porque temos apenas um empréstimo mutável (`mutable borrow`) de cada `worker` e `join` toma posse (ownership) de seu argumento. Para resolver esse problema, precisamos retirar a thread da instância de `Worker` que possui `thread` para que `join` possa consumir a thread. Uma maneira de fazer isso é adotar a mesma abordagem que tomamos na Listagem 18-15. Se `Worker` contivesse um `Option<thread::JoinHandle<()>>`, poderíamos chamar o método `take` no `Option` para mover o valor para fora da variante `Some` e deixar uma variante `None` em seu lugar. Em outras palavras, um `Worker` que está rodando teria uma variante `Some` em `thread`, e quando quiséssemos limpar um `Worker`, substituiríamos `Some` por `None` para que o `Worker` não tivesse uma thread para executar.

No entanto, a *única* vez que isso surgiria seria ao descartar o `Worker`. Em troca, teríamos que lidar com um `Option<thread::JoinHandle<()>>` em qualquer lugar onde acessássemos `worker.thread`. O Rust idiomático usa `Option` bastante, mas quando você se pega envolvendo algo que você sabe que estará sempre presente em um `Option` como uma solução alternativa como essa, é uma boa ideia procurar abordagens alternativas para tornar seu código mais limpo e menos suscetível a erros.

Nesse caso, existe uma alternativa melhor: o método `Vec::drain`. Ele aceita um parâmetro de intervalo para especificar quais itens remover do vetor e retorna um iterador desses itens. Passar a sintaxe de intervalo `..` removerá *todos* os valores do vetor.

Portanto, precisamos atualizar a implementação de `drop` do `ThreadPool` assim:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Isso resolve o erro do compilador e não requer nenhuma outra alteração em nosso código. Note que, como o drop pode ser chamado durante um pânico, o `unwrap` também pode entrar em pânico e causar um pânico duplo, o que trava imediatamente o programa e encerra qualquer limpeza em andamento. Tudo bem para um programa de exemplo, mas não é recomendado para código de produção.

### Sinalizando para as Threads Pararem de Escutar por Tarefas

Com todas as alterações que fizemos, nosso código compila sem nenhum aviso. No entanto, a má notícia é que este código ainda não funciona da maneira que queremos. A chave é a lógica nas closures executadas pelas threads das instâncias de `Worker`: no momento, chamamos `join`, mas isso não vai desligar as threads, porque elas entram em um `loop` infinito procurando por tarefas. Se tentarmos descartar nosso `ThreadPool` com nossa implementação atual de `drop`, a thread principal bloqueará para sempre, esperando que a primeira thread termine.

Para corrigir esse problema, precisaremos de uma alteração na implementação de `drop` do `ThreadPool` e, em seguida, uma alteração no loop do `Worker`.

Primeiro, mudaremos a implementação de `drop` do `ThreadPool` para descartar explicitamente o `sender` antes de esperar que as threads terminem. A Listagem 21-23 mostra as alterações no `ThreadPool` para descartar explicitamente o `sender`. Ao contrário da thread, aqui _precisamos_ usar um `Option` para poder mover o `sender` para fora de `ThreadPool` com `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Descartando explicitamente o `sender` antes de fazer join nas threads do `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Descartar o `sender` fecha o canal, o que indica que nenhuma outra mensagem será enviada. Quando isso acontece, todas as chamadas para `recv` que as instâncias de `Worker` fazem no loop infinito retornarão um erro. Na Listagem 21-24, mudamos o loop do `Worker` para sair graciosamente do loop nesse caso, o que significa que as threads terminarão quando a implementação de `drop` do `ThreadPool` chamar `join` nelas.

<Listing number="21-24" file-name="src/lib.rs" caption="Saindo explicitamente do loop quando `recv` retorna um erro">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Para ver este código em ação, vamos modificar `main` para aceitar apenas duas requisições antes de desligar graciosamente o servidor, como mostrado na Listagem 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Desligando o servidor após atender duas requisições ao sair do loop">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Você não iria querer que um servidor web do mundo real fosse desligado após atender apenas duas requisições. Este código apenas demonstra que o encerramento gracioso e a limpeza estão funcionando corretamente.

O método `take` é definido na trait `Iterator` e limita a iteração aos dois primeiros itens no máximo. O `ThreadPool` sairá de escopo no final de `main`, e a implementação de `drop` será executada.

Inicie o servidor com `cargo run` e faça três requisições. A terceira requisição deve apresentar erro, e no seu terminal, você deve ver uma saída semelhante a esta:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Você pode ver uma ordem diferente de IDs de `Worker` e mensagens impressas. Podemos ver como este código funciona através das mensagens: as instâncias de `Worker` 0 e 3 pegaram as duas primeiras requisições. O servidor parou de aceitar conexões após a segunda conexão, e a implementação de `Drop` no `ThreadPool` começa a ser executada antes mesmo de o `Worker 3` começar seu trabalho. O descarte do `sender` desconecta todas as instâncias de `Worker` e diz a elas para desligarem. Cada instância de `Worker` imprime uma mensagem quando se desconecta, e então o pool de threads chama `join` para esperar que cada thread do `Worker` termine.

Note um aspecto interessante desta execução específica: o `ThreadPool` descartou o `sender`, e antes que qualquer `Worker` recebesse um erro, tentamos fazer o `join` do `Worker 0`. O `Worker 0` ainda não havia recebido um erro de `recv`, então a thread principal bloqueou, esperando o `Worker 0` terminar. Enquanto isso, o `Worker 3` recebeu uma tarefa e então todas as threads receberam um erro. Quando o `Worker 0` terminou, a thread principal esperou o restante das instâncias de `Worker` terminarem. Nesse ponto, todas elas já tinham saído de seus loops e parado.

Parabéns! Agora concluímos nosso projeto; temos um servidor web básico que usa um pool de threads para responder de forma assídua. Somos capazes de realizar um encerramento gracioso do servidor, que limpa todas as threads no pool.

Aqui está o código completo para referência:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Poderíamos fazer mais aqui! Se você quiser continuar aprimorando este projeto, aqui estão algumas ideias:

- Adicionar mais documentação ao `ThreadPool` e seus métodos públicos.
- Adicionar testes para a funcionalidade da biblioteca.
- Mudar as chamadas para `unwrap` para um tratamento de erros mais robusto.
- Usar o `ThreadPool` para executar alguma tarefa que não seja atender requisições web.
- Encontrar uma crate de pool de threads no [crates.io](https://crates.io/) e implementar um servidor web semelhante usando a crate em vez disso. Depois, compare sua API e robustez com o pool de threads que implementamos.

## Resumo

Muito bem! Você chegou ao final do livro! Queremos agradecer por se juntar a nós nesta jornada pelo Rust. Agora você está pronto para implementar seus próprios projetos em Rust e ajudar com os projetos de outras pessoas. Lembre-se de que há uma comunidade acolhedora de outros Rustacean que adorariam ajudá-lo com qualquer desafio que você encontrar em sua jornada no Rust.