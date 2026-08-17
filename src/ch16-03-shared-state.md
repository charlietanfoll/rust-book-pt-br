## Concorrência de Estado Compartilhado

A passagem de mensagens é uma boa maneira de lidar com concorrência, mas não é a única.
Outro método seria permitir que múltiplas threads acessem os mesmos dados compartilhados.
Considere novamente esta parte do slogan da documentação da linguagem Go: “Não
se comunique compartilhando memória.”

Como seria se comunicar compartilhando memória? Além disso, por que os entusiastas
da passagem de mensagens alertam para não se usar o compartilhamento de memória?

De certa forma, canais em qualquer linguagem de programação são semelhantes à
posse única, porque, assim que você transfere um valor por um canal, você não
deve mais usar esse valor. A concorrência de memória compartilhada é como a
posse múltipla: Múltiplas threads podem acessar o mesmo local de memória ao mesmo
tempo. Como você viu no Capítulo 15, onde os ponteiros inteligentes tornaram a
posse múltipla possível, a posse múltipla pode adicionar complexidade porque
esses diferentes proprietários precisam ser gerenciados. O sistema de tipos e as
regras de posse do Rust ajudam enormemente a fazer esse gerenciamento da forma
correta. Como exemplo, vamos analisar os mutexes, uma das primitivas de
concorrência mais comuns para memória compartilhada.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### Controlando o Acesso com Mutexes

_Mutex_ é uma abreviação para _mutual exclusion_ (exclusão mútua), o que significa
que um mutex permite que apenas uma thread acesse determinados dados em um dado
momento. Para acessar os dados em um mutex, uma thread deve primeiro sinalizar
que deseja acesso pedindo para adquirir o bloqueio (_lock_) do mutex. O _lock_ é
uma estrutura de dados que faz parte do mutex e controla quem atualmente tem
acesso exclusivo aos dados. Portanto, diz-se que o mutex está _guardando_ (protegendo)
os dados que ele contém através do sistema de bloqueio.

Os mutexes têm a reputação de serem difíceis de usar porque você precisa se
lembrar de duas regras:

1. Você deve tentar adquirir o bloqueio antes de usar os dados.
2. Quando terminar com os dados que o mutex protege, você deve desbloquear os
   dados para que outras threads possam adquirir o bloqueio.

Para uma metáfora do mundo real sobre um mutex, imagine um painel de discussão
em uma conferência com apenas um microfone. Antes que um participante possa
falar, ele deve pedir ou sinalizar que deseja usar o microfone. Quando obtêm o
microfone, eles podem falar pelo tempo que quiserem e, em seguida, entregar o
microfone para o próximo participante que solicitar a palavra. Se um participante
esquece de entregar o microfone quando termina, mais ninguém consegue falar.
Se o gerenciamento do microfone compartilhado der errado, o painel não funcionará
como planejado!

O gerenciamento de mutexes pode ser incrivelmente difícil de acertar, razão pela
qual tantas pessoas são entusiastas de canais. No entanto, graças ao sistema de
tipos e às regras de posse do Rust, você não pode errar o bloqueio e o desbloqueio.

#### A API de `Mutex<T>`

Como exemplo de como usar um mutex, vamos começar usando um mutex em um contexto
de thread única (single-threaded), conforme mostrado na Listagem 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="Explorando a API de `Mutex<T>` em um contexto de thread única para simplificar">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Assim como com muitos tipos, criamos um `Mutex<T>` usando a função associada `new`.
Para acessar os dados dentro do mutex, usamos o método `lock` para adquirir o
bloqueio. Esta chamada irá bloquear a thread atual para que ela não possa realizar
nenhum trabalho até que seja a nossa vez de ter o bloqueio.

A chamada para `lock` falharia se outra thread que estivesse segurando o bloqueio
entrasse em pânico (_panic_). Nesse caso, ninguém jamais seria capaz de obter o
bloqueio, então escolhemos usar `unwrap` e fazer com que esta thread entre em
pânico se estivermos nessa situação.

Depois de adquirir o bloqueio, podemos tratar o valor de retorno, chamado `num`
neste caso, como uma referência mutável aos dados internos. O sistema de tipos
garante que adquiramos um bloqueio antes de usar o valor em `m`. O tipo de `m` é
`Mutex<i32>`, e não `i32`, então _devemos_ chamar `lock` para poder usar o valor
`i32`. Não podemos esquecer; o sistema de tipos não nos deixará acessar o `i32`
interno de outra forma.

A chamada para `lock` retorna um tipo chamado `MutexGuard`, envolvido em um
`LockResult` que tratamos com a chamada para `unwrap`. O tipo `MutexGuard`
implementa `Deref` para apontar para os nossos dados internos; o tipo também tem
uma implementação de `Drop` que libera o bloqueio automaticamente quando um
`MutexGuard` sai de escopo, o que acontece no final do escopo interno. Como
resultado, não corremos o risco de esquecer de liberar o bloqueio e impedir que o
mutex seja usado por outras threads, porque a liberação do bloqueio acontece
automaticamente.

Após o término do escopo do bloqueio, podemos imprimir o valor do mutex e ver que
conseguimos alterar o `i32` interno para `6`.

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### Acesso Compartilhado a `Mutex<T>`

Agora vamos tentar compartilhar um valor entre múltiplas threads usando `Mutex<T>`.
Vamos iniciar 10 threads e fazer com que cada uma incremente o valor de um
contador em 1, para que o contador vá de 0 a 10. O exemplo na Listagem 16-13 terá
um erro de compilação, e usaremos esse erro para aprender mais sobre o uso de
`Mutex<T>` e como o Rust nos ajuda a usá-lo corretamente.

<Listing number="16-13" file-name="src/main.rs" caption="Dez threads, cada uma incrementando um contador protegido por um `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Criamos uma variável `counter` para conter um `i32` dentro de um `Mutex<T>`, como
fizemos na Listagem 16-12. Em seguida, criamos 10 threads iterando sobre um
intervalo de números. Usamos `thread::spawn` e fornecemos a todas as threads a
mesma closure: uma que move o contador para dentro da thread, adquire um bloqueio
no `Mutex<T>` chamando o método `lock`, e então adiciona 1 ao valor no mutex.
Quando uma thread termina de executar sua closure, `num` sairá de escopo e
liberará o bloqueio para que outra thread possa adquiri-lo.

Na thread principal, coletamos todos os manipuladores de junção (_join handles_).
Então, como fizemos na Listagem 16-2, chamamos `join` em cada manipulador para
garantir que todas as threads terminem. Nesse ponto, a thread principal adquirirá
o bloqueio e imprimirá o resultado deste programa.

Sugerimos que este exemplo não compilaria. Agora vamos descobrir o porquê!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

A mensagem de erro afirma que o valor `counter` foi movido na iteração anterior
do loop. O Rust está nos dizendo que não podemos mover a propriedade do bloqueio
`counter` para múltiplas threads. Vamos corrigir o erro de compilação com o
método de posse múltipla que discutimos no Capítulo 15.

#### Posse Múltipla com Múltiplas Threads

No Capítulo 15, demos um valor a múltiplos proprietários usando o ponteiro
inteligente `Rc<T>` para criar um valor com contagem de referências. Vamos fazer
o mesmo aqui e ver o que acontece. Vamos envolver o `Mutex<T>` em `Rc<T>` na
Listagem 16-14 e clonar o `Rc<T>` antes de mover a propriedade para a thread.

<Listing number="16-14" file-name="src/main.rs" caption="Tentando usar `Rc<T>` para permitir que múltiplas threads sejam proprietárias do `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Mais uma vez, compilamos e obtemos... erros diferentes! O compilador está nos
ensinando muito:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Nossa, essa mensagem de erro é muito longa! Aqui está a parte importante em que
devemos nos concentrar: `` `Rc<Mutex<i32>>` não pode ser enviado entre threads com segurança ``.
O compilador também está nos dizendo o motivo: `` a trait `Send` não está implementada
para `Rc<Mutex<i32>>` ``. Falaremos sobre `Send` na próxima seção: É uma das traits
que garante que os tipos que usamos com threads sejam destinados ao uso em
situações concorrentes.

Infelizmente, `Rc<T>` não é seguro para ser compartilhado entre threads. Quando o
`Rc<T>` gerencia a contagem de referências, ele adiciona à contagem para cada
chamada de `clone` e subtrai da contagem quando cada clone é descartado (_dropped_).
No entanto, ele não usa nenhuma primitiva de concorrência para garantir que as
alterações na contagem não possam ser interrompidas por outra thread. Isso
poderia levar a contagens incorretas — bugs sutis que, por sua vez, poderiam
levar a vazamentos de memória ou a um valor sendo descartado antes que terminemos
de usá-lo. O que precisamos é de um tipo que seja exatamente como o `Rc<T>`, mas
que faça alterações na contagem de referências de maneira segura para threads
(_thread-safe_).

#### Contagem de Referências Atômica com `Arc<T>`

Felizmente, `Arc<T>` _é_ um tipo como o `Rc<T>` que é seguro para uso em situações
concorrentes. O _a_ significa _atomic_ (atômico), o que significa que é um tipo
com _contagem de referências atômica_. Operações atômicas são um tipo adicional
de primitiva de concorrência que não abordaremos em detalhes aqui: Consulte a
documentação da biblioteca padrão para [`std::sync::atomic`][atomic]<!-- ignore -->
para mais detalhes. Neste ponto, você só precisa saber que as operações atômicas
funcionam como tipos primitivos, mas são seguras para serem compartilhadas entre
threads.

Você pode então se perguntar por que todos os tipos primitivos não são atômicos e
por que os tipos da biblioteca padrão não são implementados para usar `Arc<T>` por
padrão. A razão é que a segurança de threads vem com um custo de desempenho que
você só quer pagar quando realmente precisa. Se você está apenas realizando
operações em valores dentro de uma única thread, seu código pode rodar mais
rápido se não precisar impor as garantias que as operações atômicas fornecem.

Vamos voltar ao nosso exemplo: `Arc<T>` e `Rc<T>` têm a mesma API, então
corrigimos nosso programa alterando a linha `use`, a chamada para `new` e a chamada
para `clone`. O código na Listagem 16-15 finalmente compilará e executará.

<Listing number="16-15" file-name="src/main.rs" caption="Usando um `Arc<T>` para envolver o `Mutex<T>` para poder compartilhar a propriedade entre múltiplas threads">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Este código imprimirá o seguinte:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Conseguimos! Contamos de 0 a 10, o que pode não parecer muito impressionante, mas
nos ensinou muito sobre `Mutex<T>` e segurança de threads. Você também pode usar
a estrutura deste programa para fazer operações mais complexas do que apenas
incrementar um contador. Usando esta estratégia, você pode dividir um cálculo em
partes independentes, espalhar essas partes entre as threads e, em seguida, usar
um `Mutex<T>` para fazer com que cada thread atualize o resultado final com a
sua parte.

Observe que, se você estiver fazendo operações numéricas simples, existem tipos
mais simples do que `Mutex<T>` fornecidos pelo [módulo `std::sync::atomic` da
biblioteca padrão][atomic]<!-- ignore -->. Esses tipos fornecem acesso atômico,
concorrente e seguro a tipos primitivos. Escolhemos usar `Mutex<T>` com um tipo
primitivo para este exemplo para que pudéssemos nos concentrar em como o `Mutex<T>`
funciona.

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### Comparando `RefCell<T>`/`Rc<T>` e `Mutex<T>`/`Arc<T>`

Você deve ter notado que `counter` é imutável, mas conseguimos obter uma referência
mutável para o valor dentro dele; isso significa que `Mutex<T>` fornece mutabilidade
interior, assim como a família `Cell` faz. Da mesma forma que usamos `RefCell<T>`
no Capítulo 15 para nos permitir mutar conteúdos dentro de um `Rc<T>`, usamos
`Mutex<T>` para mutar conteúdos dentro de um `Arc<T>`.

Outro detalhe a notar é que o Rust não pode protegê-lo de todos os tipos de erros
de lógica quando você usa `Mutex<T>`. Lembre-se do Capítulo 15 que o uso de `Rc<T>`
vinha com o risco de criar ciclos de referência, onde dois valores `Rc<T>`
apontam um para o outro, causando vazamentos de memória. Da mesma forma, `Mutex<T>`
vem com o risco de criar _deadlocks_ (impasses). Eles ocorrem quando uma operação
precisa bloquear dois recursos e duas threads adquiriram cada uma um dos
bloqueios, fazendo com que fiquem esperando uma pela outra para sempre. Se você se
interessa por deadlocks, tente criar um programa em Rust que tenha um deadlock;
então, pesquise estratégias de mitigação de deadlock para mutexes em qualquer
linguagem e tente implementá-las em Rust. A documentação da API da biblioteca
padrão para `Mutex<T>` e `MutexGuard` oferece informações úteis.

Vamos concluir este capítulo falando sobre as traits `Send` e `Sync` e como podemos
usá-las com tipos personalizados.

[atomic]: ../std/sync/atomic/index.html
