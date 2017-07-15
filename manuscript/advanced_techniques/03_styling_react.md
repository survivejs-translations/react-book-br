# Estilos em React

Tradicionalmente, as páginas web foram divididas em marcação (HTML), estilos (CSS) e lógica (JavaScript). Graças ao React, e abordagens semelhantes, começamos a questionar essa divisão. Ainda podemos separar nossos interesses de alguma forma. Mas essa separação pode ser em eixos diferentes.

Essa mudança de mentalidade levou a novas maneiras de pensar sobre o CSS. Com o React, ainda estamos descobrindo as melhores práticas. Contudo, alguns padrões iniciais começaram a surgir. Como resultado, é difícil fornecer recomendações definitivas no momento. Em vez disso, vamos ver várias abordagens para que você possa tomar uma decisão com base em suas necessidades.

## Modo antigo de estilos

A abordagem da "velha guarda" para o estilo é para polvilhar alguns *ids* e *classes*, configurar regras CSS e esperar o melhor. No CSS, tudo é global por padrão. Definições de aninhamento (por ex., `.main .sidebar .button`.), cria uma lógica implícita para o seu estilo. Ambos os recursos levam a muita complexidade à medida que seu projeto cresce. Essa abordagem pode ser aceitável no início, mas, à medida que você desenvolve, você provavelmente quer melhorar ela.

## Metodologias CSS

O que acontece quando o aplicativo começa a se expandir e novos conceitos são adicionados? Os seletores CSS são amplos e globais. O problema fica ainda pior se você tiver que lidar com a ordem de carregamento. Se os seletores acabarem em conflitos, a última declaração ganha, a menos que haja `!important` em algum lugar. Tudo se torna complexo muito rápido.

Podemos combater este problema, tornando os seletores mais específicos, usando algumas regras de nomeação, e assim por diante. Isso acaba atrasando o inevitável. À medida que a comunidade CSS foi lutando com esses problemas por muito tempo, várias metodologias surgiram.

Em particular, [OOCSS](http://oocss.org/) (CSS Orientado a Objetos), [SMACSS](https://smacss.com/) (Abordagem Escalável e Modular para CSS) e [BEM](https://en.bem.info/method/) (Bloco, Elemento e Modificador) são bem conhecidos. Cada um deles resolvem problemas de CSS "puro", à sua maneira.

### BEM

BEM foi criado no Yandex. O objetivo do BEM é permitir componentes reutilizáveis e compartilhamento de código. Sites, como [Get BEM](http://getbem.com/) ajudam você a entender a metodologia com mais detalhes.

Manter nomes de classes longos, que o BEM requer, pode ser tedioso. Sendo assim, várias bibliotecas apareceram para tornar isso mais fácil. Para React, alguns exemplos disso são [react-bem-helper](https://www.npmjs.com/package/react-bem-helper), [react-bem-render](https://www.npmjs.com/package/reagir-bem-render) e [bem-react](https://www.npmjs.com/package/bem-react).

Vale notar que o [postcss-bem-linter](https://www.npmjs.com/package/postcss-bem-linter) adiciona regras de lint para o seu CSS, validando suas declarações BEM.

### OOCSS e SMACSS

Assim como o BEM, o OOCSS e o SMACSS trazem suas próprias convenções e metodologias. Até o momento que escrevo isso, não existem bibliotecas auxiliares específicas em React para OOCSS e SMACSS.

### Prós e contras

O principal benefício de adotar uma metodologia é trazer estrutura ao seu projeto. Ao invés de escrever regras a mão e esperar que tudo funcione, você terá algo mais estável para te ajudar. As metodologias superam alguns dos problemas básicos e o ajudam a desenvolver bons softwares a longo prazo. As convenções que eles trazem para um projeto ajudam com a manutenção e são menos propensas a criar uma bagunça.

A desvantagem, uma vez que você adote um, você estará preso com ele e será difícil migrar. Mas se você está disposto a se cometer, há bons benefícios dessa relação.

As metodologias também trazem suas peculiaridades (por exemplo, esquemas de nomeação complexos). Isso pode tornar certas coisas mais complicadas do que devem ser. Eles não resolvem necessariamente nenhum dos maiores problemas do CSS. Apenas oferecem uma alternativa, evitando-os a curto prazo.

Existem várias abordagens que vão além e conseguem resolver algum desses problemas fundamentais, dito isso, não é uma situação de escolha entre um ou outro, você pode adotar uma metodologia mesmo se você estiver usando um processador CSS.

## Processadores CSS

![Processadores CSS](../images/css.png)

CSS Puro sempre falta alguma funcionalidade que facilita a manutenção. Considere algo básico como variáveis, aninhamento, mixins, matemática ou funções de cores. Também seria bom poder esquecer os prefixos específicos do navegador. Estas são pequenas coisas que se somam bastante rápido e tornam irritante escrever CSS Puro.

Às vezes, você pode ver termos *pré-processador* ou *pós-processador*. [Stefan Baumgartner](https://medium.com/@ddprrt/deconfusing-pre-and-post-processing-d68e3bd078a3) chama essas ferramentas de *Processadores CSS*. A imagem acima, adaptada com base no trabalho de Stefan, chega ao ponto que queremos. A ferramenta funciona tanto no nível de autoria quanto na otimização. Por autoria, queremos dizer recursos que facilitam a escrita do CSS. Os recursos de otimização operam com base no CSS escrito convertendo-o em algo mais ideal para os navegadores.

O interessante é que você realmente pode usar vários processadores CSS. A imagem de Stefan ilustra como você pode escrever seu código usando SASS e ainda se beneficiar do processamento feito através do PostCSS. Por exemplo, ele pode *autoprefixar* seu código CSS para que você não precise se preocupar mais com o prefixo por navegador.

Você pode usar processadores comuns, como [Less](http://lesscss.org/), [Sass](http://sass-lang.com/), [Stylus](https://learnboost.github.io/stylus/), ou [PostCSS](http://postcss.org/) com React.

[cssnext](https://cssnext.github.io/) é um plugin PostCSS que nos permite experimentar o futuro agora. Existem algumas restrições, mas pode valer a pena. A vantagem do PostCSS e do cssnext é que você, literalmente, estará codificando CSS do futuro. À medida que os navegadores melhorem e adotem os padrões, você não precisa se preocupar com a portabilidade.

### Prós e contras

Comparado ao CSS puro, os processadores trazem muitas facilidades. Eles lidam com certos aborrecimentos (por exemplo, *autoprefixing*) enquanto melhoram sua produtividade. O PostCSS é mais flexível, por definição, e permite que você use apenas os recursos desejados. Os processadores, como Less ou Sass, tem sua própria gama de ferramentas. Essas abordagens podem ser usadas em conjunto, no entanto, para que você possa, por exemplo, escrever seus estilos em Sass e, em seguida, aplicar alguns plugins PostCSS, como você desejar.

Em nosso projeto, nós poderíamos nos beneficiar do cssnext, mesmo não fazendo nenhuma alteração em nosso CSS. Graças ao *autoprefixing*, os cantos arredondados de nossas pistas ficariam bem mesmo em navegadores legados. Além disso, podemos parametrizar o estilo graças às variáveis.

## Abordagens baseadas em React

Com o React, temos algumas alternativas. E se a maneira como pensamos sobre o estilo estiver defasada? O CSS é poderoso, mas pode se tornar uma bagunça incessante sem alguma disciplina. Onde desenhamos a linha entre CSS e JavaScript?

Existem várias abordagens para React que nos permitem empurrar estilo para o nível de componente. Pode parecer maluco, mas React, e a mentalidade de componentização, pode liderar o caminho aqui.

### Estilos inline ao salvamento!

Ironicamente, a forma como as soluções com base em React resolvem isso é através de estilos *inline*. Em primeiro lugar, o não uso de estilos inline foi um dos principais motivos para o uso de arquivos CSS separados. Agora estamos fazendo o caminho de volta. Isso significa que em vez de algo assim:

```javascript
render(props, context) {
  const notes = this.props.notes;

  return <ul className='notes'>{notes.map(this.renderNote)}</ul>;
}
```

seria acompanhado de CSS, ficando algo assim:

```javascript
render(props, context) {
  const notes = this.props.notes;
  const style = {
    margin: '0.5em',
    paddingLeft: 0,
    listStyle: 'none'
  };

  return <ul style={style}>{notes.map(this.renderNote)}</ul>;
}
```

Assim como os atributos HTML, estamos usando a mesma convenção camelCase para propriedades CSS.

Com estilos no nível do componente, podemos implementar a lógica que irá alterar os estilos com mais facilidade. Uma maneira clássica de fazer isso é alterar os nomes das classes com base nas mudanças que queremos. Agora, podemos ajustar as propriedades que queremos diretamente.

No entanto, perdemos algo no processo. Agora, todo o nosso estilo está vinculado ao nosso código JavaScript. Vai ser difícil realizar grandes mudanças na nossa base de códigos, pois precisamos ajustar muitos componentes para conseguir isso.

Nós podemos tentar contornar isso, injetando uma parte do estilo através de *props*. Um componente poderia alterar seu estilo com base em uma prop fornecida. Isso pode ser aprimorado ainda mais, com convenções que permitem que pedaços de configuração de estilos sejam mapeadas para alguma parte específica da sua aplicação. Acabamos de reinventar os seletores CSS, em pequena escala.

E como fica *media queries*? Essa abordagem, ingênua, não disponibiliza suporte. Felizmente, algumas pessoas criaram bibliotecas específicas para resolver esses problemas difíceis para nós.

De acordo com Michele Bertoli, as características básicas dessas bibliotecas são:

* Autoprefixing - ex., para `border`, `animation`, `flex`.
* Pseudo classes - ex., `:hover`, `:active`.
* Media queries - ex., `@media (max-width: 200px)`.
* Estilos como Objeto literal - Veja os exemplos acima.
* Extração de CSS Inline - É útil extrair os estilos para arquivos CSS separados, pois ajuda com o carregamento inicial da página. Isso evitará um flash de conteúdo não estilizado (*FOUC - Flash of unstyled content*).

Iremos cobrir algumas das bibliotecas disponíveis, para lhe dar uma melhor idéia de como elas funcionam. Veja a [lista completa do Michele](https://github.com/MicheleBertoli/css-in-js) para saber mais sobre isso.

### Radium

[Radium](http://projects.formidablelabs.com/radium/) traz certas idéias valiosas que merecem destaque. O mais importante, fornece abstrações necessárias para lidar com *media queries* e *pseudo-classes* (ex. `:hover`). Ele expande a sintaxe básica da seguinte maneira:

```javascript
const styles = {
  button: {
    padding: '1em',

    ':hover': {
      border: '1px solid black'
    },

    '@media (max-width: 200px)': {
      width: '100%',

      ':hover': {
        background: 'white',
      }
    }
  },
  primary: {
    background: 'green'
  },
  warning: {
    background: 'yellow'
  },
};

...

<button style={[styles.button, styles.primary]}>Confirm</button>
```

Para a propriedade `style` funcionar, você precisar anotar sua classe usando o *decorator* `@Radium`.

### React Style

[React Style](https://github.com/js-next/react-style) usa a mesma sintaxe do [Reative Native StyleSheet](https://facebook.github.io/react-native/docs/stylesheet.html#content). Ele expande a definição básica através de chaves adicionais para criar fragmentos.

```javascript
import StyleSheet from 'react-style';

const styles = StyleSheet.create({
  primary: {
    background: 'green'
  },
  warning: {
    background: 'yellow'
  },
  button: {
    padding: '1em'
  },
  // media queries
  '@media (max-width: 200px)': {
    button: {
      width: '100%'
    }
  }
});

...

<button styles={[styles.button, styles.primary]}>Confirm</button>
```

Como você pode ver, podemos usar fragmentos individuais para obter o mesmo efeito que os modificadores do Radium. *Media queries* também são suportadas. React Style espera que você manipule os estados do navegador (ex. `:hover`) através do JavaScript. Animações CSS não funcionarão. Ao invés disso, é preferível usar outra solução para isso.

T> [Plugin React Style para Webpack](https://github.com/js-next/react-style-webpack-plugin) pode extrair declarações CSS em um arquivo separado. Agora estamos mais perto do mundo ao qual estamos acostumados, mas sem o efeito cascata. Também temos nossas declarações de estilo ao nível do componente.

### JSS

[JSS](https://github.com/jsstyles/jss) é um compilador JSON para StyleSheets. Pode ser conveniente representar seus estilos usando estruturas JSON, pois isso nos facilita o uso de nomes. Além disso, é possível realizar transformações sobre o JSON para obter outros recursos, como autoprefixing. O JSS fornece uma interface de plugin apenas para isso.

JSS pode ser usado com React através do [react-jss](https://www.npmjs.com/package/react-jss). Podemos usar JSS com *react-jss* dessa maneira:

```javascript
...
import classnames from 'classnames';
import useSheet from 'react-jss';

const styles = {
  button: {
    padding: '1em'
  },
  'media (max-width: 200px)': {
    button: {
      width: '100%'
    }
  },
  primary: {
    background: 'green'
  },
  warning: {
    background: 'yellow'
  }
};

@useSheet(styles)
export default class ConfirmButton extends React.Component {
  render() {
    const {classes} = this.props.sheet;

    return <button
      className={classnames(classes.button, classes.primary)}>
        Confirm
      </button>;
  }
}
```

A abordagem suporta *pseudo-selectors*, ou seja, você pode definir um seletor implícito, como `&:hover`, dentro de uma definição, e ela irá funcionar.

T> Existe um Webpack loader para isso, o [jss-loader](https://www.npmjs.com/package/jss-loader).

### React Inline

[React Inline](https://github.com/martinandert/react-inline) é uma abordagem interessante para StyleSheet. Ele gera CSS com base na propriedade `className` de elementos onde ele é usado. O exemplo acima pode ser adaptado para React Inline dessa maneira:

```javascript
import cx from 'classnames';
...

class ConfirmButton extends React.Component {
  render() {
    const {className} = this.props;
    const classes = cx(styles.button, styles.primary, className);

    return <button className={classes}>Confirm</button>;
  }
}
```

Diferente do React Style, essa abordagem oferece suporte a diferentes estados (por exemplo, `:hover`). Infelizmente, ele depende de suas próprias ferramentas de tooling para gerar código React e CSS.

### jsxstyle

A biblioteca do Pete Hunt, [jsxstyle](https://github.com/petehunt/jsxstyle), visa mitigar alguns problemas da abordagem do React Style. Como você viu em exemplos anteriores, ainda temos definições de estilo separadas da marcação de componente. **jsxstyle** combina esses dois conceitos. Considere o seguinte exemplo:

```javascript
// PrimaryButton component
<button
  padding='1em'
  background='green'
>Confirm</button>
```

Essa abordagem ainda está em seus primeiros dias. Por exemplo, o suporte para **media queries** está faltando. Em vez de definir modificadores como no exemplo acima, você acabará definindo mais componentes para seus casos de uso.

T> Assim como React Style, jsxstyle tem um Webpack loader que pode extrair o CSS em um arquivo separado.

## CSS Modules

Como se não houvesse opções de estilo suficientes para React, há mais um que vale a pena mencionar. [CSS Modules](https://github.com/css-modules/css-modules) parte da premissa de que as regras CSS devem ser locais por padrão. Declarações globais devem ser tratadas como um caso especial. O post do Mark Dalgleish, que você pode conferir em Português, [O fim do CSS global](https://medium.com/tableless/o-fim-do-css-global-ddcd80bd6334), fala mais detalhadamente sobre isso.

Resumindo, se você dificultar o uso de declarações globais, você consegue resolver o maior problema do CSS. A abordagem ainda nos permite desenvolver CSS do modo que estamos acostumados. Dessa vez, estamos usando um contexto local, mais seguro por padrão.

Isso já resolve uma grande quantidade de problemas que as bibliotecas acima tentam resolver em seus próprios modos. Se precisarmos de estilos globais, ainda podemos usá-los. Afinal, para alguns estilos mais genéricos, nós ainda queremos declarações globais. Mas nessa abordagem, estamos sendo explícitos sobre isso.

Para dar uma ideia melhor, considere o exemplo abaixo:

**style.css**

```css
.primary {
  background: 'green';
}

.warning {
  background: 'yellow';
}

.button {
  padding: 1em;
}

.primaryButton {
  composes: primary button;
}

@media (max-width: 200px) {
  .primaryButton {
    composes: primary button;

    width: 100%;
  }
}
```

**button.jsx**

```javascript
import styles from './style.css';

...

<button className=`${styles.primaryButton}`>Confirm</button>
```

Como você pode ver, essa abordagem fornece um equilíbrio entre o que as pessoas estão familiarizadas e o que as bibliotecas específicas em React, fazem. Não me surpreenderia muito se essa abordagem ganhasse popularidade apesar de ainda estar em seus primeiros dias. Veja o [demo de CSS Modules com Webpack](https://css-modules.github.io/webpack-demo/) para mais detalhes.

T> Você pode usar outros processadores, como SASS, antes do CSS Modules, caso você queira mais funcionalidades.

T> [gajus/react-css-modules](https://github.com/gajus/react-css-modules) deixa ainda mais conveniente usar CSS Modules com React. Usando essa biblioteca, você não precisa mais se referir ao objeto `styles`, e você não é obrigado a usar camelCase para nome de classes.

T> Glen Maddern discute o tema em maior detalhe em seu artigo chamado [CSS Modules - Welcome to the Future](http://glenmaddern.com/articles/css-modules).

## Conclusão

É simples experimentar várias abordagens de estilos em React. Você pode fazer tudo, variando de CSS puro para configurações mais complexas. Várias ferramentas de tooling específicas para React trazem *loaders* específicos para isso. Facilitando a experimentação de diferentes alternativas.

A abordagen base de estilos em React nos permitem mover nossos estilos para o nível do componente. Isso proporciona um contraste interessante com as abordagens convencionais onde o CSS é mantido separado. Lidar com a lógica específica do componente torna-se mais fácil. Você perderá algum poder fornecido pelo CSS. Mas em troca, você ganha algo que é mais simples de entender e também, mais difícil de quebrar.

CSS Modules encontra um equilíbrio entre uma abordagem convencional e abordagens específicas do React. Embora seja um recém-chegado, trás muita promessa. O maior benefício, parece que não se perde muito do processo já comumente usado. É um bom passo adiante.

Ainda não existem boas práticas, e ainda estamos descobrindo as melhores maneiras de fazer isso em React. Você provavelmente terá que fazer algumas experiências próprias para descobrir quais maneiras se encaixam melhor em seu caso de uso.
