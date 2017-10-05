# Implementando um aplicativo de Notas

Agora que temos uma configuração agradável para desenvolvimento, podemos realmente fazer algum trabalho. Nosso objetivo aqui é construir uma breve aplicação de notas. Ela terá operações básicas de manipulação. Vamos expandir nossa aplicação a partir do zero e entrar em alguns problemas. Desta forma, você entenderá por que são necessárias arquiteturas, como o Flux.

## Modelo de dados inicial

Muitas vezes, uma boa maneira de começar a projetar um aplicativo é começar com o formato dos dados. Podemos modelar uma lista de notas da seguinte forma:

```javascript
[
  {
    id: '4e81fc6e-bfb6-419b-93e5-0242fb6f3f6a',
    task: 'Aprenda React'
  },
  {
    id: '11bbffc8-5891-4b45-b9ea-5c99aadf870f',
    task: 'Lavar roupa'
  }
];
```

Cada nota é um objeto que conterá os dados que precisamos, incluindo um `id` e uma `task` que queremos realizar. Mais tarde, é possível ampliar essa definição de dados para incluir coisas como a cor da nota ou o proprietário.

Poderíamos ignorar ids em nossa definição. Isso se tornaria problemático à medida que aumentamos nossa aplicação e adicionamos o conceito de referências. Cada coluna do Kanban precisa referenciar algumas notas. Ao adotar uma indexação adequada no início, nos salvamos algum esforço mais tarde.

T> Outra maneira interessante de abordar os dados seria normalizá-los. Nesse caso, acabaríamos com uma estrutura similar a `[<id> -> { id: '...', task: '...' }]`. Mesmo que haja alguma redundância, é conveniente operar usando uma estrutura com acesso via índices. A estrutura torna-se ainda mais útil uma vez que começamos a obter referências entre entidades de dados.

## Renderização de dados iniciais

Agora que temos um modelo de dados, podemos tentar renderizá-lo através do React. Nós vamos precisar de um componente para armazenar os dados. Vamos chamá-lo de `Notes`, por enquanto. Podemos melhorar a partir disso, pois queremos mais funcionalidades. Configure um arquivo com um pequeno componente da seguinte maneira:

**app/components/Notes.jsx**

```javascript
import React from 'react';

const notes = [
  {
    id: '4e81fc6e-bfb6-419b-93e5-0242fb6f3f6a',
    task: 'Aprenda React'
  },
  {
    id: '11bbffc8-5891-4b45-b9ea-5c99aadf870f',
    task: 'Lavar roupa'
  }
];

export default () => (
  <ul>{notes.map(note =>
    <li key={note.id}>{note.task}</li>
  )}</ul>
)
```

Estamos usando vários recursos importantes do JSX no trecho acima. As partes difíceis abaixo:

* `<ul>{notes.map(note => ...)}</ul>` - `{}` nos permitem misturar a sintaxe do JavaScript dentro do JSX. `map` retorna uma lista de elementos `li` para o React renderizar.
* `<li key={note.id}>{note.task}</li>` - Para dizer ao React quais itens foram alterados, adicionados ou removidos, usamos a propriedade `key`. É importante que isso seja único, ou o React não poderá descobrir a ordem correta para renderizar. Se não for definido, React dará um aviso. Veja [múltiplos componentes](https://facebook.github.io/react/docs/lists-and-keys.html#rendering-multiple-components) para mais informações.

Também precisamos adicionar o componente no ponto de entrada do nosso aplicativo:

**app/index.jsx**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
leanpub-start-insert
import Notes from './components/Notes';
leanpub-end-insert

if(process.env.NODE_ENV !== 'production') {
  React.Perf = require('react-addons-perf');
}

ReactDOM.render(
leanpub-start-delete
  <div>Hello world</div>,
leanpub-end-delete
leanpub-start-insert
  <Notes />,
leanpub-end-insert
  document.getElementById('app')
);
```

Se você executar o aplicativo agora, você deve ver uma lista de notas. Ainda não é bonito ou útil, mas é um começo:

![A list of notes](images/react_03.png)

T> Nós precisamos fazer o `import` do React no *Notes.jsx* dado que existe JSX, para realizar a transformação do JavaScript. Sem ele, o código falharia.

## Gerando Ids

Normalmente, o problema de gerar os ids é resolvido por um back-end. Como ainda não temos um, usaremos um padrão conhecido como [RFC4122](https://www.ietf.org/rfc/rfc4122.txt). Isso nos permite gerar ids únicos. Usaremos uma implementação Node.js conhecida como *uuid* e sua função `uuid.v4`. Ele nos dará ids, como `1c8e7a12-0b4c-4f23-938c-00d7161f94fc` e eles tem uma garantia, bem alta por sinal, de serem únicos.

Para conectar o gerador ao nosso aplicativo, vamos usá-lo da seguinte maneira:

**app/components/Notes.jsx**

```javascript
import React from 'react';
leanpub-start-insert
import uuid from 'uuid';
leanpub-end-insert

const notes = [
  {
leanpub-start-delete
    id: '4e81fc6e-bfb6-419b-93e5-0242fb6f3f6a',
leanpub-end-delete
leanpub-start-insert
    id: uuid.v4(),
leanpub-end-insert
    task: 'Aprenda React'
  },
  {
leanpub-start-delete
    id: '11bbffc8-5891-4b45-b9ea-5c99aadf870f',
leanpub-end-delete
leanpub-start-insert
    id: uuid.v4(),
leanpub-end-insert
    task: 'Lavar roupa'
  }
];

...
```

A configuração de desenvolvimento instalará a depêndencia `uuid` automaticamente. Uma vez instalado e o aplicativo atualizado, tudo deve permanecer o mesmo. Se você tentar debugar, você pode ver os IDs ao atualizar. Você pode verificar isso facilmente, inserindo uma declaração `console.log(notes);` ou [debugger;](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger) dentro da função do componente.

A declaração `debugger;` é particularmente útil porque diz ao navegador que interrompa a execução atual. Desta forma, você pode examinar a pilha de chamadas atual e as variáveis disponíveis. Se você não tiver certeza de algo, esta é uma ótima maneira de debugar e descobrir o que está acontecendo.

`console.log` é uma alternativa mais leve. Você pode até projetar um sistema de registro em torno dele e usar essas técnicas juntas. Acesse [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Console) e [Chrome documentation](https://developers.google.com/web/tools/chrome-devtools/debug/console/console-reference) para ver a API completa.

T> Se você estiver interessado na matemática por trás da geração de IDs, confira [os cálculos na Wikipedia](https://en.wikipedia.org/wiki/Universally_unique_identifier#Random_UUID_probability_of_duplicates) para mais detalhes. Você verá que a possibilidade de colisões é bem pequena e algo que não precisamos nos preocupar.

## Adicionando novas notas à lista

Agora, conseguimos mostrar notas individuais, mas ainda está faltando muita lógica para tornar nossa aplicação útil. Uma maneira lógica de começar isso seria implementar a adição de novas notas à lista. Para fazer isso, precisamos expandir o aplicativo um pouco.

### Definindo um Stub para nosso `App`

Para criar a adição de novas notas, devemos ter um botão para isso em algum lugar. Atualmente, nosso componente `Notes` faz apenas uma coisa, exibe notas. E isso é perfeitamente aceitável. Para ter mais espaço para mais funcionalidades, poderíamos adicionar um conceito conhecido como `App` em cima disso. Esse componente orquestará a execução de nossa aplicação. Podemos adicionar o botão que queremos por lá, e também, gerenciar o estado de como adicionar notas. Em um nível básico, o `App` poderia ficar assim:

**app/components/App.jsx**

```javascript
import React from 'react';
import Notes from './Notes';

export default () => <Notes />;
```

Tudo o que ele faz até o momento, é renderizar `Notes`, vai levar um tempo para torná-lo útil. Vamos adicionar o `App` na nossa aplicação, também precisamos ajustar o ponto de entrada da seguinte maneira:

**app/index.jsx**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
leanpub-start-delete
import Notes from './components/Notes';
leanpub-end-delete
leanpub-start-insert
import App from './components/App';
leanpub-end-insert

if(process.env.NODE_ENV !== 'production') {
  React.Perf = require('react-addons-perf');
}

ReactDOM.render(
leanpub-start-delete
  <Notes />,
leanpub-end-delete
leanpub-start-insert
  <App />,
leanpub-end-insert
  document.getElementById('app')
);
```

Agora, ao executar o aplicativo, ele deve ser exatamente o mesmo que antes. Mas agora temos mais espaço para crescer.

### Adicionando um Stub para o botão *Adicionar*

Um bom passo para algo mais funcional é adicionar um *stub* para um botão *Adicionar*. Para conseguir isso, o `App` precisa evoluir:

**app/components/App.jsx**

```
import React from 'react';
import Notes from './Notes';

leanpub-start-delete
export default () => <Notes />;
leanpub-end-delete
leanpub-start-insert
export default () => (
  <div>
    <button onClick={() => console.log('adicionar nota')}>+</button>
    <Notes />
  </div>
);
leanpub-end-insert
```

Se você clicar no botão que adicionamos, você deve ver uma mensagem "adicionar nota" no console do navegador. Ainda temos que conectar o botão com nossos dados de alguma forma. Atualmente, os dados estão no componente `Notes`, então, antes que possamos fazer isso, precisamos extraí-lo para o nível` App`.

T> Apesar de componentes React devolverem um único elemento, devemos envolver o nosso aplicativo dentro de um `div`.

### Pushing Data to `App`

To push the data to `App` we need to make two modifications. First we need to literally move it there and pass the data through a prop to `Notes`. After that we need to tweak `Notes` to operate based on the new logic. Once we have achieved this, we can start thinking about adding new notes.

The `App` side is simple:

**app/components/App.jsx**

```javascript
import React from 'react';
leanpub-start-insert
import uuid from 'uuid';
leanpub-end-insert
import Notes from './Notes';

leanpub-start-insert
const notes = [
  {
    id: uuid.v4(),
    task: 'Aprenda React'
  },
  {
    id: uuid.v4(),
    task: 'Lavar roupa'
  }
];
leanpub-end-insert

export default () => (
  <div>
    <button onClick={() => console.log('adicionar nota')}>+</button>
leanpub-start-delete
    <Notes />
leanpub-end-delete
leanpub-start-insert
    <Notes notes={notes} />
leanpub-end-insert
  </div>
);
```

This won't do much until we tweak `Notes` as well:

**app/components/Notes.jsx**

```javascript
import React from 'react';
leanpub-start-delete
import uuid from 'uuid';

const notes = [
  {
    id: uuid.v4(),
    task: 'Aprenda React'
  },
  {
    id: uuid.v4(),
    task: 'Lavar roupa'
  }
];

export default () => {
leanpub-end-delete
leanpub-start-insert
export default ({notes}) => (
leanpub-end-insert
  <ul>{notes.map(note =>
    <li key={note.id}>{note.task}</li>
  )}</ul>
);
```

Our application should look exactly the same as it did before these changes. Now we are ready to add some logic to it.

T> The way we extract `notes` from `props` (the first parameter) is a standard trick you see with React. If you want to access the remaining `props`, you can use `{notes, ...props}` kind of syntax. We'll use this later so you can get a better feel for how this works and why you might use it.

### Pushing State to `App`

Now that we have everything in the right place, we can start to worry about modifying the data. If you have used JavaScript before, the intuitive way to handle it would be to set up an event handler like `() => notes.push({id: uuid.v4(), task: 'New task'})`. If you try this, you'll see that nothing happens.

The reason why is simple. React cannot notice the structure has changed and won't react accordingly (that is, trigger `render()`). To overcome this issue, we can implement our modification through React's own API. This makes it notice that the structure has changed. As a result it is able to `render()` as we might expect.

As of the time of writing, the function based component definition doesn't support the concept of state. The problem is that these components don't have a backing instance. It is something in which you would attach state. We might see a way to solve this through functions only in the future but for now we have to use a heavy duty alternative.

In addition to functions, you can create React components through `React.createClass` or a class based component definition. In this book we'll use function based components whenever possible. If there's a good reason why those can't work, then we'll use the class based definition instead.

In order to convert our `App` to a class based component, adjust it as follows to push the state within:

**app/components/App.jsx**

```javascript
import React from 'react';
import uuid from 'uuid';
import Notes from './Notes';

leanpub-start-delete
const notes = [
  {
    id: uuid.v4(),
    task: 'Aprenda React'
  },
  {
    id: uuid.v4(),
    task: 'Lavar roupa'
  }
];

export default () => (
  <div>
    <button onClick={() => console.log('adicionar nota')}>+</button>
    <Notes notes={notes} />
  </div>
);
leanpub-end-delete

leanpub-start-insert
export default class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      notes: [
        {
          id: uuid.v4(),
          task: 'Aprenda React'
        },
        {
          id: uuid.v4(),
          task: 'Lavar roupa'
        }
      ]
    };
  }
  render() {
    const {notes} = this.state;

    return (
      <div>
        <button onClick={() => console.log('adicionar nota')}>+</button>
        <Notes notes={notes} />
      </div>
    );
  }
}
leanpub-end-insert
```

After this change `App` owns the state even though the application still should look the same as before. We can begin to use React's API to modify the state.

T> Data management solutions, such as [MobX](https://mobxjs.github.io/mobx/), solve this problem in their own way. Using them you annotate your data structures and React components and leave the updating problem to them. We'll get back to the topic of data management later in this book in detail.

T> We're passing `props` to `super` by convention. If you don't pass it, `this.props` won't get set! Calling `super` invokes the same method of the parent class and you see this kind of usage in object oriented programming often.

### Implementing `Note` Adding Logic

All the effort will pay off soon. We have just one step left. We will need to use React's API to manipulate the state and to finish our feature. React provides a method known as `setState` for this purpose. In this case we will need to call it like this: `this.setState({... new state goes here ...}, () => ...)`.

The callback is optional. React will call it when the state has been set and often you don't need to care about it at all. Once `setState` has gone through, React will call `render()`. The asynchronous API might feel a little strange at first but it will allow React to optimize its performance by using techniques such as batching updates. This all ties back to the concept of Virtual DOM.

One way to trigger `setState` would be to push the associated logic to a method of its own and then call it when a new note is added. The class based component definition doesn't bind custom methods like this by default so we will need to handle the binding somehow. It would be possible to do that at the `constructor`, `render()`, or by using a specific syntax. I'm opting for the syntax option in this book. Read the *Language Features* appendix to learn more.

To tie the logic with the button, adjust `App` as follows:

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
leanpub-start-delete
        <button onClick={() => console.log('adicionar nota')}>+</button>
leanpub-end-delete
leanpub-start-insert
        <button onClick={this.addNote}>+</button>
leanpub-end-insert
        <Notes notes={notes} />
      </div>
    );
  }
leanpub-start-insert
  addNote = () => {
    // It would be possible to write this in an imperative style.
    // I.e., through `this.state.notes.push` and then
    // `this.setState({notes: this.state.notes})` to commit.
    //
    // I tend to favor functional style whenever that makes sense.
    // Even though it might take more code sometimes, I feel
    // the benefits (easy to reason about, no side effects)
    // more than make up for it.
    //
    // Libraries, such as Immutable.js, go a notch further.
    this.setState({
      notes: this.state.notes.concat([{
        id: uuid.v4(),
        task: 'New task'
      }])
    });
  }
leanpub-end-insert
}
```

Given we are binding to an instance here, the hot loading setup cannot pick up the change. To try out the new feature, refresh the browser and try clicking the `+` button. You should see something:

![Notes with a plus](images/react_05.png)

T> If we were operating with a back-end, we would trigger a query here and capture the id from the response. For now it's enough to just generate an entry and a custom id.

T> You could use `this.setState({notes: [...this.state.notes, {id: uuid.v4(), task: 'New task'}]})` to achieve the same result. This [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) can be used with function parameters as well. See the *Language Features* appendix for more information.

T> Using [autobind-decorator](https://www.npmjs.com/package/autobind-decorator) would be a valid alternative for property initializers. In this case we would use `@autobind` annotation either on class or method level. To learn more about decorators, read the *Understanding Decorators* appendix.

## Conclusion

Even though we have a rough application together now, we are still missing two crucial features: editing and deleting notes. It's a good time to focus on those next. Let's do deletion first and handle editing after that.
