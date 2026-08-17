## Instalação

O primeiro passo é instalar o Rust. Nós vamos baixar o Rust através do `rustup`,
uma ferramenta de linha de comando para gerenciar versões do Rust e ferramentas
associadas. Você precisará de uma conexão com a internet para o download.

> Nota: Se por algum motivo você preferir não usar o `rustup`, consulte a página
> [Outros Métodos de Instalação do Rust][otherinstall] para mais opções.

Os passos a seguir instalam a versão estável mais recente do compilador Rust.
As garantias de estabilidade do Rust asseguram que todos os exemplos do livro
que compilan continuarão compilando com versões mais recentes do Rust. A saída
pode diferir ligeiramente entre as versões porque o Rust frequentemente melhora
mensagens de erro e avisos. Em outras palavras, qualquer versão nova e estável do
Rust que você instalar usando estes passos deve funcionar conforme o esperado com
o conteúdo deste livro.

> ### Notação de Linha de Comando
>
> Neste capítulo e ao longo do livro, mostraremos alguns comandos usados no
> terminal. As linhas que você deve digitar em um terminal começam com `$`. Você
> não precisa digitar o caractere `$`; ele é o prompt de linha de comando exibido
> para indicar o início de cada comando. Linhas que não começam com `$` tipicamente
> mostram a saída do comando anterior. Além disso, exemplos específicos do PowerShell
> usarão `>` em vez de `$`.

### Instalando o `rustup` no Linux ou macOS

Se você estiver usando Linux ou macOS, abra um terminal e digite o seguinte comando:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

O comando baixa um script e inicia a instalação da ferramenta `rustup`, que
instala a versão estável mais recente do Rust. Pode ser solicitado que você
insira sua senha. Se a instalação for bem-sucedida, a seguinte linha aparecerá:

```text
Rust is installed now. Great!
```

Você também precisará de um _linker_ (ligador), que é um programa que o Rust
usa para juntar suas saídas compiladas em um único arquivo. É provável que você
já tenha um. Se você receber erros de linker, você deve instalar um compilador
C, que tipicamente incluirá um linker. Um compilador C também é útil porque
alguns pacotes comuns do Rust dependem de código C e precisarão de um compilador C.

No macOS, você pode obter um compilador C executando:

```console
$ xcode-select --install
```

Usuários de Linux devem geralmente instalar o GCC ou o Clang, de acordo com a
documentação de sua distribuição. Por exemplo, se você usa o Ubuntu, pode instalar
o pacote `build-essential`.

### Instalando o `rustup` no Windows

No Windows, vá para [https://www.rust-lang.org/tools/install][install]<!-- ignore
--> e siga as instruções para instalar o Rust. Em algum momento da instalação,
será solicitado que você instale o Visual Studio. Isso fornece um linker e as
bibliotecas nativas necessárias para compilar programas. Se você precisar de mais
ajuda com esta etapa, consulte
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore -->.

O resto deste livro usa comandos que funcionam tanto no _cmd.exe_ quanto no PowerShell.
Se houver diferenças específicas, explicaremos qual usar.

### Solução de Problemas

Para verificar se você instalou o Rust corretamente, abra um shell e digite esta
linha:

```console
$ rustc --version
```

Você deve ver o número da versão, o hash do commit e a data do commit para a
última versão estável lançada, no seguinte formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Se você vir esta informação, você instalou o Rust com sucesso! Se você não
vir esta informação, verifique se o Rust está na sua variável de sistema `%PATH%`
da seguinte forma.

No CMD do Windows, use:

```console
> echo %PATH%
```

No PowerShell, use:

```powershell
> echo $env:Path
```

No Linux e macOS, use:

```console
$ echo $PATH
```

Se tudo estiver correto e o Rust ainda não estiver funcionando, há vários lugares
onde você pode obter ajuda. Descubra como entrar em contato com outros Rustaceans
(um apelido divertido que usamos para nós mesmos) na [página da comunidade][community].

### Atualizando e Desinstalando

Uma vez que o Rust esteja instalado via `rustup`, atualizar para uma versão recém-lançada
é fácil. A partir do seu shell, execute o seguinte script de atualização:

```console
$ rustup update
```

Para desinstalar o Rust e o `rustup`, execute o seguinte script de desinstalação
a partir do seu shell:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### Lendo a Documentação Local

A instalação do Rust também inclui uma cópia local da documentação para que
você possa lê-la offline. Execute `rustup doc` para abrir a documentação local
no seu navegador.

Sempre que um tipo ou função for fornecido pela biblioteca padrão e você não
tiver certeza do que ele faz ou como usá-lo, use a documentação da interface
de programação de aplicações (API) para descobrir!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### Usando Editores de Texto e IDEs

Este livro não faz suposições sobre quais ferramentas você usa para escrever código Rust.
Quase qualquer editor de texto servirá! No entanto, muitos editores de texto e
ambientes de desenvolvimento integrado (IDEs) têm suporte integrado para o Rust.
Você sempre pode encontrar uma lista razoavelmente atual de muitos editores e
IDEs na [página de ferramentas][tools] no site do Rust.

### Trabalhando Offline com Este Livro

Em vários exemplos, usaremos pacotes do Rust além da biblioteca padrão. Para
acompanhar esses exemplos, você precisará ter uma conexão com a internet ou
ter baixado essas dependências com antecedência. Para baixar as dependências com
antecedência, você pode executar os seguintes comandos. (Explicaremos o que é o
`cargo` e o que cada um desses comandos faz em detalhes mais tarde.)

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.10.1 trpl@0.2.0
```

Isso fará o cache dos downloads para esses pacotes para que você não precise
baixá-los mais tarde. Uma vez que você tenha executado este comando, você não
precisa manter a pasta `get-dependencies`. Se você executou este comando, você
pode usar a flag `--offline` com todos os comandos do `cargo` no resto do livro
para usar essas versões em cache em vez de tentar usar a rede.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
