# Enums e Correspondência de Padrões (Pattern Matching)

Neste capítulo, veremos as enumerações, também chamadas de _enums_.
As enums permitem que você defina um tipo enumerando suas variantes possíveis. Primeiro,
definiremos e usaremos uma enum para mostrar como ela pode codificar significado junto com
dados. Em seguida, exploraremos uma enum particularmente útil, chamada `Option`, que
expressa que um valor pode ser algo ou nada. Depois, veremos
como a correspondência de padrões na expressão `match` facilita a execução de códigos
diferentes para valores diferentes de uma enum. Por fim, abordaremos como a construção
`if let` é outro idioma conveniente e conciso disponível para lidar com enums no
seu código.