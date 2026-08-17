# Ponteiros Inteligentes

Um ponteiro é um conceito geral para uma variável que contém um endereço na
memória. Esse endereço se refere a, ou "aponta para", algum outro dado. O tipo
mais comum de ponteiro em Rust é a referência, sobre a qual você aprendeu no
Capítulo 4. As referências são indicadas pelo símbolo `&` e emprestam o valor
para o qual apontam. Elas não têm nenhuma capacidade especial além de se
referir a dados, e não possuem custo adicional (*overhead*).

Os _ponteiros inteligentes_ (*smart pointers*), por outro lado, são estruturas de
dados que agem como um ponteiro, mas também possuem metadados e capacidades
adicionais. O conceito de ponteiros inteligentes não é exclusivo do Rust: os
ponteiros inteligentes originaram-se em C++ e existem em outras linguagens
também. O Rust possui uma variedade de ponteiros inteligentes definidos na
biblioteca padrão que fornecem funcionalidades além daquelas fornecidas pelas
referências. Para explorar o conceito geral, veremos alguns exemplos
diferentes de ponteiros inteligentes, incluindo um tipo de ponteiro inteligente
de _contagem de referências_. Esse ponteiro permite que você permita que os
dados tenham múltiplos proprietários, rastreando o número de proprietários e,
quando não restarem proprietários, limpando os dados.

No Rust, com seu conceito de propriedade e empréstimo, há uma diferença
adicional entre referências e ponteiros inteligentes: enquanto as referências
apenas emprestam dados, em muitos casos os ponteiros inteligentes _possuem_
(`own`) os dados para os quais apontam.

Ponteiros inteligentes são geralmente implementados usando structs. Ao
contrário de uma struct comum, os ponteiros inteligentes implementam os traits
`Deref` e `Drop`. O trait `Deref` permite que uma instância da struct do
ponteiro inteligente se comporte como uma referência para que você possa
escrever seu código para funcionar com referências ou ponteiros inteligentes.
O trait `Drop` permite que você personalize o código que é executado quando uma
instância do ponteiro inteligente sai de escopo. Neste capítulo, discutiremos
ambos os traits e demonstraremos por que eles são importantes para os ponteiros
inteligentes.

Dado que o padrão de ponteiro inteligente é um padrão de projeto geral usado
frequentemente em Rust, este capítulo não cobrirá todos os ponteiros
inteligentes existentes. Muitas bibliotecas têm seus próprios ponteiros
inteligentes, e você pode até escrever os seus próprios. Cobriremos os ponteiros
inteligentes mais comuns na biblioteca padrão:

- `Box<T>`, para alocar valores no *heap*
- `Rc<T>`, um tipo de contagem de referências que permite múltiplos proprietários
- `Ref<T>` e `RefMut<T>`, acessados através de `RefCell<T>`, um tipo que impõe
  as regras de empréstimo em tempo de execução em vez de em tempo de compilação

Além disso, cobriremos o padrão de _mutabilidade interior_ (*interior mutability*),
no qual um tipo imutável expõe uma API para mutar um valor interno. Também
discutiremos ciclos de referência: como eles podem vazar memória e como evitá-los.

Vamos mergulhar no assunto!