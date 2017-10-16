# Cuidando das depêndencias entre dados

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

### Configurando `attachToLane`

Para começar, devemos adicionar `attachToLane` nas nossas ações:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'attachToLane'
);
```

A fim de implementar `attachToLane`, nós precisamos encontrar a `Lane` correspondente ao id e em seguida, anexar o id da nota nesse modelo. Além disso, cada nota deve pertencer há apenas uma pista ao mesmo tempo. Podemos realizar uma verificação parecida com:

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

Poder anexar notas a uma faixa não é o suficiente. Nós também precisamos, de alguma maneira, separá-las. Isso irá acontecer quando removermos alguma nota.

T> Podemos dar um aviso aqui caso você esteja tentando anexar uma nota a uma faixa que não existe, `console.warn` funciona bem para isso.

### Configurando `detachFromLane`

Podemos modelar uma operação inversa, semelhante a `detachFromLane` usando uma API como:

```javascript
LaneActions.detachFromLane({noteId, laneId});
NoteActions.delete(noteId);
```

T> Assim como `attachToLane`, você poderia modelar a API usando parâmetros posicionados, ficando assim: `LaneActions.detachFromLane(laneId, noteId)`.

Novamente, devemos adicionar uma nova ação:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'attachToLane', 'detachFromLane'
);
```

A implementação será semelhante a `attachToLane`. Mas nesse caso, iremos remover a `Note`, se encontrada, ficando assim:

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

Agora, nós temos lógicas e métodos suficientes, podemos começar a conectá-los com a interface do usuário.

T> Em certos cenários, é possível que `detachFromLane` não encontre nada para remover. Seria bom usar `console.warn` para informar o desenvolvedor sobre essa situação.

### Conectando `Lane` com nossa lógica

Para fazer isso funcionar, há alguns lugares para atualizar em `Lane`:

* Ao adicionar uma nota, precisamos **adicionar** na `Lane` atual.
* Ao excluir uma nota, precisamos **remover** ela da `Lane` atual.
* Ao renderizar uma `Lane`, precisamos **selecionar** notas que pertencem a ela. É importante renderizar as notas na ordem em que elas estão salvas. Isso precisa de uma lógica extra.

Essas mudanças são definidas em `Lane`:

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
  // `reduce` é um método poderoso que nos permite
  // reduzir dados a um único valor. Você pode implementar
  // `filter` e `map` com ele. Aqui estamos usando ele
  // para concatenar notas correspondentes aos ids.
  return noteIds.reduce((notes, id) =>
    // Concatenando possíveis ids correspondentes ao resultado
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

Se você tentar usar o aplicativo agora, você deve ver que cada `Lane` está mantendo suas próprias notas:

![Separate notes](../images/kanban_02.png)

The current structure allows us to keep singleton stores and a flat data structure. Dealing with references is a little awkward, but that's consistent with the Flux architecture. You can see the same theme in the
A estrutura atual nos permite manter `singleton stores` e uma estrutura de dados plana. Lidar com referências é um pouco estranho, mas isso é consistente com a arquitetura Flux. Você pode ver o mesmo exemplo na [implementação com Redux](https://github.com/survivejs-demos/redux-demo). O exemplo com [MobX](https://github.com/survivejs-demos/mobx-demo), evita o problema, dado que podemos usar referências adequadas lá.

T> `selectNotesByIds` poderia ter sido escrito com `map` e `find`. Nesse caso, você teria acabado com `noteIds.map(id => allNotes.find(note => note.id === id));`. No entando, você precisaria de um polyfill para `find` para suportar navegadores mais antigos.

T> Uma normalização de dados aqui teria tornado `selectNotesByIds` desnecessário. Se você estiver usando uma solução como o Redux, a normalização pode tornar essas operações fáceis.

## Extraíndo `LaneHeader` de `Lane`

`Lane` está começando a ficar um grande componente. Esta é uma boa chance de dividi-lo, para manter nossa aplicação mais fácil de manter. Especialmente, o cabeçalho, poderia ser  um componente próprio. Para começar, defina `LaneHeader` com base no código atual:

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

Também precisamos conectar o componente extraído em `Lane`:

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

Após essas mudanças, temos algo que é um pouco mais fácil de se trabalhar. Seria possível manter todo esse código em um único componente. Muitas vezes, essas situações são baseadas no seu julgamento, pois é você quem percebe se há maneiras mais legais de dividir seus componentes. Às vezes, a necessidade de reutilização ou desempenho irá forçá-lo a dividir.

## Conclusão

Conseguimos resolver o problema das dependências entre dados neste capítulo. É um problema frequente quando sua aplicação começa a crescer. Cada solução de gerenciamento de dados fornece uma maneira própria de lidar com isso. Alternativas baseadas em Flux e Redux esperam que você gerencie as referências. Soluções como MobX possuem gerenciamento de referência integrado. A normalização dos dados pode tornar este tipo de operações mais fácil.

No próximo capítulo, ieremos nos concentraremos em adicionar mais funcionalidades ao nosso aplicativo, como a possibilidade de editar uma `Lane`. Também podemos tornar o aplicativo um pouco mais agradável visualmente e felizmente, uma grande parte da lógica já foi desenvolvida até aqui.
