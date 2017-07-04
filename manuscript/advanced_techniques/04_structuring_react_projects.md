# Estruturando seu Projeto React

React não impõe nenhuma estrutura de projeto específica. A coisa boa sobre isso é que ele permite que você faça uma estrutura para atender às suas necessidades. O ruim é que não é possível fornecer uma estrutura ideal para todos os projetos. Em vez disso, vou mostrar algumas inspirações que você pode usar para pensar sobre a sua estrutura.

## Diretório por Conceito

Nosso aplicativo Kanban tem uma estrutura um tanto plana:

```bash
├── actions
│   ├── LaneActions.js
│   └── NoteActions.js
├── components
│   ├── App.jsx
│   ├── Editable.jsx
│   ├── Lane.jsx
│   ├── Lanes.jsx
│   ├── Note.jsx
│   └── Notes.jsx
├── constants
│   └── itemTypes.js
├── index.jsx
├── libs
│   ├── alt.js
│   ├── persist.js
│   └── storage.js
├── main.css
└── stores
    ├── LaneStore.js
    └── NoteStore.js
```

É o suficiente para este projeto, mas existem algumas alternativas interessantes em torno:

* Arquivo por conceito - perfeito para pequenos protótipos. Você pode dividir isso quando seu aplicativo ficar mais sério.
* Diretório por Componente - É possível organizar componentes em diretórios próprios. Embora esta seja uma abordagem mais complexa, existem algumas vantagens interessantes, como veremos em breve.
* Diretório por Página - Essa abordagem torna-se relevante quando você deseja introduzir o roteamento na sua aplicação.

Existem mais alternativas, mas estas cobrem alguns dos casos comuns. Há sempre espaço para ajustes com base nas necessidades da sua aplicação.

## Diretório por Componente

Se dividirmos nossos componentes em diretórios próprios, seria algo assim:

```bash
├── actions
│   ├── LaneActions.js
│   └── NoteActions.js
├── components
│   ├── App
│   │   ├── App.jsx
│   │   ├── app.css
│   │   ├── app_test.jsx
│   │   └── index.js
│   ├── Editable
│   │   ├── Editable.jsx
│   │   ├── editable.css
│   │   ├── editable_test.jsx
│   │   └── index.js
...
│   └── index.js
├── constants
│   └── itemTypes.js
├── index.jsx
├── libs
│   ├── alt.js
│   ├── persist.js
│   └── storage.js
├── main.css
└── stores
    ├── LaneStore.js
    └── NoteStore.js
```

Comparado com a nossa solução atual, isso seria mais complexo. Os arquivos *index.js* estão disponíveis para fornecer pontos de entrada para os componentes. Mesmo que eles sejam opcionais, eles simplificam as importações.

Há, ainda, alguns benefícios interessantes nesta abordagem:

* Podemos aproveitar outras tecnologias, como os Módulos CSS, para estilizar cada componente separadamente.
* Dado que cada componente é um pequeno "pacote" próprio, agora seria mais fácil extraí-los do projeto. Você pode mover componentes genéricos para outros lugares e consumi-los em várias aplicações.
* Podemos definir testes unitários ao lado do componente. Essa abordagem encoraja você a criar testes específicos para seu componente e, ainda podemos ter testes de nível superior, na raiz do projeto, como anteriormente.

Pode ser interessante tentar mover suas *actions* e *stores* para "componentes" também. Ou eles poderiam seguir um esquema de diretório semelhante. O benefício disso é que ele permitiria que você definisse testes unitários de forma semelhante.

Esta configuração não é suficiente quando você quer adicionar várias páginas ao aplicativo. É necessário outra abordagem para suportar isso.

T> [gajus/create-index](https://github.com/gajus/create-index) pode gerar arquivos *index.js* automaticamente à medida que você desenvolve.

## Diretório por Página

Várias páginas trazem desafios próprios. Em primeiro lugar, você precisará definir um esquema de roteamento. [react-router](https://github.com/rackt/react-router) é uma alternativa popular para este propósito. Além de um esquema de roteamento, você precisará definir o que exibir em cada página. Você poderia ter páginas separadas para a página inicial do aplicativo, registro, quadro do Kanban, e assim por diante, combinando cada rota.

Esses requisitos significam que novos conceitos precisam ser introduzidos na estrutura. Uma maneira de lidar com o roteamento é move-lo para um componente `Routes` que coordena a exibição que é exibida a qualquer momento com base na rota atual. Ao invés de `App`, nós teríamos várias páginas. Aqui está uma possível estrutura:

```bash
├── components
│   ├── Note
│   │   ├── Note.jsx
│   │   ├── index.js
│   │   ├── note.css
│   │   └── note_test.jsx
│   ├── Routes
│   │   ├── Routes.jsx
│   │   ├── index.js
│   │   └── routes_test.jsx
│   └── index.js
...
├── index.jsx
├── main.css
└── views
    ├── Home
    │   ├── Home.jsx
    │   ├── home.css
    │   ├── home_test.jsx
    │   └── index.js
    ├── Register
    │   ├── Register.jsx
    │   ├── index.js
    │   ├── register.css
    │   └── register_test.jsx
    └── index.js
```

A idéia é a mesma que antes. Desta vez, nós temos mais partes para coordenar. O aplicativo começa a partir de `index.jsx` que irá desencadear rotas que, por sua vez, escolhem uma página para exibir. Depois disso, é o fluxo que nos estamos acostumados.

Esta estrutura pode ser escalada ainda mais, mas mesmo ela, tem seus limites. Uma vez que seu projeto começa a crescer, você pode querer introduzir outros conceitos junto dela. Poe exemplo, é comum introduzir diretórios por conceitos dentro de cada página e também os componentes.

Por exemplo, você pode ter um `LoginModal` que é exibido em determinadas páginas se a sessão do usuário tiver expirado. Ele seria composto de componentes primitivos. Esses recursos comuns podem até, ser excluídos do próprio projeto, em pacotes próprios, aumentando o potencial de reutilização.

## Conclusão

Não existe uma maneira única de estruturar seu projeto React. É um desses aspectos que vale a pena investir tempo para pensar. Escrever uma estrutura que lhe sirva bem, vale a pena. Uma estrutura clara ajuda no esforço de manutenção e torna o seu projeto mais compreensível para os outros.

Você pode mudar essa estrutura no decorrer do seu projeto. Uma estrutura complexa no início pode diminuir sua velocidade. À medida que o projeto evolui, também deve evoluir sua estrutura. É uma dessas coisas que vale a pena pensar, dado que afeta nossa produtividade e desenvolvimento.
