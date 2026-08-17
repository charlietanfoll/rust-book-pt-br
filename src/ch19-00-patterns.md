# Padrões e Correspondência (Patterns and Matching)

Os padrões são uma sintaxe especial em Rust para corresponder à estrutura de
tipos, sejam eles complexos ou simples. Usar padrões em conjunto com expressões
`match` e outras construções oferece maior controle sobre o fluxo de execução
de um programa. Um padrão consiste em alguma combinação dos seguintes elementos:

- Literais
- Arrays, enums, structs ou tuplas desestruturados
- Variáveis
- Curingas (Wildcards)
- Espaços reservados (Placeholders)

Alguns exemplos de padrões incluem `x`, `(a, 3)` e `Some(Color::Red)`. Nos
contextos em que os padrões são válidos, esses componentes descrevem a forma dos
dados. Nosso programa então compara os valores com os padrões para determinar se
eles possuem a forma correta de dados para continuar executando um determinado
trecho de código.

Para usar um padrão, nós o comparamos com algum valor. Se o padrão corresponder
ao valor, nós usamos as partes do valor em nosso código. Lembre-se das
expressões `match` no Capítulo 6 que usavam padrões, como o exemplo da máquina de
classificação de moedas. Se o valor se encaixar na forma do padrão, podemos usar
as partes nomeadas. Se não se encaixar, o código associado ao padrão não será
executado.

Este capítulo é uma referência sobre tudo relacionado a padrões. Vamos abordar
os locais válidos para usar padrões, a diferença entre padrões refutáveis e
inrefutáveis, e os diferentes tipos de sintaxe de padrões que você pode
encontrar. Ao final do capítulo, você saberá como usar padrões para expressar
muitos conceitos de maneira clara.