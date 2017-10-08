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

React [refs](https://facebook.github.io/react/docs/more-about-refs.html) permitem que você acesse facilmente o elemento DOM. Seu código será exclusivo para web, mas às vezes, não há como contornar isso, por exemplo, se você estiver medindo seus componentes.

Refs precisa de uma instância. Isso significa que eles só funcionarão com `React.createClass` ou definições baseadas em classes. A idéia básica é a seguinte:

```javascript
<input type="text" ref="input" />

...

// Access somewhere
this.refs.input
```

Além de *strings*, refs suportam uma função de *callback* que é chamada logo após o componente ser montado. Você pode fazer alguma inicialização aqui ou capturar a referência:

```javascript
<input type="text" ref={element => element.focus()} />
```

## Propriedades e métodos personalizados

Além dos refs e métodos de ciclo de vida, há uma variedade de [propriedades e métodos](https://facebook.github.io/react/docs/component-specs.html) você deve conhecer, especialmente se você vai usar `React.createClass`:

* `displayName` - Ao configurar `displayName`, irá melhorar o nome dos seus componentes para debug. Para as classes ES6, isso é criado automaticamente com base no nome da classe. Você pode adicionar `displayName` para um componente baseado em função anônima também.
* `getInitialState()` - Em componentes baseados em classe, você pode alcançar o mesmo resultao dentro do `constructor`.
* `getDefaultProps()` - Em componentes baseados em classe, você pode alcançar o mesmo resultao dentro do `constructor`.
* `render()` - Esse é o método principal do React. Ele [deve retornar um único nó](https://facebook.github.io/react/tips/maximum-number-of-jsx-root-nodes.html), se você retornar múltiplos nós, ele não irá funcionar!
* `mixins` - `mixins` contém uma série de mixins para se aplicar ao componente.
* `statics` - `statics` contém propriedades estáticas e métodos para um componente. No ES6 você pode atribuí-los à classe conforme abaixo:

```javascript
class Note {
  render() {
    ...
  }
}
Note.willTransitionTo = () => {...};

export default Note;
```

Isso também pode ser escrito como:

```javascript
class Note {
  static willTransitionTo() {...}
  render() {
    ...
  }
}

export default Note;
```

Algumas bibliotecas, como React DnD, dependem de métodos estáticos para fornecer ações de transição. Eles permitem que você controle o que acontece quando um componente é mostrado ou oculto. Por definição, o métoo estático está disponível através da própria classe.

Os componentes React permitem documentar a interface do seu componente usando `propTypes`.

```javascript
const Note = ({task}) => <div>{task}</div>;
Note.propTypes = {
  task: React.PropTypes.string.isRequired
}
```

Para entender `propTypes` melhor, leia o capítulo *Tipagem em React*.

## React Component Conventions

I prefer to have the `constructor` first, followed by lifecycle methods, `render()`, and finally, methods used by `render()`. This top-down approach makes it straightforward to follow code. There is also an inverse convention that leaves `render()` as the last method. Naming conventions vary as well. You will have to find conventions which work the best for you.

You can enforce a convention by using a linter such as [ESLint](http://eslint.org/). Using a linter decreases the amount of friction when working on code written by others. Even on personal projects, using tools to verify syntax and standards for you can be useful. It lessens the amount and severity of mistakes and allows you to spot them early.

By setting up a continuous integration system you can test against multiple platforms and catch possible regressions early. This is particularly important if you are using lenient version ranges. Sometimes dependencies might have problems and it's good to catch those.

## Conclusion

Even though React's component definition is fairly simple, it's also powerful and pragmatic. Especially the advanced parts can take a while to master, but it's good to know they are there.

We'll continue the implementation in the next chapter as we allow the user to edit individual notes.
