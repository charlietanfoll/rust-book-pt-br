## Dar `panic!` ou Não Dar `panic!`

Então, como você decide quando deve chamar `panic!` e quando deve retornar
`Result`? Quando o código entra em pânico (`panics`), não há maneira de se recuperar. Você poderia chamar `panic!`
para qualquer situação de erro, haja ou não uma maneira possível de se recuperar, mas
nesse caso você está tomando a decisão de que uma situação é irrecuperável em nome do
código chamador. Quando você escolhe retornar um valor `Result`, você dá opções ao
código chamador. O código chamador pode escolher tentar se recuperar de
uma maneira apropriada para a sua situação, ou pode decidir que um valor `Err`
neste caso é irrecuperável, podendo assim chamar `panic!` e transformar o seu erro
recuperável em um irrecuperável. Portanto, retornar `Result` é uma
boa escolha padrão quando você está definindo uma função que pode falhar.

Em situações como exemplos, código de protótipo e testes, é mais
apropriado escrever código que entra em pânico em vez de retornar um `Result`. Vamos
explorar o motivo e, em seguida, discutir situações em que o compilador não consegue dizer que a
falha é impossível, mas você, como ser humano, consegue. O capítulo concluirá com
algumas diretrizes gerais sobre como decidir se deve causar um pânico (`panic`) no código de biblioteca.

### Exemplos, Código de Protótipo e Testes

Quando você está escrevendo um exemplo para ilustrar algum conceito, incluir também
código robusto de tratamento de erros pode tornar o exemplo menos claro. Em exemplos, entende-se
que uma chamada a um método como `unwrap`, que pode entrar em pânico, serve como
um espaço reservado para a maneira como você gostaria que sua aplicação lidasse com erros, o que pode
diferir com base no que o resto do seu código está fazendo.

Da mesma forma, os métodos `unwrap` e `expect` são muito úteis quando você está
prototipando e ainda não está pronto para decidir como lidar com erros. Eles deixam
marcadores claros no seu código para quando você estiver pronto para tornar seu programa mais
robusto.

Se a chamada de um método falhar em um teste, você vai querer que o teste inteiro falhe, mesmo que
esse método não seja a funcionalidade sob teste. Como `panic!` é a forma como um teste
é marcado como falho, chamar `unwrap` ou `expect` é exatamente o que deve
acontecer.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Quando Você Tem Mais Informações do que o Compilador

Também seria apropriado chamar `expect` quando você tiver alguma outra lógica
que garanta que o `Result` terá um valor `Ok`, mas a lógica não for algo
que o compilador entende. Você ainda terá um valor `Result` com o qual
precisa lidar: Qualquer operação que você esteja chamando ainda tem a possibilidade de
falhar em geral, mesmo que seja logicamente impossível na sua situação específica. Se você puder garantir por inspeção manual do código que nunca terá
uma variante `Err`, é perfeitamente aceitável chamar `expect` e documentar
o motivo pelo qual você acha que nunca terá uma variante `Err` no texto do argumento.
Aqui está um exemplo:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Estamos criando uma instância de `IpAddr` analisando (*parsing*) uma string fixa no código. Podemos ver
que `127.0.0.1` é um endereço IP válido, então é aceitável usar `expect`
aqui. No entanto, ter uma string válida fixada no código não altera o tipo de retorno
do método `parse`: Nós ainda recebemos um valor `Result`, e o compilador ainda
nos obrigará a lidar com o `Result` como se a variante `Err` fosse uma possibilidade,
porque o compilador não é inteligente o suficiente para ver que esta string é sempre um
endereço IP válido. Se a string do endereço IP viesse de um usuário em vez de estar
fixada no código do programa e, portanto, _tivesse_ uma possibilidade de falha,
nós definitivamente gostaríamos de lidar com o `Result` de uma forma mais robusta.
Mencionar a suposição de que este endereço IP está fixado no código nos incitará a
alterar o `expect` para um código de tratamento de erros melhor se, no futuro, precisarmos obter
o endereço IP de alguma outra fonte.

### Diretrizes para Tratamento de Erros

É aconselhável fazer seu código entrar em pânico quando for possível que o seu código
acerbe em um estado ruim. Neste contexto, um _estado ruim_ é quando alguma suposição,
garantia, contrato ou invariante foi quebrada, como quando valores inválidos,
valores contraditórios ou valores ausentes são passados para o seu código — além de uma ou
mais das seguintes condições:

- O estado ruim é algo inesperado, em oposição a algo que
  provavelmente acontecerá ocasionalmente, como um usuário inserindo dados no formato
  errado.
- Seu código a partir deste ponto precisa depender de não estar neste estado ruim,
  em vez de verificar o problema a cada passo.
- Não há uma boa maneira de codificar essa informação nos tipos que você usa. Vamos
  trabalhar em um exemplo do que queremos dizer em [“Codificando Estados e Comportamento como Tipos”][encoding]<!-- ignore --> no Capítulo 18.

Se alguém chama seu código e passa valores que não fazem sentido, é
melhor retornar um erro, se possível, para que o usuário da biblioteca possa decidir
o que deseja fazer nesse caso. No entanto, em casos onde continuar pode ser
inseguro ou prejudicial, a melhor escolha pode ser chamar `panic!` e alertar a
pessoa que usa sua biblioteca sobre o bug em seu código para que ela possa corrigi-lo
durante o desenvolvimento. Da mesma forma, `panic!` geralmente é apropriado se você estiver chamando
código externo que está fora do seu controle e retorna um estado inválido que você
não tem como corrigir.

No entanto, quando uma falha é esperada, é mais apropriado retornar um `Result`
do que fazer uma chamada a `panic!`. Exemplos incluem um *parser* recebendo dados malformados
ou uma requisição HTTP retornando um status que indica que você atingiu um limite de taxa (*rate limit*). Nestes casos, retornar um `Result` indica que a falha é uma
possibilidade esperada que o código chamador deve decidir como lidar.

Quando o seu código realiza uma operação que pode colocar um usuário em risco se for
chamada usando valores inválidos, seu código deve verificar primeiro se os valores são válidos
e entrar em pânico caso não sejam. Isso é principalmente por razões de segurança:
Tentar operar em dados inválidos pode expor seu código a vulnerabilidades.
Esta é a principal razão pela qual a biblioteca padrão chamará `panic!` se você tentar
um acesso à memória fora dos limites (*out-of-bounds*): Tentar acessar memória que não pertence à
estrutura de dados atual é um problema de segurança comum. As funções frequentemente têm
_contratos_: seu comportamento só é garantido se as entradas atenderem a requisitos específicos.
Entrar em pânico quando o contrato é violado faz sentido porque uma violação de contrato
sempre indica um bug do lado do chamador, e não é o tipo de erro que você quer que o código
chamador tenha que tratar explicitamente. De fato, não há maneira razoável para o código chamador
se recuperar; os _programadores_ chamadores precisam corrigir o código. Os contratos de uma
função, especialmente quando uma violação causa um pânico, devem ser explicados na
documentação da API da função.

No entanto, ter muitas verificações de erro em todas as suas funções seria prolixo
e irritante. Felizmente, você pode usar o sistema de tipos do Rust (e, portanto, a
verificação de tipos feita pelo compilador) para fazer muitas das verificações para você. Se a sua
função tem um determinado tipo como parâmetro, você pode prosseguir com a lógica do seu código
sabendo que o compilador já garantiu que você tem um valor válido. Por exemplo, se você tem um tipo em vez de um `Option`, seu programa
espera ter _algo_ em vez de _nada_. Seu código então não precisa lidar com dois casos para as variantes `Some` e `None`: Ele terá apenas um caso para ter um valor com certeza. O código que tentar passar nada para a sua
função nem sequer compilará, de modo que sua função não precisa verificar esse
caso em tempo de execução. Outro exemplo é usar um tipo de inteiro sem sinal como
`u32`, o que garante que o parâmetro nunca seja negativo.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Tipos Personalizados para Validação

Vamos levar a ideia de usar o sistema de tipos do Rust para garantir que temos um valor
válido um passo além e analisar a criação de um tipo personalizado para validação.
Lembre-se do jogo de adivinhação do Capítulo 2, em que nosso código pedia ao usuário para adivinhar
um número entre 1 e 100. Nunca validamos se o palpite do usuário estava
entre esses números antes de compará-lo com o nosso número secreto; nós apenas
validamos se o palpite era positivo. Nesse caso, as consequências não foram
muito graves: Nossa saída de “Muito alto” ou “Muito baixo” ainda estaria correta. Mas seria
uma melhoria útil guiar o usuário em direção a palpites válidos e ter um comportamento
diferente quando o usuário adivinha um número fora do intervalo em comparação
a quando o usuário digita, por exemplo, letras.

Uma maneira de fazer isso seria analisar o palpite como um `i32` em vez de apenas um
`u32` para permitir números potencialmente negativos, e então adicionar uma verificação para o
número estar dentro do intervalo, assim:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

A expressão `if` verifica se o nosso valor está fora do intervalo, avisa o usuário
sobre o problema e chama `continue` para iniciar a próxima iteração do loop
e pedir outro palpite. Após a expressão `if`, podemos prosseguir com as
comparações entre `guess` e o número secreto sabendo que `guess` está
entre 1 e 100.

No entanto, esta não é uma solução ideal: Se fosse absolutamente crucial que o
programa operasse apenas com valores entre 1 e 100, e ele tivesse muitas funções
com esse requisito, ter uma verificação como essa em cada função seria
tedioso (e poderia impactar o desempenho).

Em vez disso, podemos criar um novo tipo em um módulo dedicado e colocar as validações
em uma função para criar uma instância do tipo, em vez de repetir as
validações em todos os lugares. Dessa forma, é seguro para as funções usar o novo tipo
em suas assinaturas e usar com confiança os valores que recebem. A Listagem 9-13
mostra uma maneira de definir um tipo `Guess` que só criará uma instância de
`Guess` se a função `new` receber um valor entre 1 e 100.

<Listing number="9-13" caption="Um tipo `Guess` que só continuará com valores entre 1 e 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Note que este código em *src/guessing_game.rs* depende da adição de uma declaração de módulo
`mod guessing_game;` em *src/lib.rs* que não mostramos aqui.
Dentro do arquivo deste novo módulo, definimos uma estrutura (`struct`) chamada `Guess` que tem um
campo chamado `value` que armazena um `i32`. É aqui que o número será
armazenado.

Em seguida, implementamos uma função associada chamada `new` em `Guess` que cria
instâncias de valores `Guess`. A função `new` é definida para ter um
parâmetro chamado `value` do tipo `i32` e retornar um `Guess`. O código no
corpo da função `new` testa o `value` para ter certeza de que ele está entre 1 e 100.
Se o `value` não passar neste teste, fazemos uma chamada a `panic!`, o que alertará
o programador que está escrevendo o código chamador de que ele tem um bug que precisa
corrigir, porque criar um `Guess` com um `value` fora desse intervalo violaria
o contrato no qual `Guess::new` está confiando. As condições sob as quais
`Guess::new` pode entrar em pânico devem ser discutidas na documentação de sua API pública;
vamos abordar convenções de documentação que indicam a possibilidade de um `panic!`
na documentação da API que você criar no Capítulo 14. Se
`value` passar no teste, criamos um novo `Guess` com seu campo `value` definido
como o parâmetro `value` e retornamos o `Guess`.

Em seguida, implementamos um método chamado `value` que empresta (`borrows`) `self`, não tem
quaisquer outros parâmetros e retorna um `i32`. Esse tipo de método às vezes é chamado
de _getter_ (leitor), porque seu propósito é obter alguns dados de seus campos e retorná-los.
Este método público é necessário porque o campo `value` da estrutura `Guess`
é privado. É importante que o campo `value` seja privado para que o código que usa
a estrutura `Guess` não tenha permissão para definir `value` diretamente: O código
fora do módulo `guessing_game` _deve_ usar a função `Guess::new` para
criar uma instância de `Guess`, garantindo assim que não há como um
`Guess` ter um `value` que não tenha sido verificado pelas condições na
função `Guess::new`.

Uma função que tem um parâmetro ou retorna apenas números entre 1 e 100 pode
então declarar em sua assinatura que recebe ou retorna um `Guess` em vez de um
`i32` e não precisaria fazer nenhuma verificação adicional em seu corpo.

## Resumo

Os recursos de tratamento de erros do Rust são projetados para ajudá-lo a escrever código mais robusto.
A macro `panic!` sinaliza que seu programa está em um estado que ele não pode manipular e
permite que você diga para o processo parar em vez de tentar prosseguir com valores inválidos ou
incorretos. O enum `Result` usa o sistema de tipos do Rust para indicar que
operações podem falhar de uma forma da qual seu código pode se recuperar. Você pode usar
`Result` para dizer ao código que chama o seu que ele precisa lidar com o sucesso ou a falha potenciais
também. Usar `panic!` e `Result` nas situações apropriadas tornará seu código
mais confiável diante de problemas inevitáveis.

Agora que você viu maneiras úteis pelas quais a biblioteca padrão usa genéricos com
os enums `Option` e `Result`, falaremos sobre como os genéricos funcionam e como você
pode usá-los no seu código.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
