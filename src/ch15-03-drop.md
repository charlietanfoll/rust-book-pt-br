## Executando Código na Limpeza com a Trait `Drop`

A segunda trait importante para o padrão de ponteiro inteligente é a `Drop`, que permite que você customize o que acontece prestes a um valor sair de escopo. Você pode fornecer uma implementação para a trait `Drop` em qualquer tipo, e esse código pode ser usado para liberar recursos como arquivos ou conexões de rede.

Estamos introduzindo `Drop` no contexto de ponteiros inteligentes porque a funcionalidade da trait `Drop` é quase sempre usada ao implementar um ponteiro inteligente. Por exemplo, quando um `Box<T>` é descartado (*dropped*), ele desaloca o espaço na heap para o qual o ponteiro aponta.

Em algumas linguagens, para alguns tipos, o programador deve chamar código para liberar memória ou recursos toda vez que terminar de usar uma instância desses tipos. Exemplos incluem manipuladores de arquivos (*file handles*), sockets e locks. Se o programador esquecer, o sistema pode ficar sobrecarregado e travar. Em Rust, você pode especificar que um trecho específico de código seja executado sempre que um valor sair de escopo, e o compilador inserirá esse código automaticamente. Como resultado, você não precisa se preocupar em colocar código de limpeza em todos os lugares do programa em que o uso de uma instância de um determinado tipo terminou — você ainda assim não deixará recursos vazarem (*leak resources*)!

Você especifica o código a ser executado quando um valor sai de escopo implementando a trait `Drop`. A trait `Drop` requer que você implemente um método chamado `drop` que aceita uma referência mutável para `self`. Para ver quando o Rust chama `drop`, vamos implementar `drop` com declarações `println!` por enquanto.

A Listagem 15-14 mostra uma struct `CustomSmartPointer` cuja única funcionalidade customizada é que ela imprimirá `Dropping CustomSmartPointer!` quando a instância sair de escopo, para mostrar quando o Rust executa o método `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="Uma struct `CustomSmartPointer` que implementa a trait `Drop` onde colocaríamos nosso código de limpeza">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

A trait `Drop` está incluída no prelúdio (*prelude*), então não precisamos trazê-la para o escopo. Implementamos a trait `Drop` em `CustomSmartPointer` e fornecemos uma implementação para o método `drop` que chama `println!`. O corpo do método `drop` é onde você colocaria qualquer lógica que quisesse executar quando uma instância do seu tipo saísse de escopo. Estamos imprimindo algum texto aqui para demonstrar visualmente quando o Rust chamará `drop`.

Em `main`, criamos duas instâncias de `CustomSmartPointer` e depois imprimimos `CustomSmartPointers created`. No final de `main`, nossas instâncias de `CustomSmartPointer` sairão de escopo, e o Rust chamará o código que colocamos no método `drop`, imprimindo nossa mensagem final. Note que não precisamos chamar o método `drop` explicitamente.

Quando executamos este programa, veremos a seguinte saída:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

O Rust chamou `drop` automaticamente para nós quando nossas instâncias saíram de escopo, chamando o código que especificamos. As variáveis são descartadas na ordem inversa de sua criação, então `d` foi descartado antes de `c`. O propósito deste exemplo é dar a você um guia visual de como o método `drop` funciona; normalmente você especificaria o código de limpeza que o seu tipo precisa executar em vez de uma mensagem de impressão.

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

Infelizmente, não é simples desativar a funcionalidade automática de `drop`. Desativar `drop` geralmente não é necessário; o objetivo principal da trait `Drop` é que ela cuida disso automaticamente. Ocasionalmente, no entanto, você pode querer limpar um valor mais cedo. Um exemplo é ao usar ponteiros inteligentes que gerenciam locks: você pode querer forçar o método `drop` que libera o lock para que outro código no mesmo escopo possa adquiri-lo. O Rust não permite que você chame o método `drop` da trait `Drop` manualmente; em vez disso, você deve chamar a função `std::mem::drop` fornecida pela biblioteca padrão se quiser forçar um valor a ser descartado antes do fim de seu escopo.

Tentar chamar o método `drop` da trait `Drop` manualmente modificando a função `main` da Listagem 15-14 não funcionará, como mostrado na Listagem 15-15.

<Listing number="15-15" file-name="src/main.rs" caption="Tentando chamar o método `drop` da trait `Drop` manualmente para fazer a limpeza mais cedo">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Quando tentamos compilar este código, obtemos este erro:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Esta mensagem de erro afirma que não temos permissão para chamar `drop` explicitamente. A mensagem de erro usa o termo _destrutor_ (*destructor*), que é o termo geral de programação para uma função que limpa uma instância. Um _destrutor_ é análogo a um _construtor_, que cria uma instância. A função `drop` em Rust é um destrutor específico.

O Rust não nos deixa chamar `drop` explicitamente porque o Rust ainda chamaria `drop` automaticamente no valor ao final de `main`. Isso causaria um erro de liberação dupla (*double free error*) porque o Rust estaria tentando limpar o mesmo valor duas vezes.

Não podemos desativar a inserção automática de `drop` quando um valor sai de escopo, e não podemos chamar o método `drop` explicitamente. Portanto, se precisarmos forçar a limpeza antecipada de um valor, usamos a função `std::mem::drop`.

A função `std::mem::drop` é diferente do método `drop` na trait `Drop`. Nós a chamamos passando como argumento o valor que queremos forçar o descarte. A função está no prelúdio, então podemos modificar `main` na Listagem 15-15 para chamar a função `drop`, como mostrado na Listagem 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="Chamando `std::mem::drop` para descartar explicitamente um valor antes que ele saia de escopo">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Executar este código imprimirá o seguinte:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

O texto ``Dropping CustomSmartPointer with data `some data`!`` é impresso entre o texto `CustomSmartPointer created` e `CustomSmartPointer dropped before the end of main`, mostrando que o código do método `drop` é chamado para descartar `c` naquele ponto.

Você pode usar o código especificado em uma implementação da trait `Drop` de muitas maneiras para tornar a limpeza conveniente e segura: por exemplo, você poderia usá-lo para criar seu próprio alocador de memória! Com a trait `Drop` e o sistema de propriedade (*ownership*) do Rust, você não precisa se lembrar de limpar, porque o Rust faz isso automaticamente.

Você também não precisa se preocupar com problemas decorrentes de limpar acidentalmente valores ainda em uso: o sistema de propriedade que garante que as referências sejam sempre válidas também assegura que `drop` seja chamado apenas uma vez quando o valor não estiver mais sendo usado.

Agora que examinamos `Box<T>` e algumas das características dos ponteiros inteligentes, vamos dar uma olhada em alguns outros ponteiros inteligentes definidos na biblioteca padrão.
