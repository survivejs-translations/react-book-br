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

![A list of notes](../images/react_03.png)

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

### Movendo dados para o `App`

Para mover nossos dados para o `App` precisamos fazer duas modificações. Primeiro, precisamos literalmente mover e passar os dados através de uma *prop* para `Notes`. Depois disso, precisamos ajustar as `Notes` para operar com base na nova lógica. Uma vez que conseguimos isso, podemos começar a pensar em adicionar novas notas.

O `App` se parecerá com:

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

Isso não fará muito, precisamos ajustar `Notes` também:

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

Nosso aplicativo deve ser exatamente o mesmo que antes após essas mudanças. Agora estamos prontos para adicionar alguma lógica nele.

T> A maneira como extraímos `notes` de`props` (o primeiro parâmetro) é um truque que você vê com o React. Se quiser acessar as `props` restantes, você pode usar a sintaxe `{notes, ... props}`. Usaremos isso mais tarde para que você possa ter uma melhor sensação de como isso funciona e por que você pode usá-lo.

### Movendo estados para o `App`

Agora que temos tudo no lugar certo, podemos começar a nos preocupar com a modificação dos nossos dados. Se você já usou JavaScript antes, a maneira intuitiva de lidar com isso seria configurar um manipulador de eventos como `() => notes.push({id: uuid.v4(), task: 'Nova tarefa'})`. Se você tentar isso, verá que nada acontece.

A razão pela qual, é simples. React pode não perceber que a estrutura mudou e não irá atualizar seu estado (isto é, executar o `render()`). Para corrigir este problema, podemos implementar nossa modificação através da própria API do React. Isso faz com que ele perceba que a estrutura mudou. Como resultado, o `render()` será executado, como esperado.

No momento dessa escrita, a componentes baseados em função, não suportam o conceito de estado. O problema é que esses componentes não possuem uma instância. Isso é, algo em que você eles poderiam anexar um estado. Talvez no futuro, isso seja resolvido, mas por enquanto, temos que usar uma alternativa para isso.

Além das funções, você pode criar componentes no React através de `React.createClass` ou componente baseado em classes. Neste livro, usaremos componentes baseados em funções sempre que possível. Se houver uma boa razão, então usaremos a definição baseada em classe.

Para converter nosso `App` para um componente baseado em classe, vamos altera-lo da seguinte forma:

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

Após esta mudança, `App` possui um estado, mesmo que o aplicativo ainda seja o mesmo de antes. Podemos começar a usar a API do React para modificar o estado.

T> Soluções de gerenciamento de dados, como [MobX](https://mobxjs.github.io/mobx/), resolvem esse problema à sua maneira. Usando-os, você anota suas estruturas de dados nos componentes React e deixa o problema de atualização para essas bibliotecas. Voltaremos ao tópico de gerenciamento de dados mais adiante neste livro.

T> Estamos passando `props` para `super`, por convenção. Se você não passar, `this.props` não será configurado! Chamar `super` invoca o mesmo método da classe pai e você vê esse tipo de uso em programação orientada a objetos com freqüência.

### Implementando lógica para adicionar `Note`

Todo o esforço irá se pagar em breve. Temos apenas um passo à faltando. Precisamos usar a API do React para manipular o estado e terminar nosso recurso. React fornece um método conhecido como `setState` para isso. Nesse caso, iremos chamá-lo dessa maneira: `this.setState({... new state goes here ...}, () => ...)`.

A função de *callback* é opcional. React irá chamá-lo quando o estado for alterado e muitas vezes você não precisa se preocupar com isso. Uma vez que o `setState` terminado, o React executará `render()`. A API assíncrona pode ser um pouco estranha no início, mas permitirá que o React otimize seu desempenho usando técnicas como atualizações por lotes (*batch updates*). Isso tudo vincula o conceito do Virtual DOM.

Uma maneira de executar o `setState` seria mover a lógica de um próprio método e chamá-lo quando uma nova nota for adicionada. A definição de componente baseada em classe não vincula métodos personalizados como este por padrão, então, precisamos lidar com isso de alguma forma. Seria possível fazer isso no `constructor`, `render()`, ou usando uma sintaxe específica. Estou optando pela opção de sintaxe neste livro. Leia mais no apêndice *Caracteristicas da Linguagem* para aprender mais.

Para chamar a lógica ao clicar no botão, iremos mudar o `App` da seguinte maneira:

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
    // Seria possível escrever isso em um estilo imperativo.
    // por exemplo, através de `this.state.notes.push` e então
    // `this.setState({notes: this.state.notes})` para persistir.
    //
    // Eu prefiro favorecer o estilo funcional sempre que possível.
    // Embora possa ser mais verboso as vezes, eu sinto
    // que os benefícios (fácil de entender, sem efeitos colaterais)
    // valem a pena.
    //
    // Bibliotecas como Immutable.js, vão um passo além.
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

Dado que estamos vinculando uma instância aqui, a configuração de *hot loading* não irá refletir a alteração. Para ver os novos recursos, atualize o navegador e tente clicar no botão `+`. Você deve ver algo:

![Notes with a plus](../images/react_05.png)

T> Se estivéssemos operando com um back-end, iríamos fazer uma consulta e capturar o id da resposta. Por enquanto, basta apenas gerar uma entrada e um ID personalizado.

T> Você poderia usar `this.setState({notes: [...this.state.notes, {id: uuid.v4(), task: 'New task'}]})` para coneguir o mesmo resultado. O [operador spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) também pode ser usado com parâmetros de função. Veja o apêndice *Caracteristicas da Linguagem* para mais detalhes.

T> Usando [autobind-decorator](https://www.npmjs.com/package/autobind-decorator) seria uma alternativa válida para inicializadores de propriedade. Neste caso, usaríamos a anotação `@autobind` na classe ou método. Para aprender mais sobre *decorators*, leia o apêndice *Entendendo Decorators*.

## Conclusão

Com uma quase- aplicação, ainda está faltando duas características cruciais: edição e exclusão de notas. É um bom momento para se concentrar nos próximos passos. Vamos fazer a exclusão primeiro e lidar com a edição depois disso.
