# Alterando nosso esquema de dados

Até o momento, nós desenvolvemos uma aplicação para salvar notas no `localStroage`. Para ficar mais próximo de um quadro Kanban, nós precisamos modelar o conceito de `Lane`. Uma `Lane` é algo que deve ser capaz de salvar várias `Notes` e sua ordem. Um modo de modelar dessa idéia seria manter um array de `Note` ids em `Lane` para conversar com `Notes`.

Essa relação poderia ser inversa. Uma `Note` poderia conter um id de uma `Lane` e manter informação sobre sua posição naquela `Lane`. Nesse caso, nós iremos utilizar a primeira abordagem, pois ao reordenar as notas, essa estrutura irá facilitar nosso trabalho.

## Definindo `Lanes`

Como antes, iremos dividir a implementação em dois componentes. Um componente de alto nível, `Lanes`, e um outro de baixo nível, `Lane`. Nosso componente de alto nível ficará responsável pela ordenação. Uma `Lane` será responsável por renderizar conteúdo (ex: nome e `Notes`) e ter algumas operações de manipulações.

ar adicionar algumas ações. Por hora, vamos implementar a ação de criar uma novas `Lanes`, como abaixo:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions('create');
```

Além disso, nós iremos precisar de uma `LaneStore` e um método `create`. A idéia é e implementação é similar ao `NoteStore`. O método `create` irá concatenar uma nova `Lane` na lista de `Lanes`. Depois disso, a mudança será propagada para os ouvintes (ex: `FinalStore` e componentes).

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  constructor() {
    this.bindActions(LaneActions);

    this.lanes = [];
  }
  create(lane) {
    // Se `notes` não estiver definido, por alguma razão
    // usamos um array em branco como padrão
    lane.notes = lane.notes || [];

    this.setState({
      lanes: this.lanes.concat(lane)
    });
  }
}
```

Para conectar `LaneStore` com nossa aplicação, nós precisamos conecta-la através do `setup`:

**app/components/Provider/setup.js**

```javascript
import storage from '../../libs/storage';
import persist from '../../libs/persist';
import NoteStore from '../../stores/NoteStore';
leanpub-start-insert
import LaneStore from '../../stores/LaneStore';
leanpub-end-insert

export default alt => {
  alt.addStore('NoteStore', NoteStore);
leanpub-start-insert
  alt.addStore('LaneStore', LaneStore);
leanpub-end-insert

  persist(alt, storage(localStorage), 'app');
}
```

Nós também iremos precisar de um *container* componente para renderizar nossas  `Lanes`:

**app/components/Lanes.jsx**

```javascript
import React from 'react';
import Lane from './Lane';

export default ({lanes}) => (
  <div className="lanes">{lanes.map(lane =>
    <Lane className="lane" key={lane.id} lane={lane} />
  )}</div>
)
```

E finalmente, nós podemos adicionar um pequeno `stub` para `Lane` e ter certeza que nossa aplicação não irá quebrar ao conectar `Lanes`. Aliás, nós iremos mover bastante lógica do `App` para cá:

**app/components/Lane.jsx**

```javascript
import React from 'react';

export default ({lane, ...props}) => (
  <div {...props}>{lane.name}</div>
)
```

## Conectando `Lanes` com `App`

Em seguida, nós precisamos adicionar `Lanes` para o `App`. Nós iremos substituir a referência de `Notes` por `Lanes`, também precisamos configurar as ações e `store` para `Lanes`. Isso significa que boa parte do antigo código irá desaparecer. Vamos atualizar `App` da seguinte maneira:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import connect from '../libs/connect';
import Lanes from './Lanes';
import LaneActions from '../actions/LaneActions';

const App = ({LaneActions, lanes}) => {
  const addLane = () => {
    LaneActions.create({
      id: uuid.v4(),
      name: 'New lane'
    });
  };

  return (
    <div>
      <button className="add-lane" onClick={addLane}>+</button>
      <Lanes lanes={lanes} />
    </div>
  );
};

export default connect(({lanes}) => ({
  lanes
}), {
  LaneActions
})(App)
```

 muita coisa. Você poderá apenas adicionar novas `Lanes` no nosso quadro Kanban e ver o botão "New lane" e cada um. Para recuperar a funcionalidade de adicionar notas, nós precisamos modelar `Lane` um pouco mais.

## Modelando `Lane`

`Lane` irá renderizar o nome e `Notes` associadas a esse modelo. A implementação de `Lane` abaixo foi alterada para esse propóisito, sendo bem diferente desde a primeira implementação do nosso `App`. Você pode substituir seu conteúdo por:

**app/components/Lane.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
import Notes from './Notes';

const Lane = ({
  lane, notes, NoteActions, ...props
}) => {
  const editNote = (id, task) => {
    NoteActions.update({id, task, editing: false});
  };
  const addNote = e => {
    e.stopPropagation();

    const noteId = uuid.v4();

    NoteActions.create({
      id: noteId,
      task: 'New task'
    });
  };
  const deleteNote = (noteId, e) => {
    e.stopPropagation();

    NoteActions.delete(noteId);
  };
  const activateNoteEdit = id => {
    NoteActions.update({id, editing: true});
  };

  return (
    <div {...props}>
      <div className="lane-header">
        <div className="lane-add-note">
          <button onClick={addNote}>+</button>
        </div>
        <div className="lane-name">{lane.name}</div>
      </div>
      <Notes
        notes={notes}
        onNoteClick={activateNoteEdit}
        onEdit={editNote}
        onDelete={deleteNote} />
    </div>
  );
};

export default connect(
  ({notes}) => ({
    notes
  }), {
    NoteActions
  }
)(Lane)
```

Se você rodar o aplicativo e tentar adicionar notas, você verá que algo está errado. Cada nota que você adiciona é compartilhada em todas as `Lanes`. Se uma nota é modificada, a outras `Lanes` também são atualizadas.

![Duplicate notes](../images/kanban_01.png)

O motivo desse problema é simples. Nossa `NoteStore` é uma instância única. Isso significa que cada componente que está escutando as mudanças em `NoteStore`, receberão os mesmos dados. Nós precisamos resolver esse problema de alguma forma.

## Deixando as `Lanes` responsáveis por `Notes`

Atualmente, nossa `Lane` contém apenas uma array de objetos. Cada objeto tem seu *id* e *name*. Nós iremos precisar de algo mais sofisticado.

Cada `Lane` precisa saber qual `Notes` pertence a ela. Se uma `Lane` conter um array de id de `Notes`, ela poderia filtrar as notas e mostra-las corretamente em `Notes`. Nós iremos implementar esse esquema a seguir.

### Entendendo `attachToLane`

Ao adicionar uma nova `Note` no sistema através de `addNote`, nós precisamos ter certeza que ela seja associada a uma `Lane`. Essa associação pode ser modelada através de um método, por exemplo `LaneActions.attachToLane({laneId: <id>, noteId: <id>})`. Veja a implementação abaixo:

```javascript
const addNote = e => {
  e.stopPropagation();

  const noteId = uuid.v4();

  NoteActions.create({
    id: noteId,
    task: 'New task'
  });
  LaneActions.attachToLane({
    laneId: lane.id,
    noteId
  });
}
```

Essa é apenas uma maneira de utilizar o `noteId`. Nós poderíamos gerar a lógica com `NoteActions.create` e retornar o id generado de lá. Nós poderíamos também, utilizar [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). Isso seria bem útil se nós adicionarmos um back-end em nossa implementação. A implementação se pareceria com:

```javascript
const addNote = e => {
  e.stopPropagation();

  NoteActions.create({
    task: 'New task'
  }).then(noteId => {
    LaneActions.attachToLane({
      laneId: lane.id,
      noteId: noteId
    });
  })
}
```

Agora declaramos as dependências entre `NoteActions.create` e `LaneActions.attachToLane`. Essa é uma alternativa válida, especialmente se precisarmos ir adiante e adicionar mais funcionalidades.

T> Você poderia modelar a API usando parâmetros posicionados e seria lago como `LaneActions.attachToLane(laneId, note.id)`. Eu prefiro o modo utitlizando um objeto, a leitura é mais simples e você não precisa se preocupar com a ordem.

T> Outra maneira de implementar essa depêndencia seria usar a funcionalidade do *dispatcher* do Flux conhecida como [waitFor](http://alt.js.org/guide/wait-for/). Isso nos permite modelar depêndencias ao nível da *store*. É melhor evitar essa funcionalidade se você puder, soluções de gerenciamento de estado como Redux, tornam isso redundante. Usando `Promises`, como acima, pode resolver muito bem!

### Setting Up `attachToLane`

To get started we should add `attachToLane` to actions as before:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'attachToLane'
);
```

In order to implement `attachToLane`, we need to find a lane matching to the given lane id and then attach note id to it. Furthermore, each note should belong only to one lane at a time. We can perform a rough check against that:

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  ...
leanpub-start-insert
  attachToLane({laneId, noteId}) {
    this.setState({
      lanes: this.lanes.map(lane => {
        if(lane.notes.includes(noteId)) {
          lane.notes = lane.notes.filter(note => note !== noteId);
        }

        if(lane.id === laneId) {
          lane.notes = lane.notes.concat([noteId]);
        }

        return lane;
      })
    });
  }
leanpub-end-insert
}
```

Just being able to attach notes to a lane isn't enough. We are also going to need some way to detach them. This comes up when we are removing notes.

T> We could give a warning here in case you are trying to attach a note to a lane that doesn't exist. `console.warn` would work nicely for that.

### Setting Up `detachFromLane`

We can model a similar counter-operation `detachFromLane` using an API like this:

```javascript
LaneActions.detachFromLane({noteId, laneId});
NoteActions.delete(noteId);
```

T> Just like with `attachToLane`, you could model the API using positional parameters and end up with `LaneActions.detachFromLane(laneId, noteId)`.

Again, we should set up an action:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'attachToLane', 'detachFromLane'
);
```

The implementation will resemble `attachToLane`. In this case, we'll remove the possibly found `Note` instead:

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  ...
leanpub-start-insert
  detachFromLane({laneId, noteId}) {
    this.setState({
      lanes: this.lanes.map(lane => {
        if(lane.id === laneId) {
          lane.notes = lane.notes.filter(note => note !== noteId);
        }

        return lane;
      })
    });
  }
leanpub-end-insert
}
```

Given we have enough logic in place now, we can start connecting it with the user interface.

T> It is possible `detachFromLane` doesn't detach anything. If this case is detected, it would be nice to use `console.warn` to let the developer know about the situation.

### Connecting `Lane` with the Logic

To make this work, there are a couple of places to tweak at `Lane`:

* When adding a note, we need to **attach** it to the current lane.
* When deleting a note, we need to **detach** it from the current lane.
* When rendering a lane, we need to **select** notes belonging to it. It is important to render then notes in the order they belong to a lane. This needs extra logic.

These changes map to `Lane` as follows:

**app/components/Lane.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
leanpub-start-insert
import LaneActions from '../actions/LaneActions';
leanpub-end-insert
import Notes from './Notes';

const Lane = ({
leanpub-start-delete
  lane, notes, NoteActions, ...props
leanpub-end-delete
leanpub-start-insert
  lane, notes, LaneActions, NoteActions, ...props
leanpub-end-insert
}) => {
  const editNote = (id, task) => {
    ...
  };
  const addNote = e => {
    e.stopPropagation();

    const noteId = uuid.v4();

    NoteActions.create({
      id: noteId,
      task: 'New task'
    });
leanpub-start-insert
    LaneActions.attachToLane({
      laneId: lane.id,
      noteId
    });
leanpub-end-insert
  };
  const deleteNote = (noteId, e) => {
    e.stopPropagation();

leanpub-start-insert
    LaneActions.detachFromLane({
      laneId: lane.id,
      noteId
    });
leanpub-end-insert
    NoteActions.delete(noteId);
  };
  const activateNoteEdit = id => {
    NoteActions.update({id, editing: true});
  };

  return (
    <div {...props}>
      <div className="lane-header">
        <div className="lane-add-note">
          <button onClick={addNote}>+</button>
        </div>
        <div className="lane-name">{lane.name}</div>
      </div>
      <Notes
leanpub-start-delete
        notes={notes}
leanpub-end-delete
leanpub-start-insert
        notes={selectNotesByIds(notes, lane.notes)}
leanpub-end-insert
        onNoteClick={activateNoteEdit}
        onEdit={editNote}
        onDelete={deleteNote} />
    </div>
  );
};

leanpub-start-insert
function selectNotesByIds(allNotes, noteIds = []) {
  // `reduce` is a powerful method that allows us to
  // fold data. You can implement `filter` and `map`
  // through it. Here we are using it to concatenate
  // notes matching to the ids.
  return noteIds.reduce((notes, id) =>
    // Concatenate possible matching ids to the result
    notes.concat(
      allNotes.filter(note => note.id === id)
    )
  , []);
}
leanpub-end-insert

export default connect(
  ({notes}) => ({
    notes
  }), {
leanpub-start-delete
    NoteActions
leanpub-end-delete
leanpub-start-insert
    NoteActions,
    LaneActions
leanpub-end-insert
  }
)(Lane)
```

If you try using the application now, you should see that each lane is able to maintain its own notes:

![Separate notes](images/kanban_02.png)

The current structure allows us to keep singleton stores and a flat data structure. Dealing with references is a little awkward, but that's consistent with the Flux architecture. You can see the same theme in the [Redux implementation](https://github.com/survivejs-demos/redux-demo). The [MobX one](https://github.com/survivejs-demos/mobx-demo) avoids the problem altogether given we can use proper references there.

T> `selectNotesByIds` could have been written in terms of `map` and `find`. In that case you would have ended up with `noteIds.map(id => allNotes.find(note => note.id === id));`. You would need to polyfill `find` in this case to support older browsers, though.

T> Normalizing the data would have made `selectNotesByIds` trivial. If you are using a solution like Redux, normalization can make operations like this easy.

## Extracting `LaneHeader` from `Lane`

`Lane` is starting to feel like a big component. This is a good chance to split it up to keep our application easier to maintain. Especially the lane header feels like a component of its own. To get started, define `LaneHeader` based on the current code like this:

**app/components/LaneHeader.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
import LaneActions from '../actions/LaneActions';

export default connect(() => ({}), {
  NoteActions,
  LaneActions
})(({lane, LaneActions, NoteActions, ...props}) => {
  const addNote = e => {
    e.stopPropagation();

    const noteId = uuid.v4();

    NoteActions.create({
      id: noteId,
      task: 'New task'
    });
    LaneActions.attachToLane({
      laneId: lane.id,
      noteId
    });
  };

  return (
    <div className="lane-header" {...props}>
      <div className="lane-add-note">
        <button onClick={addNote}>+</button>
      </div>
      <div className="lane-name">{lane.name}</div>
    </div>
  );
})
```

We also need to connect the extracted component with `Lane`:

**app/components/Lane.jsx**

```javascript
import React from 'react';
leanpub-start-delete
import uuid from 'uuid';
leanpub-end-delete
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
import LaneActions from '../actions/LaneActions';
import Notes from './Notes';
leanpub-start-insert
import LaneHeader from './LaneHeader';
leanpub-end-insert

const Lane = ({
  lane, notes, LaneActions, NoteActions, ...props
}) => {
  const editNote = (id, task) => {
    NoteActions.update({id, task, editing: false});
  };
leanpub-start-delete
  const addNote = e => {
    e.stopPropagation();

    const noteId = uuid.v4();

    NoteActions.create({
      id: noteId,
      task: 'New task'
    });
    LaneActions.attachToLane({
      laneId: lane.id,
      noteId
    });
  };
leanpub-end-delete
  const deleteNote = (noteId, e) => {
    e.stopPropagation();

    LaneActions.detachFromLane({
      laneId: lane.id,
      noteId
    });
    NoteActions.delete(noteId);
  };
  const activateNoteEdit = id => {
    NoteActions.update({id, editing: true});
  };

  return (
    <div {...props}>
leanpub-start-delete
      <div className="lane-header">
        <div className="lane-add-note">
          <button onClick={addNote}>+</button>
        </div>
        <div className="lane-name">{lane.name}</div>
      </div>
leanpub-end-delete
leanpub-start-insert
      <LaneHeader lane={lane} />
leanpub-end-insert
      <Notes
        notes={selectNotesByIds(notes, lane.notes)}
        onNoteClick={activateNoteEdit}
        onEdit={editNote}
        onDelete={deleteNote} />
    </div>
  );
};

...
```

After these changes we have something that's a little easier to work with. It would have been possible to maintain all the related code in a single component. Often these are judgment calls as you realize there are nicer ways to split up your components. Sometimes the need for reuse or performance will force you to split.

## Conclusion

We managed to solve the problem of handling data dependencies in this chapter. It is a problem that comes up often. Each data management solution provides a way of its own to deal with it. Flux based alternatives and Redux expect you to manage the references. Solutions like MobX have reference handling integrated. Data normalization can make these type of operations easier.

In the next chapter we will focus on adding missing functionality to the application. This means tackling editing lanes. We can also make the application look a little nicer again. Fortunately a lot of the logic has been developed already.
