# Implementando persistência com `localStorage`

Atualmente, nosso aplicativo não pode guardar seu estado se atualizarmos a página. Uma excelente maneira de resolver esse problema é armazenar o estado do aplicativo no [localStorage](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage) e depois restaurá-lo quando executarmos o aplicativo novamente.

Se você estivesse trabalhando com um back-end, isso não seria um problema. Mesmo tendo um cache temporário em `localStorage` pode ser útil. Certifique-se de não armazenar nenhum dado sensível, pois é fácil acessar.

## Entendendo `localStorage`

`localStorage` é uma parte da Web Storage API. A outra metade, `sessionStorage`, existe enquanto o navegador estiver aberto enquanto, `localStorage` persiste os dados, mesmo neste caso. Ambos compartilham [a mesma API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API), conforme abaixo:

* `storage.getItem(k)` - Retorna o valor da string armazenada para a chave fornecida.
* `storage.removeItem(k)` - Remove os dados que correspondem à chave.
* `storage.setItem(k, v)` - Armazena o valor dado usando a chave fornecida.
* `storage.clear()` - Esvazia todo o conteúdo armazenado.

É interessante explorar a API através das ferramentas de desenvolvimento do seu navegador. No Chrome, especialmente na aba *Resources*, é uma maneira útil porque permite que você inspecione os dados e realize operações diretas nele. Você pode até mesmo usar os atalhos `storage.key` e `storage.key = 'value'` no console para ajustes rápidos.

`localStorage` e `sessionStorage`, combinados, podem usar até 10 MB de dados. Mesmo que atualmente eles sejam bem suportados, existem certos casos em que eles podem falhar. Esses casos incluem a falta de memória no Internet Explorer (falha silenciosa) ou falhando completamente no modo privado do Safari. No entanto, é possível resolver essas falhas.

T> Você pode dar suportar ao Safari no modo privado, ao tentar escrever em `localStorage`, se isso falhar, você pode usar o espaço em memória no Safari, ou apenas notifique o usuário sobre a situação. Veja esse discussão no [Stack Overflow](https://stackoverflow.com/questions/14555347/html5-localstorage-error-with-safari-quota-exceeded-err-dom-exception-22-an).

## Implementando um serviço com `localStorage`

Para manter as coisas simples e gerenciáveis, implementaremos um pequeno serviç, `storage`, para guardar essas complexidades. A API terá `get(k)`, para buscar itens armazenados, e `set(k, v)`, para salvar um item. Dado que a API do `localStorage` é baseada em strings, usaremos `JSON.parse` e `JSON.stringify` para serialização. Como `JSON.parse` pode falhar, isso é algo que precisamos levar em consideração. Considere a implementação abaixo:

**app/libs/storage.js**

```javascript
export default storage => ({
  get(k) {
    try {
      return JSON.parse(storage.getItem(k));
    }
    catch(e) {
      return null;
    }
  },
  set(k, v) {
    storage.setItem(k, JSON.stringify(v));
  }
})
```

Essa implementação é suficiente para os nossos propósitos. Não é infalível e falhará se colocarmos muitos dados em um armazenamento. Para superar esses problemas sem ter que resolvê-los você mesmo, seria possível usar uma biblioteca como [localForage](https://github.com/mozilla/localForage) para gerenciar essa complexidade.

## Persistindo os dados da aplicação usando `FinalStore`

Ter um serviço para escrever e ler do `localStorage` não é o suficiente. Ainda precisamos conectar a aplicação de alguma forma. As soluções de gerenciamento de estado fornecem ganchos para esse propósito. Muitas vezes, você encontrará uma maneira de interceptá-los. No caso, Alt fornece uma opção embutida conhecida como `FinalStore`.

Nós já configuramos isso em nossa instância Alt. O que resta é escrever o estado do aplicativo no `localStorage` quando ele for alterado. Também precisamos carregar o estado quando começamos a executá-lo. Em outras palavras, esses processos são conhecidos como **snapshotting** e **bootstrapping**.

T> Uma maneira alternativa de lidar com o armazenamento dos dados seria tirar um **snapshot** somente quando a janela for fechada. Existe um evento chamado `window.beforeunload` que poderia ser usado. No entanto, essa abordagem é frágil. E se algo inesperado acontecer e o evento não for ativado por algum motivo? Você perderá os dados.

## Implementando a lógica de persistência

Podemos lidar com a lógica de persistência em um módulo separado, dedicado a ele. Agora, vamos conectá-lo na configuração do aplicativo.

Dado que pode ser útil para desativar temporariamente o **snapshot**, pode ser uma boa idéia implementar um parâmetro de `debug`. A idéia é que, se o parâmetro estiver configurado, ignoraremos o armazenamento dos dados.

Isso é particularmente útil se, de alguma forma, conseguirmos quebrar drasticamente o estado do aplicativo durante o desenvolvimento, assim, ele nos permite restaurá-lo para um estado em branco facilmente através de `localStorage.setItem('debug', 'true')` (ou `localStorage.debug = true`), `localStorage.clear()` e por fim, atualizar o navegador.

Dado que o `bootstrapping` também pode falhar por algum motivo desconhecido, nós poderemos ter acesso a mensagem de erro. Pode ser uma boa idéia continuar com a iniciação da aplicação, mesmo que algo horrível aconteça neste momento. A fase de `snapshot` é mais fácil, pois precisamos apenas verificar se o parâmetro `debug` está ativo, para então, salvar ou não os dados.

A implementação abaixo ilustra as idéias:

**app/libs/persist.js**

```javascript
export default function(alt, storage, storageName) {
  try {
    alt.bootstrap(storage.get(storageName));
  }
  catch(e) {
    console.error('Failed to bootstrap data', e);
  }

  alt.FinalStore.listen(() => {
    if(!storage.get('debug')) {
      storage.set(storageName, alt.takeSnapshot());
    }
  });
}
```

Você acabaria com algo semelhante em outros sistemas de gerenciamento de estado. Você precisará encontrar ganchos equivalentes para inicializar o sistema com dados carregados a partir do `localStorage` e escreva seu estado lá quando alterado.

## Conectando a lógica de persistência com a aplicação

Ainda está faltando uma parte para fazer tudo isso funcionar. Precisamos conectar a lógica com nossa aplicação. Felizmente, há um lugar adequado para isso, na nossa configuração. Vamos ajustar da seguinte maneira:

**app/components/Provider/setup.js**

```javascript
leanpub-start-insert
import storage from '../../libs/storage';
import persist from '../../libs/persist';
leanpub-end-insert
import NoteStore from '../../stores/NoteStore';

export default alt => {
  alt.addStore('NoteStore', NoteStore);

leanpub-start-insert
  persist(alt, storage(localStorage), 'app');
leanpub-end-insert
}
```

Se você tentar atualizar o navegador agora, o aplicativo deve manter seu estado. Dado que a solução é genérica, adicionar mais estado ao aplicativo não deve ser um problema. Poderíamos também integrar um back-end adequado através dos mesmos ganchos se quisermos.

Se tivéssemos um back-end real, poderíamos passar os dados iniciais como parte do HTML e carregá-lo a partir dali. Isso evitaria uma requisição de ida e volta ao servidor. E podemos renderizar também a marcação inicial do aplicativo, acabaríamos por implementar o básico da abordagem de **renderização universal**. A renderização universal é uma técnica poderosa que permite usar o React para melhorar o desempenho do seu aplicativo e obter benefícios de SEO.

W> Nosso serviço `persist`não é perfeiro, contém falahas. É fácil terminar em uma situação em que o `localStorage` contém dados inválidos devido a mudanças feitas no modelo de dados. Isso leva você ao mundo dos esquemas e migrações de banco de dados. A lição aqui é, quanto mais estados e lógica você adiciona na sua aplicação, mais complicado é de lidar com isso.

## Limpando `NoteStore`

Antes de seguir em frente, seria uma boa idéia para limpar `NoteStore` um pouco. Ainda há código dos nossos experimentos anteriores. Dado que a persistência funciona agora, podemos também começar de um estado em branco. Mesmo que quisessemos alguns dados iniciais, seria melhor lidar com isso em uma camada superior, como a inicialização do aplicativo. Vamos modificar `NoteStore` da seguinte maneira:

**app/stores/NoteStore.js**

```javascript
leanpub-start-delete
import uuid from 'uuid';
leanpub-end-delete
import NoteActions from '../actions/NoteActions';

export default class NoteStore {
  constructor() {
    this.bindActions(NoteActions);

leanpub-start-delete
    this.notes = [
      {
        id: uuid.v4(),
        task: 'Learn React'
      },
      {
        id: uuid.v4(),
        task: 'Do laundry'
      }
    ];
leanpub-end-delete
leanpub-start-insert
    this.notes = [];
leanpub-end-insert
  }
  ...
}
```

Isso é o suficiente por enquanto. Agora, nosso aplicativo deve começar a partir de um estado em branco.

## Implementações alternativas

Mesmo usando Alt nesta implementação inicial, ele não é a única opção. Para comparar várias arquiteturas, implementei o mesmo aplicativo usando diferentes técnicas. Eu os comparei brevemente abaixo:

* [Redux](http://rackt.org/redux/) é uma arquitetura inspirada no Flux que foi projetada com o **hot loading** como sua principal intenção. O Redux opera com base em uma única árvore de estado. Essa árvore é manipulada usando *funções puras* conhecidas como redutores. Mesmo que haja um código importante, o Redux te força a usar programação funcional. A implementação é bastante próxima do Alt. - [Redux demo](https://github.com/survivejs/redux-demo)
* Comparado com Redux, [Cerebral](http://www.cerebraljs.com/) tem um ponto de partida diferente. Foi desenvolvido para fornecer informações sobre *como* o aplicativo muda seu estado. O Cerebral é mais opinativo na maneira de desenvolver e, como resultado, vem com algumas soluções incluídas. - [Cerebral demo](https://github.com/survivejs/cerebral-demo)
* [MobX](https://mobxjs.github.io/mobx/) permite que suas estruturas de dados sejam observáveis. As estruturas podem então ser conectadas com os componentes React para que sempre que essas estruturas se atualizem, ela faça o mesmo com os componentes React. Dado que as referências reais entre estruturas podem ser usadas, a implementação Kanban é surpreendentemente simples. - [MobX demo](https://github.com/survivejs/mobx-demo)

## Relay?

Comparado com Flux, a solução [Relay](https://facebook.github.io/react/blog/2015/02/20/introducing-relay-and-graphql.html), criada pelo Facebook, melhora a camada de busca de dados. Ele permite que você as requisições de dados sejam feitas no nível do componente. Ele pode ser usado separadamente ou com Flux, dependendo de suas necessidades.

Dado que ainda é tecnologia em grande parte não testada, não vamos falar sobre ele neste livro. O Relay vem com requisitos especiais próprios (como uma API compatível com GraphQL). Só o tempo dirá como é adotado pela comunidade.

## Conclusão

Neste capítulo, você viu como configurar `localStorage` para fazer a persistência do estado do aplicativo. É uma técnica simples e útil para se conhecer. Agora que temos a camada de persistência resolvida, estamos prontos para começar a criar nosso quadro Kanban!
