# Configurando o Projeto

Para ser mais fácil começar, eu criei um projeto simples baseado no webpack, nos permitindo começar com React imediatamente. O esqueleto inclui um modo de desenvolvimento com um recurso conhecido como *hot loading* ativado.

*Hot loading* permite que o webpack atualize parte do código em execução no navegador sem uma atualização completa. Funciona especialmente muito bem com estilos/css, embora, o suporte para componentes React também funciona muito bem.

Infelizmente, não é uma tecnologia à prova de balas e não será capaz de detectar todas as alterações feitas no código. Isso significa que haverá momentos em que você precisa forçar uma atualização completa para que o navegador capture as mudanças recentes.

T> Editores comuns (Sublime Text, Visual Studio Code, vim, emacs, Atom and such) oferecem um bom suporte para React. Mesmo IDEs, como [WebStorm](https://www.jetbrains.com/webstorm/), suportam até certo ponto. [Nuclide](http://nuclide.io/), uma IDE baseada no Atom, foi desenvolvido com o React em mente. Certifique-se de que os plugins relacionados ao React estão instalados e habilitados.

W> Se você usar uma IDE, desative um recurso conhecido como **safe write**. Ele é conhecido por causar problemas com a configuração que usaremos neste livro.

## Configurando Node.js e Git

Para começar, certifique-se de ter as novas versões do [Node.js](https://nodejs.org) e [Git](https://git-scm.com/) estão instalado. Eu recomendo usar pelo menos a versão LTS do Node.js. Você pode encontrar problemas difíceis de debugar com versões antigas. O mesmo pode acontecer para versões mais recentes do LTS devido ao seu estado de constante desenvolvimento.

T> Uma opção interessante é gerenciar seu ambiente através do [Vagrant](https://www.vagrantup.com/) ou ferramentas como [nvm](https://www.npmjs.com/package/nvm).

### Fazendo o downloading do Boilerplate

In order to fetch the boilerplate our project needs, clone it through Git as follows at your terminal:
Para fazer o download do boilerplate, para as necessidades do nosso projeto, vamos clonar através do Git usando o seguinte comando em seu terminal:

```bash
git clone https://github.com/survivejs/react-boilerplate.git kanban-app
```

Isso criará um novo diretório, *kanban-app*. Dentro dele você pode encontrar tudo o que precisamos para avançar. Como o boilerplate pode mudar entre as versões do livro, eu recomendo que você verifique a versão específica dele:

```bash
cd kanban-app
git checkout v2.5.6
```

O repositório contém um pequeno aplicativo de exemplo que mostra o `Hello World!` e a configuração básica do webpack. Para instalar as dependências execute:

```bash
npm install
```

Após a conclusão, você deve ver um diretório `node_modules /` contendo as dependências do projeto.

### Crie um repositório Git novo para o seu projeto

O seu novo projeto inclui o histórico do projeto `react-boilerplate`. Esse histórico não é realmente relevante para o seu novo projeto, agora é um bom momento para limpar o histórico do Git e iniciar um novo repositório. Este novo repositório irá refletir a evolução do seu projeto. No seu *commit* inicial, você pode querer mencionar a versão inicial do boilerplate.

```
rm -rf .git
git init
git add .
git commit -am "New project based on react-boilerplate (v2.5.6)"
```

Após este processo, você tem um novo projeto para trabalhar.

## Executando o projeto

Para executar o projeto, execute `npm start`. Você deve ver algo assim no terminal:

```bash
> webpack-dev-server

http://localhost:8080/
webpack result is served from /
content is served from .../kanban-app
404s will fallback to /index.html
Child html-webpack-plugin for "index.html":

webpack: bundle is now VALID.
```

Caso tenha recebido um erro, certifique-se de que não haja outro processo em execução na mesma porta. Você pode executar o aplicativo através de alguma outra porta facilmente, usando o comando com `PORT=3000 npm start` (somente para Unix). A configuração irá usar essa nova porta para o ambiente. Se você deseja configurar a porta para algum número específico, ajuste seu valor no *webpack.config.js*.

Vamos supor que tudo correu bem, você deveria ver algo assim no navegador:

![Hello world](images/hello_01.png)

Você pode tentar modificar a fonte para ver o *hot loading* em funcionamento.

Eu vou discutir o boilerplate em maior detalhe a seguir, então você saberá como ele funciona. Também abordarei as funcionalidades da linguagem que usaremos brevemente.

T> As técnicas utilizadas pelo *boilerplate* são cobertas em maior detalhe em [SurviveJS - Webpack](http://survivejs.com/webpack/introduction/).

## npm `scripts` no Boilerplate

Nosso boilerplate é capaz de gerar uma compilação para produção com *hashing*. Há também um processo de compilação para [páginas do GitHub] (https://pages.github.com/). Eu listei todos os `script` abaixo:

* `npm run start` (ou `npm start`) - Inicia o projeto em modo de desenvolvimento. Navegue até `localhost:8080` para vê-lo funcionando.
* `npm run build` - Irá iniciar a compilação para produção gerando a pasta `build/`. Você pode abrir o *index.html* no seu navegador para examinar o resultado.
* `npm run deploy` - Faz o deploy da pasta `build/` para a *branch* *gh-pages* do seu projeto e a envia para o GitHub. Você pode acessar o projeto através da URL `<user>.github.io/<project>`. Mas antes, para fazer isso funcionar, você deve configurar o `publicPath` no *webpack.config.js* para combinar com o nome do seu projeto no GitHub.
* `npm run stats` - Gera estatísticas (*stats.json*) sobre o projeto. Após isso, você pode [analisar o resultado do seu projeto](http://survivejs.com/webpack/building-with-webpack/analyzing-build-statistics/).
* `npm run test` (ou `npm test`) - Executa os testes do projeto. No capítulo *Testando React*, iremos falar mais sobre esse tópico. De fato, escrever testes para seus componentes pode ser uma boa maneira de aprender e entender melhor sobre o React.
* `npm run test:tdd` - Executa os testes do projeto em TDD. Isso significa que iremos aguardar mudanças nos seus arquivos e executar os testes quando essas mudanças forem detectadas, permitindo que você desenvolva de uma maneira mais rápida, sem a necessidade de rodar os testes manualmente.
* `npm run test:lint` - Executa [ESLint](http://eslint.org/) no seu código. ESLint é capaz de capturar problemas menores. Você pode até mesmo configurar seu ambiente de desenvolvimento para trabalhar com ele. Isso permite que você capture possíveis erros o mais cedo possível. Nosso projeto já vem configurado, mesmo durante o desenvolvimento, você raramente precisa executar esse comando.

Verifique a parte dos `"scripts"` no seu *package.json* para melhor entender como cada um deles funciona. Há um quantidade considerável de configuração. Veja [SurviveJS - Webpack](http://survivejs.com/webpack/introduction/) para saber mais sobre o tópico.

## Boilerplate Language Features

![Babel](images/babel.png)

The boilerplate relies on a transpiler known as [Babel](https://babeljs.io/). It allows us to use features from the future of JavaScript. It transforms your code to a format understandable by the browsers. You can even use it to develop your own language features. It supports JSX through a plugin.

Babel provides support for certain [experimental features](https://babeljs.io/docs/plugins/#stage-x-experimental-presets-) from ES7 beyond standard ES6. Some of these might make it to the core language while some might be dropped altogether. The language proposals have been categorized within stages:

* **Stage 0** - Strawman
* **Stage 1** - Proposal
* **Stage 2** - Draft
* **Stage 3** - Candidate
* **Stage 4** - Finished

I would be very careful with **stage 0** features. The problem is that if the feature changes or gets removed you will end up with broken code and will need to rewrite it. In smaller experimental projects it may be worth the risk, though.

In addition to standard ES2015 and JSX, we'll be using a few custom features in this project. I've listed them below. See the *Language Features* appendix to learn more of each.

* [Property initializers](https://github.com/jeffmo/es-class-static-properties-and-fields) - Example: `addNote = (e) => {`. This binds the `addNote` method to an instance automatically. The feature makes more sense as we get to use it.
* [Decorators](https://github.com/wycats/javascript-decorators) - Example: `@DragDropContext(HTML5Backend)`. These annotations allow us to attach functionality to classes and their methods.
* [Object rest/spread](https://github.com/sebmarkbage/ecmascript-rest-spread) - Example: `const {a, b, ...props} = this.props`. This syntax allows us to easily extract specific properties from an object.

In order to make it easier to set up the features, I created [a specific preset](https://github.com/survivejs/babel-preset-survivejs-kanban). It also contains [babel-plugin-transform-object-assign](https://www.npmjs.com/package/babel-plugin-transform-object-assign) and [babel-plugin-array-includes](https://www.npmjs.com/package/babel-plugin-array-includes) plugins. The former allows us to use `Object.assign` while the latter provides `Array.includes` without having to worry about shimming these for older environments.

A preset is simply a npm module exporting Babel configuration. Maintaining presets like this can be useful especially if you want to share the same set of functionality across multiple projects.

T> You can [try out Babel online](https://babeljs.io/repl/) to see what kind of code it generates.

T> If you are interested in a lighter alternative, check out [Bublé](https://gitlab.com/Rich-Harris/buble).

## Conclusion

Now that we have a simple "Hello World!" application running, we can focus on development. Developing and getting into trouble is a good way to learn after all.
