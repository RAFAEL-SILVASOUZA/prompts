Você é um engenheiro 3D sênior especializado em WebGL/Three.js. Crie, do zero e em uma única resposta, uma cena interativa de um PARQUE DE DIVERSÕES MODERNO E DE PORTE MÉDIO, EM PLENO FUNCIONAMENTO, usando APENAS HTML, CSS, JavaScript puro e Three.js. O resultado deve ser um artefato autônomo que roda no navegador, com TODAS as atrações animadas e em movimento simultâneo desde o início.

## Stack e restrições
- Three.js na versão estável mais recente, importado via `importmap` de um CDN (jsDelivr ou unpkg). Use ES modules.
- `OrbitControls` de `three/addons/controls/OrbitControls.js`.
- Sem frameworks (nada de React/Vue) e sem engines de física pesadas — a animação é feita à mão com trigonometria e curvas.
- Entregue em um único `index.html` autossuficiente (HTML + CSS + JS embutidos). Como usa ES modules, avise que pode ser preciso servir via servidor local (ex.: `python -m http.server`) por causa de CORS em `file://`.
- Loop de render com `THREE.Clock` e delta time (animações independentes de FPS). Alvo: 60 FPS.

## Ambiente da cena
- Terreno plano com gramado, caminhos/calçadas conectando as atrações e uma praça central.
- Céu com gradiente (sky dome ou `Scene.background`) e névoa sutil (`fog`) ao fundo.
- Cercas no perímetro, PORTÃO DE ENTRADA com letreiro, bilheteria, bancos, lixeiras, postes de luz e barracas de comida (cachorro-quente, algodão-doce, sorvete).
- Árvores e arbustos espalhados (use `InstancedMesh` para repetição eficiente).
- Balões coloridos amarrados em alguns pontos, oscilando levemente com o "vento".

## Atrações (todas devem estar EM MOVIMENTO)
Implemente pelo menos estas 6, cada uma como função separada e com o comportamento indicado:

1. RODA-GIGANTE: estrutura de suporte + roda girando lenta e continuamente. As gôndolas ficam presas na borda e PERMANECEM SEMPRE NA HORIZONTAL (aplicar contra-rotação para não virarem de cabeça para baixo). Luzes nas hastes que acendem à noite.
2. CARROSSEL: plataforma circular com telhado listrado girando. Cavalos sobem e descem com movimento senoidal, cada um com fase diferente. Poste central iluminado.
3. MONTANHA-RUSSA: trilho fechado (loop) construído a partir de uma `CatmullRomCurve3` com subidas, descidas e pelo menos uma curva fechada. Um comboio de carrinhos percorre o trilho continuamente, POSICIONADO E ORIENTADO ao longo da curva (use `getPointAt`/`getTangentAt` e um frame de orientação para alinhar cada carrinho à direção do trilho). Estrutura de suporte visível sob o trilho.
4. XÍCARAS GIRATÓRIAS (teacups): plataforma base girando; sobre ela, várias xícaras que também giram; e cada xícara girando no próprio eixo (rotações aninhadas).
5. TORRE DE QUEDA (drop tower): gôndola que sobe devagar, pausa no topo e DESPENCA com aceleração, repetindo o ciclo. Use easing na descida.
6. CADEIRAS VOADORAS (chair swing): topo giratório com correntes e assentos que SE ABREM PARA FORA conforme a rotação (o ângulo das correntes aumenta com a velocidade angular).

(Se sobrar fôlego, adicione bate-bate se movendo numa arena.)

## Iluminação e ciclo dia/noite
- Iluminação diurna com `DirectionalLight` como sol (sombras configuradas — ajuste o `shadow.camera` para cobrir a cena e evite sombras estouradas), mais `HemisphereLight`/`AmbientLight` de preenchimento.
- Sombras ativadas (`shadowMap.enabled`, `PCFSoftShadowMap`); atrações e props projetam e recebem sombra.
- BOTÃO para alternar DIA ↔ NOITE. À noite: céu e luz ambiente escurecem e ACENDEM as luzes das atrações (lâmpadas nas rodas, no carrossel, nos postes) via `PointLight`/materiais emissivos. Use `InstancedMesh` com material emissivo para as centenas de lâmpadas.
- À noite, efeito ocasional de FOGOS DE ARTIFÍCIO (partículas) sobre o parque.

## Câmera, controles e interatividade
- `OrbitControls` com damping, limites de zoom e de ângulo (não deixar a câmera ir abaixo do chão). Enquadramento inicial mostrando o parque inteiro.
- CLIQUE em uma atração faz a câmera focar/aproximar suavemente e exibe um painel com o nome e uma descrição curta (use `Raycaster`).
- Opcional: botão de "tour" que passeia a câmera automaticamente pelas atrações.

## Ambientação viva
- MULTIDÃO: dezenas de figuras humanas simples (bonecos estilizados de cápsulas/blocos) andando pelos caminhos e formando filas. Use `InstancedMesh` e animação procedural (leve "bob" de caminhada).
- Bandeirinhas/faixas coloridas entre postes, tremulando.
- Detalhes de vida: fumacinha saindo da barraca de comida (partículas), etc.

## Interface (HUD)
- Overlay mínimo em CSS: título do parque, botão dia/noite, botão de tour e legenda curta de controles (arrastar = girar, scroll = zoom, clicar = focar).
- TELA DE CARREGAMENTO simples enquanto a cena monta.
- Contador de FPS opcional no canto.

## Qualidade visual e performance
- `renderer` com `antialias`, `ACESFilmicToneMapping` e `outputColorSpace = SRGBColorSpace`.
- Materiais PBR (`MeshStandardMaterial`) com paleta viva e coerente; nada de cor chapada sem sombreamento.
- `InstancedMesh` e geometria/material compartilhados em tudo que se repete (lâmpadas, árvores, multidão, cercas).
- `requestAnimationFrame`, redimensionamento responsivo (ajustar câmera e renderer no `resize`) e `pixelRatio` limitado (`Math.min(devicePixelRatio, 2)`).

## Estrutura do código
- Funções claras: `createGround`, `createFerrisWheel`, `createCarousel`, `createCoaster`, `createTeacups`, `createDropTower`, `createSwingRide`, `createCrowd`, `createLighting`, `setupUI`, `animate`, etc.
- Comentários em português nas partes principais, sobretudo a matemática das animações (curva da montanha-russa, contra-rotação das gôndolas, abertura das cadeiras).
- Sem erros no console. A cena inicia já animada e roda fluida.

## Critérios de aceitação (pronto quando)
- [ ] As 6 atrações estão visíveis e se movendo simultaneamente e corretamente.
- [ ] As gôndolas da roda-gigante nunca viram de cabeça para baixo.
- [ ] Os carrinhos da montanha-russa seguem o trilho colados à curva, com orientação correta.
- [ ] O botão dia/noite funciona e, à noite, as luzes das atrações acendem.
- [ ] Dá para orbitar, dar zoom e clicar numa atração para focar e ver seu nome.
- [ ] Há multidão andando e ambientação (árvores, barracas, balões, bandeirinhas).
- [ ] Roda a ~60 FPS num notebook comum, sem erros no console.

Entregue o código completo e funcional em um único arquivo.
