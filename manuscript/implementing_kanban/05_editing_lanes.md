# Editando Lanes

![Kanban board](../images/kanban_05.png)

Nós ainda temos trabalho a fazer para transformar isso em um Kanban como ilustrado acima. Ainda falta lógica e estilos no nosso aplicativo. É sobre isso que vamos nos concentrar aqui.

O componente `Editable` que nós implementamos anteriormente será útil. Podemos usá-lo para tornar possível alterar os nomes de cada `Lane`. A idéia é exatamente a mesma coisa que fizemos para `Note`.

Devemos também permitir a remoção de cada `Lane`. Para isso funcionar, precisaremos adicionar um controle na UI e anexar uma lógica a ele. Mais uma vez, é uma ideia semelhante à anterior.

## Implementando edição para os nomes das `Lane`

Para editar o nome de uma `Lane`, precisamos de um pouco de lógica e ações na UI. `Editable` pode cuidar da parte da UI. Na lógica, teremos mais trabalho. Podemos começar com `LaneHeader`:

**app/components/LaneHeader.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import connect from '../libs/connect';
import NoteActions from '../actions/NoteActions';
import LaneActions from '../actions/LaneActions';
leanpub-start-insert
import Editable from './Editable';
leanpub-end-insert

export default connect(() => ({}), {
  NoteActions,
  LaneActions
})(({lane, LaneActions, NoteActions, ...props}) => {
  const addNote = e => {
    ...
  };
leanpub-start-insert
  const activateLaneEdit = () => {
    LaneActions.update({
      id: lane.id,
      editing: true
    });
  };
  const editName = name => {
    LaneActions.update({
      id: lane.id,
      name,
      editing: false
    });
  };
leanpub-end-insert

  return (
leanpub-start-delete
    <div className="lane-header" {...props}>
leanpub-end-delete
leanpub-start-insert
    <div className="lane-header" onClick={activateLaneEdit} {...props}>
leanpub-end-insert
      <div className="lane-add-note">
        <button onClick={addNote}>+</button>
      </div>
leanpub-start-delete
      <div className="lane-name">{lane.name}</div>
leanpub-end-delete
leanpub-start-insert
      <Editable className="lane-name" editing={lane.editing}
        value={lane.name} onEdit={editName} />
leanpub-end-insert
    </div>
  );
})
```

A interface do usuário deve ser exatamente a mesma após essa alteração. Ainda precisamos implementar `LaneActions.update` para que nossa configuração funcione.

Assim como antes, devemos ajustar dois lugares, a definição de ação e `LaneStore`. Aqui está a parte da ação:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'update', 'attachToLane', 'detachFromLane'
);
```

Para adicionar a lógica restante, modifique `LaneStore` da seguinte maneira. É a mesma ideia que em `NoteStore`:

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  constructor() {
    this.bindActions(LaneActions);

    this.lanes = [];
  }
  create(lane) {
    ...
  }
leanpub-start-insert
  update(updatedLane) {
    this.setState({
      lanes: this.lanes.map(lane => {
        if(lane.id === updatedLane.id) {
          return Object.assign({}, lane, updatedLane);
        }

        return lane;
      })
    });
  }
leanpub-end-insert
  ...
}
```

Após essas alterações, você pode editar os nomes das `Lanes`. A remoção de uma `Lane` vamos implementar a seguir.

## Implementando a remoção de uma `Lane`

Remover uma `Lane` é um problema semelhante. Precisamos ampliar a interface do usuário, adicionar uma ação e anexar lógica associada a ela.

Vamos começar pela interface do usuário. Muitas vezes é uma boa idéia adicionar alguns `console.log` para verificar que seus eventos estão sendo ativados como esperado. Seria ainda melhor escrever testes para eles. Dessa forma, você terá uma especificação executável. Veja como adicionar um `stube` para remover uma `Lane`:

**app/components/LaneHeader.jsx**

```javascript
...

export default connect(() => ({}), {
  NoteActions,
  LaneActions
})(({lane, LaneActions, NoteActions, ...props}) => {
  ...
leanpub-start-insert
  const deleteLane = e => {
    // Evitando a propagação para `edit`
    e.stopPropagation();

    LaneActions.delete(lane.id);
  };
leanpub-end-insert

  return (
    <div className="lane-header" onClick={activateLaneEdit} {...props}>
      <div className="lane-add-note">
        <button onClick={addNote}>+</button>
      </div>
      <Editable className="lane-name" editing={lane.editing}
        value={lane.name} onEdit={editName} />
leanpub-start-insert
      <div className="lane-delete">
        <button onClick={deleteLane}>x</button>
      </div>
leanpub-end-insert
    </div>
  );
});
```

Novamente, precisamos expandir nossa ação:

**app/actions/LaneActions.js**

```javascript
import alt from '../libs/alt';

export default alt.generateActions(
  'create', 'update', 'delete', 'attachToLane', 'detachFromLane'
);
```

E para finalizar a implementação, vamos adicionar nossa lógica:

**app/stores/LaneStore.js**

```javascript
import LaneActions from '../actions/LaneActions';

export default class LaneStore {
  constructor() {
    this.bindActions(LaneActions);

    this.lanes = [];
  }
  create(lane) {
    ...
  }
  update(updatedLane) {
    ...
  }
leanpub-start-insert
  delete(id) {
    this.setState({
      lanes: this.lanes.filter(lane => lane.id !== id)
    });
  }
leanpub-end-insert
  ...
}
```

Supondo que tudo correu bem, você pode remover qualquer `Lane` agora.

A implementação atual contém um problema. Embora estamos removendo a referência da `Lane`, as notas guardadas nelas ainda permanecem por lá. Isso é algo que pode ser transformado em um recurso de lixeira. Ou, também podemos fazer a limpeza delas. Para essa exemplo, podemos deixar a situação como está. No entanto, é algo bom estar ciente.

## Adicionando estilos ao quadro Kanban

Como adicionamos `Lanes` ao nosso aplicativo, os estilos ficaram um pouco fora de mão. Vamos deixar a aparência mais agradável com:

**app/main.css**

```css
body {
  background-color: cornsilk;

  font-family: sans-serif;
}

leanpub-start-delete
.add-note {
  background-color: #fdfdfd;

  border: 1px solid #ccc;
}
leanpub-end-delete

leanpub-start-insert
.lane {
  display: inline-block;

  margin: 1em;

  background-color: #efefef;
  border: 1px solid #ccc;
  border-radius: 0.5em;

  min-width: 10em;
  vertical-align: top;
}

.lane-header {
  overflow: auto;

  padding: 1em;

  color: #efefef;
  background-color: #333;

  border-top-left-radius: 0.5em;
  border-top-right-radius: 0.5em;
}

.lane-name {
  float: left;
}

.lane-add-note {
  float: left;

  margin-right: 0.5em;
}

.lane-delete {
  float: right;

  margin-left: 0.5em;

  visibility: hidden;
}
.lane-header:hover .lane-delete {
  visibility: visible;
}

.add-lane, .lane-add-note button {
  cursor: pointer;

  background-color: #fdfdfd;
  border: 1px solid #ccc;
}

.lane-delete button {
  padding: 0;

  cursor: pointer;

  color: white;
  background-color: rgba(0, 0, 0, 0);
  border: 0;
}
leanpub-end-insert

...
```

Você deve ter um resultado como este:

![Styled Kanban](../images/kanban_styled.png)

Como esse é um pequeno projeto, podemos deixar o CSS em um único arquivo. Caso comece a crescer, considere separá-lo em vários arquivos. Uma maneira de fazer isso é extrair um CSS por componente e, em seguida, fazer o `require` do arquivo (ex: `require('./lane.css')` em `Lane.jsx`). Você pode até mesmo considerar usar **CSS Modules** para deixar seu CSS com escopo local como padrão. Veja o capítulo *Estilos em React* para mais ideias.

## Conclucsão

Mesmo que o nosso aplicativo esteja começando a ficar com funcionalidades básicas, ainda falta o recurso mais importante. Ainda não podemos mover as notas entre as pistas. Isso é algo que resolveremos no próximo capítulo à medida que implementamos o arrastar e soltar.
