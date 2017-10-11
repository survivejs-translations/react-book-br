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

## Conectando `Editable` com `Notes`

Ainda precisamos substituir as partes relevantes do código para usarmos o `Editable`. Ainda restam algumas `props` para conectar:

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

Se tudo correu bem, você deveria ver algo assim:

![Connected `Editable`](../images/react_06.png)

## Identificando o estado `editing` em `Note`

Ainda está em falta, lógicas necessárias para controlar o `Editable`. Dado que o estado do nosso aplicativo é mantido no `App`, precisamos conectar esse estado. Devemos configurar o estado `editable` da nota em questão para `true` quando começamos a editar e configurá-lo de novo para `false` quando concluirmos o processo de edição. Devemos também, atualizar `task` para usar o novo valor. Por enquanto, estamos interessados em obter apenas a o estado `editable`. Vamos atualizar da seguinte maneira:

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

Agora, se você tentar editar uma `Note`, você deve ver algo assim:

![Tracking `editing` state](../images/react_07.png)

Se você clicar na `Note` duas vezes para fazer a edição, você deverá ver um error como `Uncaught Invariant Violation` no console do navegador. Isso acontece porque não estamos lidando com `task` corretamente, ainda! Nós ligamos apenas o `id` e` task` apontará para um objeto `event` fornecido pelo React. Isso é algo que devemos corrigir.

T> Se usarmos uma estrutura de dados normalizada (ex: `{<id>: {id: <id>, task: <str>}}`), seria possível escrever as operações usando`Object.assign` e evitar mutações.

T> Para limpar o código, você pode extrair um método para guardar a lógica compartilhada por `activateNoteEdit` e `editNote`.

## Implementando `Edit`

Falta mais uma parte para que isso funcione. No momento, conseguimos controlar o estado `editing` por `Note`, mas ainda não podemos editá-los. Para este propósito, precisamos expandir `Edit` e fazer com que ele gere um campo de texto para nós.

Neste caso, vamos usar um componente **não controlado** e extrair seu valor do DOM somente quando precisarmos. Não há necessidade de mais controle do que isso.

Considere o código abaixo para a implementação completa. Observe como estamos lidando com a edição. Capturamos `onKeyPress` e verificamos o `Enter` para confirmar a edição. Nós também executamos a lógica em `onBlur` para que possamos terminar a edição quando o campo de texto perca o foco:

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

Se você atualizar e editar uma nota, iremos ver:

![Editing a `Note`](../images/react_08.png)

## Um pouco sobre nomenclatura de componentes

Nós poderíamos ter abordado `Editable` de uma maneira diferente. Em uma edição anterior deste livro acabei criando um componente único. Controlando a renderização do valor/campo de texto a partir de métodos (ou seja, `renderValue`). Muitas vezes, o nome do método como esse é uma pista de que é possível refatorar seu código e extrair componentes separados, como fizemos aqui.

Você pode dar um passo adiante e [criar namespaces](https://facebook.github.io/react/docs/jsx-in-depth.html#namespaced-components) para seus componentes. Seria possível definir os componentes `Editable.Value` e `Editable.Edit`. Melhor ainda, poderíamos ter permitido ao usuário trocar esses componentes através de `props`. Enquanto a interface for a mesma, os componentes devem funcionar. Isso proporcionaria uma personalização extra.

Um exemplo para isso, teríamos que fazer algo como:

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

Você também pode usar uma abordagem similar para componentes mais genéricos. Considere algo como `Form`. Você poderia facilmente ter `Form.Label`,` Form.Input`, `Form.Textarea` e assim por diante. Cada um conteria sua formatação e lógica personalizada conforme necessário. Esta é uma maneira de tornar seus projetos mais flexíveis.

## Conclusão

Foram necessários alguns passos, mas agora, podemos editar nossas anotações. O melhor de tudo, `Editable` deve ser útil sempre que precisarmos editar alguma propriedade. Poderíamos ter extraído a lógica mais tarde, conforme a duplicação aparecesse, mas essa é uma maneira de se fazer.

Mesmo que nossa aplicação funcione, ainda é bastante feio. Vamos fazer algo sobre isso no próximo capítulo, pois iremos adicionar alguns estilos básicos.
