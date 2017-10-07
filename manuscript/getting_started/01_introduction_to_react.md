# Introdução ao React

[React](https://facebook.github.io/react/), biblioteca do Facebook, mudou a forma como pensamos sobre aplicações web e desenvolvimento de interfaces de usuários. Devido ao seu design, você pode usá-lo muito além da web. Um recurso conhecido como **Virtual DOM** permite isso.

Neste capítulo, veremos algumas das idéias básicas por trás da biblioteca para que você entenda React um pouco melhor antes de seguir em frente.

## O qué é o React?

![React](../images/react_header.png)

React é uma biblioteca JavaScript que obriga você a pensar em componentes. Esse modelo de pensamento se adapta bem às interfaces do usuário. Dependendo da sua experiência anterior, pode parecer estranho no início. Você terá que pensar com muito cuidado sobre o conceito de "estado" e aonde ele pertence.

**Administrar estado** de uma aplicação, é um problema difícil, uma variedade de soluções apareceram. Neste livro, começaremos a gerenciar o estado nós mesmo e, ao avançarmos, usaremos uma implementação do Flux, conhecida como Alt. Existem várias outras alternativas, implementações como Redux, MobX e Cerebral.

React é pragmático no sentido de que contém um conjunto de possibilidades. Se o modelo padrão do React não funcionar para você, é possível usar algo de mais baixo nível. Por exemplo, existem ganchos que podem ser usados para envolver uma lógica mais antiga que depende do DOM. Isso quebra a abstração e vincula seu código a um ambiente específico, mas às vezes isso é o ponto pragmático de fazer.

## Virtual DOM

![Virtual DOM](../images/vdom.png)

Um dos problemas fundamentais da programação é como lidar com o estado da aplicação. Vamos supor que você esteja desenvolvendo uma interface de usuário e quer mostrar os mesmos dados em vários lugares. Como você se certifica de que os dados estão consistentes?

Historicamente, misturamos as declarações DOM e de estado e tentamos gerenciar tudo isso. React resolve esse problema de uma maneira diferente. Ele introduz o conceito do **Virtual DOM** na comunidade.

Virtual DOM existe em cima do DOM real, ou algum outro alvo de renderização. Ele resolve o problema de manipulação do estado de uma maneira diferente. Sempre que alguma mudança é feitas nele, ele descobre a melhor maneira de processar essas mudanças para refleti-las no DOM real. Ele é capaz de propagar mudanças em sua árvore virtual, como na imagem acima.

### Performance do Virtual DOM

Atualizar as manipulações do DOM dessa maneira pode levar a um aumento de desempenho. Manipular o DOM na mão tende a ser ineficiente e é difícil de otimizar. Ao deixar o problema da manipulação do DOM para uma boa abstração, você pode economizar muito tempo e esforço.

React permite que você ajuste o desempenho em ainda mais detalhes, implementando métodos de ciclos de vida para ajustar a maneira como a árvore virtual é atualizada. Embora este seja muitas vezes um passo opcional.

O maior custo do Virtual DOM é que sua implementação faz com que o React fique bastante grande. Você pode esperar que os tamanhos do pacote de pequenas aplicações sejam de cerca de 150-200 kB minificados, com React incluído. Gzip ajudará, mas ainda é grande.

T> Solução como [preact](https://developit.github.io/preact/) e [react-lite](https://github.com/Lucifier129/react-lite) permitem que você alcance tamanhos menores enquanto sacrifica alguma funcionalidade. Se você está preocupado com o tamanho do arquivo final, vale a pena considerar essas soluções.

T> Bibliotecas como [Matt-Esch/virtual-dom](https://github.com/Matt-Esch/virtual-dom) ou [paldepind/snabbdom](https://github.com/paldepind/snabbdom), são focadas inteiramente em Virtual DOM. Se você está interessado na teoria e quer entender isso, vale a pena dar uma olhada.

## Renderizadores do React

Como já foi mencionado, o React tem uma abordagem desacoplada da web. Você pode renderizar sua interface em diversas plataformas. No nosso caso, nós iremos utilizar o renderizador [react-dom](https://www.npmjs.com/package/react-dom). Ele suporta a renderização do lado do cliente e do servidor.

### Renderização Universal

Poderíamos usar o react-dom para implementar o chamado *renderização universal*. A idéia é que o servidor irá processar a marcação inicial e repassar os dados iniciais para o cliente. Isso melhora o desempenho, evitando chamadas de ida e volta desnecessárias, conforme cada pedido, vem uma sobrecarga. Também é útil para otimização de mecanismos de pesquisa (SEO).

Embora a técnica pareça simples, pode ser difícil de implementar para aplicações de grande escala. Mas ainda é algo que vale a pena conhecer.

Às vezes, utilizar a interface de renderização no servidor do react-dom, é suficiente. Você pode usar como exemplo o [gerardor de faturas](https://github.com/bebraw/generate-invoice). Essa é uma maneira de usar React de uma forma flexível. Gerar faturas é uma necessidade comum, afinal.

### Renderizadores disponíveis para React

Mesmo que o react-dom seja o renderizador mais utilizado, há alguns outros que você pode querer conhecer. Vou listar algumas das alternativas bem conhecidas abaixo:

* [React Native](https://facebook.github.io/react-native/) - React Native é um framework e renderizador para plataformas móveis, incluindo iOS e Android. Você também pode [rodar aplicações React Native na web](https://github.com/necolas/react-native-web).
* [react-blessed](https://github.com/Yomguithereal/react-blessed) - react-blessed permite que você crie aplicações para terminais utilizando React. É possível até, criar animações.
* [gl-react](https://projectseptemberinc.gitbooks.io/gl-react/content/) - gl-react disponibiliza métodos WebGL para React. Você pode criar *shaders*, por exemplo.
* [react-canvas](https://github.com/Flipboard/react-canvas) - react-canvas disponibiliza métodos para o elemento Canvas.

## `React.createElement` e JSX

Dado que estamos operando com Virtual DOM, existe uma [API](https://facebook.github.io/react/docs/top-level-api.html) para trabalhar com ele. Um componente React ingênuo, pode ser escrito usando a API JavaScript:

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

Como é verboso escrever componentes dessa maneira e o código é bastante difícil de ler, muitas vezes as pessoas preferem usar um idioma conhecido como [JSX](https://facebook.github.io/jsx/). Considere o mesmo componente escrito usando JSX:

```javascript
const Names = () => {
  const names = ['John', 'Jill', 'Jack'];

  return (
    <div>
      <h2>Names</h2>

      {/* Uma lista de nomes */}
      <ul className="names">{
        names.map(name =>
          <li className="name">{name}</li>
        )
      }</ul>
    </div>
  );
};
```

Agora, podemos ver o componente renderizar um conjunto de nomes dentro de uma lista HTML. Pode não ser o componente mais útil, mas é suficiente para ilustrar a idéia básica do JSX Ele nos fornece uma sintaxe que se lembra o HTML. Ele também fornece uma maneira de escrever JavaScript dentro dele usando chaves (`{}`).

Comparado com o HTML, estamos usando `className` ao invés de `class`. Isso ocorre porque a API foi modelada seguindo a nomeação DOM. Demora um pouco de acostumar e você pode experimentar algumas [surpresas ao usar JSX](https://medium.com/@housecor/react-s-jsx-the-other-side-of-the-coin-2ace7ab62b98), até começar a apreciar essa abordagem. Isso nos dá um nível adicional de validação.

T> [HyperScript](https://github.com/dominictarr/hyperscript) é uma interessante alternativa ao JSX. Ele fornece uma API baseada em JavaScript e, como tal, está mais perto da API do React. Você pode ver a sintaxe com React em [hyperscript-helpers](https://www.npmjs.com/package/hyperscript-helpers).

T> Existe uma diferença semântica entre os componentes React e Elementos React. No exemplo, cada um desses nós JSX seria convertido em um elemento. Em suma, os componentes podem ter estado, enquanto os elementos são mais simples por natureza. Eles são apenas objetos puros. Dan Abramov entra em detalhes nesse [artigo](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html).

## Conclusão

Agora que temos uma compreensão melhor do que é o React, podemos avançar para algo mais técnico. É hora de colocar um pequeno projeto em funcionamento.
