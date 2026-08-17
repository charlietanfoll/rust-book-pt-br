# Introdução

> Nota: Esta edição do livro é a mesma de [The Rust Programming
> Language][nsprust] disponível em formato impresso e e-book pela [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition
[nsp]: https://nostarch.com/

Bem-vindo ao _The Rust Programming Language_ (A Linguagem de Programação Rust), um livro introdutório sobre Rust.
A linguagem de programação Rust ajuda você a escrever software mais rápido e confiável.
Ergonomia de alto nível e controle de baixo nível estão frequentemente em conflito no design de linguagens de programação; Rust desafia esse conflito. Ao equilibrar uma capacidade técnica poderosa com uma excelente experiência de desenvolvimento, Rust oferece a opção de controlar detalhes de baixo nível (como o uso de memória) sem todo o trabalho tradicionalmente associado a esse controle.

## Para quem é o Rust

Rust é ideal para muitas pessoas por vários motivos. Vamos dar uma olhada em alguns dos grupos mais importantes.

### Equipes de Desenvolvedores

O Rust está se provando uma ferramenta produtiva para colaboração entre grandes equipes de desenvolvedores com diferentes níveis de conhecimento em programação de sistemas. Código de baixo nível é propenso a vários bugs sutis, que na maioria das outras linguagens só podem ser detectados por meio de testes extensivos e revisões de código cuidadosas por desenvolvedores experientes. Em Rust, o compilador desempenha um papel de guardião ao recusar a compilação de código com esses bugs elusivos, incluindo bugs de concorrência. Ao trabalhar junto com o compilador, a equipe pode gastar seu tempo focando na lógica do programa em vez de correr atrás de bugs.

Rust também traz ferramentas de desenvolvimento contemporâneas para o mundo da programação de sistemas:

- O Cargo, o gerenciador de dependências e ferramenta de build inclusos, torna a adição, compilação e gerenciamento de dependências fáceis e consistentes em todo o ecossistema Rust.
- A ferramenta de formatação `rustfmt` garante um estilo de código consistente entre os desenvolvedores.
- O Rust Language Server alimenta a integração com ambientes de desenvolvimento integrado (IDE) para conclusão de código e mensagens de erro em tempo real.

Ao usar essas e outras ferramentas do ecossistema Rust, os desenvolvedores podem ser produtivos ao escrever código em nível de sistema.

### Estudantes

Rust é para estudantes e para aqueles que têm interesse em aprender sobre conceitos de sistemas. Usando Rust, muitas pessoas aprenderam sobre tópicos como desenvolvimento de sistemas operacionais. A comunidade é muito acolhedora e feliz em responder às perguntas dos estudantes. Por meio de esforços como este livro, as equipes do Rust querem tornar os conceitos de sistemas mais acessíveis a mais pessoas, especialmente aquelas novas na programação.

### Empresas

Centenas de empresas, grandes e pequenas, usam Rust em produção para uma variedade de tarefas, incluindo ferramentas de linha de comando, serviços web, ferramentas de DevOps, dispositivos embarcados, análise e transcodificação de áudio e vídeo, criptomoedas, bioinformática, mecanismos de busca, aplicações de Internet das Coisas (IoT), aprendizado de máquina e até mesmo partes principais do navegador web Firefox.

### Desenvolvedores de Código Aberto

Rust é para pessoas que querem construir a linguagem de programação Rust, sua comunidade, ferramentas de desenvolvimento e bibliotecas. Adoraríamos que você contribuísse para a linguagem Rust.

### Pessoas que Valorizam Velocidade e Estabilidade

Rust é para pessoas que anseiam por velocidade e estabilidade em uma linguagem. Por velocidade, queremos dizer tanto a rapidez com que o código Rust pode ser executado quanto a velocidade com que o Rust permite que você escreva programas. As verificações do compilador do Rust garantem estabilidade durante a adição de recursos e refatoração. Isso contrasta com o código legado frágil em linguagens sem essas verificações, que os desenvolvedores frequentemente têm medo de modificar. Ao buscar abstrações de custo zero — recursos de nível superior que compilam para código de nível inferior tão rápido quanto código escrito manualmente —, o Rust se esforça para garantir que o código seguro também seja um código rápido.

A linguagem Rust espera apoiar muitos outros usuários também; aqueles mencionados aqui são apenas alguns dos maiores interessados. No geral, a maior ambição do Rust é eliminar os compromissos que os programadores aceitam há décadas, fornecendo segurança _e_ produtividade, velocidade _e_ ergonomia. Dê uma chance ao Rust e veja se as escolhas dele funcionam para você.

## Para quem é este livro

Este livro assume que você já escreveu código em outra linguagem de programação, mas não faz nenhuma suposição sobre qual. Tentamos tornar o material amplamente acessível para pessoas de uma ampla variedade de origens de programação. Não gastamos muito tempo falando sobre o que _é_ programação ou como pensar sobre ela. Se você é totalmente novo na programação, será melhor atendido pela leitura de um livro que forneça especificamente uma introdução à programação.

## Como usar este livro

Em geral, este livro assume que você o está lendo em sequência, da frente para trás. Os capítulos posteriores baseiam-se em conceitos dos capítulos anteriores, e os capítulos anteriores podem não aprofundar os detalhes de um tópico específico, mas o revisitarão em um capítulo posterior.

Você encontrará dois tipos de capítulos neste livro: capítulos de conceitos e capítulos de projetos. Nos capítulos de conceitos, você aprenderá sobre um aspecto do Rust. Nos capítulos de projetos, construiremos pequenos programas juntos, aplicando o que você aprendeu até agora. O Capítulo 2, o Capítulo 12 e o Capítulo 21 são capítulos de projetos; o restante são capítulos de conceitos.

O **Capítulo 1** explica como instalar o Rust, como escrever um programa "Olá, mundo!" e como usar o Cargo, o gerenciador de pacotes e ferramenta de build do Rust. O **Capítulo 2** é uma introdução prática à escrita de um programa em Rust, fazendo com que você construa um jogo de adivinhação de números. Aqui, cobrimos conceitos em alto nível, e capítulos posteriores fornecerão detalhes adicionais. Se você quiser colocar a mão na massa imediatamente, o Capítulo 2 é o lugar para isso. Se você for um aprendiz particularmente meticuloso que prefere aprender cada detalhe antes de passar para o próximo, pode querer pular o Capítulo 2 e ir direto para o **Capítulo 3**, que aborda recursos do Rust semelhantes aos de outras linguagens de programação; então, você pode retornar ao Capítulo 2 quando quiser trabalhar em um projeto aplicando os detalhes que aprendeu.

No **Capítulo 4**, você aprenderá sobre o sistema de propriedade (*ownership*) do Rust. O **Capítulo 5** discute estruturas (*structs*) e métodos. O **Capítulo 6** aborda enums, expressões `match` e as construções de fluxo de controle `if let` e `let...else`. Você usará estruturas e enums para criar tipos personalizados.

No **Capítulo 7**, você aprenderá sobre o sistema de módulos do Rust e sobre as regras de privacidade para organizar seu código e sua interface de programação de aplicativos (API) pública. O **Capítulo 8** discute algumas estruturas de dados de coleções comuns que a biblioteca padrão fornece: vetores (*vectors*), strings e mapas de hash (*hash maps*). O **Capítulo 9** explora a filosofia e as técnicas de tratamento de erros do Rust.

O **Capítulo 10** mergulha em genéricos (*generics*), traits e tempos de vida (*lifetimes*), que dão a você o poder de definir código que se aplica a múltiplos tipos. O **Capítulo 11** trata de testes, que, mesmo com as garantias de segurança do Rust, são necessários para garantir que a lógica do seu programa esteja correta. No **Capítulo 12**, construiremos nossa própria implementação de um subconjunto de funcionalidades da ferramenta de linha de comando `grep` que busca texto dentro de arquivos. Para isso, usaremos muitos dos conceitos que discutimos nos capítulos anteriores.

O **Capítulo 13** explora closures e iteradores: recursos do Rust que vêm de linguagens de programação funcional. No **Capítulo 14**, examinaremos o Cargo mais a fundo e falaremos sobre as melhores práticas para compartilhar suas bibliotecas com outras pessoas. O **Capítulo 15** discute ponteiros inteligentes (*smart pointers*) que a biblioteca padrão fornece e as traits que habilitam sua funcionalidade.

No **Capítulo 16**, percorreremos diferentes modelos de programação concorrente e falaremos sobre como o Rust ajuda você a programar em múltiplas threads sem medo. No **Capítulo 17**, construímos sobre isso explorando a sintaxe `async` e `await` do Rust, juntamente com tarefas (*tasks*), futuros (*futures*), fluxos (*streams*) e o modelo de concorrência leve que eles habilitam.

O **Capítulo 18** analisa como os idiomas (*idioms*) do Rust se comparam aos princípios de programação orientada a objetos com os quais você pode estar familiarizado. O **Capítulo 19** é uma referência sobre padrões e correspondência de padrões (*pattern matching*), que são maneiras poderosas de expressar ideias em programas Rust. O **Capítulo 20** contém uma variedade de tópicos avançados de interesse, incluindo Rust inseguro (*unsafe*), macros e mais sobre tempos de vida (*lifetimes*), traits, tipos, funções e closures.

No **Capítulo 21**, concluiremos um projeto no qual implementaremos um servidor web multi-thread de baixo nível!

Finalmente, alguns apêndices contêm informações úteis sobre a linguagem em um formato mais semelhante a uma referência. O **Apêndice A** cobre as palavras-chave do Rust, o **Apêndice B** cobre os operadores e símbolos do Rust, o **Apêndice C** cobre traits deriváveis fornecidas pela biblioteca padrão, o **Apêndice D** cobre algumas ferramentas de desenvolvimento úteis e o **Apêndice E** explica as edições do Rust. No **Apêndice F**, você pode encontrar traduções do livro, e no **Apêndice G** abordaremos como o Rust é feito e o que é o Rust nightly.

Não há maneira errada de ler este livro: se você quiser pular adiante, vá em frente! Você pode ter que voltar a capítulos anteriores se encontrar alguma confusão. Mas faça o que funcionar melhor para você.

<span id="ferris"></span>

Uma parte importante do processo de aprendizagem do Rust é aprender a ler as mensagens de erro que o compilador exibe: elas o guiarão em direção ao código funcional. Como tal, forneceremos muitos exemplos que não compilarão junto com a mensagem de erro que o compilador mostrará em cada situação. Saiba que, se você digitar e executar um exemplo aleatório, ele pode não compilar! Certifique-se de ler o texto ao redor para ver se o exemplo que você está tentando executar deve gerar erro. Na maioria das situações, nós o levaremos à versão correta de qualquer código que não compile. O Ferris também ajudará você a distinguir o código que não foi feito para funcionar:

| Ferris                                                                                                           | Significado                                      |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris com um ponto de interrogação"/>    | Este código não compila!                         |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris jogando as mãos para o alto"/>               | Este código entra em pânico (*panics*)!          |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris com uma garra para cima, dando de ombros"/> | Este código não produz o comportamento desejado. |

Na maioria das situações, nós o levaremos à versão correta de qualquer código que não compile.

## Código Fonte

Os arquivos fonte a partir dos quais este livro é gerado podem ser encontrados no
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
