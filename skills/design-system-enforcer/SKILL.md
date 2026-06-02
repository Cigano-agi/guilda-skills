---
name: design-system-enforcer
description: Enforces a project's design system across all UI work. Standardizes every screen using the project's design tokens (defined per project) + Minimals.cc-style layout patterns + Laws of UX. TRIGGER on UI work (React/Next/Vue/Tailwind/HTML/CSS), creation of components, dashboards, forms, pages, tables, modals. Also trigger on "padronizar design", "aplicar design system", "auditar UI", "/design-system-enforcer".
---

# Design System Enforcer

> **Missão:** qualquer tela de qualquer projeto sai com a mesma identidade, sem retrabalho. Menos decisão, mais consistência.
> **Inspirações:** [Minimals.cc](https://docs.minimals.cc/introduction/) + [Laws of UX](https://lawsofux.com/).
> **Lema:** restrição é luxo. Cada pixel tem propósito. Psicologia antes de estética.

A skill não impõe uma paleta fixa. Ela impõe um **método**: ler os tokens do projeto, aplicá-los sem exceção, e auditar pelas Laws of UX antes de entregar. A paleta, fontes e marca vêm do projeto (definidos em um arquivo de tokens local ou no `tailwind.config`/`@theme`).

---

## Ativação

Dispare esta skill sempre que:

- Criar/editar componentes React, Next, Vue, HTML/CSS, Tailwind
- Montar dashboard, formulário, landing, card, tabela, modal, menu
- Escolher cor, tipografia, espaçamento ou motion
- Usuário disser "padroniza", "aplica o design system", "audita a UI", "melhora a UX"

Sem dispensa. Se a tarefa toca pixel, a skill entra.

---

## Regras invioláveis (top 10)

1. **Zero cor hardcoded** — sempre tokens (`var(--primary)`, `text-primary`, `bg-surface`). Banido: `#fff`, `rgb(...)`, `gray-500` soltos.
2. **Paleta = a do projeto** — use só os tokens de marca definidos no projeto. Nada de cor fora da paleta exceto estados semânticos (`success`/`warning`/`danger`/`info`).
3. **Tipografia = a escala do projeto** — fonte e escala modular vêm dos tokens. Sem inventar `font-size`.
4. **Espaçamento = múltiplos de 4px** — Tailwind `p-1 p-2 p-3 p-4 p-6 p-8`.
5. **Radius padronizado** — escala única de tokens (ex.: `6px` inputs, `8px` cards, `12px` modais, `999px` pills). Nunca `rounded-[7px]` solto.
6. **Sombra contida** — no máximo 2-3 níveis (`shadow-sm`, `shadow-md`, `shadow-lg`). Nada de neon/glow fora de hover states.
7. **Motion discreto** — `ease-out`, 120-320ms. Nunca bouncy/elastic em UI de produto.
8. **Acessibilidade não é opcional** — contraste ≥ 4.5:1 texto normal, 3:1 UI chrome. Focus ring visível sempre.
9. **Mobile-first** — toda tela funciona em 360px antes de largar pra desktop.
10. **Theming consistente** — light e dark são variantes do mesmo set de role-tokens (troca por `[data-theme]` ou `.dark`), nunca cores duplicadas hardcoded.

---

## Workflow

Quando acionada, execute em ordem.

### 1. Detect
- Existe um set de tokens (`:root { --primary: ... }`, `@theme`, ou `tailwind.config` extends)? Se não → criar a partir da paleta do projeto.
- Há cor hardcoded no código? Grepar `#[0-9a-f]{3,8}`, `rgb(`, `hsl(` — mapear pros tokens.

### 2. Apply
- Substituir cores → tokens.
- Normalizar spacing (4px base), radius, shadow, motion.
- Aplicar layout Minimals se for dashboard/app (sidebar + topbar + content area).

### 3. Audit
- Rode as **Laws of UX** (abaixo) como checklist → marcar ✅/❌ em cada.
- Consertar falhas antes de entregar.

### 4. Report
- Diff resumido: o que mudou + qual lei corrigiu.
- Screenshot sugerido se via browser.

**Tom ao aplicar:** não peça permissão item-a-item; aplique tudo e relate no fim. Só pergunte em decisões arquiteturais grandes (trocar framework, reescrever página do zero).

---

## Arquitetura de tokens (estrutura, valores vêm do projeto)

Todo projeto começa com um set de **role-tokens** semânticos. Os valores brutos (hex/HSL da marca) ficam atrás dos tokens de papel — a UI nunca toca o valor bruto.

```css
:root {
  /* 1. Brand palette — defina os valores do SEU projeto aqui */
  --brand-500: /* cor primária da marca */;
  --brand-600: /* hover da primária */;

  /* 2. Grey scale neutra */
  --grey-50; --grey-100; --grey-200; --grey-300; --grey-400;
  --grey-500; --grey-600; --grey-700; --grey-800; --grey-900;

  /* 3. Semantic */
  --success; --warning; --danger; --info;

  /* 4. Role tokens — a UI usa SÓ estes */
  --bg; --surface; --surface-2; --surface-hover;
  --border; --border-subtle;
  --fg; --fg-muted; --fg-subtle;
  --primary; --primary-hover; --primary-fg; --primary-soft;
  --accent; --ring;

  /* 5. Shadows / radius / motion */
  --shadow-sm; --shadow-md; --shadow-lg;
  --r-sm: 6px; --r: 8px; --r-lg: 12px; --r-xl: 16px; --r-pill: 999px;
  --ease-out: cubic-bezier(0.22, 1, 0.36, 1);
  --duration-fast: 120ms; --duration-base: 200ms; --duration-slow: 320ms;
}

/* Dark é variante: redefine só os role-tokens, mesma marca */
[data-theme="dark"] {
  --bg; --surface; --surface-2; --surface-hover;
  --border; --fg; --fg-muted; --fg-subtle; --primary-soft;
  --shadow-sm; --shadow-md; --shadow-lg;
}
```

**Tailwind v3** — mapeie os mesmos role-tokens em `theme.extend.colors` via `var(--*)`; declare `borderRadius`, `boxShadow`, `fontFamily` a partir dos tokens.
**Tailwind v4** — declare os tokens dentro de um bloco `@theme { --color-primary: ...; --radius: 8px; ... }` no CSS global.

**Tipografia (escala modular típica):** `xs 12` · `sm 13` · `base 14` · `md 15` · `lg 18` · `xl 22` · `2xl 28` · `3xl 36` (px), pesos 400-800, line-height confortável. A fonte concreta vem do projeto.

**Espaçamento (4px base):** `1:4 · 2:8 · 3:12 · 4:16 · 6:24 · 8:32 · 12:48 · 16:64`.

**Motion:** hover/focus = `fast` + `ease-out`; modal/drawer = `slow` entrada / `base` saída; proibido bounce; `prefers-reduced-motion: reduce` → só fade.

**Ícones:** uma única lib por projeto (ex.: Lucide), `stroke-width` consistente, tamanhos `16/20/24`. Nunca emoji como UI chrome.

---

## Enforcement — auditar e refatorar UI existente

Aplicar retroativamente em projeto que já tem código:

**1. Scan** (de dentro do projeto alvo):

```bash
# Cores hardcoded (fora de tokens)
grep -rEn "#[0-9a-fA-F]{3,8}|rgba?\(|hsla?\(" src/ \
  --include="*.{ts,tsx,jsx,vue,css,scss,html}" | grep -vE "var\(--|theme\(|@theme"

# Font-family inline, radius soltos, !important, inline styles
grep -rEn "font-family:" src/ | grep -v "var(--"
grep -rEn "rounded-\[|border-radius:\s*[0-9]+px" src/
grep -rEn "!important" src/ --include="*.{css,scss}"
grep -rEn 'style=\{\{|style="' src/ --include="*.{tsx,jsx}"
```

**2. Injetar tokens** no CSS global (`globals.css`/`index.css`) e no `tailwind.config`/`@theme`. Carregar as fontes do projeto.

**3. Substituição em lote** (com revisão humana, não cega):

| Antes | Depois |
|---|---|
| `#fff`, `white` | `var(--surface)` / `text-fg` |
| `#000`, `black` | `var(--bg)` |
| `gray-50..900` | role-tokens neutros |
| `blue-500..700` (ou cor de marca solta) | `primary` |
| `bg-gray-100` | `bg-surface` |
| `border-gray-200` | `border-border` |
| `rounded-md` | `rounded` (token) |
| `shadow-xl` | `shadow-md` (máximo controlado) |

**4. Auditar** pelas Laws of UX (abaixo).

**5. Verificação final:** `grep` de cor hardcoded deve voltar 0; dark mode togglável; axe-core/Lighthouse com 0 erros de contraste; animações param com reduce-motion ON.

Commit sugerido:

```
style(design-system): apply design tokens + Laws of UX audit

- Replace N hardcoded colors with design tokens
- Normalize spacing to 4px base, standardize radius/shadow
- Fix contrast on X elements (was 3.2:1, now 5.8:1)
- Add focus rings to N interactive elements
- Apply Law of Proximity / Hick's Law where flagged
```

---

## Minimals.cc — padrões de layout

**Shell de app (dashboard/SaaS):** TopBar sticky (~56px: logo, nav, user menu) + Sidebar (~280px) + Content area (`max-w-7xl mx-auto`, padding 24px). Sidebar colapsa pra ~64px (só ícones) em `lg→xl` e vira drawer off-canvas em `< md`. Content com scroll próprio. Landing/marketing usa só os tokens (sem shell); form standalone = card centralizado.

**Dashboard card:** header com label pequeno (`text-sm text-fg-muted`) + métrica grande (`text-2xl font-bold`); ícone circular colorido pela semântica no canto; footer com delta + contexto comparativo.

**Data table:** zebra OFF, usar `hover:bg-surface-2`. Alinhamento: texto à esquerda, números à direita, status ao centro. Sticky header. Selection ≥1 → bulk-action bar ancorada. Row click → detail drawer lateral (não navega embora).

**Form:** label **acima** do input (mais escaneável, não flutuante). Required = asterisco discreto. Error text embaixo em `text-danger text-xs`; hint em `text-fg-muted text-xs`. Máx 2 colunas desktop / 1 mobile. Actions à direita: `Cancel` (ghost) à esquerda do `Primary` (cheio), separadas por `border-t`.

**Empty state:** nunca página em branco. Sempre ícone + título + explicação + CTA (só se houver ação óbvia).

**Modal vs Drawer:** confirm destrutivo / info breve → **Modal** (center, `max-w-480px`, fecha em Esc + click fora). Form médio (≥4 campos) / detail da row → **Drawer** (right-side, `480px` desktop / `100%` mobile, slide-in).

**Toast:** top-right desktop / top-center mobile. Auto-dismiss 4s success / 7s error / nunca action-required. Máx 3 stacked, resto vira fila. Copy humano ("Salvo ✓" > "Operação bem-sucedida").

**Sidebar nav:** grupos com label UPPERCASE pequena (`text-xs text-fg-subtle`). Item ativo `bg-primary/10 text-primary`, hover inativo `bg-surface-2`. Ícone 20px + label 14px, `gap-3 px-3 py-2 rounded`.

**Breadcrumb:** só em telas de detalhe (profundidade ≥2). Root clicável; atual sem link.

**Skeleton:** sempre em listas, cards de métrica e dashboard pré-render. `bg-surface-2` + `animate-pulse`, ~1.2s. Nunca spinner isolado.

**Breakpoints (Tailwind):** `sm 640 · md 768 · lg 1024 · xl 1280 · 2xl 1536`. Regra: toda tela testada em 360px antes de considerar pronta.

---

## Laws of UX — checklist obrigatório

Fonte: [lawsofux.com](https://lawsofux.com/). Para cada tela criada/alterada, passe pelas 12 leis, marque ✅/❌, conserte, entregue.

1. **Jakob's Law** — usuários esperam que funcione como os outros. Nav no topo/lateral; logo clicável → home; ícones universais (lixeira = deletar); primary action é botão cheio no canto esperado. *Analogia: porta de banheiro com ícone padrão, não um enigma.*
2. **Fitts's Law** — alvo fácil = grande e perto. Botões ≥ 44×44px (mobile) / 36×32px mín (desktop); CTAs perto da ação recém-feita; canto/borda têm área infinita (menus globais, logo). *Analogia: controle de TV — botões mais usados são os maiores.*
3. **Hick's Law** — mais opções = decisão mais lenta. Menu ≤ 7 itens; corta campos opcionais; 1 primary CTA por tela (máx 2); filtros raros atrás de "Avançado". *Analogia: cardápio de 2 páginas > de 20.*
4. **Miller's Law** — memória curta = 7±2. Listas segmentadas em grupos ≤ 5; números longos formatados; stepper mostra posição ("2 de 5").
5. **Tesler's Law** — complexidade não some, só muda de lado. Valor padrão inteligente em todo campo; tolerar formatos colados; validação no sistema, não no usuário. *Analogia: pré-assar a pizza em vez do cliente montar.*
6. **Doherty Threshold** — feedback < 400ms. Click → resposta visual ≤ 100ms; operação > 1s → skeleton/progress (nunca spinner vazio); > 10s → progress com estimativa.
7. **Aesthetic-Usability Effect** — bonito parece mais funcional. Espaçamento generoso; alinhamento rigoroso (grid, sem 1px torto); hierarquia visual clara.
8. **Von Restorff** — o diferente é lembrado. Primary CTA destoa (cor de marca cheia); estado atual destacado; **mas só 1 coisa destoa por tela**.
9. **Peak-End Rule** — pico + fim definem a memória. Primeiro valor entregue bem tratado; sucesso/erro com copy humano e próxima ação clara ("Pronto! Relatório enviado.").
10. **Zeigarnik Effect** — tarefa incompleta fica na cabeça. Auto-save em forms longos; indicador de rascunhos; onboarding com checklist visível.
11. **Serial Position Effect** — primeiro e último são lembrados. Item mais importante na 1ª/última posição; listas por relevância; CTA primário no fim (direita) ou início, nunca no meio.
12. **Law of Proximity** — coisas juntas parecem relacionadas. Label colado no campo; grupos com divisor visível; hint imediatamente abaixo do campo.

**Acessibilidade (inegociável):** contraste ≥ 4.5:1 (normal) / 3:1 (grande); focus ring visível em todo interativo; `alt` correto; keyboard-only funciona (Tab/Enter/Esc/setas); `aria-label` em botão-ícone sem texto; respeita `prefers-reduced-motion`.

**Resultado do audit** — tabela `Lei | Status | Nota` + fix list numerada e acionável.

---

## Anti-patterns (reprova automaticamente)

- Cor fora da paleta do projeto sem justificativa semântica
- Fonte fora da escala definida (exceto code → mono)
- Border-radius solto (`rounded-[7px]`)
- `!important` (exceto override de lib terceira, comentado)
- Inline style (exceto gradients dinâmicos)
- Ícones de libs diferentes na mesma tela
- Emoji como UI chrome (ícones de ação/botões) — só decorativo se o user pedir
