# React e Flux

Você pode ir bastante longe mantendo tudo em componentes. Essa é uma maneira inteiramente válida de começar. Os problemas começam quando você adiciona o estado ao seu aplicativo e precisa compartilhá-lo em diferentes partes. Esta é a razão pela qual várias soluções de gerenciamento de estado surgiram. Cada um desses tenta resolver o problema à sua maneira.

A primeira solução para esse problema foi [arquitetura Flux](https://facebook.github.io/flux/docs/overview.html). Ele permite modelar sua aplicação em termos de **Actions**, **Stores**, e **Views**. Também tem uma parte conhecida como **Dispatcher** para gerenciar ações e permitir que você modelo dependências entre diferentes chamadas.

Esta separação é particularmente útil quando você está trabalhando com grandes equipes. O fluxo unidireccional torna fácil de dizer o que está acontecendo. Esse é um tema comum em várias soluções de gerenciamento de dados disponíveis para o React.

## Introdução rápida ao Redux

Uma solução conhecida como [Redux](http://redux.js.org/) tomou as idéias centrais do Flux e as empurrou para uma certa forma. Redux é mais uma diretriz, embora poderosa, que dá à sua aplicação certa estrutura e modela preocupações relacionadas a dados de uma determinada maneira. Você manterá o estado do seu aplicativo em uma única árvore que você altera usando funções puras (sem efeitos colaterais) através de redutores.

Isso pode soar um pouco complexo, mas, na prática, o Redux torna o fluxo de dados muito explícito. Flux por si só, não é tão apreciado em certas partes. Eu acredito que entender Flux antes de mergulhar no Redux é uma boa jogada, pois você pode ver temas compartilhados em ambos.

## Introdução rápida ao MobX

[MobX](https://mobxjs.github.io/mobx/) assume uma visão completamente diferente sobre o gerenciamento de dados. Se o Redux ajuda a modelar o fluxo de dados explicitamente, o MobX faz uma grande parte disso implícito. Não o força a nenhuma estrutura específica. Ao invés disso, você anotará suas estruturas de dados com **observable** e MobX irá lidar com a atualização do modelo e suas **views**.

Enquanto a Redux abraça o conceito de imutabilidade através da idéia de redutores, a MobX faz algo oposta e depende da mutação. Isso significa que aspectos como o gerenciamento de referência podem ser surpreendentemente simples no MobX enquanto que no Redux você provavelmente será forçado a normalizar seus dados para que seja fácil manipular através de redutores.

Tanto o Redux como o MobX são valiosos em suas próprias formas. Não há uma solução certa quando se trata de gerenciamento de dados. Tenho certeza de que mais alternativas aparecerão com o passar do tempo. Cada solução vem com seus prós / contras. Ao entender as alternativas, você tem uma melhor chance de escolher uma solução que se ajuste aos seus propósitos em um determinado momento.

## Qual solução de gerenciamento de dados usar?

A situação de gerenciamento de dados está mudando constantemente. No momento [Redux](http://rackt.org/redux/) é uma opção bem usada, mas existem boas alternativas. [voronianski/flux-comparison](https://github.com/voronianski/flux-comparison) oferece uma boa comparação entre alguns dos mais populares.

Ao escolher uma biblioteca, ela se resume às suas próprias preferências pessoais. Você terá que considerar fatores, como API, recursos, documentação e suporte. Começar com uma das alternativas mais populares pode ser uma boa idéia. À medida que você começa a entender a arquitetura, você pode fazer escolhas que melhor atendam você.

Neste aplicativo, usaremos uma implementação Flux conhecida como [Alt](http://alt.js.org/). A API é pura e suficiente para nossos propósitos. Como bônus, a Alt foi projetada a renderização universal (isomórfica) em mente. Se você entender o Flux, você tem um bom ponto de partida para entender as alternativas.

O livro ainda não cobre as soluções alternativas, mas nós vamos projetar nossa arquitetura de aplicativos para que seja possível inserir alternativas mais tarde. A idéia é que isolamos nossa porção do gerenciamento de dados para que possamos trocar peças sem ajustar nosso código React. É uma maneira de projetar para a mudança.

## Introdução ao Flux

![Unidirectional Flux dataflow](../images/flux_linear.png)

Até agora, lidamos apenas com **views**. A arquitetura Flux introduz novos conceitos para a mistura. São **actions**, **dispatcher** e **store**. Flux implementa um fluxo unidirecional em contraste com estruturas populares, como Angular ou Ember. Mesmo que as ligações bidirecionais possam ser convenientes, elas vêm com um custo. Pode ser difícil deduzir o que está acontecendo e por quê.

### Actions e Stores

O Flux não é inteiramente simples de entender, pois há muitos conceitos a se preocupar. No nosso caso, vamos modelar `NoteActions` e `NoteStore`. `NoteActions` fornecerá operações concretas que podemos realizar ao longo dos nossos dados. Por exemplo, podemos ter `NoteActions.create({task: 'Learn React'})`.

### Dispatcher

Quando desencadearmos uma ação, o **dispatcher** será notificado. O **dispatcher** poderá lidar com possíveis dependências entre **stores**. É possível que uma certa ação precise acontecer antes de outra. O **dispatcher** nos permite fazer isso.

Em uma analogia simples, as ações podem apenas passar uma mensagem para o **dispatcher**. Eles também podem desencadear consultas assíncronas e chamar o **dispatcher** com base no resultado. Isso nos permite lidar com dados recebidos e possíveis erros.

Uma vez que o **dispatcher** tenha lidado com uma ação, as **stores** que estão "escutando" essas ações, são ativadas. No nosso caso, o `NoteStore` é notificado. Como resultado, ele poderá atualizar seu estado interno. Depois de fazer isso, ele notificará possíveis "ouvintes" do novo estado.

### Flux e o fluxo de dados

Isso completa o processo de fluxo unidirecional, mesmo que ainda linear, do Flux. Normalmente, porém, o processo unidirecional tem um fluxo cíclico e não necessariamente termina. O diagrama a seguir ilustra um fluxo mais comum. É a mesma idéia novamente, mas com a adição de um ciclo de retorno. Eventualmente, os componentes dependendo dos dados da nossa **store** irão sse atualizar através deste processo de looping.

![Cyclical Flux dataflow](../images/flux.png)

Isso soa como uma série de passos para alcançar algo simples como criar uma nova `Note`. A abordagem vem com seus benefícios. Dado que o fluxo está sempre em uma única direção, é fácil rastrear e depurar. Se houver algo errado, está em algum lugar dentro do ciclo.

Melhor ainda, podemos consumir os mesmos dados em nossa aplicação. Você apenas conectará sua **view** a uma **store** e é isso. Este é um dos grandes benefícios de usar uma solução de gerenciamento de estado.

### Vantagens do Flux

Mesmo que isso pareça um pouco complicado, o arranjo dá flexibilidade na aplicação. Podemos, por exemplo, implementar a comunicação com uma API, armazenamento em cache e i18n fora das nossas **views**. Desta forma, ficamos com uma lógica limpa, mantendo a aplicação mais fácil de entender.

A implementação da arquitetura Flux no seu aplicativo aumentará um pouco a quantidade de código. É importante entender que minimizar a quantidade de código escrito não é o objetivo do Flux. Ele foi projetado para permitir a produtividade em equipes maiores. Você poderia dizer que explícito é melhor do que implícito.

## Porting to Alt

![Alt](images/alt.png)

In Alt, you'll deal with actions and stores. The dispatcher is hidden, but you will still have access to it if needed. Compared to other implementations, Alt hides a lot of boilerplate. There are special features to allow you to save and restore the application state. This is handy for implementing persistency and universal rendering.

There are a couple of steps we must take to push our application state to Alt:

1. Set up an Alt instance to keep track of actions and stores and to coordinate communication.
2. Connect Alt with views.
3. Push our data to a store.
4. Define actions to manipulate the store.

We'll do this gradually. The Alt specific portions will go behind adapters. The adapter approach allows us to change our mind later easier so it's worth implementing.

### Setting Up an Alt Instance

Everything in Alt begins from an Alt instance. It keeps track of actions and stores and keeps communication going on. To keep things simple, we'll be treating all Alt components as a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern). With this pattern, we reuse the same instance within the whole application.

To achieve this we can push it to a module of its own and then refer to that from everywhere. Configure it as follows:

**app/libs/alt.js**

```javascript
import Alt from 'alt';

const alt = new Alt();

export default alt;
```

This is a standard way to implement *singletons* using ES6 syntax. It caches the modules so the next time you import Alt from somewhere, it will return the same instance again.

T> Note that `alt.js` should go below `app/libs`, not project root `libs`!

T> The singleton pattern guarantees that there can be only one instance. That is exactly the behavior we want here.

### Connecting Alt with Views

Normally state management solutions provide two parts you can use to connect them with a React application. These are a `Provider` component and a `connect` higher order function (function returning function generating a component). The `Provider` sets up a React [context](https://facebook.github.io/react/docs/context.html).

Context is an advanced feature that can be used to pass data through a component hierarchy implicitly without going through props. The `connect` function uses the context to dig the data we want and then passes it to a component.

It is possible to use a `connect` through function invocation or a decorator as we'll see soon. The *Understanding Decorators* appendix digs deeper into the pattern.

To keep our application architecture easy to modify, we'll need to set up two adapters. One for `Provider` and one for `connect`. We'll deal with Alt specific details in both places.

### Setting Up a `Provider`

In order to keep our `Provider` flexible, I'm going to use special configuration. We'll wrap it within a module that will choose a `Provider` depending on our environment. This enables us to use development tooling without including it to the production bundle. There's some additional setup involved, but it's worth it given you end up with a cleaner result.

The core of this arrangement is the index of the module. CommonJS picks up the **index.js** of a directory by default when we perform an import against the directory. Given the behavior we want is dynamic, we cannot rely on ES6 modules here.

The idea is that our tooling will rewrite the code depending on `process.env.NODE_ENV` and choose the actual module to include based on that. Here's the entry point of our `Provider`:

**app/components/Provider/index.js**

```javascript
if(process.env.NODE_ENV === 'production') {
  module.exports = require('./Provider.prod');
}
else {
  module.exports = require('./Provider.dev');
}
```

We also need the files the index is pointing at. The first part is easy. We'll need to point to our Alt instance there, connect it with a component known as `AltContainer`, and then render our application within it. That's where `props.children` comes in. It's the same idea as before.

`AltContainer` will enable us to connect the data of our application at component level when we implement `connect`. To get to the point, here's the production level implementation:

**app/components/Provider/Provider.prod.jsx**

```javascript
import React from 'react';
import AltContainer from 'alt-container';
import alt from '../../libs/alt';
import setup from './setup';

setup(alt);

export default ({children}) =>
  <AltContainer flux={alt}>
    {children}
  </AltContainer>
```

The implementation of `Provider` can change based on which state management solution we are using. It is possible it ends up doing nothing, but that's acceptable. The idea is that we have an extension point where to alter our application if needed.

We are still missing one part, the development related setup. It is like the production one except this time we can enable development specific tooling. This is a good chance to move the *react-addons-perf* setup here from the *app/index.jsx* of the application. I'm also enabling [Alt's Chrome debug utilities](https://github.com/goatslacker/alt-devtool). You'll need to install the Chrome portion separately if you want to use those.

Here's the full code of the development provider:

**app/components/Provider/Provider.dev.jsx**

```javascript
import React from 'react';
import AltContainer from 'alt-container';
import chromeDebug from 'alt-utils/lib/chromeDebug';
import alt from '../../libs/alt';
import setup from './setup';

setup(alt);

chromeDebug(alt);

React.Perf = require('react-addons-perf');

export default ({children}) =>
  <AltContainer flux={alt}>
    {children}
  </AltContainer>
```

That `setup` module allows us to perform Alt related setup that's common for both production and development environment. For now it's enough to do nothing there like this:

**app/components/Provider/setup.js**

```javascript
export default alt => {}
```

We still need to connect the `Provider` with our application by tweaking *app/index.jsx*. Perform the following changes to hook it up:

**app/index.jsx**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
leanpub-start-insert
import Provider from './components/Provider';
leanpub-end-insert

leanpub-start-delete
if(process.env.NODE_ENV !== 'production') {
  React.Perf = require('react-addons-perf');
}
leanpub-end-delete

ReactDOM.render(
leanpub-start-delete
  <App />,
leanpub-end-delete
leanpub-start-insert
  <Provider><App /></Provider>,
leanpub-end-insert
  document.getElementById('app')
);
```

If you check out Webpack output, you'll likely see it is installing new dependencies to the project. That's expected given the changes. The process might take a while to complete. Once completed, refresh the browser.

Given we didn't change the application logic in any way, everything should still look the same. A good next step is to implement an adapter for connecting data to our views.

T> You can see a similar idea in [react-redux](https://www.npmjs.com/package/react-redux). MobX won't need a Provider at all. In that case our implementation would simply return `children`.

## Understanding `connect`

The idea of `connect` is to allow us to attach specific data and actions to components. I've modeled the API after react-redux. Fortunately we can adapt various data management systems to work against it. Here's how you would connect lane data and actions with `App`:

```javascript
@connect(({lanes}) => ({lanes}), {
  laneActions: LaneActions
})
export default class App extends React.Component {
  render() {
    return (
      <div>
        <button className="add-lane" onClick={this.addLane}>+</button>
        <Lanes lanes={this.props.lanes} />
      </div>
    );
  }
  addLane = () => {
    this.props.laneActions.create({name: 'New lane'});
  }
}
```

The same could be written without decorators. This is the syntax we'll be using in our application:

```javascript
class App extends React.Component {
  ...
}

export default connect(({lanes}) => ({lanes}), {
  LaneActions
})(App)
```

In case you need to apply multiple higher order functions against a component, you could use an utility like `compose` and end up with `compose(a, b)(App)`. This would be equal to `a(b(App))` and it would read a little better.

As the examples show, `compose` is a function returning a function. That's why we call it a higher order function. In the end we get a component out of it. This wrapping allows us to handle our data connection concern.

We could use a higher order function to annotate our components to give them other special properties as well. We will see the idea again when we implement drag and drop later in this part. Decorators provide a nicer way to attach these types of annotations. The *Understanding Decorators* appendix delves deeper into the topic.

Now that we have a basic understanding of how `connect` should work, we can implement it.

### Setting Up `connect`

In this case I'm going to plug in a custom `connect` to highlight a couple of key ideas. The implementation isn't optimal when it comes to performance. It is enough for this application, though.

It would be possible to optimize the behavior with further effort. You could use one of the established connectors instead or develop your own here. That's one reason why having control over `Provider` and `connect` is useful. It allows further customization and understanding of how the process works.

In case we have a custom state transformation defined, we'll dig the data we need from the `Provider`, apply it over our data as we defined, and then pass the resulting data to the component through props:

**app/libs/connect.jsx**

```javascript
import React from 'react';

export default (state, actions) => {
  if(typeof state === 'function' ||
    (typeof state === 'object' && Object.keys(state).length)) {
    return target => connect(state, actions, target);
  }

  return target => props => (
    <target {...Object.assign({}, props, actions)} />
  );
}

// Connect to Alt through context. This hasn't been optimized
// at all. If Alt store changes, it will force render.
//
// See *AltContainer* and *connect-alt* for optimized solutions.
function connect(state = () => {}, actions = {}, target) {
  class Connect extends React.Component {
    componentDidMount() {
      const {flux} = this.context;

      flux.FinalStore.listen(this.handleChange);
    }
    componentWillUnmount() {
      const {flux} = this.context;

      flux.FinalStore.unlisten(this.handleChange);
    }
    render() {
      const {flux} = this.context;
      const stores = flux.stores;
      const composedStores = composeStores(stores);

      return React.createElement(target,
        {...Object.assign(
          {}, this.props, state(composedStores), actions
        )}
      );
    }
    handleChange = () => {
      this.forceUpdate();
    }
  }
  Connect.contextTypes = {
    flux: React.PropTypes.object.isRequired
  }

  return Connect;
}

// Transform {store: <AltStore>} to {<store>: store.getState()}
function composeStores(stores) {
  let ret = {};

  Object.keys(stores).forEach(k => {
    const store = stores[k];

    // Combine store state
    ret = Object.assign({}, ret, store.getState());
  });

  return ret;
}
```

As `flux.FinalStore` won't be available by default, we'll need to alter our Alt instance to contain it. After that we can access it whenever we happen to need it:

**app/libs/alt.js**

```javascript
import Alt from 'alt';
leanpub-start-insert
import makeFinalStore from 'alt-utils/lib/makeFinalStore';
leanpub-end-insert

leanpub-start-delete
const alt = new Alt();

export default alt;
leanpub-end-delete
leanpub-start-insert
class Flux extends Alt {
  constructor(config) {
    super(config);

    this.FinalStore = makeFinalStore(this);
  }
}

const flux = new Flux();

export default flux;
leanpub-end-insert
```

In order to see `connect` in action, we could use it to attach some dummy data to `App` and then render it. Adjust it as follows to pass data `test` to `App` and then show it in the user interface:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import Notes from './Notes';
leanpub-start-insert
import connect from '../libs/connect';
leanpub-end-insert

leanpub-start-delete
export default class App extends React.Component {
leanpub-end-delete
leanpub-start-insert
class App extends React.Component {
leanpub-end-insert
  constructor(props) {
    ...
  }
  render() {
    const {notes} = this.state;

    return (
      <div>
leanpub-start-insert
        {this.props.test}
leanpub-end-insert
        <button className="add-note" onClick={this.addNote}>+</button>
        <Notes
          notes={notes}
          onNoteClick={this.activateNoteEdit}
          onEdit={this.editNote}
          onDelete={this.deleteNote}
          />
      </div>
    );
  }
  ...
}

leanpub-start-insert
export default connect(() => ({
  test: 'test'
}))(App)
leanpub-end-insert
```

To make the text show up, refresh the browser. You should see the text that we connected to `App` now.

## Dispatching in Alt

Even though you can get far without ever using Flux dispatcher, it can be useful to know something about it. Alt provides two ways to use it. If you want to log everything that goes through your `alt` instance, you can use a snippet, such as `alt.dispatcher.register(console.log.bind(console))`. Alternatively, you could trigger `this.dispatcher.register(...)` at a store constructor. These mechanisms allow you to implement effective logging.

Other state management systems provide similar hooks. It is possible to intercept the data flow in many ways and even build custom logic on top of that.

## Conclusion

In this chapter we discussed the basic idea of the Flux architecture and started porting our application to it. We pushed the state management related concerns behind an adapter to allow altering the underlying system without having to change the view related code. The next step is to implement a store for our application data and define actions to manipulate it.
