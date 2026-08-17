## Apêndice B: Operadores e Símbolos

Este apêndice contém um glossário da sintaxe do Rust, incluindo operadores e
outros símbolos que aparecem sozinhos ou no contexto de caminhos (paths),
generics, restrições de traits (trait bounds), macros, atributos, comentários,
tuplas e colchetes.

### Operadores

A Tabela B-1 contém os operadores em Rust, um exemplo de como o operador
aparece no contexto, uma breve explicação e se esse operador é
sobrecarregável. Se um operador for sobrecarregável, a trait relevante a ser
usada para sobrecarregá-lo é listada.

<span class="caption">Tabela B-1: Operadores</span>

| Operador                  | Exemplo                                                 | Explicação                                                            | Sobrecarregável? |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | ---------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Expansão de macro                                                     |                  |
| `!`                       | `!expr`                                                 | Complemento lógico ou bit a bit                                       | `Not`            |
| `!=`                      | `expr != expr`                                          | Comparação de desigualdade                                            | `PartialEq`      |
| `%`                       | `expr % expr`                                           | Resto aritmético (módulo)                                             | `Rem`            |
| `%=`                      | `var %= expr`                                           | Resto aritmético e atribuição                                         | `RemAssign`      |
| `&`                       | `&expr`, `&mut expr`                                    | Empréstimo (Borrow)                                                   |                  |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Tipo de ponteiro emprestado                                           |                  |
| `&`                       | `expr & expr`                                           | AND bit a bit                                                         | `BitAnd`         |
| `&=`                      | `var &= expr`                                           | AND bit a bit e atribuição                                            | `BitAndAssign`   |
| `&&`                      | `expr && expr`                                          | AND lógico de curto-circuito                                          |                  |
| `*`                       | `expr * expr`                                           | Multiplicação aritmética                                              | `Mul`            |
| `*=`                      | `var *= expr`                                           | Multiplicação aritmética e atribuição                                 | `MulAssign`      |
| `*`                       | `*expr`                                                 | Desreferenciação                                                      | `Deref`          |
| `*`                       | `*const type`, `*mut type`                              | Ponteiro bruto (raw pointer)                                          |                  |
| `+`                       | `trait + trait`, `'a + trait`                           | Restrição de tipo composto                                            |                  |
| `+`                       | `expr + expr`                                           | Adição aritmética                                                     | `Add`            |
| `+=`                      | `var += expr`                                           | Adição aritmética e atribuição                                        | `AddAssign`      |
| `,`                       | `expr, expr`                                            | Separador de argumentos e elementos                                   |                  |
| `-`                       | `- expr`                                                | Negação aritmética                                                    | `Neg`            |
| `-`                       | `expr - expr`                                           | Subtração aritmética                                                  | `Sub`            |
| `-=`                      | `var -= expr`                                           | Subtração aritmética e atribuição                                     | `SubAssign`      |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipo de retorno de função e closure                                   |                  |
| `.`                       | `expr.ident`                                            | Acesso a campo                                                        |                  |
| `.`                       | `expr.ident(expr, ...)`                                 | Chamada de método                                                     |                  |
| `.`                       | `expr.0`, `expr.1`, e assim por diante                  | Indexação de tupla                                                    |                  |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Literal de intervalo exclusivo à direita                              | `PartialOrd`     |
| `..=`                     | `..=expr`, `expr..=expr`                                | Literal de intervalo inclusivo à direita                              | `PartialOrd`     |
| `..`                      | `..expr`                                                | Sintaxe de atualização de literal de struct                           |                  |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Padrão de vinculação "e o resto"                                      |                  |
| `...`                     | `expr...expr`                                           | (Descontinuado, use `..=` em vez disso) Em um padrão: intervalo inclusivo |                  |
| `/`                       | `expr / expr`                                           | Divisão aritmética                                                    | `Div`            |
| `/=`                      | `var /= expr`                                           | Divisão aritmética e atribuição                                       | `DivAssign`      |
| `:`                       | `pat: type`, `ident: type`                              | Restrições de tipo                                                    |                  |
| `:`                       | `ident: expr`                                           | Inicializador de campo de struct                                      |                  |
| `:`                       | `'a: loop {...}`                                        | Rótulo de loop                                                        |                  |
| `;`                       | `expr;`                                                 | Terminador de instrução e item                                        |                  |
| `;`                       | `[...; len]`                                            | Parte da sintaxe de array de tamanho fixo                             |                  |
| `<<`                      | `expr << expr`                                          | Deslocamento à esquerda (Left-shift)                                  | `Shl`            |
| `<<=`                     | `var <<= expr`                                          | Deslocamento à esquerda e atribuição                                  | `ShlAssign`      |
| `<`                       | `expr < expr`                                           | Comparação "menor que"                                                | `PartialOrd`     |
| `<=`                      | `expr <= expr`                                          | Comparação "menor ou igual a"                                         | `PartialOrd`     |
| `=`                       | `var = expr`, `ident = type`                            | Atribuição/equivalência                                               |                  |
| `==`                      | `expr == expr`                                          | Comparação de igualdade                                               | `PartialEq`      |
| `=>`                      | `pat => expr`                                           | Parte da sintaxe de braço de match                                    |                  |
| `>`                       | `expr > expr`                                           | Comparação "maior que"                                                | `PartialOrd`     |
| `>=`                      | `expr >= expr`                                          | Comparação "maior ou igual a"                                         | `PartialOrd`     |
| `>>`                      | `expr >> expr`                                          | Deslocamento à direita (Right-shift)                                  | `Shr`            |
| `>>=`                     | `var >>= expr`                                          | Deslocamento à direita e atribuição                                   | `ShrAssign`      |
| `@`                       | `ident @ pat`                                           | Vinculação de padrão                                                  |                  |
| `^`                       | `expr ^ expr`                                           | OU exclusivo bit a bit (XOR)                                          | `BitXor`         |
| `^=`                      | `var ^= expr`                                           | OU exclusivo bit a bit e atribuição                                   | `BitXorAssign`   |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Alternativas de padrão                                                |                  |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | OU bit a bit                                                          | `BitOr`          |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | OU bit a bit e atribuição                                             | `BitOrAssign`    |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | OU lógico de curto-circuito                                           |                  |
| `?`                       | `expr?`                                                 | Propagação de erros                                                   |                  |

### Símbolos que não são operadores

As tabelas a seguir contêm todos os símbolos que não funcionam como operadores; ou
seja, eles não se comportam como uma chamada de função ou método.

A Tabela B-2 mostra símbolos que aparecem sozinhos e são válidos em uma
variedade de locais.

<span class="caption">Tabela B-2: Sintaxe autônoma</span>

| Símbolo                                                                | Explicação                                                                               |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `'ident`                                                               | Tempo de vida (lifetime) nomeado ou rótulo de loop                                       |
| Dígitos seguidos imediatamente por `u8`, `i32`, `f64`, `usize`, etc.    | Literal numérico de tipo específico                                                      |
| `"..."`                                                                | Literal de string                                                                        |
| `r"..."`, `r#"..."#`, `r##"..."##`, etc.                               | Literal de string bruta; caracteres de escape não processados                            |
| `b"..."`                                                               | Literal de string de bytes; constrói um array de bytes em vez de uma string              |
| `br"..."`, `br#"..."#`, `br##"..."##`, etc.                            | Literal de string de bytes bruta; combinação de string bruta e de bytes                  |
| `'...'`                                                                | Literal de caractere                                                                     |
| `b'...'`                                                               | Literal de byte ASCII                                                                    |
| <code>&vert;...&vert; expr</code>                                      | Closure                                                                                  |
| `!`                                                                    | Tipo inferior (bottom type) sempre vazio para funções divergentes                        |
| `_`                                                                    | Vinculação de padrão "ignorada"; também usado para tornar literais inteiros legíveis     |

A Tabela B-3 mostra símbolos que aparecem no contexto de um caminho através da
hierarquia de módulos até um item.

<span class="caption">Tabela B-3: Sintaxe Relacionada a Caminhos (Paths)</span>

| Símbolo                                 | Explicação                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------|
| `ident::ident`                          | Caminho de namespace                                                                                         |
| `::path`                                | Caminho relativo à raiz do crate (ou seja, um caminho explicitamente absoluto)                               |
| `self::path`                            | Caminho relativo ao módulo atual (ou seja, um caminho explicitamente relativo)                               |
| `super::path`                           | Caminho relativo ao pai do módulo atual                                                                      |
| `type::ident`, `<type as trait>::ident` | Constantes, funções e tipos associados                                                                       |
| `<type>::...`                           | Item associado para um tipo que não pode ser nomeado diretamente (por exemplo, `<&T>::...`, `<[T]>::...`, etc.) |
| `trait::method(...)`                    | Desambiguando uma chamada de método nomeando a trait que a define                                            |
| `type::method(...)`                     | Desambiguando uma chamada de método nomeando o tipo para o qual ela é definido                               |
| `<type as trait>::method(...)`          | Desambiguando uma chamada de método nomeando a trait e o tipo                                                |

A Tabela B-4 mostra símbolos que aparecem no contexto do uso de parâmetros de
tipo genérico.

<span class="caption">Tabela B-4: Generics</span>

| Símbolo                        | Explicação                                                                                                                                          |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Especifica parâmetros para um tipo genérico em um tipo (por exemplo, `Vec<u8>`)                                                                     |
| `path::<...>`, `method::<...>` | Especifica parâmetros para um tipo genérico, função ou método em uma expressão; frequentemente chamado de _turbofish_ (por exemplo, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Define função genérica                                                                                                                              |
| `struct ident<...> ...`        | Define estrutura genérica                                                                                                                           |
| `enum ident<...> ...`          | Define enumeração genérica                                                                                                                          |
| `impl<...> ...`                | Define implementação genérica                                                                                                                       |
| `for<...> type`                | Restrições de tempo de vida de alta prioridade (higher ranked lifetime bounds)                                                                      |
| `type<ident=type>`             | Um tipo genérico onde um ou mais tipos associados têm atribuições específicas (por exemplo, `Iterator<Item=T>`)                                     |

A Tabela B-5 mostra símbolos que aparecem no contexto de restrição de parâmetros
de tipo genérico com trait bounds.

<span class="caption">Tabela B-5: Restrições de Trait Bounds</span>

| Símbolo                       | Explicação                                                                                                                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                        | Parâmetro genérico `T` restrito a tipos que implementam `U`                                                                                  |
| `T: 'a`                       | O tipo genérico `T` deve durar mais que o tempo de vida `'a` (significando que o tipo não pode conter transitivamente referências com tempos de vida menores que `'a`) |
| `T: 'static`                  | O tipo genérico `T` não contém referências emprestadas além das do tipo `'static`                                                            |
| `'b: 'a`                      | O tempo de vida genérico `'b` deve durar mais que o tempo de vida `'a`                                                                      |
| `T: ?Sized`                   | Permite que o parâmetro de tipo genérico seja um tipo de tamanho dinâmico (dynamically sized type)                                           |
| `'a + trait`, `trait + trait` | Restrição de tipo composto                                                                                                                   |

A Tabela B-6 mostra símbolos que aparecem no contexto de chamada ou definição de
macros e especificação de atributos em um item.

<span class="caption">Tabela B-6: Macros e Atributos</span>

| Símbolo                                     | Explicação          |
| ------------------------------------------- | ------------------- |
| `#[meta]`                                   | Atributo externo    |
| `#![meta]`                                  | Atributo interno    |
| `$ident`                                    | Substituição de macro |
| `$ident:kind`                               | Metavariável de macro |
| `$(...)...`                                 | Repetição de macro  |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Invocação de macro  |

A Tabela B-7 mostra símbolos que criam comentários.

<span class="caption">Tabela B-7: Comentários</span>

| Símbolo    | Explicação                       |
| ---------- | -------------------------------- |
| `//`       | Comentário de linha              |
| `//!`      | Comentário de documentação interno de linha |
| `///`      | Comentário de documentação externo de linha |
| `/*...*/`  | Comentário de bloco              |
| `/*!...*/` | Comentário de documentação interno de bloco |
| `/**...*/` | Comentário de documentação externo de bloco |

A Tabela B-8 mostra os contextos nos quais parênteses são usados.

<span class="caption">Tabela B-8: Parênteses</span>

| Símbolo                  | Explicação                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Tupla vazia (também conhecida como unit), tanto literal quanto tipo                         |
| `(expr)`                 | Expressão entre parênteses                                                                  |
| `(expr,)`                | Expressão de tupla de elemento único                                                        |
| `(type,)`                | Tipo de tupla de elemento único                                                             |
| `(expr, ...)`            | Expressão de tupla                                                                          |
| `(type, ...)`            | Tipo de tupla                                                                               |
| `expr(expr, ...)`        | Expressão de chamada de função; também usada para inicializar `struct`s e variantes de `enum` do tipo tupla |

A Tabela B-9 mostra os contextos nos quais chaves são usadas.

<span class="caption">Tabela B-9: Chaves</span>

| Contexto     | Explicação           |
| ------------ | -------------------- |
| `{...}`      | Expressão de bloco   |
| `Type {...}` | Literal de struct    |

A Tabela B-10 mostra os contextos nos quais colchetes são usados.

<span class="caption">Tabela B-10: Colchetes</span>

| Contexto                                           | Explicação                                                                                                                    |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Literal de array                                                                                                              |
| `[expr; len]`                                      | Literal de array contendo `len` cópias de `expr`                                                                              |
| `[type; len]`                                      | Tipo de array contendo `len` instâncias de `type`                                                                             |
| `expr[expr]`                                       | Indexação de coleção; sobrecarregável (`Index`, `IndexMut`)                                                                   |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexação de coleção fingindo ser fatiamento (slicing) de coleção, usando `Range`, `RangeFrom`, `RangeTo` ou `RangeFull` como o “índice” |
