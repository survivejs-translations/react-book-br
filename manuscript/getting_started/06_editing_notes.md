# Editando `Notes`

A edição de notas é um problema semelhante ao de eliminá-los. O fluxo de dados é exatamente o mesmo. Precisamos definir uma chamada para `onEdit` e fazer o `bind` com o id da nota que está sendo editada em` Notes`.

O que dificulta nesse cenário é a necessidade da interface do usuário. Não basta apenas ter um botão. Precisaremos de alguma maneira, permitir que o usuário insira um novo valor para podermos atualizar o modelo de dados.

Uma maneira de conseguir isso é implementar o chamado **edição em linha**. A idéia é que, quando um usuário clicar em uma nota, mostraremos um campo de texto. Depois que o usuário terminar de editar e precionar *enter* ou cicar fora do campo (disparando o evento `blur`), vamos capturar o valor e realizar a atualização.

## Implementando `Editable`

Para manter a aplicação limpa, vamos isolar esse comportamento em um componente conhecido como `Editable`. Isso nos dará uma API como esta:

```javascript
<Editable
  editing={editing}
  value={task}
  onEdit={onEdit.bind(null, id)} />
```

Isso é um exemplo de um componente **controlado**. Controlaremos o estado de edição explicitamente de fora do componente. Isso nos dá mais poder, mas também torna `Editable` mais verboso de se usar.

T> Pode ser uma boa ideia nomear seus callbacks usando o prefixo `on`. Isso permitirá que você os identifique dentre outras `props` e mantenha seu código um pouco mais limpo.

### Um pouco mais sobre controlado vs não controlado

Uma maneira alternativa de lidar com a edição teria sido deixar o controle sobre o estado de `editing` para `Editable`. Isso é uma maneira de **não controlar** o componente, é uma maneira de projetar componentes que pode ser válida se você não quer fazer nada com o estado fora do componente.

É possível usar esses dois meios juntos. Você pode até ter um componente controlado que tenha elementos não controlados dentro. No nosso caso, iremos usar um componente não controlado para o `input` e um controlado para o `Editable`. E podemos transformar isso em algo controlado, se quisermos.

O componente `Editable` consiste em duas porções separadas. Precisamos mostrar o valor padrão enquanto não estivermos no modo de `editing`. Quando estivermos no modo `editing`, queremos mostrar um controle `Edit`. Nesse caso, nos contentaremos com um campo de texto simples, isso fará o truque.

Antes de entrar nos detalhes, podemos implementar um pequeno `stub` e conectar isso ao aplicativo. Isso nos dará a estrutura básica que precisamos para crescer. Para começar, ajustaremos a hierarquia de componentes facilitar a implementação do `stub`.

T> A documentação oficial do React fala sobre [componentes controlados](https://facebook.github.io/react/docs/forms.html) em mais detalhes.

## Extraindo a renderização do `Note`

Atualmente `Note` controla o que é renderizado dentro dele. Ele processa a tarefa passada e conecta um botão de exclusão. Poderíamos colocar `Editable` dentro dele e fazer a ligação através das `props` do `Note`. Mesmo que essa possa ser uma maneira válida de se fazer, podemos empurrar a preocupação de renderização para um nível superior.

Tendo o conceito de `Note` é útil, especialmente, quando expandirmos o aplicativo, portanto, não há necessidade de removê-lo. Ao invés disso, podemos dar o controle sobre o comportamento de renderização para `Notes` e conectar tudo lá.

React fornece uma interface conhecida como `children` para isso. Ajustando `Note` e `Notes` da seguinte forma, iremos deixar o controle de renderização do `Note` para `Notes`:

**app/components/Note.jsx**

```javascript
import React from 'react';

leanpub-start-delete
export default ({task, onDelete}) => (
  <div>
    <span>{task}</span>
    <button onClick={onDelete}>x</button>
  </div>
);
leanpub-end-delete
leanpub-start-insert
export default ({children, ...props}) => (
  <div {...props}>
    {children}
  </div>
);
leanpub-end-insert
```

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';

export default ({notes, onDelete=() => {}}) => (
  <ul>{notes.map(({id, task}) =>
    <li key={id}>
leanpub-start-delete
      <Note
        onDelete={onDelete.bind(null, id)}
        task={task} />
leanpub-end-delete
leanpub-start-insert
      <Note>
        <span>{task}</span>
        <button onClick={onDelete.bind(null, id)}>x</button>
      </Note>
leanpub-end-insert
    </li>
  )}</ul>
)
```

Agora, nós temos espaço para trabalhar, podemos configurar um `stub` para `Editable`.

## Adicionando o stub de `Editable`

Podemos modelar um ponto de partida com base em nossa especificação abaixo. A ideia é que vamos fazer uma ramificação com base na `props` chamada `editing` e realizar a lógica necessária para implementar a funcionalidade:

**app/components/Editable.jsx**

```javascript
import React from 'react';

export default ({editing, value, onEdit, ...props}) => {
  if(editing) {
    return <Edit value={value} onEdit={onEdit} {...props} />;
  }

  return <span {...props}>value: {value}</span>;
}

const Edit = ({onEdit = () => {}, value, ...props}) => (
  <div onClick={onEdit} {...props}>
    <span>edit: {value}</span>
  </div>
);
```

Para ver o nosso `stub` em ação, ainda precisamos conectá-lo ao nosso aplicativo.

## Connecting `Editable` with `Notes`

We still need to replace the relevant portions of the code to point at `Editable`. There are more props to track and to connect:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
leanpub-start-insert
import Editable from './Editable';
leanpub-end-insert

leanpub-start-delete
export default ({notes, onDelete=() => {}}) => (
leanpub-end-delete
leanpub-start-insert
export default ({
  notes,
  onNoteClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => (
leanpub-end-insert
leanpub-start-delete
  <ul>{notes.map(({id, task}) =>
    <li key={id}>
      <Note>
        <span>{task}</span>
        <button onClick={onDelete.bind(null, id)}>x</button>
      </Note>
    </li>
  )}</ul>
leanpub-end-delete
leanpub-start-insert
  <ul>{notes.map(({id, editing, task}) =>
    <li key={id}>
      <Note onClick={onNoteClick.bind(null, id)}>
        <Editable
           editing={editing}
           value={task}
           onEdit={onEdit.bind(null, id)} />
        <button onClick={onDelete.bind(null, id)}>x</button>
      </Note>
    </li>
  )}</ul>
leanpub-end-insert
)
```

If everything went right, you should see something like this:

![Connected `Editable`](images/react_06.png)

## Tracking `Note` `editing` State

We are still missing logic needed to control the `Editable`. Given the state of our application is maintained at `App`, we'll need to deal with it there. It should set the `editable` flag of the edited note to `true` when we begin to edit and set it back to `false` when we complete the editing process. We should also adjust its `task` using the new value. For now we are interested in just getting the `editable` flag to work, though. Modify as follows:

**app/components/App.jsx**

```javascript
...

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
        <Notes notes={notes} onDelete={this.deleteNote} />
leanpub-end-delete
leanpub-start-insert
        <Notes
          notes={notes}
          onNoteClick={this.activateNoteEdit}
          onEdit={this.editNote}
          onDelete={this.deleteNote}
          />
leanpub-end-insert
      </div>
    );
  }
  addNote = () => {
    ...
  }
  deleteNote = (id, e) => {
    ...
  }
leanpub-start-insert
  activateNoteEdit = (id) => {
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = true;
        }

        return note;
      })
    });
  }
  editNote = (id, task) => {
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = false;
          note.task = task;
        }

        return note;
      })
    });
  }
leanpub-end-insert
}
```

If you try to edit a `Note` now, you should see something like this:

![Tracking `editing` state](images/react_07.png)

If you click a `Note` twice to confirm the edit, you should see an `Uncaught Invariant Violation` error at the browser console. This happens because we don't deal with `task` correctly yet. We have bound only `id` and `task` will actually point to an `event` object provided by React. This is something we should fix next.

T> If we used a normalized data structure (i.e., `{<id>: {id: <id>, task: <str>}}`), it would be possible to write the operations using `Object.assign` and avoid mutation.

T> In order to clean up the code, you could extract a method to contain the logic shared by `activateNoteEdit` and `editNote`.

## Implementing `Edit`

We are missing one more part to make this work. Even though we can manage the `editing` state per `Note` now, we still can't actually edit them. For this purpose we need to expand `Edit` and make it render a text input for us.

In this case we'll be using **uncontrolled** design and extract the value of the input from the DOM only when we need it. We don't need more control than that here.

Consider the code below for the full implementation. Note how we are handling finishing the editing. We capture `onKeyPress` and check for `Enter` to confirm editing. We also run the finish logic `onBlur` so that we can end the editing when the input loses focus:

**app/components/Editable.jsx**

```javascript
...


export default ({editing, value, onEdit, ...props}) => {
  if(editing) {
    return <Edit value={value} onEdit={onEdit} {...props} />;
  }

leanpub-start-delete
  return <span {...props}>value: {value}</span>;
leanpub-end-delete
leanpub-start-insert
  return <span {...props}>{value}</span>;
leanpub-end-insert
}

leanpub-start-delete
const Edit = ({onEdit = () => {}, value, ...props}) => (
  <div onClick={onEdit} {...props}>
    <span>edit: {value}</span>
  </div>
);
leanpub-end-delete
leanpub-start-insert
class Edit extends React.Component {
  render() {
    const {value, onEdit, ...props} = this.props;

    return <input
      type="text"
      autoFocus={true}
      defaultValue={value}
      onBlur={this.finishEdit}
      onKeyPress={this.checkEnter}
      {...props} />;
  }
  checkEnter = (e) => {
    if(e.key === 'Enter') {
      this.finishEdit(e);
    }
  }
  finishEdit = (e) => {
    const value = e.target.value;

    if(this.props.onEdit) {
      this.props.onEdit(value);
    }
  }
}
leanpub-end-insert
```

If you refresh and edit a note, the commits should go through:

![Editing a `Note`](images/react_08.png)

## On Namespacing Components

We could have approached `Editable` in a different way. In an earlier edition of this book I ended up developing it as a single component. I handled rendering the value and the edit control through methods (i.e., `renderValue`). Often method naming like that is a clue that it's possible to refactor your code and extract separate components like we did here.

You can go one step further and [namespace](https://facebook.github.io/react/docs/jsx-in-depth.html#namespaced-components)  your component parts. It would have been possible to define `Editable.Value` and `Editable.Edit` components. Better yet, we could have allowed the user to swap those components through props. As long as the interface is the same, the components should work. This would give an extra dimension of customizability.

Implementation-wise we would have had to do something like this in case we had gone with namespacing:

**app/components/Editable.jsx**

```javascript
import React from 'react';

// We could allow edit/value to be swapped here through props
const Editable = ({editing, value, onEdit}) => {
  if(editing) {
    return <Editable.Edit value={value} onEdit={onEdit} />;
  }

  return <Editable.Value value={value} />;
};

Editable.Value = ({value, ...props}) => <span {...props}>{value}</span>

class Edit extends React.Component {
  ...
}
Editable.Edit = Edit;

// We could export individual components too to allow modification
export default Editable;
```

You can use a similar approach for more generic components as well. Consider something like `Form`. You could easily have `Form.Label`, `Form.Input`, `Form.Textarea` and so on. Each would contain your custom formatting and logic as needed. This is one way to make your designs more flexible.

## Conclusion

It took quite a few steps, but we can edit our notes now. Best of all, `Editable` should be useful whenever we need to edit some property. We could have extracted the logic later on as we see duplication, but this is one way to do it.

Even though the application kind of works, it is still quite ugly. We'll do something about that in the next chapter as we add basic styling to it.
