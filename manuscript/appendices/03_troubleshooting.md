# Solucionando Problemas

Eu tentei abordar alguns problemas comuns aqui. Este capítulo será expandido à medida que forem encontrados outros problemas comuns.

## `EPEERINVALID`

É possível que você veja uma mensagem como esta:

```bash
npm WARN package.json kanban-app@0.0.0 No repository field.
npm WARN package.json kanban-app@0.0.0 No README data
npm WARN peerDependencies The peer dependency eslint@0.21 - 0.23 included from eslint-loader will no
npm WARN peerDependencies longer be automatically installed to fulfill the peerDependency
npm WARN peerDependencies in npm 3+. Your application will need to depend on it explicitly.

...

npm ERR! Darwin 14.3.0
npm ERR! argv "node" "/usr/local/bin/npm" "i"
npm ERR! node v0.10.38
npm ERR! npm  v2.11.0
npm ERR! code EPEERINVALID

npm ERR! peerinvalid The package eslint does not satisfy its siblings' peerDependencies requirements!
npm ERR! peerinvalid Peer eslint-plugin-react@2.5.2 wants eslint@>=0.8.0
npm ERR! peerinvalid Peer eslint-loader@0.14.0 wants eslint@0.21 - 0.23

npm ERR! Please include the following file with any support request:
...
```

Em linguagem humana, isso significa que algum pacote, `eslint-loader` nesse caso, tema uma dependência rígida, `peerDependency`, declarada. Nosso projeto já possui uma versão mais recente. Dado que a dependência necessária declarada é mais antiga do que a nossa versão, obtemos esse erro específico.

Há algumas maneiras de contornar isso:

1. Informe a falha para o autor da biblioteca e espere que a extensão de versões seja ampliada.
2. Resolva o conflito localmente, configurando-o para uma versão que satisfaça a dependência. Nesse caso, poderíamos colocar `eslint` para versão `0.23` (`"eslint": "0.23"`), e todo mundo ficaria feliz.
3. Faça um *fork* da biblioteca, corrija a extensão de versões e aponte para a sua versão personalizada. Neste caso, você teria algo como `"<package>": "<github user>/<project>#<reference>"` declarado nas suas depêndencias.

T> Observe que as `peerDependency` são tratadas de forma diferente a partir de npm 3. Depois dessa versão, cabe ao consumidor do pacote (ou seja, você) lidar com isso. Este erro, em particular, desaparecerá.

## Warning: setState(...): Cannot update during an existing state transition

You might get this warning while using React. An easy way to end up getting it is to trigger
Você pode obter esse aviso usando React. Uma maneira fácil de encontra-lo é usar `setState()` dentro de um método, como `render()`. Às vezes, isso pode acontecer indiretamente. Uma maneira de causar o aviso é executar um método ao invés de vincular sua referência. Exemplo: `<input onKeyPress={this.checkEnter()} />`. Assumindo `this.checkEnter` faça uso de `setState()`, esse código irá falhar. Ao invés disso, você deveria usar `<input onKeyPress={this.checkEnter} />`, dessa maneira, você está vinculando a referência do método ao invés de executá-lo.

## Warning: React attempted to reuse markup in a container but the checksum was invalid

Você pode obter esse aviso por vários meios. Causas comuns abaixo:

* Você tentou montar sua aplicação React, várias vezes, usando o mesmo objeto alvo. Verifique o carregamento do seu script e verifique se o seu aplicativo é carregado apenas uma vez.
* A marcação existente não corresponde ao renderizado pelo React. Isso pode acontecer especialmente se você estiver renderizando a marcação inicial no servidor.

## `Module parse failed`

Ao usar o Webpack, um erro como esse pode surgir:

```bash
ERROR in ./app/components/Demo.jsx
Module parse failed: .../app/components/Demo.jsx Line 16: Unexpected token <
```

Isso significa que há alguma coisa impedindo o Webpack de interpretar o arquivo corretamente. Você deve verificar sua configuração e seus `loaders`. Verifique se os carregadores corretos são aplicados aos arquivos corretos. Se você estiver usando `include`, você deve verificar se o arquivo está incluído dentro do caminho definido no `include`.

## O projeto não compila

Mesmo que tudo funcione, em teoria, às vezes os intervalos de versões podem ser um problema, apesar do `SemVer`. Se algum pacote principal tem uma versão com grandes mudanças, digamos `babel`, e você executa `npm i`, em um momento infeliz, você pode acabar com um projeto que não compila.

Um bom primeiro passo é executar `npm update`. Isso verificará suas dependências e puxará as versões mais recentes baseado nas declarações`SemVer`. Se isso não resolver o problema, você pode tentar remover `node_modules` (`rm -rf node_modules`) do projeto e instalar as dependências (`npm i`) novamente. Alternativamente, você pode tentar incluir, explicitamente, algumas de suas dependências em versões específicas.

Muitas vezes você não está sozinho com seu problema. Portanto, vale a pena verificar os problemas do projeto, no Github, para ver o que está acontecendo. Você provavelmente pode encontrar uma boa solução alternativa ou uma proposta de correção por lá. Essas questões tendem a ser corrigidas rapidamente para projetos populares.

Em um ambiente de produção, é possível bloquear dependências de produção usando `npm shrinkwrap`. [A documentação oficial](https://docs.npmjs.com/cli/shrinkwrap) entra em mais detalhes desse tópico.
