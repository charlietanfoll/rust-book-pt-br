## Características das Linguagens Orientadas a Objetos

Não há consenso na comunidade de programação sobre quais recursos uma
linguagem deve ter para ser considerada orientada a objetos. O Rust é influenciado por muitos
paradigmas de programação, incluindo POO; por exemplo, exploramos os recursos
vindos da programação funcional no Capítulo 13. Pode-se dizer que as linguagens de POO
compartilham certas características comuns — a saber, objetos, encapsulamento e
herança. Vamos examinar o que cada uma dessas características significa e se
o Rust a suporta.

### Objetos Contêm Dados e Comportamento

O livro _Design Patterns: Elements of Reusable Object-Oriented Software_ de
Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides (Addison-Wesley,
1994), coloquialmente conhecido como o livro do _Gang of Four_ (Gangue dos Quatro), é um catálogo de
padrões de projeto orientados a objetos. Ele define a POO da seguinte forma:

> Programas orientados a objetos são feitos de objetos. Um **objeto** empacota tanto
> dados quanto os procedimentos que operam sobre esses dados. Os procedimentos são
> tipicamente chamados de **métodos** ou **operações**.

Usando essa definição, o Rust é orientado a objetos: Structs e enums têm dados,
e blocos `impl` fornecem métodos em structs e enums. Embora structs e
enums com métodos não sejam _chamados_ de objetos, eles fornecem a mesma
funcionalidade, de acordo com a definição de objetos da Gangue dos Quatro.

### Encapsulamento que Oculta Detalhes de Implementação

Outro aspecto comumente associado à POO é a ideia de _encapsulamento_,
o que significa que os detalhes de implementação de um objeto não são acessíveis ao
código que usa esse objeto. Portanto, a única maneira de interagir com um objeto é
através de sua API pública; o código que usa o objeto não deve ser capaz de acessar
os detalhes internos do objeto e alterar dados ou comportamento diretamente. Isso permite que o
programador altere e refatore os detalhes internos de um objeto sem precisar
alterar o código que usa o objeto.

Discutimos como controlar o encapsulamento no Capítulo 7: Podemos usar a palavra-chave `pub`
para decidir quais módulos, tipos, funções e métodos em nosso código
devem ser públicos, e por padrão todo o resto é privado. Por exemplo, podemos
definir uma struct `AveragedCollection` que possui um campo contendo um vetor
de valores `i32`. A struct também pode ter um campo que contém a média dos
valores no vetor, o que significa que a média não precisa ser calculada sob
demanda sempre que alguém precisar dela. Em outras palavras, `AveragedCollection`
vai armazenar em cache a média calculada para nós. A Listagem 18-1 contém a definição da
struct `AveragedCollection`.

<Listing number="18-1" file-name="src/lib.rs" caption="Uma struct `AveragedCollection` que mantém uma lista de inteiros e a média dos itens na coleção">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

A struct é marcada como `pub` para que outro código possa usá-la, mas os campos dentro
da struct permanecem privados. Isso é importante neste caso porque queremos
garantir que, sempre que um valor for adicionado ou removido da lista, a média
também seja atualizada. Fazemos isso implementando os métodos `add`, `remove` e
`average` na struct, conforme mostrado na Listagem 18-2.

<Listing number="18-2" file-name="src/lib.rs" caption="Implementações dos métodos públicos `add`, `remove` e `average` em `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Os métodos públicos `add`, `remove` e `average` são as únicas maneiras de acessar
ou modificar dados em uma instância de `AveragedCollection`. Quando um item é adicionado à
`list` usando o método `add` ou removido usando o método `remove`, as
implementações de cada um chamam o método privado `update_average` que lida com a
atualização do campo `average` também.

Deixamos os campos `list` e `average` privados para que não haja como o código externo
adicionar ou remover itens do campo `list` diretamente;
caso contrário, o campo `average` pode ficar dessincronizado quando a `list`
mudar. O método `average` retorna o valor no campo `average`,
permitindo que o código externo leia a `average`, mas não a modifique.

Como encapsulamos os detalhes de implementação da struct
`AveragedCollection`, podemos alterar facilmente aspectos, como a estrutura de dados,
no futuro. Por exemplo, poderíamos usar um `HashSet<i32>` em vez de um
`Vec<i32>` para o campo `list`. Desde que as assinaturas dos métodos públicos `add`,
`remove` e `average` permaneçam as mesmas, o código que usa
`AveragedCollection` não precisará mudar. Se tornássemos `list` pública em vez disso,
esse não seria necessariamente o caso: `HashSet<i32>` e `Vec<i32>` têm
diferentes métodos para adicionar e remover itens, então o código externo
provavelmente teria que mudar se estivesse modificando `list` diretamente.

Se o encapsulamento for um aspecto obrigatório para que uma linguagem seja considerada orientada
a objetos, então o Rust cumpre esse requisito. A opção de usar `pub` ou não para
diferentes partes do código permite o encapsulamento de detalhes de implementação.

### Herança como um Sistema de Tipos e como Compartilhamento de Código

A _herança_ é um mecanismo pelo qual um objeto pode herdar elementos da
definição de outro objeto, ganhando assim os dados e o comportamento do objeto pai
sem que você precise defini-los novamente.

Se uma linguagem precisa ter herança para ser orientada a objetos, então o Rust não
é tal linguagem. Não há maneira de definir uma struct que herde os campos e
as implementações de métodos da struct pai sem usar uma macro.

No entanto, se você está acostumado a ter herança em sua caixa de ferramentas de programação, você
pode usar outras soluções em Rust, dependendo do seu motivo para buscar
herança em primeiro lugar.

Você escolheria a herança por dois motivos principais. Um é para reutilização de código:
Você pode implementar um comportamento específico para um tipo, e a herança permite que você
reutilize essa implementação para um tipo diferente. Você pode fazer isso de forma limitada
no código Rust usando implementações de métodos de trait padrão, que você viu na
Listagem 10-14 quando adicionamos uma implementação padrão do método `summarize`
no trait `Summary`. Qualquer tipo que implemente o trait `Summary` terá
o método `summarize` disponível nele sem nenhum código adicional. Isso é
semelhante a uma classe pai ter a implementação de um método e uma
classe filha herdeira também ter a implementação desse método. Também podemos
sobrescrever a implementação padrão do método `summarize` quando implementamos
o trait `Summary`, o que é semelhante a uma classe filha sobrescrevendo a
implementação de um método herdado de uma classe pai.

O outro motivo para usar herança está relacionado ao sistema de tipos: permitir que um
tipo filho seja usado nos mesmos locais que o tipo pai. Isso também
é chamado de _polimorfismo_, o que significa que você pode substituir vários objetos
uns pelos outros em tempo de execução se eles compartilharem certas características.

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinônimo de herança. Mas é
> na verdade um conceito mais geral que se refere a código que pode trabalhar com dados de
> múltiplos tipos. Para a herança, esses tipos são geralmente subclasses.
>
> O Rust em vez disso usa genéricos para abstrair sobre diferentes tipos possíveis e
> limites de trait (trait bounds) para impor restrições sobre o que esses tipos devem fornecer. Isso é
> às vezes chamado de _polimorfismo paramétrico delimitado_.

O Rust escolheu um conjunto diferente de concessões (trade-offs) ao não oferecer herança.
A herança está frequentemente em risco de compartilhar mais código do que o necessário.
As subclasses nem sempre devem compartilhar todas as características de sua classe pai, mas farão isso
com a herança. Isso pode tornar o design de um programa menos flexível. Também
introduz a possibilidade de chamar métodos em subclasses que não fazem
sentido ou que causam erros porque os métodos não se aplicam à subclasse. Além
disso, algumas linguagens permitem apenas _herança simples_ (o que significa que uma
subclasse só pode herdar de uma classe), restringindo ainda mais a flexibilidade
do design de um programa.

Por essas razões, o Rust adota a abordagem diferente de usar objetos de trait (trait objects)
em vez de herança para alcançar o polimorfismo em tempo de execução. Vamos examinar como
os objetos de trait funcionam.