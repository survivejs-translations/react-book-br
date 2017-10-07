# Entendo componentes em React

Como vimos até agora, os componentes em React são bastante simples. Eles podem ter guardar um `state` interno. Eles também podem aceitar `props`. Além disso, React fornece meios que permitem manipular casos avançados. Includingo métodos de ciclo de vida e `refs`. Há também um conjunto de propriedades personalizadas e métodos que você precisa saber.

## Métodos de ciclo de vida

![Lifecycle methods](../images/lifecycle.png)

A partir da imagem acima, podemos ver que um componente React possui três fases durante o ciclo de vida. Pode ser **mounting**, **mounted**, e **unmounting**. Cada uma dessas fases vem com métodos relacionados.

Durante a fase de **mounting**, você tem acesso ao seguinte:

* `componentWillMount()` é desencadeado uma vez antes de qualquer renderização. Uma maneira de usá-lo seria carregar dados de forma assíncrona e forçar a renderização através de `setState`. `render()` verá o estado atualizado e será executado apenas uma vez, apesar da mudança de estado. Esse método também é ativado realizando renderização no servidor.
* `componentDidMount()` é desencadeado após a renderização inicial. Você tem acesso ao DOM aqui. Você poderia usar esse método para usar um plugin jQuery dentro de um componente, por exemplo. Esse método **não** é ativado realizando renderização no servidor.

Depois que um componente foi montado e está funcionando, você pode operar através dos seguintes métodos na fase **mounted**:

* `componentWillReceiveProps(object nextProps)` dispara quando o componente recebe novas `props`. Você poderia, por exemplo, modificar o estado do componente com base nas `props` recebidas.
* `shouldComponentUpdate(object nextProps, object nextState)` permite otimizar a renderização. Se você verificar as `props` e ver que não há necessidade de atualização, você pode retornar `false`. Isso é onde [Immutable.js](https://facebook.github.io/immutable-js/) e bibliotecas similares são úteis graças às suas simples verificações de igualdade. [A documentação oficial](https://facebook.github.io/react/docs/optimizing-performance.html#shouldcomponentupdate-in-action) explica em mais detalhes.
* `componentWillUpdate(object nextProps, object nextState)` é disparada logo após `shouldComponentUpdate` e antes do `render()`. Não é possível usar `setState` aqui, mas você pode definir propriedades da classe atual.
* `componentDidUpdate(object nextProps, object nextState)` é disparada após a renderização. Você pode modificar o DOM aqui. Isso pode ser útil para adaptar outro código para trabalhar com o React.

Finalmente, quando um componente está desmontando, há mais um método que você pode usar na fase **unmounting**:

* `componentWillUnmount()` é disparada logo antes um componente ser desmontado do DOM. Este é o local ideal para executar a limpeza (por exemplo, remover *timers/timeouts* em execução, elementos do DOM personalizados e assim por diante).

Muitas vezes, `componentDidMount` e `componentWillUnmount` são usados em conjunto. Se você configurar algum evento DOM (ex: *window.addEventListener(..)*) no `componentDidMount`, você também precisa se lembrar de limpá-lo em `componentWillUnmount` (ex: *window.removeEventListener(..)*).

## Refs

React's [refs](https://facebook.github.io/react/docs/more-about-refs.html) allow you to access the underlying DOM structure easily. Using them will bind your code to the web, but sometimes there's no way around this if you are measuring components for instance.

Refs need a backing instance. This means they will work only with `React.createClass` or class based component definitions. The basic idea goes as follows:

```javascript
<input type="text" ref="input" />

...

// Access somewhere
this.refs.input
```

In addition to strings, refs support a callback that gets called right after the component is mounted. You can do some initialization here or capture the reference:

```javascript
<input type="text" ref={element => element.focus()} />
```

## Custom Properties and Methods

Beyond the lifecycle methods and refs, there are a variety of [properties and methods](https://facebook.github.io/react/docs/component-specs.html) you should be aware of especially if you are going to use `React.createClass`:

* `displayName` - It is preferable to set `displayName` as that will improve debug information. For ES6 classes this is derived automatically based on the class name. You can attach `displayName` to an anonymous function based component as well.
* `getInitialState()` - In class based approach the same can be achieved through `constructor`.
* `getDefaultProps()` - In classes you can set these in `constructor`.
* `render()` - This is the workhorse of React. It [must return a single node](https://facebook.github.io/react/tips/maximum-number-of-jsx-root-nodes.html) as returning multiple won't work!
* `mixins` - `mixins` contains an array of mixins to apply to components.
* `statics` - `statics` contains static properties and method for a component. In ES6 you can assign them to the class as below:

```javascript
class Note {
  render() {
    ...
  }
}
Note.willTransitionTo = () => {...};

export default Note;
```

This could also be written as:

```javascript
class Note {
  static willTransitionTo() {...}
  render() {
    ...
  }
}

export default Note;
```

Some libraries, such as React DnD, rely on static methods to provide transition hooks. They allow you to control what happens when a component is shown or hidden. By definition statics are available through the class itself.

React components allow you to document the interface of your component using `propTypes` as below.

```javascript
const Note = ({task}) => <div>{task}</div>;
Note.propTypes = {
  task: React.PropTypes.string.isRequired
}
```

To understand `propTypes` better, read the *Typing with React* chapter.

## React Component Conventions

I prefer to have the `constructor` first, followed by lifecycle methods, `render()`, and finally, methods used by `render()`. This top-down approach makes it straightforward to follow code. There is also an inverse convention that leaves `render()` as the last method. Naming conventions vary as well. You will have to find conventions which work the best for you.

You can enforce a convention by using a linter such as [ESLint](http://eslint.org/). Using a linter decreases the amount of friction when working on code written by others. Even on personal projects, using tools to verify syntax and standards for you can be useful. It lessens the amount and severity of mistakes and allows you to spot them early.

By setting up a continuous integration system you can test against multiple platforms and catch possible regressions early. This is particularly important if you are using lenient version ranges. Sometimes dependencies might have problems and it's good to catch those.

## Conclusion

Even though React's component definition is fairly simple, it's also powerful and pragmatic. Especially the advanced parts can take a while to master, but it's good to know they are there.

We'll continue the implementation in the next chapter as we allow the user to edit individual notes.
