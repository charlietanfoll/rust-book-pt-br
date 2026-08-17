## Rust Inseguro

Todo o código que discutimos até agora teve suas garantias de segurança de
memória do Rust aplicadas em tempo de compilação. No entanto, o Rust tem uma
segunda linguagem oculta dentro dele que não aplica essas garantias de
segurança de memória: ela é chamada de _Rust inseguro_ (_unsafe Rust_) e
funciona exatamente como o Rust normal, mas nos dá superpoderes extras.

O Rust inseguro existe porque, por natureza, a análise estática é conservadora.
Quando o compilador tenta determinar se o código cumpre ou não as garantias, é
melhor que ele rejeite alguns programas válidos do que aceite alguns programas
inválidos. Embora o código _possa_ estar correto, se o compilador do Rust não
tiver informações suficientes para ter certeza, ele rejeitará o código. Nesses
casos, você pode usar código inseguro para dizer ao compilador: "Confie em mim,
eu sei o que estou fazendo". Esteja avisado, no entanto, que você usa o Rust
inseguro por sua conta e risco: se você usar código inseguro incorretamente,
podem ocorrer problemas devido à insegurança de memória, como a desreferenciação
de um ponteiro nulo.

Outra razão pela qual o Rust tem um alter ego inseguro é que o hardware de
computador subjacente é inerentemente inseguro. Se o Rust não permitisse que
você realizasse operações inseguras, você não conseguiria realizar certas
tarefas. O Rust precisa permitir que você faça programação de sistemas de baixo
nível, como interagir diretamente com o sistema operacional ou até mesmo
escrever seu próprio sistema operacional. Trabalhar com programação de sistemas
de baixo nível é um dos objetivos da linguagem. Vamos explorar o que podemos
fazer com o Rust inseguro e como fazê-lo.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### Executando Superpoderes Inseguros

Para alternar para o Rust inseguro, use a palavra-chave `unsafe` e inicie um
novo bloco que contenha o código inseguro. Você pode realizar cinco ações no
Rust inseguro que não pode fazer no Rust seguro, às quais chamamos de
_superpoderes inseguros_. Esses superpoderes incluem a capacidade de:

1. Desreferenciar um ponteiro bruto.
1. Chamar uma função ou método inseguro.
1. Acessar ou modificar uma variável estática mutável.
1. Implementar uma trait insegura.
1. Acessar campos de `union`s.

É importante entender que `unsafe` não desativa o verificador de empréstimos
(`borrow checker`) nem desativa nenhuma das outras verificações de segurança do
Rust: se você usar uma referência em código inseguro, ela ainda será verificada.
A palavra-chave `unsafe` apenas dá a você acesso a esses cinco recursos, que
então não são verificados pelo compilador quanto à segurança de memória. Você
ainda terá algum grau de segurança dentro de um bloco inseguro.

Além disso, `unsafe` não significa que o código dentro do bloco seja
necessariamente perigoso ou que terá definitivamente problemas de segurança de
memória: a intenção é que, como programador, você garanta que o código dentro de
um bloco `unsafe` acessará a memória de maneira válida.

As pessoas são suscetíveis a erros e eles vão acontecer, mas ao exigir que
essas cinco operações inseguras estejam dentro de blocos anotados com `unsafe`,
você saberá que quaisquer erros relacionados à segurança de memória devem estar
dentro de um bloco `unsafe`. Mantenha os blocos `unsafe` pequenos; você ficará
grato mais tarde quando for investigar bugs de memória.

Para isolar o código inseguro o máximo possível, é melhor envolver esse código
em uma abstração segura e fornecer uma API segura, o que discutiremos mais adiante
no capítulo quando examinarmos funções e métodos inseguros. Partes da biblioteca
padrão são implementadas como abstrações seguras sobre código inseguro que foi
auditado. Envolver código inseguro em uma abstração segura evita que o uso de
`unsafe` vaze para todos os lugares onde você ou seus usuários possam querer
usar a funcionalidade implementada com código `unsafe`, porque usar uma
abstração segura é seguro.

Vamos examinar cada um dos cinco superpoderes inseguros, um por um. Também
veremos algumas abstrações que fornecem uma interface segura para códigos
inseguros.

### Desreferenciando um Ponteiro Bruto

No Capítulo 4, na seção [“Referências Vazias (Dangling References)”][dangling-references]<!-- ignore
-->, mencionamos que o compilador garante que as referências sejam sempre
válidas. O Rust inseguro tem dois novos tipos chamados _ponteiros brutos_
(_raw pointers_) que são semelhantes a referências. Assim como as referências,
os ponteiros brutos podem ser imutáveis ou mutáveis e são escritos como `*const
T` e `*mut T`, respectivamente. O asterisco não é o operador de desreferência;
ele faz parte do nome do tipo. No contexto de ponteiros brutos, _imutável_
significa que o ponteiro não pode ser atribuído diretamente após ser
desreferenciado.

Diferente de referências e ponteiros inteligentes, os ponteiros brutos:

- Têm permissão para ignorar as regras de empréstimo ao possuírem tanto ponteiros
  imutáveis quanto mutáveis, ou múltiplos ponteiros mutáveis para a mesma
  localização
- Não têm garantia de apontar para memória válida
- Têm permissão para serem nulos
- Não implementam nenhuma limpeza automática

Ao optar por não ter o Rust aplicando essas garantias, você abre mão da
segurança garantida em troca de maior desempenho ou da capacidade de se
comunicar com outra linguagem ou hardware onde as garantias do Rust não se
aplicam.

A Listagem 20-1 mostra como criar um ponteiro bruto imutável e um mutável.

<Listing number="20-1" caption="Criando ponteiros brutos com os operadores de empréstimo bruto">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Note que não incluímos a palavra-chave `unsafe` neste código. Podemos criar
ponteiros brutos em código seguro; apenas não podemos desreferenciar ponteiros
brutos fora de um bloco inseguro, como você verá em breve.

Criamos ponteiros brutos usando os operadores de empréstimo bruto: `&raw const
num` cria um ponteiro bruto imutável `*const i32`, e `&raw mut num` cria um
ponteiro bruto mutável `*mut i32`. Como os criamos diretamente a partir de uma
variável local, sabemos que esses ponteiros brutos específicos são válidos, mas
não podemos fazer essa suposição sobre qualquer ponteiro bruto.

Para demonstrar isso, a seguir criaremos um ponteiro bruto cuja validade não
podemos ter tanta certeza, usando a palavra-chave `as` para fazer um *cast* de
um valor em vez de usar o operador de empréstimo bruto. A Listagem 20-2 mostra
como criar um ponteiro bruto para uma localização arbitrária na memória. Tentar
usar memória arbitrária é indefinido: pode haver dados nesse endereço ou não, o
compilador pode otimizar o código para que não haja acesso à memória, ou o
programa pode terminar com uma falha de segmentação (segmentation fault).
Geralmente, não há um bom motivo para escrever código assim, especialmente em
casos onde você pode usar um operador de empréstimo bruto em vez disso, mas é
possível.

<Listing number="20-2" caption="Criando um ponteiro bruto para um endereço de memória arbitrário">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Lembre-se de que podemos criar ponteiros brutos em código seguro, mas não
podemos desreferenciar ponteiros brutos e ler os dados para os quais estão
apontando. Na Listagem 20-3, usamos o operador de desreferência `*` em um
ponteiro bruto, o que requer um bloco `unsafe`.

<Listing number="20-3" caption="Desreferenciando ponteiros brutos dentro de um bloco `unsafe`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Criar um ponteiro não faz mal; é apenas quando tentamos acessar o valor para o
qual ele aponta que podemos acabar lidando com um valor inválido.

Observe também que nas Listagens 20-1 e 20-3, criamos os ponteiros brutos
`*const i32` e `*mut i32` que apontavam para a mesma localização de memória,
onde `num` está armazenado. Se tentássemos criar uma referência imutável e uma
mutável para `num`, o código não teria compilado porque as regras de
propriedade do Rust não permitem uma referência mutável ao mesmo tempo que
quaisquer referências imutáveis. Com ponteiros brutos, podemos criar um ponteiro
mutável e um imutável para a mesma localização e alterar dados através do
ponteiro mutável, criando potencialmente uma condição de corrida de dados (*data
race*). Tenha cuidado!

Com todos esses perigos, por que você usaria ponteiros brutos? Um caso de uso
principal é ao interagir com código C, como você verá na próxima seção. Outro
caso é ao construir abstrações seguras que o verificador de empréstimos não
conhece. Apresentaremos funções inseguras e, em seguida, veremos um exemplo de
abstração segura que usa código inseguro.

### Chamando uma Função ou Método Inseguro

O segundo tipo de operação que você pode realizar em um bloco inseguro é chamar
funções inseguras. Funções e métodos inseguros parecem exatamente com funções e
métodos normais, mas eles têm um `unsafe` extra antes do resto da definição. A
palavra-chave `unsafe` neste contexto indica que a função tem requisitos que
precisamos cumprir quando a chamamos, porque o Rust não pode garantir que
cumprimos esses requisitos. Ao chamar uma função insegura dentro de um bloco
`unsafe`, estamos dizendo que lemos a documentação dessa função e assumimos a
responsabilidade de cumprir os contratos da função.

Aqui está uma função insegura chamada `dangerous` que não faz nada em seu
corpo:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Devemos chamar a função `dangerous` dentro de um bloco `unsafe` separado. Se
tentarmos chamar `dangerous` sem o bloco `unsafe`, receberemos um erro:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Com o bloco `unsafe`, estamos afirmando para o Rust que lemos a documentação da
função, entendemos como usá-la corretamente e verificamos que estamos cumprindo
o contrato da função.

Para realizar operações inseguras no corpo de uma função `unsafe`, você ainda
precisa usar um bloco `unsafe`, assim como em uma função normal, e o
compilador avisará se você esquecer. Isso nos ajuda a manter os blocos `unsafe`
o menor possível, já que operações inseguras podem não ser necessárias em todo
o corpo da função.

#### Criando uma Abstração Segura sobre Código Inseguro

O fato de uma função conter código inseguro não significa que precisamos marcar
a função inteira como insegura. Na verdade, envolver código inseguro em uma
função segura é uma abstração comum. Como exemplo, vamos estudar a função
`split_at_mut` da biblioteca padrão, que requer algum código inseguro. Vamos
explorar como poderíamos implementá-la. Este método seguro é definido em fatias
(_slices_) mutáveis: ele pega uma fatia e a divide em duas, fatiando a fatia no
índice fornecido como argumento. A Listagem 20-4 mostra como usar
`split_at_mut`.

<Listing number="20-4" caption="Usando a função segura `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Não podemos implementar esta função usando apenas Rust seguro. Uma tentativa
pode se parecer com a Listagem 20-5, que não compilará. Para simplificar,
implementaremos `split_at_mut` como uma função em vez de um método e apenas para
fatias de valores `i32` em vez de para um tipo genérico `T`.

<Listing number="20-5" caption="Uma tentativa de implementação de `split_at_mut` usando apenas Rust seguro">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Esta função primeiro obtém o comprimento total da fatia. Em seguida, ela
afirma que o índice fornecido como parâmetro está dentro da fatia, verificando
se é menor ou igual ao comprimento. A asserção significa que se passarmos um
índice maior do que o comprimento para fatiar a fatia, a função entrará em
pânico antes de tentar usar esse índice.

Em seguida, retornamos duas fatias mutáveis em uma tupla: uma do início da
fatia original até o índice `mid` e outra de `mid` até o final da fatia.

Quando tentamos compilar o código na Listagem 20-5, recebemos um erro:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

O verificador de empréstimos do Rust não consegue entender que estamos
emprestando partes diferentes da fatia; ele apenas sabe que estamos emprestando
da mesma fatia duas vezes. Emprestar partes diferentes de uma fatia é
fundamentalmente seguro porque as duas fatias não se sobrepõem, mas o Rust não
é inteligente o suficiente para saber disso. Quando sabemos que o código está
correto, mas o Rust não, é hora de recorrer a código inseguro.

A Listagem 20-6 mostra como usar um bloco `unsafe`, um ponteiro bruto e algumas
chamadas a funções inseguras para fazer a implementação de `split_at_mut`
funcionar.

<Listing number="20-6" caption="Usando código inseguro na implementação da função `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Lembre-se da seção [“O Tipo Fatia (The Slice Type)”][the-slice-type]<!-- ignore -->
no Capítulo 4 de que uma fatia é um ponteiro para alguns dados e o comprimento da
fatia. Usamos o método `len` para obter o comprimento de uma fatia e o método
`as_mut_ptr` para acessar o ponteiro bruto de uma fatia. Nesse caso, como temos
uma fatia mutável para valores `i32`, `as_mut_ptr` retorna um ponteiro bruto com
o tipo `*mut i32`, que armazenamos na variável `ptr`.

Mantemos a asserção de que o índice `mid` está dentro da fatia. Em seguida,
chegamos ao código inseguro: a função `slice::from_raw_parts_mut` aceita um
ponteiro bruto e um comprimento, e cria uma fatia. Usamos esta função para criar
uma fatia que começa em `ptr` e tem `mid` itens de comprimento. Em seguida,
chamamos o método `add` em `ptr` com `mid` como argumento para obter um ponteiro
bruto que começa em `mid`, e criamos uma fatia usando esse ponteiro e o número
restante de itens após `mid` como comprimento.

A função `slice::from_raw_parts_mut` é insegura porque aceita um ponteiro bruto
e precisa confiar que esse ponteiro é válido. O método `add` em ponteiros
brutos também é inseguro porque deve confiar que a localização deslocada
também é um ponteiro válido. Portanto, tivemos que colocar um bloco `unsafe` ao
redor de nossas chamadas para `slice::from_raw_parts_mut` e `add` para podermos
chamá-las. Olhando para o código e adicionando a asserção de que `mid` deve ser
menor ou igual a `len`, podemos dizer que todos os ponteiros brutos usados
dentro do bloco `unsafe` serão ponteiros válidos para dados dentro da fatia.
Este é um uso aceitável e apropriado de `unsafe`.

Observe que não precisamos marcar a função resultante `split_at_mut` como
`unsafe`, e podemos chamar essa função a partir do Rust seguro. Criamos uma
abstração segura para o código inseguro com uma implementação da função que usa
código `unsafe` de maneira segura, porque ela cria apenas ponteiros válidos a
partir dos dados aos quais esta função tem acesso.

Em contraste, o uso de `slice::from_raw_parts_mut` na Listagem 20-7 provavelmente
travaria quando a fatia for usada. Este código pega uma localização de memória
arbitrária e cria uma fatia com 10.000 itens de comprimento.

<Listing number="20-7" caption="Criando uma fatia a partir de uma localização de memória arbitrária">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Não possuímos a memória nesta localização arbitrária, e não há garantia de que a
fatia que este código cria contenha valores `i32` válidos. Tentar usar `values`
como se fosse uma fatia válida resulta em comportamento indefinido.

#### Usando Funções `extern` para Chamar Código Externo

Às vezes, seu código Rust pode precisar interagir com código escrito em outra
linguagem. Para isso, o Rust tem a palavra-chave `extern` que facilita a
criação e o uso de uma _Interface de Função Estrangeira (FFI — Foreign Function
Interface)_, que é uma maneira de uma linguagem de programação definir funções e
permitir que uma linguagem de programação diferente (estrangeira) chame essas
funções.

A Listagem 20-8 demonstra como configurar uma integração com a função `abs` da
biblioteca padrão do C. Funções declaradas dentro de blocos `extern` são
geralmente inseguras de serem chamadas a partir de código Rust, então os blocos
`extern` também devem ser marcados como `unsafe`. O motivo é que outras
linguagens não aplicam as regras e garantias do Rust, e o Rust não pode
verificá-las, portanto, a responsabilidade recai sobre o programador para garantir
a segurança.

<Listing number="20-8" file-name="src/main.rs" caption="Declarando e chamando uma função `extern` definida em outra linguagem">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Dentro do bloco `unsafe extern "C"`, listamos os nomes e assinaturas de funções
externas de outra linguagem que queremos chamar. A parte `"C"` define qual
_Interface Binária de Aplicação (ABI — Application Binary Interface)_ a função
externa usa: a ABI define como chamar a função a nível de linguagem de
montagem (assembly). A ABI `"C"` é a mais comum e segue a ABI da linguagem de
programação C. Informações sobre todas as ABIs que o Rust suporta estão
disponíveis na [Referência do Rust (The Rust Reference)][ABI].

Cada item declarado dentro de um bloco `unsafe extern` é implicitamente
inseguro. No entanto, algumas funções de FFI *são* seguras de chamar. Por
exemplo, a função `abs` da biblioteca padrão do C não possui nenhuma
consideração de segurança de memória, e sabemos que ela pode ser chamada com
qualquer `i32`. Em casos como este, podemos usar a palavra-chave `safe` para
dizer que esta função específica é segura de chamar, mesmo estando em um bloco
`unsafe extern`. Uma vez que fazemos essa alteração, chamá-la não requer mais
um bloco `unsafe`, como mostrado na Listagem 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Marcando explicitamente uma função como `safe` dentro de um bloco `unsafe extern` e chamando-a de forma segura">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Marcar uma função como `safe` não a torna inerentemente segura! Em vez disso, é
como uma promessa que você está fazendo ao Rust de que ela é segura. Ainda é
sua responsabilidade garantir que essa promessa seja cumprida!

#### Chamando Funções Rust a partir de Outras Linguagens

Também podemos usar `extern` para criar uma interface que permita que outras
linguagens chamem funções Rust. Em vez de criar um bloco `extern` inteiro,
adicionamos a palavra-chave `extern` e especificamos a ABI a ser usada logo
antes da palavra-chave `fn` para a função relevante. Também precisamos
adicionar uma anotação `#[unsafe(no_mangle)]` para dizer ao compilador do Rust
para não alterar a mangueira (fazer _mangling_) do nome desta função. O
_Mangling_ ocorre quando um compilador altera o nome que demos a uma função para
um nome diferente que contém mais informações para outras partes do processo de
compilação consumirem, mas é menos legível para humanos. O compilador de cada
linguagem de programação faz o *mangling* de nomes de forma ligeiramente
diferente, portanto, para que uma função Rust possa ser nomeada por outras
linguagens, devemos desativar o *mangling* de nomes do compilador Rust. Isso é
inseguro porque pode haver colisões de nomes entre bibliotecas sem o *mangling*
integrado, então é nossa responsabilidade garantir que o nome que escolhermos
seja seguro para exportar sem *mangling*.

No exemplo a seguir, tornamos a função `call_from_c` acessível a partir de
código C, após ela ser compilada para uma biblioteca compartilhada e vinculada a
partir do C:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Acabei de chamar uma função Rust a partir do C!");
}
```

Este uso de `extern` requer `unsafe` apenas no atributo, não no bloco `extern`.

### Acessando ou Modificando uma Variável Estática Mutável

Neste livro, ainda não falamos sobre variáveis globais, que o Rust suporta, mas
que podem ser problemáticas com as regras de propriedade do Rust. Se duas
threads estiverem acessando a mesma variável global mutável, isso pode causar
uma condição de corrida de dados (*data race*).

No Rust, variáveis globais são chamadas de variáveis _estáticas_ (_static_). A
Listagem 20-10 mostra um exemplo de declaração e uso de uma variável estática
com uma fatia de string como valor.

<Listing number="20-10" file-name="src/main.rs" caption="Definindo e usando uma variável estática imutável">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Variáveis estáticas são semelhantes a constantes, que discutimos na seção
[“Declarando Constantes”][constants]<!-- ignore --> no Capítulo 3. Os nomes
das variáveis estáticas estão em `SCREAMING_SNAKE_CASE` por convenção.
Variáveis estáticas só podem armazenar referências com o tempo de vida
`'static`, o que significa que o compilador do Rust pode descobrir o tempo de
vida e não somos obrigados a anotá-lo explicitamente. Acessar uma variável
estática imutável é seguro.

Uma diferença sutil entre constantes e variáveis estáticas imutáveis é que os
valores em uma variável estática têm um endereço fixo na memória. O uso do valor
sempre acessará os mesmos dados. Constantes, por outro lado, têm permissão para
duplicar seus dados sempre que são usadas. Outra diferença é que variáveis
estáticas podem ser mutáveis. Acessar e modificar variáveis estáticas mutáveis
é _inseguro_. A Listagem 20-11 mostra como declarar, acessar e modificar uma
variável estática mutável chamada `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Ler ou escrever em uma variável estática mutável é inseguro.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Como em variáveis normais, especificamos a mutabilidade usando a palavra-chave
`mut`. Qualquer código que leia ou escreva a partir de `COUNTER` deve estar
dentro de um bloco `unsafe`. O código na Listagem 20-11 compila e imprime
`COUNTER: 3` como esperaríamos, porque é de thread única. Ter várias threads
acessando `COUNTER` provavelmente resultaria em condições de corrida de dados,
portanto, é um comportamento indefinido. Portanto, precisamos marcar a função
inteira como `unsafe` e documentar a limitação de segurança para que qualquer
pessoa que chame a função saiba o que é ou não permitido fazer com segurança.

Sempre que escrevemos uma função insegura, é idiomático escrever um comentário
começando com `SAFETY` e explicando o que o chamador precisa fazer para chamar a
função com segurança. Da mesma forma, sempre que realizamos uma operação
insegura, é idiomático escrever um comentário começando com `SAFETY` para
explicar como as regras de segurança são mantidas.

Além disso, o compilador negará por padrão qualquer tentativa de criar
referências a uma variável estática mutável por meio de um aviso do compilador
(*lint*). Você deve explicitamente desativar as proteções desse *lint*
adicionando uma anotação `#[allow(static_mut_refs)]` ou acessar a variável
estática mutável por meio de um ponteiro bruto criado com um dos operadores de
empréstimo bruto. Isso inclui casos em que a referência é criada invisivelmente,
como quando é usada no `println!` nesta listagem de código. Exigir que
referências a variáveis estáticas mutáveis sejam criadas por meio de ponteiros
brutos ajuda a tornar os requisitos de segurança para usá-las mais óbvios.

Com dados mutáveis que são globalmente acessíveis, é difícil garantir que não
haja condições de corrida de dados, e é por isso que o Rust considera variáveis
estáticas mutáveis como inseguras. Onde possível, é preferível usar as técnicas
de concorrência e ponteiros inteligentes seguros para threads que discutimos no
Capítulo 16, para que o compilador verifique se o acesso aos dados a partir de
diferentes threads é feito de forma segura.

### Implementando uma Trait Insegura

Podemos usar `unsafe` para implementar uma trait insegura. Uma trait é insegura
quando pelo menos um de seus métodos tem algum invariante que o compilador não
pode verificar. Declaramos que uma trait é `unsafe` adicionando a palavra-chave
`unsafe` antes de `trait` e marcando a implementação da trait como `unsafe`
também, como mostrado na Listagem 20-12.

<Listing number="20-12" caption="Definindo e implementando uma trait insegura">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Ao usar `unsafe impl`, estamos prometendo que manteremos os invariantes que o
compilador não pode verificar.

Como exemplo, relembre as traits marcadoras `Send` e `Sync` que discutimos na
seção [“Concorrência Extensível com `Send` e `Sync`”][send-and-sync]<!-- ignore
--> no Capítulo 16: o compilador implementa essas traits automaticamente se
nossos tipos forem compostos inteiramente de outros tipos que implementam `Send`
e `Sync`. Se implementarmos um tipo que contenha um tipo que não implementa
`Send` ou `Sync`, como ponteiros brutos, e quisermos marcar esse tipo como `Send`
ou `Sync`, devemos usar `unsafe`. O Rust não pode verificar se nosso tipo cumpre
as garantias de que pode ser enviado com segurança entre threads ou acessado a
partir de múltiplas threads; portanto, precisamos fazer essas verificações
manualmente e indicar isso com `unsafe`.

### Acessando Campos de uma Union

A última ação que funciona apenas com `unsafe` é acessar campos de uma `union`.
Uma *union* (união) é semelhante a uma `struct`, mas apenas um campo declarado
é usado em uma instância específica de cada vez. As unions são usadas
principalmente para fazer interface com unions em código C. O acesso aos campos
da union é inseguro porque o Rust não pode garantir o tipo de dados atualmente
armazenado na instância da union. Você pode aprender mais sobre unions na
[Referência do Rust][unions].

### Usando o Miri para Verificar Código Inseguro

Ao escrever código inseguro, você pode querer verificar se o que você escreveu
é realmente seguro e correto. Uma das melhores maneiras de fazer isso é usar o
Miri, uma ferramenta oficial do Rust para detectar comportamento indefinido.
Enquanto o verificador de empréstimos é uma ferramenta _estática_ que funciona
em tempo de compilação, o Miri é uma ferramenta _dinâmica_ que funciona em tempo
de execução. Ele verifica seu código executando seu programa, ou seu conjunto
de testes, e detectando quando você viola as regras que ele entende sobre como o
Rust deve funcionar.

O uso do Miri requer uma versão *nightly* do Rust (sobre a qual falamos mais no
[Apêndice G: Como o Rust é Feito e o "Nightly Rust"][nightly]<!-- ignore -->).
Você pode instalar tanto uma versão *nightly* do Rust quanto a ferramenta Miri
digitando `rustup +nightly component add miri`. Isso não altera qual versão do
Rust seu projeto usa; apenas adiciona a ferramenta ao seu sistema para que você
possa usá-la quando quiser. Você pode executar o Miri em um projeto digitando
`cargo +nightly miri run` ou `cargo +nightly miri test`.

Para ter um exemplo de quão útil isso pode ser, considere o que acontece quando
o executamos contra a Listagem 20-7.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

O Miri nos avisa corretamente que estamos fazendo um *cast* de um inteiro para
um ponteiro, o que pode ser um problema, mas o Miri não consegue determinar se
existe um problema porque ele não sabe como o ponteiro se originou. Em
seguida, o Miri retorna um erro onde a Listagem 20-7 tem comportamento
indefinido porque temos um ponteiro pendente (*dangling pointer*). Graças ao
Miri, agora sabemos que há um risco de comportamento indefinido e podemos pensar
em como tornar o código seguro. Em alguns casos, o Miri pode até fazer
recomendações sobre como corrigir erros.

O Miri não pega tudo o que você pode errar ao escrever código inseguro. O Miri
é uma ferramenta de análise dinâmica, portanto, ele só detecta problemas com
código que realmente é executado. Isso significa que você precisará usá-lo em
conjunto com boas técnicas de teste para aumentar sua confiança sobre o código
inseguro que você escreveu. O Miri também não cobre todas as maneiras possíveis
de seu código estar incorreto (insound).

Dito de outra forma: Se o Miri _chegar a pegar_ um problema, você sabe que há um
bug, mas só porque o Miri _não pegou_ um bug não significa que não há nenhum
problema. Ele consegue pegar bastante coisa, no entanto. Tente executá-lo nos
outros exemplos de código inseguro neste capítulo e veja o que ele diz!

Você pode aprender mais sobre o Miri em [seu repositório no GitHub][miri].

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### Usando Código Inseguro Corretamente

Usar `unsafe` para usar um dos cinco superpoderes discutidos anteriormente não
é errado ou malvisto, mas é mais difícil acertar no código `unsafe` porque o
compilador não pode ajudar a manter a segurança de memória. Quando você tiver
um motivo para usar código `unsafe`, você pode fazê-lo, e ter a anotação
explícita `unsafe` torna mais fácil rastrear a origem dos problemas quando eles
ocorrem. Sempre que você escrever código inseguro, você pode usar o Miri para
ajudá-lo a ter mais certeza de que o código que você escreveu cumpre as regras
do Rust.

Para uma exploração muito mais profunda de como trabalhar efetivamente com o
Rust inseguro, leia o guia oficial do Rust para `unsafe`, [O Rustonomicon][nomicon].

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/