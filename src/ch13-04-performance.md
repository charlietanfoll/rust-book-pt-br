<!-- Old headings. Do not remove or links may break. -->

<a id="comparing-performance-loops-vs-iterators"></a>

## Desempenho em Loops vs. Iteradores

Para determinar se você deve usar loops ou iteradores, precisa saber qual
implementação é mais rápida: a versão da função `search` com um loop `for`
explícito ou a versão com iteradores.

Executamos um _benchmark_ carregando todo o conteúdo de _As Aventuras de
Sherlock Holmes_, de Sir Arthur Conan Doyle, em uma `String` e procurando pela
palavra _the_ no conteúdo. Aqui estão os resultados do _benchmark_ na
versão de `search` usando o loop `for` e na versão usando iteradores:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

As duas implementações têm desempenho semelhante! Não explicaremos o
código de _benchmark_ aqui porque o objetivo não é provar que as duas versões
são equivalentes, mas sim ter uma noção geral de como essas duas implementações
se comparam em termos de desempenho.

Para um _benchmark_ mais abrangente, você deve testar usando vários textos de
vários tamanhos como o `contents`, diferentes palavras e palavras de diferentes comprimentos
como o `query`, e todos os tipos de outras variações. O ponto é o seguinte:
Os iteradores, embora sejam uma abstração de alto nível, são compilados para praticamente o
mesmo código que você escreveria à mão em nível inferior. Os iteradores são uma
das _abstrações de custo zero_ do Rust, o que significa que o uso da abstração
não impõe nenhum custo adicional em tempo de execução. Isso é análogo a como Bjarne
Stroustrup, o designer e implementor original do C++, define
custo zero em sua palestra principal no ETAPS de 2012 “Foundations of C++”:

> Em geral, as implementações de C++ obedecem ao princípio de custo zero: O que você
> não usa, você não paga. E além disso: O que você usa, você não conseguiria escrever
> à mão de forma melhor.

Em muitos casos, o código Rust que usa iteradores é compilado para o mesmo código assembly que
você escreveria manualmente. Otimizações como desenrolamento de loops (_loop unrolling_) e eliminação
de verificação de limites no acesso a arrays são aplicadas e tornam o código resultante
extremamente eficiente. Agora que você sabe disso, pode usar iteradores e _closures_ sem medo!
Eles fazem com que o código pareça ser de alto nível, mas não impõem uma penalidade de desempenho
em tempo de execução por fazerem isso.

## Resumo

_Closures_ e iteradores são recursos do Rust inspirados em ideias de linguagens de programação
funcionais. Eles contribuem para a capacidade do Rust de expressar claramente
ideias de alto nível com desempenho de baixo nível. As implementações de _closures_ e
iteradores são feitas de forma que o desempenho em tempo de execução não seja afetado. Isso faz parte do
objetivo do Rust de se esforçar para fornecer abstrações de custo zero.

Agora que melhoramos a expressividade do nosso projeto de E/S (I/O), vamos ver
alguns recursos adicionais do `cargo` que nos ajudarão a compartilhar o projeto com o
mundo.