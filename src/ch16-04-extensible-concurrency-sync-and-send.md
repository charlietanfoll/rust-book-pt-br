<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>
<a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## Concorrência Extensível com `Send` e `Sync`

Curiosamente, quase todos os recursos de concorrência de que falamos até agora
neste capítulo fazem parte da biblioteca padrão, e não da linguagem. Suas opções
para lidar com concorrência não se limitam à linguagem ou à biblioteca padrão;
você pode escrever seus próprios recursos de concorrência ou usar aqueles escritos
por terceiros.

No entanto, entre os principais conceitos de concorrência embutidos na linguagem,
em vez de na biblioteca padrão, estão as traits marcadoras (`marker traits`) `std::marker::Send` e `Sync`.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### Transferindo Propriedade entre Threads

A trait marcadora `Send` indica que a propriedade de valores do tipo que implementa
`Send` pode ser transferida entre threads. Quase todos os tipos em Rust implementam
`Send`, mas há algumas exceções, incluindo `Rc<T>`: isso não pode implementar
`Send` porque se você clonasse um valor `Rc<T>` e tentasse transferir a propriedade
do clone para outra thread, ambas as threads poderiam atualizar a contagem de
referências ao mesmo tempo. Por esse motivo, `Rc<T>` é implementado para uso em
situações de uma única thread, onde você não quer pagar o preço de desempenho
necessário para garantir a segurança entre threads.

Portanto, o sistema de tipos e as restrições de trait (*trait bounds*) do Rust
garantem que você nunca poderá enviar acidentalmente um valor `Rc<T>` entre threads
de forma não segura. Quando tentamos fazer isso na Listagem 16-14, obtivemos o erro
`` a trait `Send` não está implementada para `Rc<Mutex<i32>>` `` (`the trait `Send` is not implemented for `Rc<Mutex<i32>>``). Quando mudamos para `Arc<T>`, que implementa
`Send`, o código compilou.

Qualquer tipo composto inteiramente por tipos `Send` também é marcado automaticamente
como `Send`. Quase todos os tipos primitivos são `Send`, com exceção de ponteiros
nus (*raw pointers*), que discutiremos no Capítulo 20.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### Acessando a partir de Múltiplas Threads

A trait marcadora `Sync` indica que é seguro que o tipo que implementa `Sync`
seja referenciado a partir de múltiplas threads. Em outras palavras, qualquer tipo
`T` implementa `Sync` se `&T` (uma referência imutável a `T`) implementar `Send`, o
que significa que a referência pode ser enviada com segurança para outra thread.
Semelhante ao `Send`, os tipos primitivos implementam `Sync`, e tipos compostos
inteiramente por tipos que implementam `Sync` também implementam `Sync`.

O ponteiro inteligente `Rc<T>` também não implementa `Sync` pelas mesmas razões
pelas quais não implementa `Send`. O tipo `RefCell<T>` (sobre o qual falamos no
Capítulo 15) e a família de tipos relacionados `Cell<T>` não implementam `Sync`.
A implementação de verificação de empréstimo (*borrow checking*) que o `RefCell<T>`
faz em tempo de execução não é segura para threads. O ponteiro inteligente `Mutex<T>`
implementa `Sync` e pode ser usado para compartilhar o acesso com múltiplas threads,
como você viu em [“Acesso Compartilhado a `Mutex<T>`”][shared-access]<!-- ignore -->.

### Implementar `Send` e `Sync` Manualmente é Inseguro

Como tipos compostos inteiramente por outros tipos que implementam as traits
`Send` e `Sync` também implementam automaticamente `Send` e `Sync`, não precisamos
implementar essas traits manualmente. Sendo traits marcadoras, elas nem sequer têm
métodos para implementar. Elas são apenas úteis para impor invariantes relacionadas
à concorrência.

Implementar manualmente essas traits envolve escrever código Rust inseguro (*unsafe*).
Falaremos sobre o uso de código Rust inseguro no Capítulo 20; por ora, a informação
importante é que criar novos tipos concorrentes que não são feitos de partes
`Send` e `Sync` exige cuidado meticuloso para manter as garantias de segurança. O
[“The Rustonomicon”][nomicon] tem mais informações sobre essas garantias e como
mantê-las.

## Resumo

Esta não é a última vez que você verá concorrência neste livro: o próximo capítulo
foca em programação assíncrona, e o projeto do Capítulo 21 usará os conceitos deste
capítulo em uma situação mais realista do que os exemplos menores discutidos aqui.

Como mencionado anteriormente, dado que muito pouco da forma como o Rust lida com
concorrência faz parte da linguagem, muitas soluções de concorrência são implementadas
como *crates*. Elas evoluem mais rapidamente do que a biblioteca padrão, portanto,
certifique-se de pesquisar online pelas *crates* atuais e de última geração para usar
em situações multithread.

A biblioteca padrão do Rust fornece canais para passagem de mensagens e tipos de
ponteiros inteligentes, como `Mutex<T>` e `Arc<T>`, que são seguros para uso em
contextos concorrentes. O sistema de tipos e o verificador de empréstimos garantem
que o código que usa essas soluções não termine com *data races* (corridas de dados)
ou referências inválidas. Assim que você conseguir fazer seu código compilar, poderá
ficar tranquilo sabendo que ele será executado com segurança em múltiplas threads,
sem o tipo de bugs difíceis de rastrear que são comuns em outras linguagens. A
programação concorrente não é mais um conceito a ser temido: vá em frente e torne
seus programas concorrentes, sem medo!

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: ../nomicon/index.html
