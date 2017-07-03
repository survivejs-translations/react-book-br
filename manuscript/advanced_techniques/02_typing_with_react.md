# Tipagem em React

Para incentivar as pessoas a apoiarem meu trabalho, decidi publicar uma versão TL;DR deste capítulo para a comunidade. Isso permitirá que eu desenvolva mais o conteúdo, uma estratégia onde todo mundo ganha.

Você pode acessar o capítulo completo comprando uma cópia no [Leanpub](https://leanpub.com/survivejs_react). Nele, nós entraremos em detalhes, enquanto aqui, será uma idéia aproximada do conteúdo do capítulo.

## TL;DR

* [propTypes](https://facebook.github.io/react/docs/reusable-components.html) são ótimos. Use-os para melhorar a manutenção da sua aplicação.
* `propTypes` retorna bons erros durante o desenvolvimento, mas será eliminado na compilação para produção, melhorarando o desempenho da aplicação final.
* [Flow](http://flowtype.org) vai um passo adiante. Ele fornece sintaxe que permite que você digite, gradualmente, tipagem em seu código JavaScript.
* Enquanto `flow` é um analisador estático que você precisa executar separadamente, [babel-plugin-typecheck](https://github.com/codemix/babel-plugin-typecheck) fornece análise estática em tempo de execução durante o desenvolvimento.
* O [TypeScript](http://www.typescriptlang.org/), da Microsoft, é mais uma alternativa. A partir da versão 1.6 ganhará suporte ao JSX.

> [Compre o livro](https://leanpub.com/survivejs-react) para acessar o conteúdo restante.
