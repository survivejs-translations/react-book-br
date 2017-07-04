# Testando seu código React

Para incentivar as pessoas a apoiarem meu trabalho, decidi publicar uma versão TL;DR deste capítulo para a comunidade. Isso permitirá que eu desenvolva mais o conteúdo, uma estratégia onde todo mundo ganha.

Você pode acessar o capítulo completo comprando uma cópia no [Leanpub](https://leanpub.com/survivejs_react). Nele, nós entraremos em detalhes, enquanto aqui, será uma idéia aproximada do conteúdo do capítulo.

## TL;DR

* As técnicas básicas de teste incluem teste de unidade, teste de aceitação, testes baseados em propriedades e teste de mutação.
* Testes unitários nos permitem verificar certas verdades *específicas*.
* O teste de aceitação nos permitem testar aspectos qualitativos do nosso sistema.
* Testes baseados em propriedades(veja [QuickCheck](https://hackage.haskell.org/package/QuickCheck)) é mais genérico e nos permite cobrir uma gama maior de valores com mais facilidade. Esses testes são ainda mais difíceis de escrever.
* O teste de mutação, nos permitem testar os testes. Infelizmente, ainda não é uma técnica particularmente popular com o JavaScript.
* O [web-app](https://github.com/cesarandreu/web-app), do Cesar Andreu, tem uma configuração de teste bacana (Mocha/Karma/Istanbul).
* A cobertura do código nos ajuda a entender quais partes do código não foram testadas. No entanto, não nos dá garantias da qualidade de nossos testes.
* [React Test Utilities](https://facebook.github.io/react/docs/test-utils.html) disponibiliza uma boa maneira de escrever testes unitários. Existe uma API mais simples, com [jquense/react-testutil-query](https://github.com/jquense/react-testutil-query).
* Alt nos fornece uma boa maneira para testar nosso código [actions](http://alt.js.org/docs/testing/actions/) e [stores](http://alt.js.org/docs/testing/stores/).
* Testing provides you confidence. This will become particularly important as your codebase grows. It will become harder to break things inadvertently.
Testes oferece confiança. Isso se tornará importante à medida que sua base de código cresce. Será mais difícil quebrar coisas inadvertidamente.

> [Compre o livro](https://leanpub.com/survivejs-react) para acessar o conteúdo restante.
