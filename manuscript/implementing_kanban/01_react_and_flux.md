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

## Portando para Alt

![Alt](../images/alt.png)

Utilizando Alt, você lida com **actions** e **stores8*. O **dispatcher** está oculto, mas você ainda terá acesso a ele, se necessário. Comparado com outras implementações, Alt esconde muito do boilerplate. Existem recursos especiais que permitem salvar e restaurar o estado do aplicativo. Isso é útil para implementar persistência e renderização universal.

Há alguns passos que devemos realizar para utilizar Alt na nossa aplicação:

1. Configurar uma instância do Alt para coordenar a comunicação entre **actions** e **stores**.
2. Conectar Alt com nossas **views**.
3. Mover nossos dados para **stores**.
4. Definir **actions** para manipular as **stores**.

Faremos isso gradualmente. As porções específicas do Alt ficarão por trás de um adaptador. A abordagem do adaptador nos permite mudar nossa opinião mais tarde, por isso vale a pena implementar.

### Configurando uma instância do Alt

Tudo em Alt começa a partir de uma instância Alt. Ele mantém a comunicação em andamento entre **actions** e **stores**. Para manter as coisas simples, trataremos todos os componentes Alt como um [singleton](https://en.wikipedia.org/wiki/Singleton_pattern). Com esse padrão, reutilizamos a mesma instância dentro de toda a aplicação.

Para conseguir isso, podemos mover esse código para um módulo próprio e depois nos referir a isso em todos os lugares. Dessa maneira:

**app/libs/alt.js**

```javascript
import Alt from 'alt';

const alt = new Alt();

export default alt;
```

Esta é uma maneira padrão de implementar *singletons* usando sintaxe ES6. Ele armazena em cache os módulos, então a próxima vez que você importar Alt de algum lugar, ele retornará a mesma instância novamente.

T> Perceba que `alt.js` deve viver em `app/libs`, e não na raiz do projeto `libs`!

T> O padrão singleton garante que pode haver apenas uma instância. Esse é exatamente o comportamento que queremos aqui.

### Conectando Alt com views

Normalmente, as soluções de gerenciamento de estado fornecem duas partes que você pode usar para conectá-las com um aplicativo React. Estes são um componente `Provider` e a **high order function** `connect` (função que retorna uma função que gera um componente). O `Provider` configura um [context](https://facebook.github.io/react/docs/context.html) em React.

`context` é um recurso avançado que pode ser usado para passar dados através de uma hierarquia de componentes de forma implícita sem passar por `props`. a função `connect` usa o `context` para conectar os dados que queremos e depois passa para um componente.

É possível usar um `connect` através da invocação dessa função ou de um **decorator**, como veremos em breve. O apêndice *Entendendo Decorators* fala mais sobre esse padrão.

Para manter nossa arquitetura de aplicativos fácil de modificar, precisamos configurar dois adaptadores. Um para `Provider` e um para `connect`. Vamos lidar com os detalhes específicos do Alt em ambos os lugares.

### Configurando `Provider`

A fim de manter nosso `Provider` flexível, eu vou usar uma configuração especial. Vamos envolvê-lo dentro de um módulo que escolherá um `Provider` dependendo do nosso ambiente. Isso nos permite usar ferramentas de desenvolvimento sem incluí-lo no pacote de produção. Há alguma configuração adicional envolvida, mas vale a pena dar uma olahda, resultando em um código mais limpo.

O principal aqui é o índice do módulo. CommonJS escolhe o **index.js** de um diretório por padrão quando realizamos uma importação a partir de um diretório. Dado o comportamento que queremos é dinâmico, não podemos confiar nos módulos ES6 aqui.

A idéia é que nossa ferramenta reescreva o código dependendo do `process.env.NODE_ENV`, includingo o módulo para esse ambiente. Aqui está o ponto de entrada do nosso `Provider`:

**app/components/Provider/index.js**

```javascript
if(process.env.NODE_ENV === 'production') {
  module.exports = require('./Provider.prod');
}
else {
  module.exports = require('./Provider.dev');
}
```

Também precisamos dos arquivos que o índice está apontando. A primeira parte é fácil. Precisamos apontar para a nossa instância Alt, conectar com um componente conhecido como `AltContainer` e, em seguida, processar nosso aplicativo dentro dele. É aí que entra `props.children`. É a mesma idéia que antes.

O `AltContainer` nos permitirá conectar os dados de nossa aplicação aos componentes quando implementarmos `connect`. Para chegar lá, aqui está a implementação do nível de produção::

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

A implementação do `Provider` pode mudar de acordo com a solução de gerenciamento de estado que estamos usando. É possível que ele acabe não fazendo nada, mas isso é aceitável. A idéia é que temos um ponto de extensão onde podemos alterar nossa aplicação, se necessário.

Ainda está faltando uma parte, a configuração relacionada ao desenvolvimento. É como a produção, exceto que desta vez podemos habilitar ferramentas específicas de desenvolvimento. Esta é uma boa chance de mover a configuração do *react-addons-perf* para cá, removendo do *app/index.jsx*. Eu também estou ativando [Alt's Chrome Debug] (https://github.com/goatslacker/alt-devtool). Você precisará instalar o plugin do Chrome separadamente se desejar usá-las.

Aqui está o código completo do `Provider` de desenvolvimento:

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

Aquele módulo `setup` nos permite realizar a configuração relacionada com Alt que é comum para o ambiente de produção e desenvolvimento. Por enquanto, basta fazer nada assim:

**app/components/Provider/setup.js**

```javascript
export default alt => {}
```

Nós ainda precisamos conectar o `Provider` com nossa aplicação ajustando *app/index.jsx*. Execute as seguintes alterações para conectá-lo:

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

Se você verificar a saída do Webpack, provavelmente verá que está instalando novas dependências no projeto. Isso é esperado dado as nossas mudanças. O processo pode demorar um pouco para ser concluído. Uma vez concluído, atualize o navegador.

Dado que não alteramos a lógica do aplicativo de forma alguma, tudo deve parecer o mesmo. Um bom próximo passo é implementar um adaptador para conectar dados às nossas **views**.

T> Você pode ver uma idéia semelhante em [react-redux](https://www.npmjs.com/package/react-redux). MobX não precisará de um `Provider`. Nesse caso, nossa implementação simplesmente retornaria `children`.

## Entendendo `connect`

A idéia do `connect` é permitir anexar dados e ações específicas aos componentes. Eu modelei a API a partir do `react-redux`. Felizmente, podemos adaptar vários sistemas de gerenciamento de dados para trabalhar com isso. Veja como você conectaria os dados e ações no `App`:

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

O mesmo pode ser escrito sem decoradores. Esta é a sintaxe que usaremos em nossa aplicação:

```javascript
class App extends React.Component {
  ...
}

export default connect(({lanes}) => ({lanes}), {
  LaneActions
})(App)
```

No caso de você precisar aplicar várias funções em um componente, você poderia usar um utilitário como `compose`, e acabaria escrevendo `compose(a, b)(App)`. Isso seria igual a `a(b(App))` e seria um pouco melhor.

Como os exemplos mostram, `compose` é uma função que retorna uma função. É por isso que chamamos de uma **higher order function**. No final, obtemos um componente disso. Esse "embrulho" nos permite lidar com a conexão de dados.

Poderíamos usar uma **higher order function** para anotar nossos componentes e possibilitar propriedades especiais também. Vamos ver a idéia novamente quando implementarmos arrastar e soltar mais tarde nesta parte. Os decoradores oferecem uma maneira mais agradável de anexar esses tipos de anotações.

Agora que temos uma compreensão básica de como `connect` deve funcionar, podemos implementá-lo.

### Configurando `connect`

Neste caso, vamos usar um `connect` personalizado para destacar algumas idéias-chave. A implementação não é ideal quando se trata de desempenho. No entanto, é suficiente para essa aplicação.

Seria possível otimizar o comportamento com mais esforço. Você pode usar um dos conectores estabelecidos ou desenvolver o seu próprio aqui. Essa é uma razão pela qual ter controle sobre `Provider` e `connect` é útil. Permitindo uma maior personalização e compreensão de como o processo funciona.

Caso tenhamos uma transformação de estado personalizada definida, iremos manipular os dados que precisamos a partir do `Provider` e, em seguida, passar os dados resultantes para nosso componente através de `props`:

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

// `connect` do Alt através de `context`. Isso não está otimizado!
// Se alguma `store` do Alt, forçará uma renderização.
//
// Veja *AltContainer* e *connect-alt* para soluções otimizadas.
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

// Transforma {store: <AltStore>} em {<store>: store.getState()}
function composeStores(stores) {
  let ret = {};

  Object.keys(stores).forEach(k => {
    const store = stores[k];

    // Combinando os estados das `store`
    ret = Object.assign({}, ret, store.getState());
  });

  return ret;
}
```

Como `flux.FinalStore` não estará disponível por padrão, precisaremos alterar nossa instância do Alt para contê-la. Depois disso, podemos acessá-la sempre que for necessário:

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

Para ver `connect` em ação, poderíamos usá-lo para anexar alguns dados temporários à` App` e depois renderizá-lo. Vamos modifica-los da seguinte forma para passar dados `test` para` App` e depois mostrar na interface:

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

Para exibir o texto, atualize o navegador. Você deve ver o texto que nos conectamos à `App`.

## Usando dispatcher em Alt

Mesmo que você possa ir longe sem nunca usar o Flux **dispatcher**, pode ser útil saber sobre isso. Alt fornece duas maneiras de usá-lo. Se você deseja registrar tudo que passa pela sua instância `alt`, você pode usar um trecho, como `alt.dispatcher.register(console.log.bind(console))`. Alternativamente, você pode chamar `this.dispatcher.register(...)` em uma **store**. Esses mecanismos permitem implementar um efetivo sistema de log.

Outros sistemas de gerenciamento de estado fornecem funcionalidades semelhantes. É possível interceptar o fluxo de dados de várias maneiras e até mesmo criar uma lógica personalizada além disso.

## Conclusão

Neste capítulo, discutimos a ideia básica da arquitetura Flux e começamos a portar nosso aplicativo para ela. Nós falamos sobre as preocupações relacionadas ao gerenciamento de estado por trás de um adaptador para permitir a alteração do sistema sem ter que alterar o código relacionado à apresentação. O próximo passo é implementar uma **store** para os dados do nosso aplicativo e definir ações para manipulá-lo.
