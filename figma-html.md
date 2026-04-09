---
name: figma-html
description: >
  Use este skill para QUALQUER geração de HTML — slides, documentos de discovery,
  propostas de redesign, layouts de referência, ou qualquer outro HTML que será
  importado no Figma via html.to.figma. O objetivo é que o HTML renderize
  corretamente no browser E produza uma estrutura de nós limpa, editável e
  organizada no Figma. Este skill substitui o frontend-design para contextos
  onde o destino final é o Figma.
---

# HTML otimizado para importação no Figma

Todo HTML gerado por este skill tem dois destinos simultâneos: browser e Figma.
O browser exige que o visual funcione. O Figma exige que a estrutura de nós seja
limpa, hierárquica e editável. Quando os dois entram em conflito, o Figma vence —
porque é lá que o trabalho de edição acontece.

---

## Por que HTML comum quebra no Figma

O plugin html.to.figma converte cada elemento DOM em um nó do Figma. Problemas
típicos que surgem com HTML gerado sem essa preocupação:

- **Texto fragmentado:** divs separadas para título e corpo viram dois nós de
  texto no Figma — impossível editar como bloco contínuo
- **Frames fantasma:** divs de wrapper decorativo (só para padding ou border)
  viram frames vazios aninhados sem função no Figma
- **Auto layout quebrado:** flex aninhado em vários níveis produz auto layout
  conflitante — o Figma não consegue inferir a direção correta
- **Pseudo-elementos ignorados:** ::before e ::after não são importados — bordas
  decorativas e ícones via pseudo simplesmente desaparecem
- **Position absolute desancorada:** elementos absolutos viram nós fora do fluxo,
  difíceis de editar e realinhar no Figma

---

## Regra central: hierarquia máxima de 3 níveis por componente

```
slide (nível 1)
  └── card (nível 2)
        └── card-head + card-body + chip (nível 3)
```

Nunca ultrapassar 3 níveis dentro de um componente. Se precisar de mais, o
componente deve ser dividido.

---

## Texto

### Consolidar blocos relacionados

```html
<!-- ERRADO: dois nós de texto no Figma, wrapper desnecessário -->
<div class="heading-group">
  <h2 class="title">Título principal</h2>
  <p class="subtitle">Subtítulo explicativo</p>
</div>

<!-- CERTO opção A: irmãos diretos sem wrapper -->
<h2 class="title">Título principal</h2>
<p class="subtitle">Subtítulo explicativo</p>

<!-- CERTO opção B: consolidado com white-space:pre-line -->
<p class="heading" style="white-space:pre-line">Título principal
Subtítulo explicativo</p>
```

### Sem spans puramente decorativos dentro de parágrafos

Usar `<b>` e `<em>` inline apenas quando a distinção é essencial para o
conteúdo. Nunca para estilo puro — fragmenta o nó de texto no Figma.

---

## Layout

### Gap em vez de margin entre irmãos

`gap` em flex/grid é importado como espaçamento de auto layout no Figma.
`margin` entre irmãos cria spacing manual que se perde.

```css
/* CERTO: gap no pai */
.card { display:flex; flex-direction:column; gap:8px; }

/* ERRADO: margin nos filhos */
.card-body { margin-top:8px; }
```

### Um nível de flex por contexto

Flex aninhado em mais de 2 níveis consecutivos quebra o auto layout no Figma.

```css
/* CERTO */
.card      { display:flex; flex-direction:column; gap:8px; }
.card-head { display:flex; align-items:center; gap:8px; }

/* ERRADO: três níveis de flex consecutivos */
.card       { display:flex; }
.card-inner { display:flex; }
.card-text  { display:flex; }
```

### Tamanhos explícitos no container raiz

```css
/* Slides 16:9 */
.slide { width:1152px; height:648px; }

/* Documentos */
.doc-frame { width:1120px; min-height:800px; }
```

### Position absolute só para overlays reais

Usar apenas para badges, numeração de slide ou anotações sobrepostas. Nunca
para posicionar elementos que poderiam estar no fluxo normal.

---

## Bordas e decoração

### Border-left como box-shadow inset — nunca como wrapper

```css
/* ERRADO: wrapper extra que vira frame vazio no Figma */
.callout-wrapper { display:flex; }
.callout-bar     { width:3px; background:#7c5cfc; }
.callout-body    { padding:10px 14px; }

/* CERTO: box-shadow inset no próprio elemento */
.callout {
  padding:10px 14px;
  box-shadow:inset 3px 0 0 #7c5cfc;
  background:rgba(124,92,252,0.12);
  border-radius:0 6px 6px 0;
}
```

### Sem pseudo-elementos estruturais

`::before` e `::after` não são importados. Nunca usar para ícones, bullets ou
numeração. Usar elementos reais no DOM.

```html
<!-- ERRADO: desaparece no Figma -->
<style>.step::before { content:'01'; }</style>
<div class="step">Conteúdo</div>

<!-- CERTO: elemento real -->
<div class="step">
  <span class="step-num">01</span>
  <p>Conteúdo</p>
</div>
```

### Dividers como border no elemento pai

```css
/* CERTO: sem elemento extra */
.section-header {
  border-bottom:1px solid rgba(255,255,255,0.06);
  padding-bottom:12px;
}
```

---

## Padrões de componente

### Card

```html
<div class="card">
  <div class="card-head">               <!-- flex row: ícone + título -->
    <div class="card-icon">🔎</div>
    <span class="card-title">Título</span>
  </div>
  <p class="card-body">Descrição...</p>
  <span class="chip">Label</span>
</div>
```

```css
.card      { display:flex; flex-direction:column; gap:8px; padding:16px; border-radius:8px; }
.card-head { display:flex; align-items:center; gap:8px; }
.card-icon { width:32px; height:32px; border-radius:50%; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
```

### Callout / highlight bar

```html
<div class="callout">
  <b>Label:</b> Texto do callout em um único parágrafo.
</div>
```

```css
.callout {
  padding:8px 16px;
  box-shadow:inset 3px 0 0 var(--ac);
  background:var(--s2);
  border-radius:0 6px 6px 0;
  font-size:13px;
  color:var(--t2);
  line-height:1.55;
}
```

### Step numerado

```html
<div class="step">
  <span class="step-num">1</span>
  <div class="step-content">
    <p class="step-title">Título do passo</p>
    <p class="step-body">Descrição.</p>
  </div>
</div>
```

### Chips

```html
<!-- Chip único: inline, sem wrapper -->
<span class="chip chip-ac">Label</span>

<!-- Múltiplos chips: um wrapper direto -->
<div class="chips">
  <span class="chip chip-gn">DADO</span>
  <span class="chip chip-am">ESTIMADO</span>
</div>
```

```css
.chips { display:flex; gap:4px; flex-wrap:wrap; }
.chip  { display:inline-flex; align-items:center; font-size:10px; font-weight:500; padding:4px 8px; border-radius:20px; white-space:nowrap; }
```

### Tabela de dados

Usar `<table>` nativo — o plugin importa melhor que divs simulando grid.

```html
<table class="data-table">
  <thead><tr><th>Coluna A</th><th>Coluna B</th></tr></thead>
  <tbody><tr><td>Dado</td><td>Valor</td></tr></tbody>
</table>
```

---

## Escala de espaçamento — múltiplos de 8

Todo espaçamento no HTML deve seguir a escala de 8px. Isso se aplica a `gap`,
`padding`, `margin` e qualquer valor de distância entre elementos.

### Escala aprovada

| Token    | Valor | Uso típico                                      |
|----------|-------|-------------------------------------------------|
| `--sp-1` | 8px   | Espaço mínimo entre elementos relacionados      |
| `--sp-2` | 16px  | Gap entre filhos de um card, padding interno    |
| `--sp-3` | 24px  | Gap entre cards, padding de seção               |
| `--sp-4` | 32px  | Separação entre blocos distintos                |
| `--sp-6` | 48px  | Gap entre slides, espaçamento de página         |
| `--sp-8` | 64px  | Padding de frame principal                      |

```css
:root {
  --sp-1: 8px;
  --sp-2: 16px;
  --sp-3: 24px;
  --sp-4: 32px;
  --sp-6: 48px;
  --sp-8: 64px;
}
```

### Exceção permitida: 4px

O valor de 4px (metade de 8) é aceito **somente** para micro-espaçamentos
entre elementos inline do mesmo grupo — gap entre chips, gap entre badges,
gap entre tags. É a única exceção à regra de múltiplos de 8.

```css
/* CORRETO: 4px só entre chips/badges inline */
.chips { gap: 4px; }

/* ERRADO: 4px em padding ou gap de container */
.card  { gap: 4px; }       /* usar 8px */
.card  { padding: 4px; }   /* usar 8px */
```

### Valores proibidos

Qualquer valor que não seja múltiplo de 8 (exceto 4px no caso acima) é proibido.

```css
/* PROIBIDO */
gap: 10px;     /* → usar 8px ou 16px */
gap: 12px;     /* → usar 8px ou 16px */
padding: 9px;  /* → usar 8px */
padding: 14px; /* → usar 16px */
padding: 18px; /* → usar 16px ou 24px */
padding: 20px; /* → usar 16px ou 24px */
```

### Mapeamento dos componentes para a escala

| Componente       | Propriedade          | Antes   | Correto |
|------------------|----------------------|---------|---------|
| `.card`          | `gap`                | 10px    | 8px     |
| `.card`          | `padding`            | 18px 20px | 16px 16px |
| `.csm`           | `padding`            | 12px 14px | 8px 16px |
| `.callout`       | `padding`            | 9px 13px  | 8px 16px |
| `.step`          | `padding-top/bottom` | 10px    | 8px     |
| `.step`          | `gap`                | 14px    | 16px    |
| `.g2` / `.cards` | `gap`                | 12px    | 16px    |
| `.g3` / `.g4`    | `gap`                | 10px    | 8px     |
| `.chip`          | `padding`            | 3px 10px  | 4px 8px |
| `.chips`         | `gap`                | 4px     | 4px ✓   |
| `.anno`          | `padding-top/bottom` | 8px     | 8px ✓   |
| `.bench-label`   | `padding`            | 8px 10px  | 8px     |

### CSS dos componentes com escala correta

```css
.card      { gap:8px;  padding:16px; }
.csm       { gap:8px;  padding:8px 16px; }
.callout   { padding:8px 16px; }
.step      { gap:16px; padding:8px 0; }
.g2, .cards { gap:16px; }
.g3        { gap:8px; }
.chip      { padding:4px 8px; }
.chips     { gap:4px; }   /* exceção permitida */
.slide-frame { padding:48px 64px; }
```

---

## Design tokens — paleta padrão

```css
:root{
  /* Superfícies */
  --bg:#0c0c0e; --s1:#141416; --s2:#1c1c1f; --s3:#242428;
  /* Bordas */
  --b1:rgba(255,255,255,0.06); --b2:rgba(255,255,255,0.11); --b3:rgba(255,255,255,0.18);
  /* Texto */
  --t1:#f0efe8; --t2:#9a9890; --t3:#5a5956;
  /* Accent — roxo */
  --ac:#7c5cfc; --ab:rgba(124,92,252,0.15); --ab2:rgba(124,92,252,0.30);
  /* Success — verde */
  --gn:#29c76f; --gb:rgba(41,199,111,0.12);
  /* Warning — âmbar */
  --am:#f5a42a; --amb:rgba(245,164,42,0.12);
  /* Danger — vermelho */
  --re:#ef4f4f; --rb:rgba(239,79,79,0.12);
  /* Info — azul */
  --bl:#4ba3d9; --blb:rgba(75,163,217,0.12);
}
```

---

## Tipografia

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

/* Inter para texto, JetBrains Mono para labels/código/tags */
.eyebrow { font-size:10px; font-weight:600; letter-spacing:.09em; text-transform:uppercase; }
.h1      { font-size:52px; font-weight:300; letter-spacing:-.028em; line-height:1.08; }
.h2      { font-size:30px; font-weight:300; letter-spacing:-.02em;  line-height:1.2; }
.h3      { font-size:17px; font-weight:600; letter-spacing:-.01em;  line-height:1.3; }
.body    { font-size:14px; color:var(--t2); line-height:1.65; }
.bsm     { font-size:12px; color:var(--t2); line-height:1.55; }
.mono    { font-family:'JetBrains Mono',monospace; font-size:10px; letter-spacing:.06em; text-transform:uppercase; }
```

---

## Checklist antes de entregar

**Estrutura**
- [ ] Nenhum componente tem mais de 3 níveis de aninhamento
- [ ] Flex não está aninhado em mais de 2 níveis consecutivos
- [ ] Espaçamento entre irmãos usa `gap` no pai, não `margin` nos filhos
- [ ] Blocos de texto relacionados são irmãos diretos ou consolidados com white-space:pre-line
- [ ] Barras laterais decorativas usam `box-shadow:inset`, não wrappers
- [ ] Nenhum `::before` / `::after` carrega informação visual essencial
- [ ] Container raiz tem `width` e `height` explícitos
- [ ] `position:absolute` usado só para overlays genuínos
- [ ] Chips são elementos inline simples sem wrapper intermediário
- [ ] Imagens embedadas em base64 (não URL externa) quando precisam ser importadas

**Espaçamento — escala de 8px**
- [ ] Todo `gap` é múltiplo de 8 (exceção: 4px só entre chips/badges inline)
- [ ] Todo `padding` é múltiplo de 8
- [ ] Nenhum valor proibido: 10px, 12px, 9px, 14px, 18px, 20px
- [ ] Variáveis `--sp-1` a `--sp-8` usadas em vez de valores hardcoded quando possível

---

## O que este skill não cobre

- Interatividade com JavaScript — o Figma importa só estrutura e estilo estático
- Responsividade — HTMLs são largura fixa dimensionados para o Figma
- Acessibilidade WCAG — destino é Figma, não produção

---

## Nota de uso

Aplicar este skill em qualquer pedido de HTML cujo destino seja importação no Figma
— slides, documentos, propostas de tela, layouts de referência. Para HTML destinado
a produção web (landing pages, componentes de produto, apps), usar o skill
`frontend-design` em vez deste. A estrutura achatada é o padrão, não a exceção.

---

## Contexto de tela de produto

Quando o HTML representa uma tela de produto — app screen, componente de UI, referência de
redesign para plataformas web ou mobile — aplicar as seguintes restrições visuais adicionais:

- Hierarquia construída por escala tipográfica e peso. Cor não diferencia seções nem categorias.
- `--ac` (e equivalentes de acento) usados com parcimônia: ação primária, estado selecionado,
  ênfase pontual. Nunca como preenchimento de área ou elemento decorativo recorrente.
- Gradientes em superfícies de conteúdo são proibidos. A escala `--bg → --s1 → --s2 → --s3`
  é suficiente para profundidade.
- Bordas decorativas substituídas por espaçamento. Quando bordas forem necessárias, usar
  `rgba` de baixa opacidade (`--b1` ou `--b2`), nunca stroke sólido.
- Sombras apenas para elevação real — `box-shadow: 0 1px 3px rgba(0,0,0,0.10)` como valor
  máximo para cards planos. Sombras expressivas ficam reservadas para modais e drawers.

Estas restrições **não se aplicam** a slides, documentos de discovery ou apresentações, onde
a paleta padrão e os gradientes existentes são intencionais e apropriados para o contexto.
