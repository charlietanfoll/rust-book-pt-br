## Estendendo o Cargo com Comandos Personalizados

O Cargo foi projetado para permitir que você o estenda com novos subcomandos sem
precisar modificá-lo. Se um binário no seu `$PATH` se chamar `cargo-algo`, você
poderá executá-lo como se fosse um subcomando do Cargo executando `cargo algo`.
Comandos personalizados como esse também são listados quando você executa
`cargo --list`. Poder usar o `cargo install` para instalar extensões e executá-las
exatamente como as ferramentas integradas do Cargo é um benefício super conveniente
do design do Cargo!

## Resumo

Compartilhar código com o Cargo e o [crates.io](https://crates.io/)<!-- ignore --> é
parte do que torna o ecossistema Rust útil para muitas tarefas diferentes. A biblioteca
padrão do Rust é pequena e estável, mas os crates são fáceis de compartilhar, usar e
melhorar em um cronograma diferente daquele da linguagem. Não tenha vergonha de
compartilhar código que seja útil para você no [crates.io](https://crates.io/)<!-- ignore
-->; é provável que seja útil para outra pessoa também!