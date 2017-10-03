# Introdução ao React

[React](https://facebook.github.io/react/), biblioteca do Facebook, mudou a forma como pensamos sobre aplicações web e desenvolvimento de interfaces de usuários. Devido ao seu design, você pode usá-lo muito além da web. Um recurso conhecido como **Virtual DOM** permite isso.

Neste capítulo, veremos algumas das idéias básicas por trás da biblioteca para que você entenda React um pouco melhor antes de seguir em frente.

## O qué é o React?

![React](images/react_header.png)

React é uma biblioteca JavaScript que obriga você a pensar em componentes. Esse modelo de pensamento se adapta bem às interfaces do usuário. Dependendo da sua experiência anterior, pode parecer estranho no início. Você terá que pensar com muito cuidado sobre o conceito de "estado" e aonde ele pertence.

**Administrar estado** de uma aplicação, é um problema difícil, uma variedade de soluções apareceram. Neste livro, começaremos a gerenciar o estado nós mesmo e, ao avançarmos, usaremos uma implementação do Flux, conhecida como Alt. Existem várias outras alternativas, implementações como Redux, MobX e Cerebral.

React é pragmático no sentido de que contém um conjunto de possibilidades. Se o modelo padrão do React não funcionar para você, é possível usar algo de mais baixo nível. Por exemplo, existem ganchos que podem ser usados para envolver uma lógica mais antiga que depende do DOM. Isso quebra a abstração e vincula seu código a um ambiente específico, mas às vezes isso é o ponto pragmático de fazer.

## Virtual DOM

![Virtual DOM](images/vdom.png)

Um dos problemas fundamentais da programação é como lidar com o estado da aplicação. Vamos supor que você esteja desenvolvendo uma interface de usuário e quer mostrar os mesmos dados em vários lugares. Como você se certifica de que os dados estão consistentes?

Historicamente, misturamos as declarações DOM e de estado e tentamos gerenciar tudo isso. React resolve esse problema de uma maneira diferente. Ele introduz o conceito do **Virtual DOM** na comunidade.

Virtual DOM existe em cima do DOM real, ou algum outro alvo de renderização. Ele resolve o problema de manipulação do estado de uma maneira diferente. Sempre que alguma mudança é feitas nele, ele descobre a melhor maneira de processar essas mudanças para refleti-las no DOM real. Ele é capaz de propagar mudanças em sua árvore virtual, como na imagem acima.

### Performance do Virtual DOM

Atualizar as manipulações do DOM dessa maneira pode levar a um aumento de desempenho. Manipular o DOM na mão tende a ser ineficiente e é difícil de otimizar. Ao deixar o problema da manipulação do DOM para uma boa abstração, você pode economizar muito tempo e esforço.

React permite que você ajuste o desempenho em ainda mais detalhes, implementando métodos de ciclos de vida para ajustar a maneira como a árvore virtual é atualizada. Embora este seja muitas vezes um passo opcional.

O maior custo do Virtual DOM é que sua implementação faz com que o React fique bastante grande. Você pode esperar que os tamanhos do pacote de pequenas aplicações sejam de cerca de 150-200 kB minificados, com React incluído. Gzip ajudará, mas ainda é grande.

T> Solução como [preact](https://developit.github.io/preact/) e [react-lite](https://github.com/Lucifier129/react-lite) permitem que você alcance tamanhos menores enquanto sacrifica alguma funcionalidade. Se você está preocupado com o tamanho do arquivo final, vale a pena considerar essas soluções.

T> Bibliotecas como [Matt-Esch/virtual-dom](https://github.com/Matt-Esch/virtual-dom) ou [paldepind/snabbdom](https://github.com/paldepind/snabbdom), são focadas inteiramente em Virtual DOM. Se você está interessado na teoria e quer entender isso, vale a pena dar uma olhada.

## React Renderers

As mentioned, React's approach decouples it from the web. You can use it to implement interfaces across multiple platforms. In this case we'll be using a renderer known as [react-dom](https://www.npmjs.com/package/react-dom). It supports both client and server side rendering.

### Universal Rendering

We could use react-dom to implement so called *universal* rendering. The idea is that the server renders the initial markup and passes the initial data to the client. This improves performance by avoiding unnecessary round trips as each request comes with an overhead. It is also useful for search engine optimization (SEO) purposes.

Even though the technique sounds simple, it can be difficult to implement for larger scale applications. But it's still something worth knowing about.

Sometimes using the server side part of react-dom is enough. You can use it to [generate invoices](https://github.com/bebraw/generate-invoice) for example. That's one way to use React in a flexible manner. Generating reports is a common need after all.

### Available React Renderers

Even though react-dom is the most used renderer, there are a few others you might want to be aware of. I've listed some of the well known alternatives below:

* [React Native](https://facebook.github.io/react-native/) - React Native is a framework and renderer for mobile platforms including iOS and Android. You can also run [React Native applications on the web](https://github.com/necolas/react-native-web).
* [react-blessed](https://github.com/Yomguithereal/react-blessed) - react-blessed allows you to write terminal applications using React. It's even possible to animate them.
* [gl-react](https://projectseptemberinc.gitbooks.io/gl-react/content/) - gl-react provides WebGL bindings for React. You can write shaders this way for example.
* [react-canvas](https://github.com/Flipboard/react-canvas) - react-canvas provides React bindings for the Canvas element.

## `React.createElement` and JSX

Given we are operating with virtual DOM, there's a [high level API](https://facebook.github.io/react/docs/top-level-api.html) for handling it. A naïve React component written using the JavaScript API could look like this:

```javascript
const Names = () => {
  const names = ['John', 'Jill', 'Jack'];

  return React.createElement(
    'div',
    null,
    React.createElement('h2', null, 'Names'),
    React.createElement(
      'ul',
      { className: 'names' },
      names.map(name => {
        return React.createElement(
          'li',
          { className: 'name' },
          name
        );
      })
    )
  );
};
```

As it is verbose to write components this way and the code is quite hard to read, often people prefer to use a language known as [JSX](https://facebook.github.io/jsx/) instead. Consider the same component written using JSX below:

```javascript
const Names = () => {
  const names = ['John', 'Jill', 'Jack'];

  return (
    <div>
      <h2>Names</h2>

      {/* This is a list of names */}
      <ul className="names">{
        names.map(name =>
          <li className="name">{name}</li>
        )
      }</ul>
    </div>
  );
};
```

Now we can see the component renders a set of names within a HTML list. It might not be the most useful component, but it's enough to illustrate the basic idea of JSX. It provides us a syntax that resembles HTML. It also provides a way to write JavaScript within it by using braces (`{}`).

Compared to vanilla HTML, we are using `className` instead of `class`. This is because the API has been modeled after the DOM naming. It takes some getting used to and you might experience a [JSX shock](https://medium.com/@housecor/react-s-jsx-the-other-side-of-the-coin-2ace7ab62b98) until you begin to appreciate the approach. It gives us an additional level of validation.

T> [HyperScript](https://github.com/dominictarr/hyperscript) is an interesting alternative to JSX. It provides a JavaScript based API and as such is closer to the metal. You can use the syntax with React through [hyperscript-helpers](https://www.npmjs.com/package/hyperscript-helpers).

T> There is a semantic difference between React components and React elements. In the example each of those JSX nodes would be converted into an element. In short, components can have state whereas elements are simpler by nature. They are just pure objects. Dan Abramov goes into further detail in a [blog post](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html) of his.

## Conclusion

Now that we have a rough understanding of what React is, we can move onto something more technical. It's time to get a small project up and running.
