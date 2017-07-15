# Entendendo Decorators

Se você já usou linguagens, como Java ou Python, você pode estar familiarizado com a idéia. Os **Decorators** são açúcares sintáticos que nos permitem anotar classes e funções. Em sua [proposta atual](https://github.com/wycats/javascript-decorators) (estágio 1) somente a anotação de classe e métodos é suportado. As funções podem ser suportadas mais adiante.

No Babel 6 você pode ativar esse comportamento através dos plugins [babel-plugin-syntax-decorators](https://www.npmjs.com/package/babel-plugin-syntax-decorators) e [babel-plugin-transform-decorators-legacy](https://www.npmjs.com/package/babel-plugin-transform-decorators-legacy). O primeiro fornece suporte ao nível da sintaxe, enquanto o último trás o tipo de comportamento que vamos discutir aqui.

O maior benefício dos decoradores é que eles nos permitem envolver o comportamento em pedaços simples e reutilizáveis, reduzindo a quantidade de ruído em nosso código. Certamente, é possível codificar sem eles. Eles apenas tornam certas tarefas mais legais, como vimos com as anotações relacionadas com o **drag and drop** na nossa aplicação.

## Implementando um decorador de Logging

Às vezes, é útil saber como os métodos são chamados. Você pode, naturalmente, adicionar um `console.log`, mas é mais divertido implementar um decorador `@log`. Essa é uma maneira mais controlável de lidar com isso. Considere o exemplo abaixo:

```javascript
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling "${name}" with`, arguments);

    return oldValue.apply(null, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```

A idéia é que, o nosso decorador `log`, envolve a função original, executando um `console.log`, e finalmente, chama a função original passando o objeto [arguments](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/arguments). Se você nunca viu o objeto `arguments` ou `apply` antes, isso pode parecer um pouco estranho.

Você pode pensar no `apply` como uma outra maneira de invocar uma função enquanto passa seu contexto (`this`) e parâmetros como array. `arguments` recebe parâmetros da função, implicitamente, por isso é ideal para este caso.

Este logger pode ser extraído para um módulo separado. Depois disso, poderíamos usá-lo em nosso aplicativo sempre que quisermos registrar alguns métodos. Uma vez implementados, decoradores se tornam blocos de construção poderosos.

O decorador recebe três parâmetros:

* `target` mapeia para a instância da classe.
* `name` contém o nome do método que está sendo decorado.
* `descriptor` é a peça mais interessante, pois nos permite anotar o método e manipular seu comportamento. Poderia ser assim:

```javascript
const descriptor = {
  value: () => {...},
  enumerable: false,
  configurable: true,
  writable: true
};
```

Como você viu acima, `value` nos permite moldar o comportamento. O resto, permite que você modifique o comportamento no nível do método. Por exemplo, um decorador `@readonly` poderia limitar o acesso. `@memoize` é outro exemplo interessante, pois isso permite que você implemente armazenamento em cache para métodos de modo mais simples.

## Implementando `@connect`

`@connect` envolverá nosso componente em outro componente. Isso, por sua vez, tratará a lógica de conexão (`listen/unlisten/setState`). Ele manterá o estado da **store** internamente e depois passará para o componente filho que estamos envolvendo. Durante esse processo, ele passará o estado através de **props**. A implementação abaixo ilustra a idéia:

**app/decorators/connect.js**

```javascript
import React from 'react';

const connect = (Component, store) => {
  return class Connect extends React.Component {
    constructor(props) {
      super(props);

      this.storeChanged = this.storeChanged.bind(this);
      this.state = store.getState();

      store.listen(this.storeChanged);
    }
    componentWillUnmount() {
      store.unlisten(this.storeChanged);
    }
    storeChanged() {
      this.setState(store.getState());
    }
    render() {
      return <Component {...this.props} {...this.state} />;
    }
  };
};

export default (store) => {
  return (target) => connect(target, store);
};
```

Você consegue ver a idéia de envolver um componente? Nosso decorador segue o estado da **store**. Depois disso, ele passa o estado para o componente envolvido através de **props**.

T> `...` é conhecido como [operador spread](https://github.com/sebmarkbage/ecmascript-rest-spread). Ele expande o objeto dado, separando-os em chave-valor, ou **props**, como neste caso.

Você pode conectar o decorador com `App` desse modo:

**app/components/App.jsx**

```javascript
...
import connect from '../decorators/connect';

...

@connect(NoteStore)
export default class App extends React.Component {
  render() {
    const notes = this.props.notes;

    ...
  }
  ...
}
```

Extraindo a lógica para um decorador nos permite manter nossos componentes simples. Se quisermos adicionar mais **store** ao sistema e conectá-las a componentes, seria trivial dessa maneira. Ainda melhor, poderíamos conectar várias lojas a um único componente facilmente.

## Idéias de decoradores

Podemos construir novos decoradores para várias funcionalidades, como **undo**, dessa maneira, eles nos permitem manter nossos componentes arrumados e facilitam a extração de lógica comum para outro lugar. Decoradores bem projetados podem ser usados em vários projetos.

### `@connectToStores` do Alt

Alt fornece um decorador semelhante conhecido como `@connectToStores`. Ele é baseado em métodos estáticos. Em vez de métodos normais que são vinculados a uma instância específica, estes são vinculados no nível da `class`. Isso significa que você pode chamá-los através da própria classe (ex., `App.getStores()`). O exemplo abaixo mostra como podemos integrar `@connectToStores` na nossa aplicação.

```javascript
...
import connectToStores from 'alt-utils/lib/connectToStores';

@connectToStores
export default class App extends React.Component {
  static getStores(props) {
    return [NoteStore];
  };
  static getPropsFromStores(props) {
    return NoteStore.getState();
  };
  ...
}
```

Esta abordagem mais detalhada é aproximadamente equivalente à nossa implementação. Na verdade, ele faz coisas a mais, pois permite que você se conecte várias **stores** ao mesmo tempo. Ele também fornece mais controle sobre a forma como você pode moldar o estado da **store** para **props**.

## Conclusão

Embora ainda um pouco experimental, os decoradores oferecem bons meios para extraír a lógica de métodos e classes. Melhor ainda, eles nos proporcionam um grau de reutilização, mantendo nossos componentes arrumados e organizados.
