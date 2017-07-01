# Introdução

O desenvolvimento front-end tem avançado muito rápido. Uma boa indicação disso é o ritmo no qual as novas tecnologias aparecem na cena. [React](https://facebook.github.io/react/) é um desses recentes recém-chegados. Mesmo que a tecnologia em si seja simples, há muito acontecendo em torno dele.

O objetivo deste livro é ajudá-lo a começar com React e fornecer uma compreensão do ecossistema em torno dele, para que você saiba onde procurar.

Nossa configuração de desenvolvimento é baseada no Webpack. Há [um livro separado](http://survivejs.com/webpack/introduction/) que fala sobre ele, mas não espero que você entenda bem sobre Webpack ler este livro.

## O que é React?

React é uma biblioteca JavaScript criada pelo Facebook, é uma abstração da interface do usuário, baseada em componentes. Um componente pode ser uma entrada de formulário, um botão ou qualquer outro elemento em sua interface. Isso fornece um contraste interessante para abordagens anteriores, pois o React não está vinculado ao DOM, por design. Você pode usá-lo para implementar aplicativos móveis, por exemplo.

### React é apenas Uma Parte do Todo

Tendo que, o React foca apenas na interface do usuário, você provavelmente terá que complementá-la com outras bibliotecas, para lhe dar os bits que faltam. Isso proporciona um contraste interessante com as abordagens baseadas em frameworks, que lhe dão muito mais recursos por padrão. Ambas as abordagens têm seus méritos. Neste livro, vamos nos concentrar na abordagem orientada à bibliotecas.

As idéias introduzidas pelo React influenciaram o desenvolvimento dos frameworks. O mais importante é que nos ajudou a entender o quão bem o pensamento baseado em componentes se adapta às aplicações web.

## O que você vai aprender?

![Aplicação Kanban](images/kanban_05.png)

Este livro lhe ensinará a construir um aplicativo [Kanban](https://en.wikipedia.org/wiki/Kanban). Além disso, são discutidos mais aspectos teóricos do desenvolvimento web. Ao concluir o projeto, você terá uma boa idéia de como implementar algo por conta própria. Durante o processo, você aprenderá por que certas bibliotecas são úteis e poderá justificar melhor suas escolhas de tecnologia.

## Como este livro é organizado?

Para começar, desenvolveremos um pequeno clone de uma famosa [Aplicação de Notas](http://todomvc.com/). Isso nos levará a problemas relacionado à como escalar sua aplicação. Às vezes, você precisa fazer as coisas da maneira burra para entender por que são necessárias melhores soluções.

A partir daí, vamos generalizar e colocar [uma arquitetura Flux](https://facebook.github.io/flux/docs/overview.html) em jogo. Vamos aplicar algumas mágicas com [Drag and Drop (DnD)](https://gaearon.github.io/react-dnd/) e começar a arrastar as coisas. Finalmente, obteremos uma aplicação de qualidade de produção feita.

A parte final e teórica do livro aborda tópicos mais avançados. Se você está lendo a edição comercial deste livro, há algo extra para você. Vou mostrar-lhe como organizar sua aplicação React para produzir código com uma qualidade superior. Você também aprenderá a testar seus componentes e lógica. Você aprenderá a modelar sua aplicação React de maneiras mais inteligente e a ter uma idéia melhor de como estruturar seus projetos.

Os apêndices no final destinam-se a semear novos pensamentos e explicar aspectos como, características de linguagem, em maiores detalhes. Se houver algum sintaxe no livro que, em um primeiro momento, pareçam estranhas para você, você provavelmente encontrará mais informações por lá.

## O que é Kanban?

![Kanban por Dennis Hamilton (CC BY)](images/kanban_intro.jpg)

Kanban, desenvolvido originalmente na Toyota, permite que você acompanhe o status das tarefas. Ele pode ser modelado em termos de "Colunas" e "Notas". `Notas` movem-se através de `Colunas` representando estágios, da esquerda para a direita, à medida que se tornam completos. As "Notas" podem conter informações sobre a própria tarefa, sua prioridade e assim por diante, conforme necessário.

O sistema pode ser estendido de várias maneiras. Uma maneira simples é aplicar um limite de trabalho em andamento (*working in progress/WIP*) por coluna. O efeito disso é que você é forçado a se concentrar em fazer tarefas. Essa é uma das boas consequências do uso do Kanban. Mover essas notas ao redor é satisfatório. Como um bônus, você ganha visibilidade e sabe o que ainda está por fazer.

### Onde usar Kanban?

Este sistema pode ser usado para vários fins, incluindo software e gerenciamento de tarefas na sua vida. Você poderia usá-lo para organizar seus projetos pessoais ou metas de vida, por exemplo. Mesmo que seja uma ferramenta simples, é bastante poderosa, e você pode encontrar uso para isso em muitos lugares.

### Como construir um Kanban?

A maneira mais simples de construir um Kanban é criar um monte de notas em **post-it** e encontrar uma parede. Depois disso, você os separa em colunas. Estas `Colunas` podem consistir das seguintes etapas: **A Fazer**, **Fazendo**, **Feito**. Todos as `Notas` vão para o **A Fazer** inicialmente. À medida que você começar a trabalhar nelas, você as move para **Fazendo**, e, finalmente, para **Feito**, quando concluídas. Esta é a maneira mais simples de começar.

Este é apenas um exemplo de organização de colunas. As colunas podem ser organizdas para corresponder ao seu processo. Pode haver etapas de **Aprovação**, por exemplo. Se você estiver modelando para um processo de desenvolvimento de software, você poderia ter colunas separadas para **Testes** e **Implantação**, por exemplo.

### Implementações de Kanban disponíveis

[Trello](https://trello.com/) é, talvez, a implementação online mais conhecida de Kanban. Sprintly abriu o código fonte da sua implementação de [Kanban em React](https://github.com/sprintly/sprintly-kanban). Existe uma soluçõa usando Meteor, [wekan](https://github.com/wekan/wekan), que é outro bom exemplo. O nosso não será tão sofisticado como estes, mas será suficiente para começar.

## Para quem é este livro?

Eu espero que você tenha um conhecimento básico de JavaScript e Node.js. Você deve poder usar **npm** em um nível elementar. Se você souber algo sobre Raect, ou ES6, é ótimo! Ao ler este livro, você aprofundará sua compreensão nessas tecnologias.

Uma das coisas mais difíceis ao escrever um livro, é escrevê-lo no nível certo. Dado que o livro cobre uma variedade de tecnologias, existem apêndices que cobrem tópicos básicos, como detalhes da linguagem, com maior detalhe do que o conteúdo principal. Então, se você não está seguro de alguma coisa, você pode conferi-los primeiramente.

Há também um [chat da comunidade](https://gitter.im/survivejs/react) disponível. Se você quiser perguntar algo diretamente, estamos lá para ajudar. Todos os comentários que você possa ter, serão para melhorar o conteúdo do livro. A última coisa que eu quero, é ter alguém lutando com o conteúdo do livro.

## Como posso ler esse livro?

Embora, de uma maneira natural, ler um livro seja começar a partir do primeiro capítulo e ler os capítulos sequencialmente, essa não é a única opção neste livro. A ordem dos capítulos é apenas uma sugestão de leitura. Dependendo da sua experiência, você pode começar pela primeira parte e depois ir direto para os tópicos avançados.

O livro não cobre tudo o que você precisa saber para desenvolver aplicativos front-end. Isso é simplesmente demais para um único livro. Acredito que, no entanto, ele possa ser capaz de empurrá-lo na direção certa. O ecossistema em torno do React é bastante grande, e fiz o meu melhor para cobrir um bom pedaço dele.

Dado que o livro se baseia em uma variedade de novos recursos do JavaScript, resolvi reunir os mais importantes em um *anexo sobre a linguagem*, que fornece uma revisão rápida sobre eles. Se você quer entender os recursos isoladamente ou está se sentindo inseguro sobre alguma coisa, esse é um bom lugar para começar!

## Versão do livro

Como este livro recebe uma quantidade razoável de manutenção e melhorias, devido ao ritmo das inovações, há um esquema de versão que uso. Eu mantenho notas sobre os lançamento de cada nova versão no [blog do livro](http://survivejs.com/blog/), para falar sobre o que mudou entre as versões. Você também pode examinar o repositório no GitHub, isso pode ser benéfico para sua curiosidade. Eu recomendo usar o GitHub *compare* para este propósito. Exemplo:

```
https://github.com/survivejs/react/compare/v2.1.0...v2.5.8
```

A página irá mostrarlhe os detalhes individuais que foram alterados no projeto entre asa versões fornecidas. Você também pode ver as linhas que mudaram no livro. Isso exclui os capítulos privados, mas é o suficiente para lhe dar uma boa idéia das principais mudanças feitas no livro.

A versão atual do livro é **2.5.8**.

## Material extra

O conteúdo e código fonte do livro estão disponíveis no repositório [do livro no GitHub](https://github.com/survivejs/react). Observe que o repositório usa a branch `dev` como padrão do projeto, isso facilita na contribuição. Para encontrar o código que combina com a versão do livro que você está lendo, use o selecionador de tags na interface do GitHub, como na imagem abaixo:

![Selecionador de tag do GitHub](images/github.png)

O repositório do livro contém códigos por capítulos. Isso significa que você pode começar de qualquer lugar que quiser, sem ter que digitar as alterações você mesmo. Se você não tiver certeza de algo, você sempre pode usar isso como referência.

Você pode encontrar vários materiais complementarres na [organização SurviveJS](https://github.com/survivejs/). Exemplos disso são implementações alternativas do aplicativo em [MobX](https://github.com/survivejs/mobx-demo), [Redux](https://github.com/survivejs/redux-demo), e [Cerebral / Baobab](https://github.com/survivejs/cerebral-demo). Estudar o código fonte pode dar uma boa idéia de como diferentes arquiteturas funcionam usando o mesmo exemplo.

## Dúvidas e Suporte

Como nenhum livro é perfeito, talvez você encontre algum problem ou tenha alguma dúvida relacionada ao conteúdo. Existem várias formas para resolver isso:

* Entre em contato comigo através do [GitHub Issue Tracker](https://github.com/survivejs/react/issues)
* Participe do grupo no [Gitter Chat](https://gitter.im/survivejs/react)
* Siga [@survivejs](https://twitter.com/survivejs) no Twitter para ficar por dentro das novidades oficiais ou me envie um tweet diretamente em [@bebraw](https://twitter.com/bebraw)
* Me envie um email em [info@survivejs.com](mailto:info@survivejs.com)
* Me pergunte qualquer coisa sobre Webpack ou React em [SurviveJS AmA](https://github.com/survivejs/ama/issues)

Se você postar uma pergunta no [Stack Overflow](http://stackoverflow.com/search?q=survivejs), use a tag [**survivejs**](https://stackoverflow.com/questions/tagged/survivejs), assim, eu serei notificado. Você também pode usar a hashtag **#survivejs** no Twitter.

Tentei abordar alguns problemas comuns no apêndice *Troubleshooting*. Ele será expandido à medida que forem encontrados mais problemas comuns.

## Novidades

Eu divulgo novidades do SurviveJS através de alguns canais:

* [Newsletter](http://eepurl.com/bth1v5)
* [Twitter](https://twitter.com/survivejs)
* [Blog RSS](http://survivejs.com/atom.xml)

Fique à vontade para participar!

## Agradecimentos

Um esforço como este não seria possível sem o apoio da comunidade. Há muitas pessoas a agradecer como resultado!

Um grande obrigado para [Christian Alfoni](http://www.christianalfoni.com/) por criar o [react-webpack-cookbook](https://github.com/christianalfoni/react-webpack-cookbook) junto comigo. Esse trabalho foi o ponta pé inicial, que levou a este livro e, eventualmente, tornou-se [um livro a parte](http://survivejs.com/webpack/introduction).

O livro não seria metade do que é sem a edição e comentários do meu editor [Jesús Rodríguez Rodríguez](https://github.com/Foxandxss). Obrigado.

Agradecimentos especiais a Steve Piercy por inúmeras contribuições. Obrigado a [Prospect One](http://prospectone.pl/) e [Dixon & Moe](http://dixonandmoe.com/) na ajuda com o logotipo e os detalhes gráficos. Obrigado pela revisão de Ava Mallory e EditorNancy do fiverr.com.

Ao longo do caminho, várias contribuições e feedback valiosos, gostaria de agradecer, Vitaliy Kotov, @af7, Dan Abramov, @dnmd, James Cavanaugh, Josh Perez, Nicholas C. Zakas, Ilya Volodin, Jan Nicklas, Daniel de la Cruz, Robert Smith, Andreas Eldh, Brandon Tilley, Braden Evans, Daniele Zannotti, Partick Forringer, Rafael Xavier de Souza, Dennis Bunskoek, Ross Mackay, Jimmy Jia, Michael Bodnarchuk, Ronald Borman, Guy Ellis, Mark Penner, Cory House, Sander Wapstra, Nick Ostrovsky, Oleg Chiruhin, Matt Brookes, Devin Pastoor, Yoni Weisbrod, Guyon Moree, Wilson Mock, Herryanto Siatono, Héctor Cascos, Erick Bazán, Fabio Bedini, Gunnari Auvinen, Aaron McLeod, John Nguyen, Hasitha Liyanage, Mark Holmes, Brandon Dail, Ahmed Kamal, Jordan Harband, Michel Weststrate, Ives van Hoorne, Luca DeCaprio, @dev4Fun, Fernando Montoya, Hu Ming, @mpr0xy, David "@davegomez" Gómez, Aleksey Guryanov, Elio D'antoni, Yosi Taguri, Ed McPadden, Wayne Maurer, Adam Beck, Omid Hezaveh, Connor Lay, Nathan Grey, Avishay Orpaz, Jax Cavalera, Juan Diego Hernández, Peter Poulsen, Harro van der Klauw, Tyler Anton, Michael Kelley, @xuyuanme, @RogerSep, Jonathan Davis, @snowyplover, Tobias Koppers, Diego Toro, George Hilios, Jim Alateras, @atleb, Andy Klimczak, James Anaipakos, Christian Hettlage, Sergey Lukin, Matthew Toledo, Talha Mansoor, Pawel Chojnacki, @eMerzh, Gary Robinson, Omar van Galen, Jan Van Bruggen, Savio van Hoi, Alex Shepard, Derek Smith, Tetsushi Omi, Maria Fisher, Rory Hunter, Dario Carella, Toni Laukka, Blake Dietz, Felipe Almeida, Greg Kedge, Deepak Kannan, Jake Peyser, Alfred Lau, Tom Byrer, Stefanos Grammenos, Lionel Ringenbach, Hamilton Greene, Daniel Robinson, @karloxyz, Nicolò Ribaudo, Andrew Wooldridge, Francois Constant, Wes Price, Dawid Karabin, @alavkx, Aitor Gómez-Goiri, P.E. Butler III, @TomV, John Korzhuk, @markfox1, Jaime Liz, Richard C. Davis, e muitos outros! Se seu nome estiver faltando, talvez eu tenha esquecido de adicioná-lo.
