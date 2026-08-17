## Apêndice G - Como o Rust é Feito e o "Rust Nightly"

Este apêndice aborda como o Rust é desenvolvido e como isso afeta você como um
desenvolvedor Rust.

### Estabilidade Sem Estagnação

Como linguagem, o Rust se importa _muito_ com a estabilidade do seu código. Nós
queremos que o Rust seja uma base sólida sobre a qual você possa construir, e se
as coisas estivessem mudando constantemente, isso seria impossível. Ao mesmo
tempo, se não pudermos experimentar com novos recursos, podemos não descobrir
falhas importantes até depois de seu lançamento, quando não pudermos mais mudar
as coisas.

Nossa solução para esse problema é o que chamamos de "estabilidade sem
estagnação", e nosso princípio orientador é este: você nunca deve temer atualizar
para uma nova versão estável do Rust. Cada atualização deve ser indolor, mas
também deve trazer novos recursos, menos bugs e tempos de compilação mais
rápidos.

### Piu-í! Canais de Lançamento e Pegando os Trens

O desenvolvimento do Rust opera em um _cronograma de trens_. Ou seja, todo o
desenvolvimento é feito no branch principal (main) do repositório do Rust. Os
lançamentos seguem um modelo de trem de lançamento de software, que tem sido
usado pelo Cisco IOS e outros projetos de software. Existem três _canais de
lançamento_ para o Rust:

- Nightly
- Beta
- Stable

A maioria dos desenvolvedores Rust usa principalmente o canal estável, mas
aqueles que desejam testar novos recursos experimentais podem usar o nightly
ou o beta.

Aqui está um exemplo de como o processo de desenvolvimento e lançamento
funciona: digamos que a equipe do Rust esteja trabalhando no lançamento do Rust
1.5. Esse lançamento aconteceu em dezembro de 2015, mas ele nos fornecerá números
de versão realistas. Um novo recurso é adicionado ao Rust: um novo commit é
enviado para o branch principal. Toda noite, uma nova versão nightly do Rust é
produzida. Todos os dias são dias de lançamento, e esses lançamentos são criados
automaticamente pela nossa infraestrutura de lançamento. Portanto, com o passar
do tempo, nossos lançamentos se parecem com isso, uma vez por noite:

```text
nightly: * - - * - - *
```

A cada seis semanas, é hora de preparar um novo lançamento! O branch `beta` do
repositório do Rust é ramificado a partir do branch principal usado pelo nightly.
Agora, existem dois lançamentos:

```text
nightly: * - - * - - *
                     |
beta:                *
```

A maioria dos usuários do Rust não usa ativamente os lançamentos beta, mas os
testa em seu sistema de CI para ajudar o Rust a descobrir possíveis regressões.
Enquanto isso, ainda há um lançamento nightly todas as noites:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Digamos que uma regressão seja encontrada. Que bom que tivemos algum tempo para
testar o lançamento beta antes que a regressão se esgueirasse para um lançamento
estável! A correção é aplicada ao branch principal, de modo que o nightly é
corrigido, e então a correção é portada de volta para o branch `beta`, e um novo
lançamento do beta é produzido:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Seis semanas após a criação do primeiro beta, é hora de um lançamento estável! O
branch `stable` é produzido a partir do branch `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Eba! O Rust 1.5 está pronto! No entanto, esquecemos de uma coisa: como as seis
semanas se passaram, também precisamos de um novo beta da _próxima_ versão do
Rust, a 1.6. Portanto, depois que o `stable` é ramificado a partir do `beta`, a
próxima versão do `beta` é ramificada a partir do `nightly` novamente:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Isso é chamado de "modelo de trem" porque a cada seis semanas, um lançamento
"deixa a estação", mas ainda precisa fazer uma jornada pelo canal beta antes de
chegar como um lançamento estável.

O Rust lança atualizações a cada seis semanas, pontualmente como um relógio. Se
você sabe a data de um lançamento do Rust, você pode saber a data do próximo: é
seis semanas depois. Um aspecto legal de ter lançamentos agendados a cada seis
semanas é que o próximo trem está chegando logo. Se um recurso acabar perdendo
um lançamento específico, não há necessidade de se preocupar: outro está
acontecendo em pouco tempo! Isso ajuda a reduzir a pressão para enfiar recursos
possivelmente inacabados perto do prazo de lançamento.

Graças a esse processo, você sempre pode conferir a próxima compilação do Rust e
verificar por si mesmo que é fácil de atualizar: se um lançamento beta não
funcionar como esperado, você pode relatá-lo à equipe e corrigi-lo antes que o
próximo lançamento estável aconteça! Quebras em um lançamento beta são
relativamente raras, mas o `rustc` ainda é um pedaço de software, e bugs existem.

### Tempo de manutenção

O projeto Rust dá suporte à versão estável mais recente. Quando uma nova versão
estável é lançada, a versão antiga chega ao fim de sua vida útil (EOL — *End of
Life*). Isso significa que cada versão tem suporte por seis semanas.

### Recursos Instáveis

Há mais um detalhe com este modelo de lançamento: recursos instáveis. O Rust usa
uma técnica chamada "feature flags" (sinalizadores de recursos) para determinar
quais recursos estão habilitados em um determinado lançamento. Se um novo
recurso está em desenvolvimento ativo, ele chega ao branch principal e, portanto,
ao nightly, mas atrás de um _feature flag_. Se você, como usuário, deseja testar
o recurso em andamento, você pode, mas deve estar usando um lançamento nightly
do Rust e anotar seu código-fonte com o sinalizador apropriado para ativá-lo.

Se você estiver usando um lançamento beta ou estável do Rust, não poderá usar
nenhum feature flag. Esta é a chave que nos permite obter uso prático com novos
recursos antes de declará-los estáveis para sempre. Aqueles que desejam optar
pela tecnologia de ponta podem fazê-lo, e aqueles que querem uma experiência
sólida como uma rocha podem ficar com o estável e saber que seu código não vai
quebrar. Estabilidade sem estagnação.

Este livro contém apenas informações sobre recursos estáveis, pois recursos em
andamento ainda estão mudando e certamente serão diferentes entre o momento em
que este livro foi escrito e o momento em que forem habilitados em compilações
estáveis. Você pode encontrar documentação para recursos exclusivos do nightly
online.

### Rustup e o Papel do Rust Nightly

O Rustup facilita a alteração entre diferentes canais de lançamento do Rust, de
forma global ou por projeto. Por padrão, você terá o Rust estável instalado.
Para instalar o nightly, por exemplo:

```console
$ rustup toolchain install nightly
```

Você também pode ver todas as _toolchains_ (lançamentos do Rust e componentes
associados) que você instalou com o `rustup`. Aqui está um exemplo no
computador Windows de um dos seus autores:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Como você pode ver, a toolchain estável é a padrão. A maioria dos usuários do
Rust usa a estável na maior parte do tempo. Você pode querer usar a estável na
maior parte do tempo, mas usar o nightly em um projeto específico, porque você
se importa com um recurso de ponta. Para fazer isso, você pode usar o `rustup
override` no diretório desse projeto para definir a toolchain nightly como a que
o `rustup` deve usar quando você estiver nesse diretório:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Agora, toda vez que você chamar `rustc` ou `cargo` dentro de
_~/projects/needs-nightly_, o `rustup` garantirá que você esteja usando o Rust
nightly, em vez do seu padrão Rust estável. Isso é muito útil quando você tem
muitos projetos Rust!

### O Processo de RFC e as Equipes

Então, como você fica sabendo desses novos recursos? O modelo de desenvolvimento
do Rust segue um _Processo de Solicitação de Comentários (RFC — Request For
Comments)_. Se você quiser uma melhoria no Rust, você pode escrever uma
proposta, chamada de RFC.

Qualquer pessoa pode escrever RFCs para melhorar o Rust, e as propostas são
revisadas e discutidas pela equipe do Rust, que é composta por muitas subequipes
de tópicos. Há uma lista completa das equipes [no site do Rust](https://www.rust-lang.org/governance), que inclui equipes para cada área do
projeto: design de linguagem, implementação de compilador, infraestrutura,
documentação e muito mais. A equipe apropriada lê a proposta e os comentários,
escreve alguns comentários próprios e, eventualmente, há consenso para aceitar
ou rejeitar o recurso.

Se o recurso for aceito, um *issue* é aberto no repositório do Rust e alguém
pode implementá-lo. A pessoa que o implementa muito bem pode não ser a pessoa
que propôs o recurso em primeiro lugar! Quando a implementação estiver pronta,
ela chega ao branch principal protegida por um *feature gate*, como discutimos
na seção [“Recursos Instáveis”](#unstable-features)<!-- ignore -->.

Após algum tempo, uma vez que os desenvolvedores do Rust que usam lançamentos
nightly tenham conseguido testar o novo recurso, os membros da equipe discutirão
o recurso, como ele funcionou no nightly e decidirão se ele deve entrar no Rust
estável ou não. Se a decisão for seguir em frente, o *feature gate* é removido e
o recurso agora é considerado estável! Ele pega os trens para um novo
lançamento estável do Rust.