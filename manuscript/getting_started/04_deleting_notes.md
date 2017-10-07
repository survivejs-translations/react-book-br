# Deletando `Notes`

Uma maneira fácil de fazer a exclusão de notas é renderizar um botão "x" para cada `Note`. Ao clicar, simplesmente eliminamos a nota em questão da nossa estrutura de dados. Como antes, podemos começar adicionando *strubs*. Será um bom lugar para separar o conceito de `Note` do atual componente `Notes`.

Muitas vezes você trabalha dessa maneira no React. Você configura componentes apenas para perceber que eles são compostos de componentes menores que podem ser extraídos. Esse processo de separação é barato. Às vezes, pode até melhorar o desempenho do seu aplicativo, pois você pode otimizar a renderização de peças menores.

## Separando `Note`

Para manter o aspecto de formatação da lista separado em uma `Note`, podemos usar um `div` como este:

**app/components/Note.jsx**

```javascript
import React from 'react';

export default ({task}) => <div>{task}</div>;
```

Lembre-se de que esta declaração é equivalente a:

```javascript
import React from 'react';

export default (props) => <div>{props.task}</div>;
```

Como você pode ver, usar *destructuring*, remove ruídos do código e mantém nossa implementação simples.

Para fazer com que nosso aplicativo use o novo componente, precisamos atualizar o `Notes`:

**app/components/Notes.jsx**

```javascript
import React from 'react';
leanpub-start-insert
import Note from './Note';
leanpub-end-insert

export default ({notes}) => (
  <ul>{notes.map(note =>
leanpub-start-delete
    <li key={note.id}>{note.task}</li>
leanpub-end-delete
leanpub-start-insert
    <li key={note.id}><Note task={note.task} /></li>
leanpub-end-insert
  )}</ul>
)
```

O aplicativo deve ser exatamente o mesmo após essas mudanças. Agora temos espaço para expandir ainda mais.

## Adicionando um stub para `onDelete`

Para capturar a intenção de excluir uma `Note`, precisamos incluir um botão que desencadeie uma chamada a `onDelete`. Podemos conectar nossa lógica com isso depois que este passo for concluído. Considere o código abaixo:

**app/components/Note.jsx**

```javascript
import React from 'react';

leanpub-start-delete
export default ({task}) => <div>{task}</div>;
leanpub-end-delete
leanpub-start-insert
export default ({task, onDelete}) => (
  <div>
    <span>{task}</span>
    <button onClick={onDelete}>x</button>
  </div>
);
leanpub-end-insert
```

Você deve ver um pequeno "x" perto de cada nota:

![Notes with delete controls](../images/react_06.png)

Eles ainda não farão nada. Vamos corrigir isso no próximo passo.

## Comunicando a exclusão para `App`

Agora que temos os controles que precisamos, podemos começar a pensar sobre como conectá-los com os dados no `App`. Para excluir uma `Note`, precisamos saber seu id. Depois disso, podemos implementar a lógica no `App`. Para ilustrar a ideia, queremos acabar com uma situação como esta:

![`onDelete` flow](../images/bind.png)

T> O `e` representa um evento DOM que você pode usar. Nós podemos fazer coisas como parar a propagação de eventos através dela. Isso será útil, pois queremos mais controle sobre o comportamento do aplicativo.

T> [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) nos permite definir o contexto da função (primeiro parâmetro) e os argumentos (parâmetros seguintes). Isso nos dá uma técnica conhecida como **aplicação parcial** (*partial application*).

Vamos precisar de uma nova `prop` em `Notes`. Também precisamos fazer o `bind` do id de cada nota na chamada para `onDelete`. Aqui está a implementação completa do `Notes`:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';

leanpub-start-delete
export default ({notes}) => (
  <ul>{notes.map(note =>
    <li key={note.id}><Note task={note.task} /></li>
  )}</ul>
)
leanpub-end-delete
leanpub-start-insert
export default ({notes, onDelete=() => {}}) => (
  <ul>{notes.map(({id, task}) =>
    <li key={id}>
      <Note
        onDelete={onDelete.bind(null, id)}
        task={task} />
    </li>
  )}</ul>
)
leanpub-end-insert
```

Para evitar que o nosso código falhe se `onDelete` não for fornecido, eu defini uma *prop* padrão para ao realizar o *destructuring*. Outra boa maneira de lidar com isso, seria o uso de `propTypes` como discutido no capítulo *Tipagem em React*.

Agora que temos as ações no lugar, podemos usá-los no `App`:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import Notes from './Notes';

export default class App extends React.Component {
  constructor(props) {
    ...
  }
  render() {
    const {notes} = this.state;

    return (
      <div>
        <button onClick={this.addNote}>+</button>
leanpub-start-delete
        <Notes notes={notes} />
leanpub-end-delete
leanpub-start-insert
        <Notes notes={notes} onDelete={this.deleteNote} />
leanpub-end-insert
      </div>
    );
  }
  addNote = () => {
    ...
  }
leanpub-start-insert
  deleteNote = (id, e) => {
    // Evitando a propagação para `edit`
    e.stopPropagation();

    this.setState({
      notes: this.state.notes.filter(note => note.id !== id)
    });
  }
leanpub-end-insert
}
```

Depois de atualizar o navegador, você poderá apagar suas notas. Já pensando no futuro, adicionei uma linha extra na com `e.stopPropagation ()`. A idéia é dizer ao DOM que pare de propagar nosso evento, evitando acionar possíveis outros eventos na nossa estrutura ao excluirmos uma nota.

## Conclusão

Trabalhar com o React, muitas vezes é assim. Você identifica o componentes e fluxos com base nas suas necessidades. Aqui precisávamos modelar uma `Note` e, em seguida, projetar um fluxo de dados para que possamos ter controle suficiente no lugar certo.

Falta mais um recurso para fecharmos a primeira parte do Kanban. A edição é mais difícil de todas. Uma maneira de implementá-la seria através de um *editor em linha*. Ao implementar um componente apropriado agora, economizaremos tempo depois, quando tivermos que editar outra coisa. Antes de continuar com a implementação, vamos dar uma olhada melhor nos componentes do React para entender o tipo de funcionalidade que eles fornecem.
