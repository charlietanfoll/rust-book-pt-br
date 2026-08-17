# Coleções Comuns

A biblioteca padrão do Rust inclui várias estruturas de dados muito úteis
chamadas _coleções_. A maioria dos outros tipos de dados representa um valor
específico, mas as coleções podem conter múltiplos valores. Ao contrário dos
tipos de matriz (array) e tupla embutidos, os dados para os quais essas coleções
apontam são armazenados no *heap*, o que significa que a quantidade de dados
não precisa ser conhecida em tempo de compilação e pode crescer ou diminuir à
medida que o programa é executado. Cada tipo de coleção tem diferentes
capacidades e custos, e escolher a mais adequada para a sua situação atual é
uma habilidade que você desenvolverá com o tempo. Neste capítulo, discutiremos
três coleções que são usadas com muita frequência em programas Rust:

- Um _vector_ (vetor) permite armazenar um número variável de valores lado a lado.
- Uma _string_ é uma coleção de caracteres. Já mencionamos o tipo `String`
  anteriormente, mas neste capítulo falaremos sobre ele em profundidade.
- Um _hash map_ (mapa hash) permite associar um valor a uma chave específica. É
  uma implementação particular da estrutura de dados mais geral chamada
  _map_.

Para saber sobre os outros tipos de coleções fornecidos pela biblioteca padrão,
consulte [a documentação][collections].

Discutiremos como criar e atualizar vetores, strings e mapas hash, bem como o
que torna cada um deles especial.

[collections]: ../std/collections/index.html
