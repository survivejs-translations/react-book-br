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

## Connecting Persistency Logic with the Application

We are still missing one part to make this work. We'll need to connect the logic with our application. Fortunately there's a suitable place for this, the setup. Tweak as follows:

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

If you try refreshing the browser now, the application should retain its state. Given the solution is generic, adding more state to the system shouldn't be a problem. We could also integrate a proper back-end through the same hooks if we wanted.

If we had a real back-end, we could pass the initial payload as a part of the HTML and load it from there. This would avoid a round trip. If we rendered the initial markup of the application as well, we would end up implementing basic **universal rendering** approach. Universal rendering is a powerful technique that allows you to use React to improve the performance of your application while gaining SEO benefits.

W> Our `persist` implementation isn't without its flaws. It is easy to end up in a situation where `localStorage` contains invalid data due to changes made to the data model. This brings you to the world of database schemas and migrations. The lesson here is that the more you inject state and logic to your application, the more complicated it gets to handle.

## Cleaning Up `NoteStore`

Before moving on, it would be a good idea to clean up `NoteStore`. There's still some code hanging around from our earlier experiments. Given persistency works now, we might as well start from a blank slate. Even if we wanted some initial data, it would be better to handle that at a higher level, such as application initialization. Adjust `NoteStore` as follows:

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

This is enough for now. Now our application should start from a blank slate.

## Alternative Implementations

Even though we ended up using Alt in this initial implementation, it's not the only option. In order to benchmark various architectures, I've implemented the same application using different techniques. I've compared them briefly below:

* [Redux](http://rackt.org/redux/) is a Flux inspired architecture that was designed with hot loading as its primary constraint. Redux operates based on a single state tree. The state of the tree is manipulated using *pure functions* known as reducers. Even though there's some boilerplate code, Redux forces you to dig into functional programming. The implementation is quite close to the Alt based one. - [Redux demo](https://github.com/survivejs/redux-demo)
* Compared to Redux, [Cerebral](http://www.cerebraljs.com/) had a different starting point. It was developed to provide insight on *how* the application changes its state. Cerebral provides more opinionated way to develop, and as a result, comes with more batteries included. - [Cerebral demo](https://github.com/survivejs/cerebral-demo)
* [MobX](https://mobxjs.github.io/mobx/) allows you to make your data structures observable. The structures can then be connected with React components so that whenever the structures update, so do the React components. Given real references between structures can be used, the Kanban implementation is surprisingly simple. - [MobX demo](https://github.com/survivejs/mobx-demo)

## Relay?

Compared to Flux, Facebook's [Relay](https://facebook.github.io/react/blog/2015/02/20/introducing-relay-and-graphql.html) improves on the data fetching department. It allows you to push data requirements to the view level. It can be used standalone or with Flux depending on your needs.

Given it's still largely untested technology, we won't be covering it in this book yet. Relay comes with special requirements of its own (GraphQL compatible API). Only time will tell how it gets adopted by the community.

## Conclusion

In this chapter, you saw how to set up `localStorage` for persisting the application state. It is a useful little technique to know. Now that we have persistency sorted out, we are ready to start generalizing towards a full blown Kanban board.
