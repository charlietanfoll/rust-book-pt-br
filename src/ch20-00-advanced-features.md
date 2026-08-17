# Recursos Avançados

Até agora, você aprendeu as partes mais comumente usadas da linguagem de programação
Rust. Antes de fazermos mais um projeto, no Capítulo 21, veremos alguns
aspectos da linguagem que você pode encontrar de vez em quando, mas que talvez não
use todos os dias. Você pode usar este capítulo como referência para quando encontrar
qualquer coisa desconhecida. Os recursos abordados aqui são úteis em situações muito específicas.
Embora você possa não recorrer a eles com frequência, queremos garantir que você tenha
uma compreensão de todos os recursos que o Rust tem a oferecer.

Neste capítulo, abordaremos:

- Rust inseguro (*Unsafe Rust*): Como abrir mão de algumas das garantias do Rust e assumir
  a responsabilidade de manter manualmente essas garantias
- Traits avançadas: Tipos associados, parâmetros de tipo padrão, sintaxe
  totalmente qualificada, *supertraits* e o padrão *newtype* em relação a traits
- Tipos avançados: Mais sobre o padrão *newtype*, apelidos de tipos (*type aliases*), o tipo *never* e tipos de tamanho dinâmico
- Funções e *closures* avançadas: Ponteiros de função e retorno de *closures*
- Macros: Maneiras de definir código que define mais código em tempo de compilação

É uma variedade de recursos do Rust com algo para todos! Vamos mergulhar de cabeça!