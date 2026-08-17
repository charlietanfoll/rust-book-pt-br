# Recursos de Linguagem Funcional: Iteradores e Closures

O design do Rust foi inspirado em muitas linguagens e técnicas existentes, e uma influência significativa é a _programação funcional_. Programar em um estilo funcional frequentemente inclui o uso de funções como valores, passando-as como argumentos, retornando-as de outras funções, atribuindo-as a variáveis para execução posterior, e assim por diante.

Neste capítulo, não vamos debater a questão do que é ou não a programação funcional, mas sim discutir alguns recursos do Rust que são semelhantes a recursos em muitas linguagens frequentemente chamadas de funcionais.

Mais especificamente, vamos cobrir:

- _Closures_, uma construção semelhante a uma função que você pode armazenar em uma variável
- _Iterators_ (Iteradores), uma maneira de processar uma série de elementos
- Como usar closures e iteradores para melhorar o projeto de E/S (I/O) no Capítulo 12
- O desempenho de closures e iteradores (alerta de spoiler: eles são mais rápidos do que você imagina!)

Já cobrimos outros recursos do Rust, como pattern matching (casamento de padrões) e enums, que também são influenciados pelo estilo funcional. Como dominar closures e iteradores é uma parte importante da escrita de código Rust rápido e idiomático, dedicaremos este capítulo inteiro a eles.