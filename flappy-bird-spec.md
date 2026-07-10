# Prompt — Réplica Flappy Bird (Single HTML)

## OBJETIVO
Criar uma réplica funcional e visualmente polida do jogo Flappy Bird contida em um **único arquivo HTML** (HTML + CSS + JavaScript inline). O resultado deve ser executável abrindo o arquivo diretamente no navegador, sem dependências externas. O jogo precisa ser **fluido**, com **movimentos sutis**, **zero travamentos** e uma **experiência amigável**.

---

## REGRAS ABSOLUTAS
1. **Um único arquivo** — nada de arquivos separados, CDNs ou imports externos.
2. **Zero bibliotecas** — apenas HTML5 Canvas, CSS e JavaScript vanilla.
3. **Funcional no navegador** — abrir o .html → jogo roda sem erros.
4. **Código comentado** — seções principais identificadas com comentários.
5. **Fluidez obrigatória** — 60 FPS alvo, zero travamentos, animações suaves.
6. **Movimentos sutis** — transições, rotações e efeitos visuais que dão vida ao jogo.

---

## FASE 1 — ESPECIFICAÇÃO TÉCNICA

### 1.1 Canvas e Performance

| Parâmetro | Valor |
|-----------|-------|
| Resolução do canvas | 400 × 600 px |
| FPS alvo | 60 (via `requestAnimationFrame`) |
| Delta time | Calcular a cada frame: `dt = (now - lastFrame) / 1000`, cap em 0.05s |
| Escalamento responsivo | CSS `max-width: 100%; max-height: 100vh; object-fit: contain` |

### 1.2 Loop de Jogo

```
gameLoop(timestamp):
  dt = calcDelta(timestamp)          ← evita velocidade variável
  clearCanvas()
  update(dt)                         ← física + lógica
  draw()                             ← renderização
  requestAnimationFrame(gameLoop)    ← recursão, nunca setInterval
```

**Anti-travamento:**
- Nunca usar `setInterval` ou `setTimeout` para o loop principal.
- Cap no delta time (máx 0.05s) para evitar saltos após perda de foco.
- Limpar arrays de canos/partículas que saíram da tela a cada frame.
- Minimizar alocações dentro do loop (reutilizar objetos quando possível).

### 1.3 Estados do Jogo

| Estado | Comportamento | Input |
|--------|---------------|-------|
| `IDLE` | Pássaro flutua suavemente no centro, cenário estático, tela de boas-vindas | Space/Click → transiciona para PLAYING |
| `PLAYING` | Física ativa, canos movem, pontuação visível, chão rola | Space/Click → pulo do pássaro |
| `GAME_OVER` | Pássaro cai até o chão, overlay escurece, placar aparece | Space/Click → reinicia para IDLE |

---

## FASE 2 — VISUAL E AMBIENTE

### 2.1 Paleta de Cores

| Elemento | Cor | Hex |
|----------|-----|-----|
| Céu (gradiente topo) | Azul claro | `#4EC0CA` |
| Céu (gradiente base) | Azul celeste | `#71C8D4` |
| Nuvens | Branco suave | `#D4EFCB` |
| Canos (corpo) | Verde clássico | `#5CBF2A` |
| Canos (borda/sombra) | Verde escuro | `#3D8C20` |
| Canos (capa/borda superior) | Verde médio | `#4CA922` |
| Chão (superfície) | Marrom claro | `#DED895` |
| Chão (faixa de grama) | Verde grama | `#5CBF2A` |
| Terra (abaixo da grama) | Marrom terra | `#C8B76E` |
| Pássaro (corpo) | Amarelo vivo | `#F8C83C` |
| Pássaro (barriga) | Amarelo claro | `#FCE38A` |
| Pássaro (asa) | Laranja suave | `#E89E3D` |
| Pássaro (bico) | Laranja escuro | `#E06B20` |
| Pássaro (olho) | Branco + pupila preta | `#FFFFFF` / `#000000` |
| Placar (texto) | Branco com contorno preto | `#FFFFFF` stroke `#000000` |
| Overlay Game Over | Cinza semi-transparente | `rgba(0,0,0,0.45)` |
| Fundo das montanhas/cityscape | Verde suave | `#82B83A` |

### 2.2 Cenário de Fundo (Camadas com Parallax)

O cenário é composto por **camadas que se movem em velocidades diferentes** para criar profundidade:

| Camada | Velocidade | Conteúdo |
|--------|------------|----------|
| Céu | Estática | Gradiente vertical `#4EC0CA` → `#71C8D4` |
| Nuvens | 0.2× velocidade base | 3-5 nuvens desenhadas com arcos/elipses brancas semi-transparentes |
| Cidade/bosque | 0.4× velocidade base | Silhueta de prédios ou árvores em `#82B83A` com opacidade 0.6 |
| Chão | 1× velocidade base | Faixa de grama (`#5CBF2A`, 25px de altura) + terra texturizada (`#C8B76E`) |

**Detalhes do chão:**
- Altura total: 80px na parte inferior do canvas.
- Faixa verde (grama): topo dos 80px, 25px de altura.
- Padrão de listras diagonais na terra para dar sensação de movimento.
- O chão se move horizontalmente junto com os canos (mesma velocidade).

### 2.3 Nuvens (Decoração)
- Desenhadas como grupos de 3-4 elipses sobrepostas em branco com `alpha: 0.7`.
- Posições Y aleatórias na metade superior do canvas.
- Movimento lento para a esquerda (parallax 0.2×).
- Quando saem da tela, reposicionam no lado direito com nova altura Y.

---

## FASE 3 — COMPONENTES JOGÁVEIS

### 3.1 Pássaro

**Dimensões:**
- Largura: 34px | Altura: 24px
- Posição X fixa: 80px (20% da largura do canvas)
- Posição Y inicial (IDLE): centro vertical do canvas (~300px)

**Anatomia visual (desenhado via Canvas API):**
- Corpo: elipse amarela (`#F8C83C`) com barriga mais clara (`#FCE38A`)
- Asa: elipse laranja (`#E89E3D`) na lateral — anima sobe/desce
- Bico: triângulo laranja escuro (`#E06B20`) apontando para a direita
- Olho: círculo branco com pupila preta, pequeno highlight branco

**Física:**
| Parâmetro | Valor |
|-----------|-------|
| Gravidade | 0.5 px/frame² (ajustável via constante) |
| Força do pulo | -8 px/frame (impulso instantâneo para cima) |
| Velocidade máxima de queda | 10 px/frame (cap para não cair demais) |
| Rotação (inclinação) | Baseada na velocidade vertical: subindo → -25°, caindo → +90° máx |

**Movimentos sutis:**
- **Flutuação idle**: em IDLE, o pássaro sobe e desce suavemente com `sin(time)` — amplitude de 8px, período de 1.5s.
- **Rotação dinâmica**: ao pular, inclina para cima (-25°). Ao cair, inclina progressivamente até +90° (bico para baixo). A transição de ângulo é suave, não instantânea.
- **Animação da asa**: a asa sobe e desce em ritmo rápido durante o voo (cada 0.12s alterna posição). Em IDLE, ritmo mais lento (0.3s).
- **"Squash & stretch"**: no momento do pulo, o pássaro estica levemente na vertical (scaleY 1.15, scaleX 0.9). Ao colidir, esmaga (scaleY 0.7, scaleX 1.3).

### 3.2 Canos

**Dimensões:**
- Largura do cano: 56px
- Gap vertical entre canos: 140px (ajustável)
- Distância horizontal entre pares: 220px
- Velocidade de movimento: 2.5 px/frame para a esquerda

**Visual:**
- Corpo: retângulo verde (`#5CBF2A`) com borda escura (`#3D8C20`).
- Capa (borda do cano): retângulo mais largo (62px) na extremidade, cor `#4CA922` com borda `#3D8C20`.
- Sombra: faixa vertical escura no lado direito do corpo para dar profundidade.
- Brilho: faixa vertical clara no lado esquerdo para efeito 3D.

**Geração:**
- Gap Y mínimo: 60px do topo (cano superior não fica muito curto).
- Gap Y máximo: canvas.height - groundHeight - 60px (cano inferior não fica muito curto).
- Posição Y do gap: aleatória entre os limites acima.
- Spawn: quando o último cano atinge X < (canvas.width - 220px), gera um novo par em X = canvas.width + 10px.

### 3.3 Chão

**Comportamento:**
- Altura fixa: 80px na parte inferior do canvas.
- Move para a esquerda na mesma velocidade dos canos (2.5 px/frame).
- Padrão repetitivo: listras diagonais ou losangos que criam sensação de rolagem.
- Quando um tile sai da tela, reposiciona no lado direito (loop infinito).

---

## FASE 4 — REGRAS DE COLISÃO

### 4.1 Hitbox do Pássaro

A hitbox é **menor que o sprite visual** para uma sensação de jogo mais justa:

| Parâmetro | Valor |
|-----------|-------|
| Hitbox largura | 28px (sprite tem 34px — margem de 3px cada lado) |
| Hitbox altura | 20px (sprite tem 24px — margem de 2px cada lado) |
| Offset X | +3px do centro do sprite |
| Offset Y | +2px do centro do sprite |

### 4.2 Algoritmo — AABB (Axis-Aligned Bounding Box)

```javascript
function aabb(rect1, rect2):
  return (rect1.x < rect2.x + rect2.w AND
          rect1.x + rect1.w > rect2.x AND
          rect1.y < rect2.y + rect2.h AND
          rect1.y + rect1.h > rect2.y)
```

### 4.3 Regras de Colisão

| Colisão | Condição | Resultado |
|---------|----------|-----------|
| Pássaro vs cano superior | AABB(hitbox pássaro, retângulo cano superior) | GAME_OVER |
| Pássaro vs cano inferior | AABB(hitbox pássaro, retângulo cano inferior) | GAME_OVER |
| Pássaro vs chão | `bird.y + bird.h >= canvas.height - groundHeight` | GAME_OVER |
| Pássaro vs teto | `bird.y <= 0` | GAME_OVER (não deixa sair do canvas) |

### 4.4 Comportamento no Game Over

- Ao detectar colisão: congela a pontuação, desativa a física de pulo.
- O pássaro **cai livremente** até o chão (gravidade continua, sem pulo).
- Ao atingir o chão: aplica efeito "squash" (esmaga visualmente).
- Overlay escuro (`rgba(0,0,0,0.45)`) aparece com **fade-in** de 0.3s.
- Placar de resultado aparece com **slide-down** de 0.4s.

---

## FASE 5 — REGRAS DE PONTUAÇÃO

### 5.1 Mecânica Básica

| Regra | Detalhe |
|-------|---------|
| Ganho de ponto | +1 quando o pássaro **passa completamente** um par de canos |
| Condição de disparo | `bird.x > pipe.x + pipe.width` E `pipe.scored === false` |
| Anti-duplicação | Após pontuar, marca `pipe.scored = true` — nunca conta o mesmo cano duas vezes |
| Pontuação mínima | 0 (início do jogo) |
| Pontuação máxima | Ilimitada |

### 5.2 Placar Durante o Jogo (PLAYING)

- Posição: topo central do canvas (x = canvas.width/2, y = 50).
- Fonte: Arial Black / Impact, 48px, bold.
- Cor: branco (`#FFFFFF`) com **stroke preto** de 3px para legibilidade.
- Atualização: em tempo real, sem delay.

### 5.3 Placar no Game Over

| Elemento | Detalhe |
|----------|---------|
| Painel | Retângulo arredondado amarelo claro (`#F8E87A`), borda marrom (`#8B6914`) |
| Posição | Centro do canvas, aparece com animação slide-down |
| Pontuação atual | Exibida em grande (fonte 36px) com label "Score" |
| Melhor pontuação | Exibida abaixo (fonte 28px) com label "Best" |
| Medalha | Pontuação ≥ 10 → bronze 🥉, ≥ 20 → prata 🥈, ≥ 40 → ouro 🥇, ≥ 80 → diamante 💎 |
| Botão de reinício | Texto "Tap to Restart" com animação de pulso (opacidade oscila) |

### 5.4 Persistência

- Melhor pontuação salva em `localStorage` (chave: `flappy_best_score`).
- Se `localStorage` não estiver disponível, usa variável em memória durante a sessão.

---

## FASE 6 — TELAS E UI AMIGÁVEL

### 6.1 Tela IDLE (Boas-vindas)

```
┌──────────────────────────┐
│                          │
│       🌤️  céu + nuvens    │
│                          │
│         [pássaro]        │
│      (flutuando suave)   │
│                          │
│                          │
│     ┌──────────────┐     │
│     │  FLAPPY BIRD  │     │
│     │   (título)    │     │
│     └──────────────┘     │
│                          │
│   "Clique ou pressione   │
│        SPACE para        │
│         jogar!"          │
│   (texto pulsante)       │
│                          │
│      🌿  chão + grama    │
└──────────────────────────┘
```

- **Título**: "FLAPPY BIRD" em fonte grossa, cor branca com sombra preta, posição central-superior.
- **Instrução**: texto "Clique ou SPACE para jogar" com animação de pulso (opacidade 0.5 → 1 → 0.5 a cada 2s).
- **Pássaro**: flutuando suavemente com `sin(time)`, asas animando devagar.
- **Cenário**: nuvens e cidade visíveis mas estáticos (chão não se move em IDLE).

### 6.2 Tela PLAYING

```
┌──────────────────────────┐
│         7                │  ← placar central topo
│                          │
│   [pássaro]    ||||      │  ← canos entrando
│              ↑ ||||      │
│                          │
│                          │
│                          │
│      ||||                │
│                          │
│      🌿  chão rolante    │
└──────────────────────────┘
```

- **Placar**: número branco com contorno preto, topo central.
- **Cenário**: tudo em movimento (nuvens, cidade, chão, canos).
- **Sem distrações**: apenas o placar sobre o jogo limpo.

### 6.3 Tela GAME_OVER

```
┌──────────────────────────┐
│         7                │
│   [pássaro caindo]       │
│                          │
│     ┌────────────────┐   │
│     │   GAME OVER    │   │  ← overlay com fade-in
│     │                │   │
│     │  Score:   7    │   │  ← slide-down
│     │  Best:    12   │   │
│     │     🥈        │   │  ← medalha (se aplicável)
│     │                │   │
│     │ Tap to Restart │   │  ← pulsante
│     └────────────────┘   │
│                          │
│      🌿  chão estático    │
└──────────────────────────┘
```

- **Overlay**: escurece o fundo com `rgba(0,0,0,0.45)`, fade-in de 0.3s.
- **Painel de resultado**: retângulo amarelo arredondado, slide-down de 0.4s com easing ease-out.
- **Medalha**: aparece com animação de "pop" (escala 0 → 1.2 → 1.0) se pontuação ≥ 10.
- **Botão restart**: texto pulsante (opacidade oscila a cada 1.5s).

### 6.4 Transições Suaves

| Transição | Efeito | Duração |
|-----------|--------|---------|
| IDLE → PLAYING | Pássaro faz um pulo inicial, cenário começa a mover | Instantâneo (no input) |
| PLAYING → GAME_OVER | Colisão detectada, pássaro congela e começa a cair | Gravidade continua até o chão |
| GAME_OVER → IDLE | Overlay some com fade-out, painel desaparece | 0.2s fade-out |

---

## FASE 7 — IMPLEMENTAÇÃO (Ordem Obrigatória)

Implementar **nesta ordem**. Cada passo deve estar funcional antes de prosseguir.

### Passo 1 — Estrutura HTML + CSS + Canvas
- `<canvas width="400" height="600">` com fallback text.
- CSS: centralizar na tela (`display:flex; justify-content:center; align-items:center; min-height:100vh`).
- Fundo da página: cor escura (`#2C3E50`) para contraste com o canvas.
- Escalamento responsivo: `max-width: 100vw; max-height: 100vh; image-rendering: pixelated`.

### Passo 2 — Configurações Globais
- Bloco de constantes no topo do script: gravidade, pulo, velocidade dos canos, dimensões, cores.
- Variáveis de estado: `currentState`, `score`, `bestScore`, `frames`.
- Inicialização de `bestScore` a partir de `localStorage`.

### Passo 3 — Sistema de Renderização (Loop + Delta Time)
- `requestAnimationFrame` com cálculo de delta time.
- Funções `update(dt)` e `draw()` separadas.
- Cap no delta time (máx 0.05s).
- Limpeza do canvas a cada frame (`clearRect`).

### Passo 4 — Céu + Cenário de Fundo
- Gradiente vertical para o céu.
- Nuvens: array de objetos `{x, y, scale, speed}`, desenhadas com elipses.
- Cidade/bosque: silhueta desenhada com retângulos sobrepostos em tom verde.
- Parallax: cada camada tem sua própria velocidade multiplicadora.

### Passo 5 — Chão Rolante
- Retângulo de grama (topo, 25px) + terra (resto, 55px).
- Padrão de listras/diagonais que se repetem a cada 24px.
- Movimento sincronizado com a velocidade dos canos.
- Loop infinito: quando um tile sai, reposiciona no direito.

### Passo 6 — Pássaro (Visual + Física)
- `draw()`: corpo elipse, barriga, asa animada, bico triângulo, olho com pupila.
- `update()`: gravidade → velocidade → posição → rotação.
- `jump()`: impulso vertical instantâneo.
- Flutuação idle: `sin(time * frequency)` com amplitude de 8px.
- Rotação dinâmica: mapear velocidade vertical para ângulo (-25° a +90°).
- Squash & stretch no pulo e na colisão.

### Passo 7 — Canos (Geração + Visual + Movimento)
- Array de canos com `{x, gapY, gapHeight, scored}`.
- `spawn()`: cria par quando o último atinge X < (width - spacing).
- `update()`: move para esquerda, remove os que saíram.
- `draw()`: corpo verde + capa mais larga + sombra + brilho.

### Passo 8 — Colisão (AABB)
- Hitbox reduzida (28×20px dentro do sprite 34×24px).
- Verifica: cano superior, cano inferior, chão, teto.
- On colisão: `currentState = GAME_OVER`, pássaro entra em queda livre.

### Passo 9 — Pontuação + Placar
- Incrementa quando `bird.x > pipe.x + pipe.w` e `!pipe.scored`.
- Placar PLAYING: texto branco com stroke preto, topo central.
- Melhor pontuação: salva no `localStorage` ao final do jogo.

### Passo 10 — Telas e UI (IDLE / GAME_OVER)
- IDLE: título + instrução pulsante + pássaro flutuando.
- GAME_OVER: overlay com fade-in, painel slide-down, medalha com pop, botão restart pulsante.
- Transições suaves entre estados.

### Passo 11 — Input (Mouse + Touch + Teclado)
- `keydown` → Space: previne scroll da página (`e.preventDefault()`).
- `mousedown` no canvas: pulo ou reinício conforme estado.
- `touchstart` no canvas: mesmo comportamento do mouse, com `preventDefault()`.
- Input mapeado por estado: IDLE→iniciar, PLAYING→pulo, GAME_OVER→reiniciar (com delay de 0.5s após colisão para evitar restart acidental).

### Passo 12 — Sons (Web Audio API — Opcional)
- Gerar sons via `AudioContext` (sem arquivos externos):
  - **Pulo**: beep curto ascendente (800→1200Hz, 0.1s).
  - **Pontuação**: beep agudo (1200Hz, 0.08s).
  - **Colisão**: ruído/buzz descendente (400→100Hz, 0.2s).
- Volume moderado, não intrusivo.

### Passo 13 — Partículas (Opcional mas Recomendado)
- Ao pontuar: burst de 5-8 partículas douradas que dissipam em 0.5s.
- Ao colidir: burst de 8-12 partículas brancas/cinza que dissipam em 0.4s.
- Partículas são objetos simples `{x, y, vx, vy, life, color, size}`.
- Remover partículas mortas a cada frame (garbage collection).

---

## FASE 8 — VALIDAÇÃO

Verificar **cada item** antes de considerar o jogo concluído:

### Funcionalidade
- [ ] Arquivo abre no navegador e exibe o canvas centralizado
- [ ] Tela IDLE mostra título, instrução pulsante e pássaro flutuando
- [ ] Space/Click/Toque inicia o jogo (IDLE → PLAYING)
- [ ] Pássaro responde ao pulo com impulso vertical imediato
- [ ] Gravidade puxa o pássaro para baixo de forma consistente
- [ ] Canos são gerados, movem para a esquerda e são removidos ao sair
- [ ] Gap entre canos é consistente (140px) e passável
- [ ] Colisão com canos → GAME_OVER imediato
- [ ] Colisão com chão/teto → GAME_OVER imediato
- [ ] Pássaro cai livremente após colisão até o chão

### Pontuação
- [ ] Pontuação incrementa +1 ao passar cada par de canos
- [ ] Mesmo cano não é contado duas vezes
- [ ] Placar é visível durante o jogo (topo central, branco com stroke preto)
- [ ] GAME_OVER exibe pontuação final e melhor pontuação
- [ ] Melhor pontuação persiste no localStorage entre recarregamentos
- [ ] Medalha aparece corretamente (bronze ≥10, prata ≥20, ouro ≥40, diamante ≥80)

### Visual e Fluidez
- [ ] Jogo roda a 60 FPS sem travamentos ou quedas de frame
- [ ] Pássaro rotaciona suavemente conforme velocidade (-25° subindo, +90° caindo)
- [ ] Asa do pássaro anima (sobe/desce) durante o voo
- [ ] Flutuação idle é suave (senoidal, 8px amplitude)
- [ ] Squash & stretch no pulo e na colisão
- [ ] Cenário de fundo com parallax (nuvens lentas, cidade média, chão rápido)
- [ ] Chão rola sincronizado com os canos
- [ ] Overlay GAME_OVER com fade-in suave
- [ ] Painel de resultado com slide-down animado
- [ ] Instrução em IDLE pulsa (opacidade oscila)

### Performance
- [ ] Sem erros no console do navegador
- [ ] Memória estável após 5+ minutos de jogo (sem vazamento)
- [ ] Canos e partículas removidos quando saem da tela
- [ ] Delta time calculado e capado (máx 0.05s)
- [ ] Funciona em Chrome, Firefox e Edge

### Responsividade
- [ ] Canvas centralizado na tela
- [ ] Escala corretamente em telas menores (mobile)
- [ ] Touch funciona em dispositivos móveis

---

## FASE 9 — ENTREGA

Produzir o arquivo HTML completo com:

- **Código organizado** em seções comentadas:
  ```
  // === CONFIGURAÇÃO ===
  // === ESTADO DO JOGO ===
  // === PÁSSARO ===
  // === CANOS ===
  // === CENÁRIO (céu, nuvens, cidade, chão) ===
  // === COLISÃO ===
  // === PONTUAÇÃO ===
  // === PARTÍCULAS ===
  // === UI / TELAS ===
  // === INPUT ===
  // === LOOP PRINCIPAL ===
  ```

- **Constantes no topo** para fácil ajuste (gravidade, velocidade, cores, dimensões).
- **Sem dependências externas** — tudo inline.
- **Pronto para execução**: abrir → jogar.

---

## ESTRATÉGIA ANTI-FALHA

1. **Implemente em camadas** — não pule passos. Cada passo depende do anterior estar funcional.
2. **Valide antes de avançar** — ao terminar um passo, verifique se funciona antes de ir ao próximo.
3. **Constantes no topo** — facilitar debug e balanceamento sem buscar valores no código.
4. **Delta time capado** — usar tempo entre frames para movimento consistente; cap em 0.05s para evitar saltos após perda de foco da aba.
5. **Hitbox menor que o sprite** — margem de 3px horizontal e 2px vertical para sensação de jogo mais justa.
6. **Garbage collection ativa** — remover canos, partículas e nuvens que saíram da tela a cada frame.
7. **requestAnimationFrame exclusivo** — nunca usar setInterval/setTimeout para o loop do jogo.
8. **Delay no restart** — 0.5s após colisão para evitar reinício acidental.
9. **Fallback localStorage** — se indisponível, usar variável em memória sem quebrar o jogo.
10. **Output único** — gerar o HTML completo de uma vez, não em partes parciais.
