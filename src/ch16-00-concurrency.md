# Concorrência Sem Medo

Lidar com programação concorrente com segurança e eficiência é outro dos grandes
objetivos do Rust. A _programação concorrente_, em que diferentes partes de um
programa executam de forma independente, e a _programação paralela_, em que
diferentes partes de um programa executam ao mesmo tempo, estão se tornando cada
vez mais importantes à medida que mais computadores aproveitam seus múltiplos
processadores. Historicamente, programar nesses contextos tem sido difícil e
suscetível a erros. O Rust espera mudar isso.

Inicialmente, a equipe do Rust pensava que garantir a segurança de memória e
prevenir problemas de concorrência eram dois desafios separados a serem resolvidos
com métodos diferentes. Com o tempo, a equipe descobriu que os sistemas de
posse (*ownership*) e de tipos formam um conjunto poderoso de ferramentas para
ajudar a gerenciar problemas de segurança de memória _e_ de concorrência! Ao
aproveitar o sistema de posse e a verificação de tipos, muitos erros de
concorrência no Rust tornam-se erros de compilação em vez de erros em tempo de
execução. Portanto, em vez de fazer você gastar muito tempo tentando reproduzir
as circunstâncias exatas sob as quais um bug de concorrência em tempo de
execução ocorre, o código incorreto simplesmente se recusará a compilar e
apresentará um erro explicando o problema. Como resultado, você pode corrigir
seu código enquanto está trabalhando nele, em vez de potencialmente após ele ter
sido enviado para produção. Apelidamos esse aspecto do Rust de _concorrência sem
medo_ (*fearless concurrency*). A concorrência sem medo permite que você escreva
código livre de bugs sutis e fácil de refatorar sem introduzir novos bugs.

> Nota: Por questão de simplicidade, nos referiremos a muitos dos problemas
> como _concorrentes_ em vez de sermos mais precisos dizendo _concorrentes e/ou
> paralelos_. Para este capítulo, substitua mentalmente por _concorrente e/ou
> paralelo_ sempre que usarmos _concorrente_. No próximo capítulo, onde a
> distinção importa mais, seremos mais específicos.

Muitas linguagens são dogmáticas sobre as soluções que oferecem para lidar com
problemas concorrentes. Por exemplo, o Erlang possui uma funcionalidade elegante
para concorrência baseada em passagem de mensagens, mas tem apenas formas
obscuras de compartilhar estado entre threads. Suportar apenas um subconjunto de
soluções possíveis é uma estratégia razoável para linguagens de nível superior,
porque uma linguagem de nível superior promete benefícios ao abrir mão de certo
controle para ganhar abstrações. No entanto, espera-se que linguagens de nível
inferior forneçam a solução com o melhor desempenho em qualquer situação dada e
tenham menos abstrações sobre o hardware. Portanto, o Rust oferece uma variedade
de ferramentas para modelar problemas da maneira que for apropriada para a sua
situação e requisitos.

Aqui estão os tópicos que cobriremos neste capítulo:

- Como criar threads para executar múltiplos pedaços de código ao mesmo tempo
- Concorrência por _passagem de mensagens_, onde canais enviam mensagens entre threads
- Concorrência de _estado compartilhado_, onde múltiplas threads têm acesso a algum pedaço de dados
- Os traits `Sync` e `Send`, que estendem as garantias de concorrência do Rust para tipos definidos pelo usuário, bem como para tipos fornecidos pela biblioteca padrão