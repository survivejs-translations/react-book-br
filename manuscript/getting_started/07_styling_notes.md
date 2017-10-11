# Estilizando nossa aplicação `Notes`

Esteticamente, nosso aplicativo atual é muito básico. Como aplicações bonitas são mais divertidas de usar, podemos melhorar isso. No nosso caso, iremos usar o jeito tradicional para estilos.

Em outras palavras, vamos utilizar algumas classes CSS ao redor e depois aplicar seletores CSS com base nelas. O capítulo *Estilos em React* discute várias outras abordagens em maior detalhe.

## Estilizando o botão `Add Note`

Para estilizar o botão `Add Note`, precisamos adicionar uma classe primeiro:

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
leanpub-start-delete
        <button onClick={this.addNote}>+</button>
leanpub-end-delete
leanpub-start-insert
        <button className="add-note" onClick={this.addNote}>+</button>
leanpub-end-insert
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
```

Também precisamos adicionar o correspondente estilo:

**app/main.css**

```css
...

leanpub-start-insert
.add-note {
  background-color: #fdfdfd;

  border: 1px solid #ccc;
}
leanpub-end-insert
```

Uma maneira mais geral de lidar com isso seria configurar um componente `Button` e estiliza-lo. Isso nos daria botões estilizados em toda a aplicação.

## Estilizando `Notes`

Atualmente, a lista `Notes` parece um pouco áspera. Podemos melhorar isso escondendo o estilo específico de lista. Também podemos corrigir a largura `Notes`, portanto, se o usuário entrar em uma tarefa longa, nossa interface ainda permanecerá fixa com alguma largura máxima. Um bom primeiro passo é anexar algumas classes a `Notes`, ficando mais fácil adicionar seus estilos:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
import Editable from './Editable';

export default ({
  notes,
  onNoteClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => (
leanpub-start-delete
  <ul>{notes.map(({id, editing, task}) =>
leanpub-end-delete
leanpub-start-insert
  <ul className="notes">{notes.map(({id, editing, task}) =>
leanpub-end-insert
    <li key={id}>
leanpub-start-delete
      <Note onClick={onNoteClick.bind(null, id)}>
leanpub-end-delete
leanpub-start-insert
      <Note className="note" onClick={onNoteClick.bind(null, id)}>
leanpub-end-insert
        <Editable
leanpub-start-insert
          className="editable"
leanpub-end-insert
          editing={editing}
          value={task}
          onEdit={onEdit.bind(null, id)} />
leanpub-start-delete
        <button onClick={onDelete.bind(null, id)}>x</button>
leanpub-end-delete
leanpub-start-insert
        <button
          className="delete"
          onClick={onDelete.bind(null, id)}>x</button>
leanpub-end-insert
      </Note>
    </li>
  )}</ul>
)
```

Para eliminar o estilo específico da lista, podemos aplicar regras como estas:

**app/main.css**

```css
...

leanpub-start-insert
.notes {
  margin: 0.5em;
  padding-left: 0;

  max-width: 10em;
  list-style: none;
}
leanpub-end-insert
```

## Estilizando notas individuais

Ainda está faltando os estilos para nosso componente `Note`. Antes de criar qualquer regra, devemos ter certeza de que temos nossas classes em `Editable`:

**app/components/Editable.jsx**

```javascript
import React from 'react';
leanpub-start-insert
import classnames from 'classnames';
leanpub-end-insert

leanpub-start-delete
export default ({editing, value, onEdit, ...props}) => {
leanpub-end-delete
leanpub-start-insert
export default ({editing, value, onEdit, className, ...props}) => {
leanpub-end-insert
  if(editing) {
leanpub-start-delete
    return <Edit value={value} onEdit={onEdit} {...props} />;
leanpub-end-delete
leanpub-start-insert
    return <Edit
      className={className}
      value={value}
      onEdit={onEdit}
      {...props} />;
leanpub-end-insert
  }

leanpub-start-delete
  return <span {...props}>{value}</span>;
leanpub-end-delete
leanpub-start-insert
  return <span className={classnames('value', className)} {...props}>
    {value}
  </span>;
leanpub-end-insert
}

class Edit extends React.Component {
  render() {
leanpub-start-delete
    const {value, onEdit, ...props} = this.props;
leanpub-end-delete
leanpub-start-insert
    const {className, value, onEdit, ...props} = this.props;
leanpub-end-insert

    return <input
      type="text"
leanpub-start-insert
      className={classnames('edit', className)}
leanpub-end-insert
      autoFocus={true}
      defaultValue={value}
      onBlur={this.finishEdit}
      onKeyPress={this.checkEnter}
      {...props} />;
  }
  ...
}
```

Dado que `className` aceita apenas uma *string*, pode ser difícil trabalhar com várias classes dependendo de alguma lógica. Este é o lugar onde um pacote conhecido como [classnames](https://www.npmjs.org/package/classnames) pode ser útil. Ele aceita valores quase arbitrários e converte isso em uma *string*, resolvendo o problema.

Agora, existem classes suficientes para estilizarmos o restante da aplicação. Podemos mostrar uma sombra abaixo da nota ao colocar o mouse. Também é um bom detalhe para mostrar o controle de exclusão no *hover*. Infelizmente, isso não funcionará em interfaces baseadas em toque, mas é bom o suficiente para essa demo:

**app/main.css**

```css
...

leanpub-start-insert
.note {
  overflow: auto;

  margin-bottom: 0.5em;
  padding: 0.5em;

  background-color: #fdfdfd;
  box-shadow: 0 0 0.3em .03em rgba(0,0,0,.3);
}
.note:hover {
  box-shadow: 0 0 0.3em .03em rgba(0,0,0,.7);

  transition: .6s;
}

.note .value {
  /* Força usar inline-block para que ele obtenha altura mínima */
  display: inline-block;
}

.note .editable {
  float: left;
}
.note .delete {
  float: right;

  padding: 0;

  background-color: #fdfdfd;
  border: none;

  cursor: pointer;

  visibility: hidden;
}
.note:hover .delete {
  visibility: visible;
}
leanpub-end-insert
```

Supondo que tudo correu bem, sua aplicação deve parecer mais ou menos assim agora:

![Styled Notes Application](../images/style_01.png)

## Conclusão

Esta é apenas uma forma de estilizar uma aplicação de React. Confiar em classes como essa, pode se tornar um problema à medida que a escala do seu aplicativo cresce. É por isso que existem formas alternativas de estilo que abordam esse problema específico. O capítulo *Estilos em React* toca muitas dessas técnicas.

Pode ser uma boa idéia experimentar algumas maneiras alternativas, e encontrar algo com o qual você se sinta confortável. Particularmente, **CSS Modules** é uma idéia promissora, à medida que resolvem o problema fundamental do CSS - o problema dos estados globais. A técnica permite criar um escopo local por componente. E isso se encaixa muito bem com React, já que estamos trabalhando com componentes por padrão.

Agora que temos um aplicativo simples e funcionando, podemos começar a implementar nosso quadro Kanban. Isso levará muita paciência, pois precisamos melhorar a maneira como lidamos com o estado da aplicação. Também precisamos adicionar alguma estrutura que está em falta, e ter certeza de que é possível arrastar e soltar as coisas. Esses são bons objetivos para a próxima parte do livro.
