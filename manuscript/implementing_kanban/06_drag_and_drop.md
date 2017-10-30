# Implementando arrastar e soltar

Nosso aplicativo Kanban está quase utilizável. Tem um bom estilo e há funcionalidades básicas instaladas. Neste capítulo, integraremos a funcionalidade de arrastar e soltar utilizando [React DnD](https://gaearon.github.io/react-dnd/).

No final deste capítulo, você pode organizar as notas dentro de uma `Lane` e arrastá-las de uma `Lane` para outra. Embora isso pareça simples, teremos um pouco de trabalho, pois precisamos anotar nossos componentes do jeito certo e desenvolver a lógica necessária.

## Configurando o React DnD

Nosso primeiro passo será conectar o `React DnD` com nosso projeto. Iremos utilizar o back-end `HTML5 Drag and Drop`. Existem back-ends específicos para testes e [touch](https://github.com/yahoo/react-dnd-touch-backend).

Para configurá-lo, precisamos usar o decorador `DragDropContext` e fornecer o back-end HTML5 para ele. Para evitar verbosidade, usarei o Redux `compose` para manter o código mais limpo e mais legível:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
leanpub-start-insert
import {compose} from 'redux';
import {DragDropContext} from 'react-dnd';
import HTML5Backend from 'react-dnd-html5-backend';
leanpub-end-insert
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

leanpub-start-delete
export default connect(({lanes}) => ({
  lanes
}), {
  LaneActions
})(App)
leanpub-end-delete
leanpub-start-insert
export default compose(
  DragDropContext(HTML5Backend),
  connect(
    ({lanes}) => ({lanes}),
    {LaneActions}
  )
)(App)
leanpub-end-insert
```

Após essas alterações, nosso aplicativo deve ser exatamente o mesmo que antes. Estamos prontos para adicionar algumas funcionalidades agora.

## Permitindo que `Notes` sejam arrastadas

Permitir que as notas sejam arrastadas é um bom primeiro passo. Antes disso, precisamos configurar uma constante para que o React DnD possa diferenciar diferentes tipos de arrastadores. Configure um arquivo para rastreamento das notas:

**app/constants/itemTypes.js**

```javascript
export default {
  NOTE: 'note'
};
```

Esta definição pode ser expandida futuramente, à medida que adicionamos novos tipos, como `Lane`, ao nosso sistema.

Em seguida, precisamos dizer a nossa `Note` que é possível arrastá-la. Podemos resolver isso utilizando a anotação `DragSource`. Vamos alterar nosso componente `Note` com a seguinte implementação:

**app/components/Note.jsx**

```javascript
import React from 'react';
import {DragSource} from 'react-dnd';
import ItemTypes from '../constants/itemTypes';

const Note = ({
  connectDragSource, children, ...props
}) => {
  return connectDragSource(
    <div {...props}>
      {children}
    </div>
  );
};

const noteSource = {
  beginDrag(props) {
    console.log('begin dragging note', props);

    return {};
  }
};

export default DragSource(ItemTypes.NOTE, noteSource, connect => ({
  connectDragSource: connect.dragSource()
}))(Note)
```

Agora, se você tentar arrastar uma `Note`, você deve ver algo assim no console do navegador:

```bash
begin dragging note Object {className: "note", children: Array[2]}
```

Poder apenas arrastar notas não é suficiente. Precisamos anotá-las para que elas possam receber o evento de soltar. Isso nos permitirá alterar suas posições, pois podemos ter uma lógica para quando estamos tentando soltar uma nota em cima de outra.

T> No caso, querermos implementar o arrastar com base em um elemento específico, também podemos aplicar `connectDragSource` em apenas uma parte de uma `Note`.

W> Vale lembrar que o React DnD não suporta o *hot loading* perfeitamente. Talvez seja necessário atualizar o navegador para ver as mensagens de log que você espera!

## Detectando quando uma Nota é arrastada sobre outra

Vamos anotar nossas Notas para que elas possam reagir quando uma outra nota esteja sendo suspensa em cima delas. Neste caso, teremos que usar a anotação `DropTarget`:

**app/components/Note.jsx**

```javascript
import React from 'react';
leanpub-start-delete
import {DragSource} from 'react-dnd';
leanpub-end-delete
leanpub-start-insert
import {compose} from 'redux';
import {DragSource, DropTarget} from 'react-dnd';
leanpub-end-insert
import ItemTypes from '../constants/itemTypes';

const Note = ({
leanpub-start-delete
  connectDragSource, children, ...props
leanpub-end-delete
leanpub-start-insert
  connectDragSource, connectDropTarget,
  children, ...props
leanpub-end-insert
}) => {
leanpub-start-delete
  return connectDragSource(
leanpub-end-delete
leanpub-start-insert
  return compose(connectDragSource, connectDropTarget)(
leanpub-end-insert
    <div {...props}>
      {children}
    </div>
  );
};

const noteSource = {
  beginDrag(props) {
    console.log('begin dragging note', props);

    return {};
  }
};

leanpub-start-insert
const noteTarget = {
  hover(targetProps, monitor) {
    const sourceProps = monitor.getItem();

    console.log('dragging note', sourceProps, targetProps);
  }
};
leanpub-end-insert

leanpub-start-delete
export default DragSource(ItemTypes.NOTE, noteSource, connect => ({
  connectDragSource: connect.dragSource()
}))(Note)
leanpub-end-delete
leanpub-start-insert
export default compose(
  DragSource(ItemTypes.NOTE, noteSource, connect => ({
    connectDragSource: connect.dragSource()
  })),
  DropTarget(ItemTypes.NOTE, noteTarget, connect => ({
    connectDropTarget: connect.dropTarget()
  }))
)(Note)
leanpub-end-insert
```

Se você tentar arrastar uma nota por cima da outra agora, você deve ver mensagens como esta no console:

```bash
dragging note Object {} Object {className: "note", children: Array[2]}
```

Ambos os decoradores nos dão acesso as `props` das `Note`. Neste caso, estamos usando `monitor.getItem()` para acessá-los em `noteTarget`. Este é o segredo para que isso tudi funcione corretamente.

## Desenvolvendo a API `onMove` para `Notes`

Agora que podemos mover nossas notas, vamos começar a definir a lógica necessária com as seguintes etapas:

1. Capturar o id da `Note` que está em `beginDrag`.
2. Capturar o id do alvo `Note` em `hover`.
3. Executar o método' `onMove` em `hover` para que possamos executar a lógica em outros lugares. `LaneStore` seria o lugar ideal para isso.

Com base na idéia acima, podemos ver que devemos passar o id com a `Note` através das `props`. Também precisamos criar um método `onMove`, definindo `LaneActions.move`, e o `stub` para `LaneStore.move`.

### Recebendo `id` e `onMove` em `Note`

Nos podemos passar `id` e `onMove` como `props` para `Note`. Há uma verificação extra em `noteTarget` que precisamos fazer, pois não precisamos executar `hover` quando estivermos arrastando a nota em cima de si mesma:

**app/components/Note.jsx**

```javascript
...

const Note = ({
  connectDragSource, connectDropTarget,
leanpub-start-delete
  children, ...props
leanpub-end-delete
leanpub-start-insert
  onMove, id, children, ...props
leanpub-end-insert
}) => {
  return compose(connectDragSource, connectDropTarget)(
    <div {...props}>
      {children}
    </div>
  );
};

leanpub-start-delete
const noteSource = {
  beginDrag(props) {
    console.log('begin dragging note', props);

    return {};
  }
};
leanpub-end-delete
leanpub-start-insert
const noteSource = {
  beginDrag(props) {
    return {
      id: props.id
    };
  }
};
leanpub-end-insert

leanpub-start-delete
const noteTarget = {
  hover(targetProps, monitor) {
    const sourceProps = monitor.getItem();

    console.log('dragging note', sourceProps, targetProps);
  }
};
leanpub-end-delete
leanpub-start-insert
const noteTarget = {
  hover(targetProps, monitor) {
    const targetId = targetProps.id;
    const sourceProps = monitor.getItem();
    const sourceId = sourceProps.id;

    if(sourceId !== targetId) {
      targetProps.onMove({sourceId, targetId});
    }
  }
};
leanpub-end-insert

...
```

Agora, precisamos atualizar `Notes` para passar as verdadeiras `props` para `Note`. Iremos fazer no próximo tópico.

### Passando `id` e `onMove` de `Notes`

Passando o `id` de uma nota e `onMove`, é bem simples:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
import Editable from './Editable';

export default ({
  notes,
  onNoteClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => (
  <ul className="notes">{notes.map(({id, editing, task}) =>
    <li key={id}>
leanpub-start-delete
      <Note className="note" onClick={onNoteClick.bind(null, id)}>
leanpub-end-delete
leanpub-start-insert
      <Note className="note" id={id}
        onClick={onNoteClick.bind(null, id)}
        onMove={({sourceId, targetId}) =>
          console.log('moving from', sourceId, 'to', targetId)}>
leanpub-end-insert
        <Editable
          className="editable"
          editing={editing}
          value={task}
          onEdit={onEdit.bind(null, id)} />
        <button
          className="delete"
          onClick={onDelete.bind(null, id)}>x</button>
      </Note>
    </li>
  )}</ul>
)
```

Se você arrastar uma nota em cima da outra, você deve ver as mensagens no console:

```bash
moving from 3310916b-5b59-40e6-8a98-370f9c194e16 to 939fb627-1d56-4b57-89ea-04207dbfb405
```

## Adicionando novas ações e métodos ao arrastar

A lógica do arrastar e soltar é a seguinte, suponha que temos uma coluna contendo notas A, B, C. No caso de nos arrastarmos A em cima de C devemos terminar com B, C, A. Caso arrastarmos ela para outra lista, vamos dizer D, E, F, e arrastarmos A para o começo dela, nós teremos B, C e A, D, E, F.

No nosso caso, teremos alguma complexidade extra para arrastar de uma `Lane` para outra. Quando movemos uma `Note`, Nós conhecemos sua posição original e a posição alvo. `Lane` sabe qual `Notes` o id pertence. De alguma maneira, precisamos dizer a `LaneStore` para executar uma lógica na nota fornecida ao arrastar. Um bom ponto de partida é definir `LaneActions.move`:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'update', 'delete',
  'attachToLane', 'detachFromLane',
  'move'
);
```

Devemos conectar esta ação com o `onMove`:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
import Editable from './Editable';
leanpub-start-insert
import LaneActions from '../actions/LaneActions';
leanpub-end-insert

export default ({
  notes,
  onNoteClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => (
  <ul className="notes">{notes.map(({id, editing, task}) =>
    <li key={id}>
      <Note className="note" id={id}
        onClick={onNoteClick.bind(null, id)}
leanpub-start-delete
        onMove={({sourceId, targetId}) =>
          console.log('moving from', sourceId, 'to', targetId)}>
leanpub-end-delete
leanpub-start-insert
        onMove={LaneActions.move}>
leanpub-end-insert
        <Editable
          className="editable"
          editing={editing}
          value={task}
          onEdit={onEdit.bind(null, id)} />
        <button
          className="delete"
          onClick={onDelete.bind(null, id)}>x</button>
      </Note>
    </li>
  )}</ul>
)
```

T> Poderia ser uma boa idéia refatorar `onMove` como uma `props`, para tornar o sistema mais flexível. Em nossa implementação, o componente `Notes` é acoplado com `LaneActions`. Impossibilitando você, de utilizar essa lógica em outro lugar.

Também devemos definir um `stub` em `LaneStore` para ver que nós o conectamos corretamente:

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  ...
  detachFromLane({laneId, noteId}) {
    ...
  }
leanpub-start-insert
  move({sourceId, targetId}) {
    console.log(`source: ${sourceId}, target: ${targetId}`);
  }
leanpub-end-insert
}
```

Você deve ver as mesmas mensagens de log anteriormente discutidas.

Em seguida, precisamos adicionar alguma lógica para que isso funcione. Podemos usar a lógica descrita no início desse tópico. Temos dois casos para se preocupar: mover dentro de uma própria coluna e passando de uma coluna para outra.

## Implementando lógica das notas ao arrastar e soltar

Mover uma nota dentro de uma `Lane`, já é complicado. Quando estamos operando com base em ids e uma operação de cada vez, você precisa levar em conta alterações de índices. Como resultado, estou usando `update` [utilitário para estruturas de dados imutáveis](https://facebook.github.io/react/docs/update.html) do React como para resolver o problema em uma única chamada.

It is possible to solve the lane to lane case using
Também é possível resolver esse problema usando [splice](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/splice). Primeiro, nós usamos `splice` na nota e novamnete, fazemos `splice` da `Lane` que será alvo. E por último, um `update` funcionaria aqui, mas não faz muito sentido, já que `slice` é mais simples e elegante. O código abaixo mostra uma solução baseda em mutação:

**app/stores/LaneStore.js**

```javascript
leanpub-start-insert
import update from 'react-addons-update';
leanpub-end-insert
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  ...
leanpub-start-delete
  move({sourceId, targetId}) {
    console.log(`source: ${sourceId}, target: ${targetId}`);
  }
leanpub-end-delete
leanpub-start-insert
  move({sourceId, targetId}) {
    const lanes = this.lanes;
    const sourceLane = lanes.filter(lane => lane.notes.includes(sourceId))[0];
    const targetLane = lanes.filter(lane => lane.notes.includes(targetId))[0];
    const sourceNoteIndex = sourceLane.notes.indexOf(sourceId);
    const targetNoteIndex = targetLane.notes.indexOf(targetId);

    if(sourceLane === targetLane) {
      // move at once to avoid complications
      sourceLane.notes = update(sourceLane.notes, {
        $splice: [
          [sourceNoteIndex, 1],
          [targetNoteIndex, 0, sourceId]
        ]
      });
    }
    else {
      // get rid of the source
      sourceLane.notes.splice(sourceNoteIndex, 1);

      // and move it to target
      targetLane.notes.splice(targetNoteIndex, 0, sourceId);
    }

    this.setState({lanes});
  }
leanpub-end-insert
}
```

Se você recarregar o aplicativo, você pode realmente arrastar as notas e deve funcionar do jeito que você espera. Arrastar notas para esvaziar uma `Lane` não funciona e a apresentação poderia ser melhor.

Seria melhor se indicássemos a localização da nota arrastada com mais clareza. Podemos fazer isso escondendo a nota arrastada da lista. Reagir DnD nos fornece os ganchos que precisamos para este propósito.

## Indicando aonde mover

Reagir DnD fornece um recurso conhecido como monitores de estado. Através dele, podemos usar `monitor.isDragging ()` e `monitor.isOver ()` para detectar qual nota estamos atualmente arrastando. Pode ser configurado da seguinte forma:

**app/components/Note.jsx**

```javascript
import React from 'react';
import {compose} from 'redux';
import {DragSource, DropTarget} from 'react-dnd';
import ItemTypes from '../constants/itemTypes';

const Note = ({
leanpub-start-delete
  connectDragSource, connectDropTarget,
  onMove, id, children, ...props
leanpub-end-delete
leanpub-start-insert
  connectDragSource, connectDropTarget, isDragging,
  isOver, onMove, id, children, ...props
leanpub-end-insert
}) => {
  return compose(connectDragSource, connectDropTarget)(
leanpub-start-delete
    <div {...props}>
      {children}
    </div>
leanpub-end-delete
leanpub-start-insert
    <div style={{
      opacity: isDragging || isOver ? 0 : 1
    }} {...props}>{children}</div>
leanpub-end-insert
  );
};

...

export default compose(
leanpub-start-delete
  DragSource(ItemTypes.NOTE, noteSource, connect => ({
    connectDragSource: connect.dragSource()
  })),
  DropTarget(ItemTypes.NOTE, noteTarget, connect => ({
    connectDropTarget: connect.dropTarget()
  }))
leanpub-end-delete
leanpub-start-insert
  DragSource(ItemTypes.NOTE, noteSource, (connect, monitor) => ({
    connectDragSource: connect.dragSource(),
    isDragging: monitor.isDragging()
  })),
  DropTarget(ItemTypes.NOTE, noteTarget, (connect, monitor) => ({
    connectDropTarget: connect.dropTarget(),
    isOver: monitor.isOver()
  }))
leanpub-end-insert
)(Note)
```

Se você arrastar uma nota dentro de uma `Lane`, a nota arrastada deve ser mostrada em branco.

Há um pequeno problema no nosso sistema. Não podemos arrastar as notas para uma `Lane` vazia.

## Arrastando Notas para `Lanes` vazias

Para arrastar notas para uma `Lane` vazia, precisamos permitir que elas recebam notas. Assim como acima, podemos configurar `DropTarget` com uma lógica para isso. Primeiro, precisamos capturar o evento de arrastar em `Lane`:

**app/components/Lane.jsx**

```javascript
import React from 'react';
leanpub-start-insert
import {compose} from 'redux';
import {DropTarget} from 'react-dnd';
import ItemTypes from '../constants/itemTypes';
leanpub-end-insert
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
import LaneActions from '../actions/LaneActions';
import Notes from './Notes';
import LaneHeader from './LaneHeader';

const Lane = ({
leanpub-start-delete
  lane, notes, LaneActions, NoteActions, ...props
leanpub-end-delete
leanpub-start-insert
  connectDropTarget, lane, notes, LaneActions, NoteActions, ...props
leanpub-end-insert
}) => {
  ...

leanpub-start-delete
  return (
leanpub-end-delete
leanpub-start-insert
  return connectDropTarget(
leanpub-end-insert
    ...
  );
};

function selectNotesByIds(allNotes, noteIds = []) {
  ...
}

leanpub-start-insert
const noteTarget = {
  hover(targetProps, monitor) {
    const sourceProps = monitor.getItem();
    const sourceId = sourceProps.id;

    // Se a `Lane` alvo não tiver notas,
    // vamos anexar a nota a ela.
    //
    // `attachToLane` executa uma
    // limpeza por padrão e garante
    // que uma nota pode pertencer
    // apenas a uma única `Lane` de uma vez.
    if(!targetProps.lane.notes.length) {
      LaneActions.attachToLane({
        laneId: targetProps.lane.id,
        noteId: sourceId
      });
    }
  }
};
leanpub-end-insert

leanpub-start-delete
export default connect(
  ({notes}) => ({
    notes
  }), {
    NoteActions,
    LaneActions
  }
)(Lane)
leanpub-end-delete
leanpub-start-insert
export default compose(
  DropTarget(ItemTypes.NOTE, noteTarget, connect => ({
    connectDropTarget: connect.dropTarget()
  })),
  connect(({notes}) => ({
    notes
  }), {
    NoteActions,
    LaneActions
  })
)(Lane)
leanpub-end-insert
```

Depois de adicionar essa lógica, você pode arrastar as notas para uma `Lane` vazia.

Nossa implementação atual de `attachToLane` faz o trabalho duro para nós. Se esse método não garantir que uma nota possa pertencer há apenas uma única `Lane` de cada vez, nós iremos precisar ajustar nossa lógica. É bom ter esses tipos de invariantes dentro do sistema de gerenciamento do estado.

### Corrigindo a edição durante o arrastar

A implementação atual tem uma pequena falha. Se você editar uma nota, você ainda pode arrastá-la enquanto ela está sendo editada. Isso não é ideal porque substitui o comportamento padrão que a maioria das pessoas está acostumada. Você não pode, por exemplo, clicar duas vezes em um campo de texto para selecionar todo o texto.

Felizmente, isso é simples de corrigir. Precisamos usar o estado `editing` em cada `Note` para ajustar seu comportamento. Primeiro, precisamos passar o estado `editing` para cada `Note`:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
import Editable from './Editable';
import LaneActions from '../actions/LaneActions';

export default ({
  notes,
  onNoteClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => (
  <ul className="notes">{notes.map(({id, editing, task}) =>
    <li key={id}>
      <Note className="note" id={id}
leanpub-start-insert
        editing={editing}
leanpub-end-insert
        onClick={onNoteClick.bind(null, id)}
        onMove={LaneActions.move}>
        <Editable
          className="editable"
          editing={editing}
          value={task}
          onEdit={onEdit.bind(null, id)} />
        <button
          className="delete"
          onClick={onDelete.bind(null, id)}>x</button>
      </Note>
    </li>
  )}</ul>
)
```

Em seguida, precisamos levar isso em consideração ao renderizar o componente:

**app/components/Note.jsx**

```javascript
import React from 'react';
import {compose} from 'redux';
import {DragSource, DropTarget} from 'react-dnd';
import ItemTypes from '../constants/itemTypes';

const Note = ({
  connectDragSource, connectDropTarget, isDragging,
leanpub-start-delete
  isOver, onMove, id, children, ...props
leanpub-end-delete
leanpub-start-insert
  isOver, onMove, id, editing, children, ...props
leanpub-end-insert
}) => {
leanpub-start-insert
  // Pass through if we are editing
  const dragSource = editing ? a => a : connectDragSource;
leanpub-end-insert

leanpub-start-delete
  return compose(connectDragSource, connectDropTarget)(
leanpub-end-delete
leanpub-start-insert
  return compose(dragSource, connectDropTarget)(
leanpub-end-insert
    <div style={{
      opacity: isDragging || isOver ? 0 : 1
    }} {...props}>{children}</div>
  );
};

...
```

Esta pequena mudança nos dá o comportamento que queremos. Se você tentar editar uma nota, agora, a campo irá funcionar da maneira que você esperar.

Foi uma boa idéia manter o estado `editing` fora do `Editable`. Se não tivéssemos feito isso, essa mudança teria sido muito mais difícil, pois teríamos que extrair o estado fora do componente.

Agora, temos um quadro Kanban que é realmente útil! Podemos criar novas `Lanes` e `Notes`, editá-las e removê-las. Além disso, podemos mover as notas ao redor. Missão cumprida!

## Conclusion

In this chapter, you saw how to implement drag and drop for our little application. You can model sorting for lanes using the same technique. First, you mark the lanes to be draggable and droppable, then you sort out their ids, and finally, you'll add some logic to make it all work together. It should be considerably simpler than what we did with notes.

I encourage you to expand the application. The current implementation should work just as a starting point for something greater. Besides extending the DnD implementation, you can try adding more data to the system. You could also do something to the visual outlook. One option would be to try out various styling approaches discussed at the *Styling React* chapter.

To make it harder to break the application during development, you can also implement tests as discussed at *Testing React*. *Typing with React* discussed yet more ways to harden your code. Learning these approaches can be worthwhile. Sometimes it may be worth your while to design your applications test first. It is a valuable approach as it allows you to document your assumptions as you go.
