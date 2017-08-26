# Caracteristicas da Linguagem

ES6 (ou ES2015) foi, sem dúvida, a maior mudança no JavaScript em muito tempo. Como resultado, recebemos uma grande variedade de novas funcionalidades. O objetivo deste apêndice é ilustrar os recursos utilizados no livro de forma isolada para que eles sejam mais claros, e também, para entender melhor como eles funcionam. Ao invés de falarmos de [toda a especificação](http://www.ecma-international.org/ecma-262/6.0/index.html), Vou focar apenas no subconjunto de recursos usados no livro.

## Módulos

ES6 introduziu declarações de módulos adequadas. Antes disso, nós faziamos alguns *hacks* em determinados formatos, como AMD ou CommonJS. As declarações do módulo ES6 são analisáveis estaticamente. Isso é altamente útil para autores de ferramentas. Isso significa que podemos obter recursos como *tree shaking*. Isso permite que a ferramenta ignore o código não utilizado, simplesmente analisando a estrutura de importação.

### Único `import` e `export`

Vamos ver um exemplo de como exportar um módulo diretamente, considere abaixo:

**persist.js**

```javascript
import makeFinalStore from 'alt-utils/lib/makeFinalStore';

export default function(alt, storage, storeName) {
  ...
}
```

**index.js**

```javascript
import persist from './persist';

...
```

### Múltiplos `import` e `export`

Às vezes, queremos usar módulos como escopo para várias funções:

**math.js**

```javascript
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export function square(a) {
  return a * a;
}
```

Poderíamos escrever o módulo dessa forma:

**math.js**

```javascript
const add = (a, b) => a + b;
const multiple = (a, b) => a * b;

// Se você quiser, você pode omitir os () quando utilizar apenas um parâmetro.
const square = a => a * a;

export {
  add,
  multiple,
  // Criar "alias" também funciona
  multiple as mul
};
```

O exemplo usa as *fat arrows*. Esta definição pode ser consumida através de uma importação como esta:

**index.js**

```javascript
import {add} from './math';

// Também podemos vincular os métodos a uma chave
// import * as math from './math';
// math.add, math.multiply, ...

...
```

 `export default` é especialmente útil se você preferir manter seus módulos focados. A função `persist` é um exemplo. Já o comum `export`, é útil para agrupar múltiplas funções debaixo do mesmo módulo.

T> Dado que a sintaxe de módulos ES6 são analisáveis estaticamente, ele permite ferramentas como [analyze-es6-modules](https://www.npmjs.com/package/analyze-es6-modules).

### Criando alias para imports

Muitas vezes, vale a pena criar um alias para os módulos que você está importando. Exemplo:

```javascript
import {actions as TodoActions} from '../actions/todo'

...
```

`as` permite que você evite conflitos entre nomes de variáveis.

### Webpack `resolve.alias`

Bundlers, como o Webpack, podem fornecer alguns recursos além disso. Você pode definir um `resolve.alias` para alguns de seus diretórios de módulos, por exemplo. Isso permitiria que você use-o diretamente na importação, como `import from 'libs/';`, independentemente de onde você escreva essa declaração. Um simples `resolve.alias` poderia ser assim:

```javascript
...
resolve: {
  alias: {
    libs: path.join(__dirname, 'libs')
  }
}
```

A documentação oficial descreve [Possíveis variações](https://webpack.github.io/docs/configuration.html#resolve-alias) em mais detalhes.

## Classes

Ao contrário de muitas outras linguages de programação, o JavaScript usa herança baseada em protótipo ao invés de baseada em classe. Ambas as abordagens têm seus méritos. Na verdade, você pode imitar um modelo baseado em classe através de um protótipo. As classes ES6 oferecem um açúcar sintático em cima desse mecanismo básico do JavaScript. Internamente, ele ainda usa o mesmo sistema. Mas para o programador, a sintaxe é diferente.

Atualmente, React suporta definições de componentes baseadas em classes. Nem todos concordam que é uma coisa boa. Dito isto, a definição pode ser bastante limpa, desde que você não abuse. Um exemplo simples, considere o código abaixo:

```javascript
import React from 'react';

export default class App extends React.Component {
  constructor(props) {
    super(props);

    // Esta é uma propriedade regular da sintaxe JavaScript, não é React.
    // Se você não precisar atualizar em cada render(),
    // ela pode não funcionar.
    this.privateProperty = 'private';

    // React specific state. Alter this through `this.setState`. That
    // Estado específico do React. Altere ele através de `this.setState`.
    // E que, eventualmente irá chamar `render()`.
    this.state = {
      name: 'Class demo'
    };
  }
  render() {
    // Usando as propriedades de alguma forma.
    const privateProperty = this.privateProperty;
    const name = this.state.name
    const notes = this.props.notes;

    ...
  }
}
```

Talvez a maior vantagem da abordagem baseada em classe seja o fato de que reduz a complexidade, especialmente quando se trata de métodos de ciclo de vida do React. No entando, é importante notar que os métodos de classe não serão obtidos por padrão! É por isso que o livro baseia-se em um recurso experimental conhecido como ***property initializers***.

### Classes e Módulos

Conforme mencionado acima, os módulos ES6 permitem usar `export` e `import` de objetos únicos/múltiplos, funções ou mesmo classes. No último, você pode usar `export default class` para exportar uma classe anônima ou exportar várias classes do mesmo módulo usando `export class className`.

Para exportar e importar uma única classe, você pode usar `export default class` para exportar uma classe anônima e chama-lá de qualquer coisa na hora da importação:

**Note.jsx**

```javascript
export default class extends React.Component { ... };
```

**Notes.jsx**

```javascript
import Note from './Note.jsx';
...
```

Ou use `export class className` para exportar várias classes nomeadas de um único módulo:

**Components.jsx**

```javascript
export class Note extends React.Component { ... };

export class Notes extends React.Component { ... };
```

**App.jsx**

```javascript
import {Note, Notes} from './Components.jsx';
...
```

Recomenda-se manter suas classes separadas em diferentes módulos.

## Propriedades de Classe e Inicializadores de Propriedades

As classes ES6 não vinculam seus métodos por padrão. Isso, muitas vezes, pode ser problemático, pois você ainda pode tentar acessar as propriedades da instância. Existe uma novidade, que ainda é experimental, chamados de [propriedades de classe e inicializadores de propriedades](https://github.com/jeffmo/es-class-static-properties-and-fields)(do inglês _class properties and property initializers_) que irão resolver esse problema. Sem eles, podemos escrever algo parecido com:

```javascript
import React from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.renderNote = this.renderNote.bind(this);
  }
  render() {
    // Irá usar `renderNote` de alguma maneira.
    ...

    return this.renderNote();
  }
  renderNote() {
    // renderNote foi vinculado à instância, podemos usar `this`
    return <div>{this.props.value}</div>;
  }
}
App.propTypes = {
  value: React.PropTypes.string
};
App.defaultProps = {
  value: ''
};

export default App;
```

Usando propriedades de classe e inicializações de propriedades, poderíamos escrever algo mais simples:

```javascript
import React from 'react';

export default class App extends React.Component {
  // definições de propType como métodos estáticos da classe
  static propTypes = {
    value: React.PropTypes.string
  }
  static defaultProps = {
    value: ''
  }
  render() {
    // Irá usar `renderNote` de alguma maneira.
    ...

    return this.renderNote();
  }
  // Inicializadores de propriedades removem a necessidade do `.bind`
  renderNote = () => {
    // renderNote foi vinculado à instância, podemos usar `this`
    return <div>{this.props.note}</div>;
  }
}
```

Agora que nós empurramos a declaração para o nível do método, o código fica mais simples. Eu decidi usar o recurso neste livro principalmente por esse motivo. Temos algo a menos para se preocupar.

## Funções

Tradicionalmente, o JavaScript é flexível com suas funções. Para lhe dar uma ideia melhor, veja a implementação do `map` abaixo:

```javascript
function map(cb, values) {
  var ret = [];
  var i, len;

  for(i = 0, len = values.length; i < len; i++) {
    ret.push(cb(values[i]));
  }

  return ret;
}

map(function(v) {
  return v * 2;
}, [34, 2, 5]); // yields [68, 4, 10]
```

In ES6 we could write it as follows:

```javascript
function map(cb, values) {
  const ret = [];
  const i, len;

  for(i = 0, len = values.length; i < len; i++) {
    ret.push(cb(values[i]));
  }

  return ret;
}

map((v) => v * 2, [34, 2, 5]); // yields [68, 4, 10]
```

A implementação do `map` é mais ou menos a mesmo coisa. O interessante está na forma como o chamamos essa função. Especialmente na parte, `(v) => v * 2`, é intrigante. Ao invés de ter que escrever `function` em todos os lugares, a sintaxe ES6 _arrow functions_, nos fornece uma pequena abreviação, bem útil! Para dar mais exemplos de uso, considere abaixo:

```javascript
// Estes são os mesmos
v => v * 2;
(v) => v * 2; // Prefiro esta variante para funções curtas
(v) => { // Use isso se precisar de várias declarações
  return v * 2;
}

// Podemos vinculá-los a uma variável
const double = (v) => v * 2;

console.log(double(2));

// Se você quiser usar um método curto retornando um objeto,
// Você precisa envolver o objeto em ().
v => ({
  foo: 'bar'
});
```

### Contexto das Arrow Functions

As _arrow functions_ são especiais, elas não têm o contexto do `this`. Ao invés disso, `this` indicará o escopo superior. Considere o exemplo abaixo:

```javascript
var obj = {
  context: function() {
    return this;
  },
  name: 'demo object 1'
};

var obj2 = {
  context: () => this,
  name: 'demo object 2'
};

console.log(obj.context()); // { context: [Function], name: 'demo object 1' }
console.log(obj2.context()); // {} em Node.js, e `window` no navegador
```

Como você pode notar no fragmento acima, a função anônima possui um `this` apontando para a função `context` no objeto` obj`. Em outras palavras, o escopo superior, do objeto `obj`, é vinculado ao `this`, para a função` context`.

Isso acontece porque `this` não aponta para o escopo do objeto que o contém, mas o escopo do objeto que está executando-o, como você pode ver no próximo fragmento de código:

```javascript
console.log(obj.context.call(obj2)); // { context: [Function], name: 'demo object 2' }
```

A _arrow function_ no `obj2` não vincula nenhum escopo ao seu contexto, seguindo as regras do **escopo lexical** (do inglês, _lexical scoping_), eles resolvem a referência ao escopo externo mais próximo. Neste caso, é o objeto `global` do Node.js.

Embora o comportamento possa parecer um pouco estranho, é realmente útil. No passado, se você quisesse acessar o contexto superior, você precisava armazena-lo em uma variável `var that = this;`. A introdução da sintaxe _arrow functions_, resolveu esse problema.

### Parâmetros de Função

Historicamente, lidar com parâmetros de uma função tem sido um pouco limitado. Existem vários hacks, como `values = values || [];`, mas eles não são agradáveis e são propensos a erros. Por exemplo, usar `||` pode causar problemas com zeros. O ES6 resolve este problema introduzindo parâmetros predefinidos. Agora, podemos simplesmente escrever `function map(cb, values=[])`.

Falando mais sobre isso, valores predefinidos podem até mesmo depender uns dos outros. Você também pode passar uma quantidade arbitrária de parâmetros através de `function map(cb, ...values)`. Nesse caso, você chamaria a função através do `map(a => a * 2, 1, 2, 3, 4)`. A API pode não ser perfeita para `map`, mas pode ter mais sentido em algum outro cenário.

Existe outros meios mais convenientes para extrair valores de objetos passados. Isso é altamente útil com componentes React, usando a sintaxe de função:

```javascript
export default ({name}) => {
  // Interpolação ES6. Usando back-ticks!
  return <div>{`Hello ${name}!`}</div>;
};
```

## Interpolação de String

Sabemos que lidar com strings era um tanto doloroso em JavaScript. Normalmente, usamos uma sintaxe como `'Hello' + name + '!'`. Abusando o uso do `+` para este propósito, talvez não seja o modo mais inteligente, pois pode levar a um comportamento estranho, devido à coerção de tipo. Por exemplo, `0 + ' world` irá retornar o resultado `0 world` como string.

Além de ser mais claro, a interpolação de strings em ES6 nos fornece strings de múltiplas linhas. Isso é algo que a sintaxe antiga não suportava. Considere os exemplos abaixo:

```javascript
const hello = `Hello ${name}!`;
const multiline = `
mútiplas
linhas de
coisas boas
`;
```

A sintaxe back-tick pode demorar um pouco para ser algo familiar, mas é poderosa e menos propensa a erros.

## Desestruturação

O `...` está relacionado à idéia de desestruturação. Por exemplo, `const {lane, ...props} = this.props;` irá extrair `lane` de `this.props` enquanto as chaves restantes do objeto irão para `props`. Essa sintaxe ainda é experimental. ES6 especifica uma maneira oficial de executar o mesmo para arrays:

```javascript
const [lane, ...rest] = ['foo', 'bar', 'baz'];

console.log(lane, rest); // 'foo', ['bar', 'baz']
```

O operador **spread** (`...`) é útil, também, para concatenar. Você verá uma sintaxe assim com freqüência nos exemplos de Redux. Eles dependem da proposta experimental [Object rest/spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread):

```javascript
[...state, action.lane];

// Isso é igual ao:
state.concat([action.lane])
```

A mesma ideia aplica-se aos componentes React:

```javascript
...

render() {
  const {value, onEdit, ...props} = this.props;

  return <div {...props}>Spread demo</div>;
}

...
```

W> Existem alguns problemas relacionadas ao operador de propagação. Dado que é *raso* por padrão, pode levar a um comportamento interessante que pode ser inesperado. Isto é mais problemático quando você estiver tentando usá-lo para clonar um objeto. Josh Black discute este problema detalhadamente em seu post no Medium, intitulado [Gotchas in ES2015+ Spread](https://medium.com/@joshblack/gotchas-in-es2015-spread-5db06dfb1e10).

## Inicializadores de Objetos

Para facilitar o trabalho com objetos, o ES6 oferece uma variedade de recursos apenas para isso. Como é mostrado no [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer), considere os exemplos abaixo:

```javascript
const a = 'demo';
const shorthand = {a}; // Same as {a: a}

// Métodos de atalho
const o = {
  get property() {},
  set property(value) {},
  demo() {}
};

// Propriedades computadas
const computed = {
  [a]: 'testing' // demo -> testing
};
```

## `const`, `let`, `var`

Em JavaScript, as variáveis são globais por padrão. `var` tem o escopo relacionado ao *nível da função*. Isso vai de encontro com outras linguagens que implementam o escopo de *nível de bloco*. ES6 introduz escopos de nível de bloco através do `let`.

Há também suporte para `const`, que garante que a referência para a variável não seja alterada. Isso não significa, no entanto, que você não pode modificar o conteúdo da variável. Se você está apontando para um objeto, você ainda pode alterar seu conteúdo!

Eu uso como padrão, `const`, sempre que possível. Se eu precisar de algo mutável, `let` resolve o problema. É difícil encontrar um bom uso para `var`, já que` const` e `let` resolvem necessidade de uma forma mais compreensível. Na verdade, todo o código desse livro, além deste apêndice, depende de `const`. Isso mostra como é fácil de utiliza-las.

## Decoradores

Given decorators are still an experimental feature and there's a lot to cover about them, there's an entire appendix dedicated to the topic. Read *Understanding Decorators* for more information.
Decoradores ainda são um recurso experimental e há muito a se discutir sobre, há um apêndice inteiro dedicado ao tópico. Leia *Entendendo Decorators * para mais informações.

## Conclusão

Há muito mais sobre ES6 e suas próximas especificações do que esse apêndice. Se você deseja entender melhor a especificação, [ES6 Katas](http://es6katas.org/) é um bom ponto de partida para aprender mais. Ter uma boa idéia do básico, já é o suficiente para te levar adiante.
