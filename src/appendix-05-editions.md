## Apêndice E: Edições

No Capítulo 1, você viu que o comando `cargo new` adiciona um pouco de metadados ao seu
arquivo _Cargo.toml_ sobre uma edição. Este apêndice fala sobre o que isso significa!

A linguagem Rust e seu compilador possuem um ciclo de lançamento de seis semanas, o que significa que os usuários recebem
um fluxo constante de novos recursos. Outras linguagens de programação lançam mudanças maiores
com menos frequência; o Rust lança atualizações menores com mais frequência. Depois de um
tempo, todas essas pequenas mudanças se acumulam. Mas, de um lançamento para outro, pode ser
difícil olhar para trás e dizer: "Nossa, entre o Rust 1.10 e o Rust 1.31, o Rust
mudou muito!"

A cada três anos, mais ou menos, a equipe do Rust produz uma nova _edição_ do Rust. Cada
edição reúne os recursos que foram implementados em um pacote claro, com
documentação e ferramentas totalmente atualizadas. As novas edições são lançadas como parte do
processo normal de lançamento de seis semanas.

As edições servem a propósitos diferentes para pessoas diferentes:

- Para usuários ativos do Rust, uma nova edição reúne mudanças incrementais em
  um pacote fácil de entender.
- Para não-usuários, uma nova edição sinaliza que grandes avanços foram
  implementados, o que pode fazer com que valha a pena dar uma nova olhada no Rust.
- Para aqueles que desenvolvem o Rust, uma nova edição fornece um ponto de união
  para o projeto como um todo.

No momento em que este livro foi escrito, quatro edições do Rust estavam disponíveis: Rust 2015, Rust
2018, Rust 2021 e Rust 2024. Este livro foi escrito usando os
idiomas da edição Rust 2024.

A chave `edition` no arquivo _Cargo.toml_ indica qual edição o compilador deve
usar para o seu código. Se a chave não existir, o Rust usa `2015` como o valor
da edição por motivos de compatibilidade com versões anteriores.

Cada projeto pode optar por usar uma edição diferente da edição padrão de 2015.
As edições podem conter alterações incompatíveis, como a inclusão de uma nova palavra-chave que
entra em conflito com identificadores no código. No entanto, a menos que você opte por essas
mudanças, seu código continuará a compilar mesmo quando você atualizar a versão do
compilador Rust que utiliza.

Todas as versões do compilador Rust oferecem suporte a qualquer edição que existia antes do lançamento
desse compilador, e elas podem vincular crates de quaisquer edições suportadas.
As mudanças de edição afetam apenas a forma como o compilador analisa inicialmente o
código. Portanto, se você estiver usando o Rust 2015 e uma de suas dependências usar o
Rust 2018, seu projeto compilará e poderá usar essa dependência. A
situação oposta, em que seu projeto usa o Rust 2018 e uma dependência usa o
Rust 2015, funciona da mesma forma.

Para deixar claro: a maioria dos recursos estará disponível em todas as edições. Os desenvolvedores que usam
qualquer edição do Rust continuarão a ver melhorias à medida que novos lançamentos estáveis
forem feitos. No entanto, em alguns casos, principalmente quando novas palavras-chave são adicionadas,
alguns novos recursos podem estar disponíveis apenas em edições posteriores. Você precisará alterar
as edições se quiser aproveitar esses recursos.

Para mais detalhes, consulte [_The Rust Edition Guide_][edition-guide] (Guia de Edições do Rust). Este é um
livro completo que enumera as diferenças entre as edições e explica como
atualizar automaticamente o seu código para uma nova edição usando o comando `cargo fix`.

[edition-guide]: https://doc.rust-lang.org/stable/edition-guide
