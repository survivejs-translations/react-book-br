# Implementando `NoteStore` e `NoteActions`

Agora que temos nossas preocupações relacionadas ao gerenciamento de dados nos lugares certos, podemos nos concentrar na implementação das porções restantes - `NoteStore` e `NoteActions`. Isso encapsulará os dados e a lógica do aplicativo.

Independentemente da solução de gerenciamento de estado que você acabar usando, geralmente há algo equivalente ao redor. No Redux você acabaria usando ações que desencadeiam uma mudança de estado através de um redutor. No MobX, você pode modelar uma API de ação dentro de uma classe ES6. A idéia é que você manipulará os dados dentro da classe e isso fará com que o MobX atualize seus componentes conforme necessário.

A idéia é similar aqui. Vamos configurar ações que irão desencadear métodos nas **stores** que modificam o estado. À medida que o estado muda, nossas **views** serão atualizadas. Para começar, podemos implementar um `NoteStore` e depois definer a lógica para manipulá-la. Uma vez que fizemos isso, iremos concluir nossa implantação da arquitetura Flux.

## Configurando `NoteStore`

Atualmente mantemos o estado da aplicação em `App`. O primeiro passo para portar isso para Alt é definir uma **store** e depois consumi-la a partir daí. Isso irá quebrar a lógica do nosso aplicativo temporariamente, precisamos conectá-lo ao Alt. A criação de uma **store** inicial é um bom passo para esse objetivo geral.

Para configurar uma **store**, precisamos executar três etapas. Precisamos configurá-la, conecte-la com Alt em `Provider`, e finalmente, conecta-la no `App`.

Em Alt, modelamos **stores** usando classes ES6. Aqui está uma implementação mínima do nosso estado atual:

**app/stores/NoteStore.js**

```javascript
import uuid from 'uuid';

export default class NoteStore {
  constructor() {
    this.notes = [
      {
        id: uuid.v4(),
        task: 'Learn React'
      },
      {
        id: uuid.v4(),
        task: 'Do laundry'
      }
    ];
  }
}
```

O próximo passo é conectar a **store** com `Provider`. É aí que o módulo `setup` é útil:

**app/components/Provider/setup.js**

```javascript
leanpub-start-delete
export default alt => {}
leanpub-end-delete
leanpub-start-insert
import NoteStore from '../../stores/NoteStore';

export default alt => {
  alt.addStore('NoteStore', NoteStore);
}
leanpub-end-insert
```

Para provar que nossa configuração funciona, podemos ajustar a `App` para consumir seus dados na **store**. Isso vai quebrar a lógica, uma vez que ainda não temos como atualizar os dados da **store**, mas isso é algo que vamos ver na próxima seção. Mude `App` da seguinte forma para permitir que as notas fiquem a disposição:

**app/components/App.jsx**

```javascript
...

class App extends React.Component {
leanpub-start-delete
  constructor(props) {
    super(props);

    this.state = {
      notes: [
        {
          id: uuid.v4(),
          task: 'Learn React'
        },
        {
          id: uuid.v4(),
          task: 'Do laundry'
        }
      ]
    }
  }
leanpub-end-delete
  render() {
leanpub-start-delete
    const {notes} = this.state;
leanpub-end-delete
leanpub-start-insert
    const {notes} = this.props;
leanpub-end-insert

    return (
      <div>
leanpub-start-delete
        {this.props.test}
leanpub-end-delete
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

leanpub-start-delete
export default connect(() => ({
  test: 'test'
}))(App)
leanpub-end-delete
leanpub-start-insert
export default connect(({notes}) => ({
  notes
}))(App)
leanpub-end-insert
```

Se você atualizar o aplicativo, você deve ver exatamente os mesmos dados que antes. Desta vez, no entanto, estamos consumindo os dados da nossa **store**. Como resultado, nossa lógica está quebrada. Isso é algo que precisamos corrigir na próxima parte, onde iremos definir `NoteActions` e fazer a manipulação de estado para `NoteStore`.

T> Dado que `App` não contém mais estado, seria possivel portá-lo como um componente baseado em função. Muitas vezes, a maioria dos seus componentes será baseado em funções, principalmente por esse motivo. Se você não estiver usando estado ou refs, então é seguro usar funções como padrão.

## Entendendo ações

Ações são um dos conceitos fundamentais da arquitetura Flux. Para ser exato, é uma boa idéia separar **actions** de **action creators**. Muitas vezes, os termos podem ser usados de forma intercambiável, mas há uma diferença considerável.

**action creators** são literalmente funções que *despacham* ações. Os dados dessa ação serão então entregues às **stores** interessadas. Pode ser fácil de pensar neles como mensagens envolvidas em um envelope e depois entregues.

Esta divisão é útil quando você precisa executar ações assíncronas. Você pode, por exemplo, querer obter os dados iniciais do seu quadro Kanban. A operação pode então ter êxito ou falhar. Isso lhe dá três ações separadas para despachar. Você pode despachar ao iniciar a consulta e quando receber alguma resposta.

Todos esses dados são valiosos, pois permitem controlar a interface do usuário. Você pode exibir um widget de progresso enquanto uma consulta está sendo realizada e, em seguida, atualizar o estado do aplicativo uma vez que ele foi buscado do servidor. Se a consulta falhar, você pode deixar o usuário saber sobre isso.

Você pode ver esse tema em diferentes soluções de gerenciamento de estado. Muitas vezes você modela uma ação como uma função que retorna uma função (conhecida como *thunk*) que despacha ações individuais à medida que a consulta assíncrona progride. Em um caso síncrono e ingênuo, é suficiente retornar os dados da ação diretamente.

T> A documentação do Alt fala sobre [ações assíncronas](http://alt.js.org/docs/createActions/) em grandes detalhes.

## Configurando `NoteActions`

Alt fornece um pequeno método auxiliar conhecido como `alt.generateActions` que pode gerar **action creators** para nós. Eles simplesmente enviarão os dados forem passados. Em seguida, conectaremos essas ações nas **stores** relevantes. Neste caso, será o `NoteStore` que definimos anteriormente.

Se tratando da nossa aplicação, vamos modelar as operações básicas de CRUD (Create, Read, Update, Delete). Como a leitura (Read) está implícita, podemos ignorar isso. Mas ter o restante disponível como ações é útil. Configure `NoteActions` usando a abreviatura` alt.generateActions` como a seguir:

**app/actions/NoteActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions('create', 'update', 'delete');
```

Por si só, isso não faz muito. Dado que precisamos fazer o `connect`  das ações com `App` para realmente ativá-las, isso seria um bom lugar para fazer isso. Podemos começar a nos preocupar com ações individuais depois, ao expandir nossa **store**. Para fazer o `connect` das ações, vamos mudar `App` da seguinte maneira:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import Notes from './Notes';
import connect from '../libs/connect';
leanpub-start-insert
import NoteActions from '../actions/NoteActions';
leanpub-end-insert

class App extends React.Component {
  ...
}

leanpub-start-delete
export default connect(({notes}) => ({
  notes
}))(App)
leanpub-end-delete
leanpub-start-insert
export default connect(({notes}) => ({
  notes
}), {
  NoteActions
})(App)
leanpub-end-insert
```

Isso nos dá a API `this.props.NoteActions.create` para desencadear várias ações. Com isso, podemos expandir ainda mais a implementação.

## Conectando `NoteActions` com `NoteStore`

Alt fornece algumas maneiras convenientes de conectar ações a uma loja:

* `this.bindAction(NoteActions.CREATE, this.create)` - Vincula uma ação específica a um método específico.
* `this.bindActions(NoteActions)`- Vincula todas as ações aos métodos por convenção. ex:, ação `create` irá ser mapeada para o método `create`.
* `reduce(state, { action, data })` - É possível implementar um método personalizado conhecido como `reduce`. Isso imita o modo como os redutores do Redux funcionam. A idéia é que você retornará um novo estado com base no estado e dados fornecidos.

Vamos usar `this.bindActions` neste caso, basta confiar na convenção. Ajuste a **store** da seguinte maneira para conectar as ações e adicionar `stubs` iniciais para nossa lógica:

**app/stores/NoteStore.js**

```javascript
import uuid from 'uuid';
leanpub-start-insert
import NoteActions from '../actions/NoteActions';
leanpub-end-insert

export default class NoteStore {
  constructor() {
leanpub-start-insert
    this.bindActions(NoteActions);
leanpub-end-insert

    this.notes = [
      {
        id: uuid.v4(),
        task: 'Learn React'
      },
      {
        id: uuid.v4(),
        task: 'Do laundry'
      }
    ];
  }
leanpub-start-insert
  create(note) {
    console.log('create note', note);
  }
  update(updatedNote) {
    console.log('update note', updatedNote);
  }
  delete(id) {
    console.log('delete note', id);
  }
leanpub-end-insert
}
```

Para realmente vê-lo funcionar, teremos de começar a conectar nossas ações na `App` e começar a controlar a lógica.

## Portando `App.addNote` para Flux

`App.addNote` é um bom ponto de partida. O primeiro passo é associar a ação (`NoteActions.create`) do método e ver se algo aparece no console do navegador. Ao fazermos isso, podemos então, manipular o estado. Adicione a ação dessa maneira:

**app/components/App.jsx**

```javascript
...

class App extends React.Component {
  render() {
    ...
  }
  addNote = () => {
leanpub-start-delete
    // Seria possível escrever isso em um estilo imperativo.
    // ex: através de `this.state.notes.push` e depois
    // `this.setState({notes: this.state.notes})` para
    // persistir a mudança.
    // Eu prefiro favorecer o estilo funcional sempre que possível.
    // Embora possa levar mais código às vezes, eu sinto
    // que os benefícios (fácil de entender, sem efeitos colaterais) valem a pena!
    //
    // Bibliotecas, como `Immutable.js`, vão um passo além!
    this.setState({
      notes: this.state.notes.concat([{
        id: uuid.v4(),
        task: 'New task'
      }])
    });
leanpub-end-delete
leanpub-start-insert
    this.props.NoteActions.create({
      id: uuid.v4(),
      task: 'New task'
    });
leanpub-end-insert
  }
  ...
}

...
```

Se você atualizar e clicar no botão "add note", você deve ver uma mensagem como esta no console do navegador:

```bash
create note Object {id: "62098959-6289-4894-9bf1-82e983356375", task: "New task"}
```

Isso significa que temos os dados que precisamos no método `create` da `NoteStore`. Nós ainda precisamos manipular os dados. Depois disso, iremos concluir o loop e devemos ver novas notas através da interface do usuário. Alt usa uma API semelhante ao React. Veja a implementação abaixo:

**app/stores/NoteStore.js**

```javascript
import uuid from 'uuid';
import NoteActions from '../actions/NoteActions';

export default class NoteStore {
  constructor() {
    ...
  }
  create(note) {
leanpub-start-delete
    console.log('create note', note);
leanpub-end-delete
leanpub-start-insert
    this.setState({
      notes: this.notes.concat(note)
    });
leanpub-end-insert
  }
  ...
}
```

Se você tentar adicionar uma nota agora, a atualização deve ser realizada. Alt está guardando o estado e a edição está funcionando, graças à arquitetura que configuramos. Ainda temos que repetir o processo para os métodos restantes para completar o trabalho.

## Portando `App.deleteNote` para Flux

O processo é exatamente o mesmo para `App.deleteNote`. Precisamos conectá-lo com a nossa ação e depois transferi-lo. Aqui está as mudanças do `App`:

**app/components/App.jsx**

```javascript
...

class App extends React.Component {
  ...
  deleteNote = (id, e) => {
    // Evitando a propagação para `edit`
    e.stopPropagation();

leanpub-start-delete
    this.setState({
      notes: this.state.notes.filter(note => note.id !== id)
    });
leanpub-end-delete
leanpub-start-insert
    this.props.NoteActions.delete(id);
leanpub-end-insert
  }
  ...
}

...
```

Se você atualizar e tentar excluir uma nota, você deve ver uma mensagem como essa no console do navegador:

```bash
delete note 501c13e0-40cb-47a3-b69a-b1f2f69c4c55
```

Para finalizar, precisaremos mover o a lógica do `setState` para o método `delete`. Lembre-se de remover `this.state.notes` e usar apenas `this.notes`:

**app/stores/NoteStore.js**

```javascript
import uuid from 'uuid';
import NoteActions from '../actions/NoteActions';

export default class NoteStore {
  ...
  delete(id) {
leanpub-start-delete
    console.log('delete note', id);
leanpub-end-delete
leanpub-start-insert
    this.setState({
      notes: this.notes.filter(note => note.id !== id)
    });
leanpub-end-insert
  }
}
```

Após essa alteração, você pode apagar as notas como antes. Ainda há alguns métodos para modificar.

## Portando `App.activateNoteEdit` para Flux

`App.activateNoteEdit` é essencialmente, a operação `update`. Nós só precisamos mudar o valor de `editing` da nota para `true`. E isso iniciará o processo de edição. Como de costume, podemos atualizar o `App` da seguinte maneira:

**app/components/App.jsx**

```javascript
...

class App extends React.Component {
  ...
  activateNoteEdit = (id) => {
leanpub-start-delete
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = true;
        }

        return note;
      })
    });
leanpub-end-delete
leanpub-start-insert
    this.props.NoteActions.update({id, editing: true});
leanpub-end-insert
  }
  ...
}

...
```

Se você atualizar e tentar editar uma nota, você deve ver uma mensagem como essa no console do navegador:

```bash
update note Object {id: "2c91ba0f-12f5-4203-8d60-ea673ee00e03", editing: true}
```

Ainda precisamos persistir a mudança para que isso funcione. A lógica é a mesma que em `App`, exceto que, antes, o ela estava ainda mais genérica pois usávamos `Object.assign`:

**app/stores/NoteStore.js**

```javascript
import uuid from 'uuid';
import NoteActions from '../actions/NoteActions';

export default class NoteStore {
  ...
  update(updatedNote) {
leanpub-start-delete
    console.log('update note', updatedNote);
leanpub-end-delete
leanpub-start-insert
    this.setState({
      notes: this.notes.map(note => {
        if(note.id === updatedNote.id) {
          return Object.assign({}, note, updatedNote);
        }

        return note;
      })
    });
leanpub-end-insert
  }
  ...
}
```

Agora, deve ser possível começar a editar uma nota. Se você tentar terminar uma edição, você deve obter um erro como `Uncaught TypeError: Cannot read property 'notes' of null`. Isso acontece porque está faltando uma parte final desse processo, o `App.editNote`.

## Portando `App.editNote` para Flux

Esta parte final é fácil. Já temos a lógica de que precisamos. Agora é apenas uma questão de conectar`App.editNote` da maneira correta. Nós precisaremos chamar o método `update` da seguinte maneira:

**app/components/App.jsx**

```javascript
...

class App extends React.Component {
  ...
  editNote = (id, task) => {
leanpub-start-delete
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = false;
          note.task = task;
        }

        return note;
      })
    });
leanpub-end-delete
leanpub-start-insert
    this.props.NoteActions.update({id, task, editing: false});
leanpub-end-insert
  }
}

...
```

Após a atualização, você pode modificar as tarefas novamente e o aplicativo deve funcionar como antes. Como nós alteramos `NoteStore` através de ações, Isso leva a uma atualização cascata do nosso `App` através do `setState`. Que por sua vez, fará com que o componente execute o método `render`. Esse é o fluxo unidirecional do Flux na prática.

Na verdade, temos mais código agora do que antes, mas está tudo bem. `App` está um pouco mais limpo e vai ser mais fácil de desenvolver, como veremos em breve. O mais importante, conseguimos implementar a arquitetura Flux para nossa aplicação.

T> A implementação atual é ingênua na medida em que não valida os parâmetros de forma alguma. Seria uma boa idéia validar a forma do objeto para evitar incidentes durante o desenvolvimento. [Flow](http://flowtype.org/) fornece uma maneira de fazer isso. Além disso, você pode escrever testes unitários.

### What's the Point?

Even though integrating a state management system took a lot of effort, it was not all in vain. Consider the following questions:

1. Suppose we wanted to persist the notes within `localStorage`. Where would you implement that? One approach would be to handle that at the `Provider` `setup`.
2. What if we had many components relying on the data? We would just consume the data through `connect` and display it, however we want.
3. What if we had many, separate Note lists for different types of tasks? We could set up another store for tracking these lists. That store could refer to actual Notes by id. We'll do something like this in the next chapter, as we generalize the approach.

Adopting a state management system can be useful as the scale of your React application grows. The abstraction comes with some cost as you end up with more code. But on the other hand if you do it right, you'll end up with something that's easy to reason and develop further. Especially the unidirectional flow embraced by these systems helps when it comes to debugging and testing.

## Conclusion

In this chapter, you saw how to port our simple application to use Flux architecture. In the process we learned more about **actions** and **stores** of Flux. Now we are ready to start adding more functionality to our application. We'll add `localStorage` based persistency to the application next and perform a little clean up while at it.
