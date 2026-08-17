## Apêndice A: Palavras-chave

As listas a seguir contêm palavras-chave que estão reservadas para uso atual ou
futuro pela linguagem Rust. Sendo assim, elas não podem ser usadas como
identificadores (exceto como identificadores crus, conforme discutimos na
seção [“Identificadores Crus”][raw-identifiers]<!-- ignore -->).
*Identificadores* são nomes de funções, variáveis, parâmetros, campos de struct,
módulos, crates, constantes, macros, valores estáticos, atributos, tipos,
traits ou tempos de vida (*lifetimes*).

[raw-identifiers]: #raw-identifiers

### Palavras-chave Atualmente em Uso

A seguir, apresenta-se uma lista de palavras-chave atualmente em uso, com suas
funcionalidades descritas.

- **`as`**: Realiza conversões primitivas (*casting*), desambigua um trait específico
  que contenha um item ou renomeia itens em declarações `use`.
- **`async`**: Retorna uma `Future` em vez de bloquear a thread atual.
- **`await`**: Suspende a execução até que o resultado de uma `Future` esteja pronto.
- **`break`**: Sai de um loop imediatamente.
- **`const`**: Define itens constantes ou ponteiros crus constantes (*constant raw pointers*).
- **`continue`**: Continua para a próxima iteração do loop.
- **`crate`**: Em um caminho de módulo (*module path*), refere-se à raiz do crate.
- **`dyn`**: Despacho dinâmico (*dynamic dispatch*) para um objeto trait.
- **`else`**: Alternativa para os constructos de fluxo de controle `if` e `if let`.
- **`enum`**: Define uma enumeração.
- **`extern`**: Vincula uma função ou variável externa.
- **`false`**: Literal booleano falso.
- **`fn`**: Define uma função ou o tipo ponteiro de função.
- **`for`**: Faz um loop sobre os itens de um iterador, implementa um trait ou especifica um
  tempo de vida de ordem superior (*higher-ranked lifetime*).
- **`if`**: Realiza desvio condicional com base no resultado de uma expressão.
- **`impl`**: Implementa funcionalidade inerente ou de trait.
- **`in`**: Parte da sintaxe do loop `for`.
- **`let`**: Associa uma variável.
- **`loop`**: Executa um loop incondicional.
- **`match`**: Compara um valor com padrões (*patterns*).
- **`mod`**: Define um módulo.
- **`move`**: Faz com que uma closure assuma a propriedade de todas as suas captações.
- **`mut`**: Denota mutabilidade em referências, ponteiros crus ou associações de padrões.
- **`pub`**: Denota visibilidade pública em campos de struct, blocos `impl` ou
  módulos.
- **`ref`**: Associa por referência.
- **`return`**: Retorna de uma função.
- **`Self`**: Um apelido de tipo (*type alias*) para o tipo que estamos definindo ou implementando.
- **`self`**: Sujeito do método ou módulo atual.
- **`static`**: Variável global ou tempo de vida que dura toda a execução do programa.
- **`struct`**: Define uma estrutura.
- **`super`**: Módulo pai do módulo atual.
- **`trait`**: Define um trait.
- **`true`**: Literal booleano verdadeiro.
- **`type`**: Define um apelido de tipo ou tipo associado.
- **`union`**: Define uma [união][union]<!-- ignore -->; é uma palavra-chave apenas quando
  usada em uma declaração de união.
- **`unsafe`**: Denota código, funções, traits ou implementações não seguras.
- **`use`**: Traz símbolos para o escopo.
- **`where`**: Denota cláusulas que restringem um tipo.
- **`while`**: Executa um loop condicional com base no resultado de uma expressão.

[union]: ../reference/items/unions.html

### Palavras-chave Reservadas para Uso Futuro

As seguintes palavras-chave ainda não possuem nenhuma funcionalidade, mas estão
reservadas pelo Rust para possível uso futuro:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Identificadores Crus

*Identificadores crus* (*raw identifiers*) são a sintaxe que permite usar palavras-chave onde elas
normalmente não seriam permitidas. Você usa um identificador cru colocando `r#`
como prefixo de uma palavra-chave.

Por exemplo, `match` é uma palavra-chave. Se você tentar compilar a seguinte função
que usa `match` como seu nome:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

você receberá este erro:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

O erro mostra que você não pode usar a palavra-chave `match` como o identificador
da função. Para usar `match` como nome de função, você precisa usar a sintaxe de
identificador cru, desta forma:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Este código compilará sem nenhum erro. Note o prefixo `r#` no nome da função
tanto em sua definição quanto no local onde a função é chamada em `main`.

Identificadores crus permitem que você use qualquer palavra que escolher como
identificador, mesmo que essa palavra seja uma palavra-chave reservada. Isso nos
dá mais liberdade para escolher nomes de identificadores, além de nos permitir
integrar com programas escritos em uma linguagem onde essas palavras não são
palavras-chave. Além disso, identificadores crus permitem que você use bibliotecas
escritas em uma edição do Rust diferente daquela que o seu crate usa. Por exemplo,
`try` não é uma palavra-chave na edição de 2015, mas é nas edições de 2018, 2021 e
2024. Se você depende de uma biblioteca escrita usando a edição de 2015 e que possui
uma função chamada `try`, você precisará usar a sintaxe de identificador cru,
`r#try` neste caso, para chamar essa função a partir do seu código em edições mais
recentes. Veja o [Apêndice E][appendix-e]<!-- ignore --> para mais informações sobre
edições.

[appendix-e]: appendix-05-editions.html
