Você é um agente de desenvolvimento frontend especializado em JavaScript e Three.js.

Sua tarefa é criar uma aplicação web completa que represente uma sala de estar 3D navegável em primeira pessoa.

## Objetivo

Crie uma réplica visualmente agradável de uma sala de estar contendo:

* quatro paredes;
* piso;
* teto;
* um sofá;
* uma televisão;
* um painel ou móvel para a televisão;
* dois ou mais quadros nas paredes;
* um tapete;
* uma mesa de centro;
* iluminação interna;
* uma porta;
* elementos decorativos simples.

O usuário deve conseguir entrar na sala e caminhar livremente pelo ambiente utilizando teclado e mouse.

## Restrições técnicas

Utilize apenas:

* HTML;
* CSS;
* JavaScript;
* Three.js.

Não utilize:

* React;
* Vue;
* Angular;
* TypeScript;
* frameworks adicionais;
* backend;
* banco de dados;
* modelos 3D externos;
* arquivos GLTF, GLB, OBJ ou FBX;
* texturas externas obrigatórias;
* APIs externas além da importação do Three.js.

Todos os objetos da sala devem ser construídos com geometrias nativas do Three.js, como:

* BoxGeometry;
* PlaneGeometry;
* CylinderGeometry;
* SphereGeometry.

A aplicação deve funcionar abrindo o arquivo por meio de um servidor HTTP local.

## Estrutura obrigatória

Crie os seguintes arquivos:

```text
/index.html
/style.css
/main.js
```

Use Three.js por CDN com ES Modules.

Não concentre todo o código no arquivo HTML.

## Dimensões do ambiente

Considere uma sala retangular com aproximadamente:

* largura: 10 unidades;
* comprimento: 8 unidades;
* altura: 3 unidades.

Use uma escala coerente entre os objetos.

A câmera deve iniciar dentro da sala, em uma posição equivalente à altura dos olhos de uma pessoa.

## Navegação em primeira pessoa

Implemente navegação utilizando `PointerLockControls`.

Controles obrigatórios:

* `W`: andar para frente;
* `S`: andar para trás;
* `A`: andar para a esquerda;
* `D`: andar para a direita;
* mouse: movimentar a câmera;
* `Esc`: liberar o cursor.

Mostre uma tela inicial com a mensagem:

```text
Clique para entrar na sala
Use W, A, S e D para caminhar
Use o mouse para olhar ao redor
```

Ao clicar nessa tela, ative o Pointer Lock.

## Movimento

Implemente o movimento usando:

* vetor de direção;
* velocidade;
* delta time obtido por `THREE.Clock`;
* aceleração e desaceleração suaves.

O movimento não deve depender da taxa de quadros.

Não implemente pulo.

A câmera deve permanecer em uma altura fixa, simulando uma pessoa caminhando.

## Colisão obrigatória

Implemente colisões simples para impedir que o jogador:

* atravesse as paredes;
* saia da sala;
* atravesse o sofá;
* atravesse a mesa de centro;
* atravesse o móvel da televisão.

A colisão pode ser baseada em:

* limites mínimos e máximos no eixo X e Z;
* caixas delimitadoras com `THREE.Box3`;
* verificação da próxima posição antes de mover a câmera.

Não utilize biblioteca de física.

## Composição da sala

### Piso

Crie um piso cobrindo toda a sala.

Use uma cor semelhante a madeira, cimento ou porcelanato.

O piso deve receber sombras.

### Paredes

Crie quatro paredes com espessura visível.

Use cores claras e neutras.

As paredes devem bloquear o movimento do jogador.

### Teto

Crie um teto simples.

Evite que o interior fique completamente escuro.

### Sofá

Crie o sofá utilizando várias caixas:

* base;
* encosto;
* braços laterais;
* duas ou três almofadas.

O sofá deve estar voltado para a televisão.

Use materiais com aparência fosca.

### Televisão

Crie uma televisão formada por:

* moldura preta;
* tela;
* suporte ou painel traseiro.

A tela deve usar um material escuro com leve emissão de luz, simulando que está ligada.

Adicione uma imagem abstrata à tela utilizando um `CanvasTexture` gerado no próprio JavaScript.

Não carregue imagens externas.

### Móvel ou painel da televisão

Crie um móvel baixo ou painel utilizando caixas.

Inclua detalhes simples, como portas, gavetas ou prateleiras.

### Quadros

Crie pelo menos dois quadros nas paredes.

Cada quadro deve possuir:

* moldura;
* superfície interna;
* arte abstrata gerada com `CanvasTexture`.

As artes devem ser criadas programaticamente no JavaScript.

### Tapete

Crie um tapete fino entre o sofá e a televisão.

### Mesa de centro

Crie uma mesa de centro com:

* tampo;
* quatro pés ou uma base central.

### Porta

Crie uma porta visualmente identificável em uma das paredes.

A porta não precisa abrir.

### Decoração

Adicione pelo menos dois elementos decorativos simples, como:

* vaso;
* planta feita com cilindros e esferas;
* luminária;
* livros;
* almofadas;
* pequenas prateleiras.

## Iluminação

Implemente uma iluminação equilibrada utilizando:

* `AmbientLight` ou `HemisphereLight`;
* uma ou mais `PointLight`;
* uma `DirectionalLight`, caso necessário.

Ative sombras no renderer:

```javascript
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

Configure os principais objetos para projetar e receber sombras usando:

```javascript
object.castShadow = true;
object.receiveShadow = true;
```

Evite iluminação excessivamente escura ou estourada.

## Renderização

Configure:

```javascript
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
```

Use uma cor de fundo compatível com o interior da sala.

Implemente atualização correta do tamanho da câmera e do renderer no evento `resize`.

## Organização do código

Divida o código em funções claras, como:

```javascript
init();
createRoom();
createSofa();
createTelevision();
createTvStand();
createCoffeeTable();
createPictures();
createDecorations();
createLights();
setupControls();
setupCollisions();
animate();
```

Você pode ajustar os nomes, mas mantenha separação equivalente de responsabilidades.

Evite funções excessivamente longas.

Use constantes para dimensões e configurações importantes.

Comente apenas trechos que realmente precisem de explicação.

## Qualidade visual

A cena deve ter:

* proporções coerentes;
* objetos alinhados;
* sofá direcionado para a TV;
* quadros posicionados corretamente nas paredes;
* iluminação agradável;
* sombras suaves;
* materiais com roughness e metalness adequados;
* ausência de objetos flutuando;
* ausência de objetos atravessando paredes ou piso.

Utilize principalmente `MeshStandardMaterial`.

## Interface

Além da tela inicial, adicione uma pequena instrução no canto inferior da tela:

```text
WASD para andar | Mouse para olhar | ESC para sair
```

A interface deve ser discreta e não bloquear a visualização.

## Tratamento de erros

Antes de finalizar, verifique:

* se todos os imports estão corretos;
* se `PointerLockControls` foi importado corretamente;
* se não existem variáveis não declaradas;
* se a aplicação não gera erros no console;
* se o loop de animação foi iniciado;
* se o delta time está sendo calculado;
* se o redimensionamento da janela funciona;
* se as colisões impedem atravessar os objetos principais;
* se a câmera inicia dentro da sala;
* se todos os objetos estão visíveis.

## Critérios de aceitação

A tarefa só estará concluída quando:

1. A página abrir sem erros no console.
2. A sala estiver totalmente visível e iluminada.
3. O usuário conseguir olhar ao redor com o mouse.
4. O usuário conseguir caminhar usando W, A, S e D.
5. O usuário não conseguir atravessar as paredes.
6. O usuário não conseguir atravessar sofá, mesa e móvel da TV.
7. A sala possuir sofá, televisão, quadros, tapete, mesa, porta e decoração.
8. A aplicação se adaptar ao redimensionamento da janela.
9. Todo o código necessário estiver presente nos três arquivos solicitados.
10. Não houver dependência de modelos ou texturas externas.

## Formato da resposta

Entregue a resposta exatamente nesta ordem:

1. Estrutura de arquivos.
2. Conteúdo completo de `index.html`.
3. Conteúdo completo de `style.css`.
4. Conteúdo completo de `main.js`.
5. Instruções curtas para executar localmente.
6. Checklist final confirmando cada critério de aceitação.

Não entregue pseudocódigo.

Não omita partes usando comentários como:

```javascript
// restante do código
```

Não resuma o código.

Não explique conceitos antes de entregar os arquivos.

Produza uma implementação completa, funcional e diretamente executável.
