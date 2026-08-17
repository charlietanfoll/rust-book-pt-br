# Projeto Final: Construindo um Servidor Web Multithreaded

Tem sido uma longa jornada, mas chegamos ao fim do livro. Neste capítulo,
construiremos mais um projeto juntos para demonstrar alguns dos conceitos que
cobrimos nos capítulos finais, além de recapitular algumas lições anteriores.

Para o nosso projeto final, faremos um servidor web que diz "Hello!" e se parece
com a Figura 21-1 em um navegador web.

Aqui está o nosso plano para construir o servidor web:

1. Aprender um pouco sobre TCP e HTTP.
2. Escutar por conexões TCP em um socket.
3. Analisar (*parse*) um pequeno número de requisições HTTP.
4. Criar uma resposta HTTP adequada.
5. Melhorar a vazão (*throughput*) do nosso servidor com um pool de threads.

<img alt="Captura de tela de um navegador web acessando o endereço 127.0.0.1:8080 exibindo uma página web com o conteúdo de texto “Hello! Hi from Rust”" src="img/trpl21-01.png" class="center" style="width: 50%;" />

<span class="caption">Figura 21-1: Nosso projeto compartilhado final</span>

Antes de começarmos, devemos mencionar dois detalhes. Primeiro, o método que
usaremos não será a melhor maneira de construir um servidor web com Rust.
Membros da comunidade publicaram vários crates prontos para produção disponíveis
em [crates.io](https://crates.io/) que fornecem implementações de servidores web
e pools de threads mais completas do que a que vamos construir. No entanto, nossa
intenção neste capítulo é ajudar você a aprender, e não seguir pelo caminho mais
fácil. Como o Rust é uma linguagem de programação de sistemas, podemos escolher
o nível de abstração com o qual queremos trabalhar e podemos ir a um nível mais
baixo do que é possível ou prático em outras linguagens.

Segundo, não usaremos `async` e `await` aqui. Construir um pool de threads já é
um desafio grande o suficiente por si só, sem adicionar a construção de um
runtime assíncrono! No entanto, observaremos como `async` e `await` podem ser
aplicados a alguns dos mesmos problemas que veremos neste capítulo. Em última
análise, como observamos no Capítulo 17, muitos runtimes assíncronos usam pools
de threads para gerenciar seu trabalho.

Portanto, escreveremos o servidor HTTP básico e o pool de threads manualmente para
que você possa aprender as ideias e técnicas gerais por trás dos crates que você
poderá usar no futuro.