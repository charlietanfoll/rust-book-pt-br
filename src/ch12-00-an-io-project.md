# Um Projeto de E/S: Construindo um Programa de Linha de Comando

Este capítulo é uma recapitulação das muitas habilidades que você aprendeu até agora e uma exploração de alguns outros recursos da biblioteca padrão. Vamos construir uma ferramenta de linha de comando que interage com arquivos e entrada/saída de linha de comando para praticar alguns dos conceitos de Rust que você já domina.

A velocidade, segurança, saída de binário único e suporte multiplataforma do Rust o tornam uma linguagem ideal para a criação de ferramentas de linha de comando. Portanto, para o nosso projeto, faremos nossa própria versão da clássica ferramenta de busca de linha de comando `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint — busca globalmente por uma expressão regular e imprime). No caso de uso mais simples, o `grep` busca por uma string especificada em um arquivo especificado. Para fazer isso, o `grep` recebe como argumentos um caminho de arquivo e uma string. Em seguida, ele lê o arquivo, encontra as linhas nesse arquivo que contêm a string fornecida e imprime essas linhas.

Ao longo do caminho, mostraremos como fazer nossa ferramenta de linha de comando usar os recursos de terminal que muitas outras ferramentas de linha de comando usam. Vamos ler o valor de uma variável de ambiente para permitir que o usuário configure o comportamento da nossa ferramenta. Também imprimiremos mensagens de erro no fluxo do console de erro padrão (`stderr`) em vez da saída padrão (`stdout`) para que, por exemplo, o usuário possa redirecionar a saída bem-sucedida para um arquivo enquanto ainda vê as mensagens de erro na tela.

Um membro da comunidade Rust, Andrew Gallant, já criou uma versão muito rápida e completa do `grep`, chamada `ripgrep`. Em comparação, nossa versão será bastante simples, mas este capítulo fornecerá parte do conhecimento de fundo necessário para entender um projeto do mundo real como o `ripgrep`.

Nosso projeto `grep` combinará vários conceitos que você aprendeu até agora:

- Organização de código ([Capítulo 7][ch7]<!-- ignore -->)
- Uso de vetores e strings ([Capítulo 8][ch8]<!-- ignore -->)
- Tratamento de erros ([Capítulo 9][ch9]<!-- ignore -->)
- Uso de *traits* e tempos de vida onde apropriado ([Capítulo 10][ch10]<!-- ignore -->)
- Escrita de testes ([Capítulo 11][ch11]<!-- ignore -->)

Também apresentaremos brevemente *closures*, iteradores e objetos de *trait*, que os [Capítulos 13][ch13]<!-- ignore --> e [18][ch18]<!-- ignore --> cobrirão em detalhes.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
