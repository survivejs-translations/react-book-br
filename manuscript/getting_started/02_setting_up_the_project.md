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

## Running the Project

To get the project running, execute `npm start`. You should see something like this at the terminal if everything went right:

```bash
> webpack-dev-server

http://localhost:8080/
webpack result is served from /
content is served from .../kanban-app
404s will fallback to /index.html
Child html-webpack-plugin for "index.html":

webpack: bundle is now VALID.
```

In case you received an error, make sure there isn't something else running in the same port. You can run the application through some other port easily using an invocation such as `PORT=3000 npm start` (Unix only). The configuration will pick up the new port from the environment. If you want to fix the port to something specific, adjust its value at *webpack.config.js*.

Assuming everything went fine, you should see something like this at the browser:

![Hello world](images/hello_01.png)

You can try modifying the source to see how hot loading works.

I'll discuss the boilerplate in greater detail next so you know how it works. I'll also cover the language features we are going to use briefly.

T> The techniques used by the boilerplate are covered in greater detail at [SurviveJS - Webpack](http://survivejs.com/webpack/introduction/).

## Boilerplate npm `scripts`

Our boilerplate is able to generate a production grade build with hashing. There's also a deployment related target so that you can show your project to other people through [GitHub Pages](https://pages.github.com/). I've listed all of the `scripts` below:

* `npm run start` (or `npm start`) - Starts the project in the development mode. Surf to `localhost:8080` in your browser to see it running.
* `npm run build` - Generates a production build below `build/`. You can open the generated *index.html* through the browser to examine the result.
* `npm run deploy` - Deploys the contents of `build/` to the *gh-pages* branch of your project and pushes it to GitHub. You can access the project below `<user>.github.io/<project>` after that. Before this can work correctly, you should set `publicPath` at *webpack.config.js* to match your project name on GitHub.
* `npm run stats` - Generates statistics (*stats.json*) about the project. You can [analyze the build output](http://survivejs.com/webpack/building-with-webpack/analyzing-build-statistics/) further.
* `npm run test` (or `npm test`) - Executes project tests. The *Testing React* chapter digs deeper into the topic. In fact, writing tests against your components can be a good way to learn to understand React better.
* `npm run test:tdd` - Executes project tests in TDD mode. This means it will watch for changes and run the tests when changes are detected allowing you to develop fast without having to run the tests manually.
* `npm run test:lint` - Executes [ESLint](http://eslint.org/) against the code. ESLint is able to catch smaller issues. You can even configure your development environment to work with it. This allows you to catch potential mistakes as you make them. Our setup lints even during development so you rarely need to execute this command yourself.

Study the `"scripts"` section of *package.json* to understand better how each of these works. There is quite a bit configuration. See [SurviveJS - Webpack](http://survivejs.com/webpack/introduction/) to dig deeper into the topic.

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
