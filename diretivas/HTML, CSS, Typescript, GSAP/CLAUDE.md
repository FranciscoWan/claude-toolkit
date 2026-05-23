# CLAUDE.md — Projeto Frontend: Landing Page Fintech PIX

## Contexto do Projeto

Landing page de uma fintech fictícia de pagamentos PIX.
Foco em design premium, animações profissionais e código limpo.
Sem framework JavaScript — HTML, CSS e TypeScript puros.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Markup | HTML5 semântico |
| Estilização | CSS puro com custom properties |
| Lógica / Tipos | TypeScript (compilado via `tsc` ou Vite) |
| Animações | GSAP 3 + ScrollTrigger + Observer |
| Ícones | Lucide Icons (via CDN ou SVG inline) |
| Gráficos | Canvas API ou Chart.js puro |
| QR Code | `qrcodejs` (vanilla) |
| Fontes | Inter + Space Grotesk (Google Fonts) |
| Bundler | Vite (dev server + build) |
| Deploy | GitHub Pages ou Vercel (static) |

---

## Estrutura de Pastas

```
/
├── index.html               ← entrada única
├── tsconfig.json
├── vite.config.ts
├── src/
│   ├── main.ts              ← inicializa tudo
│   ├── styles/
│   │   ├── reset.css
│   │   ├── tokens.css       ← custom properties globais
│   │   ├── typography.css
│   │   └── sections/        ← um .css por section
│   ├── sections/            ← um .ts por section
│   │   ├── hero.ts
│   │   ├── how-it-works.ts
│   │   ├── dashboard.ts
│   │   ├── benefits.ts
│   │   ├── logos.ts
│   │   ├── qr-simulation.ts
│   │   ├── stats.ts
│   │   ├── security.ts
│   │   ├── testimonials.ts
│   │   ├── pricing.ts
│   │   ├── final-cta.ts
│   │   └── footer.ts
│   ├── animations/          ← utilitários GSAP reutilizáveis
│   │   ├── scroll-reveal.ts
│   │   ├── count-up.ts
│   │   ├── floating.ts
│   │   └── marquee.ts
│   └── lib/
│       ├── gsap.ts          ← registra plugins GSAP
│       └── qr.ts            ← wrapper do QR Code
└── public/
    └── assets/
```

---

## Convenções de Código

### HTML
- Usar tags semânticas: `<section>`, `<nav>`, `<article>`, `<header>`, `<footer>`
- Cada section tem `id` correspondente ao nome: `id="hero"`, `id="dashboard"`
- Atributos `data-*` para targets GSAP: `data-animate`, `data-stagger`
- Não usar `div` quando existe tag semântica adequada

### CSS
- Todas as cores, espaçamentos e tamanhos em custom properties em `tokens.css`
- Naming em kebab-case: `.hero__title`, `.card--featured`
- Metodologia BEM para componentes reutilizáveis
- Nenhuma cor ou tamanho hardcoded fora de `tokens.css`
- Media queries mobile-first
- Usar `clamp()` para tipografia fluida

```css
/* tokens.css — padrão obrigatório */
:root {
  --color-primary: #2563eb;
  --color-accent: #10b981;
  --color-bg: #0a0a0f;
  --color-surface: #111118;
  --color-text: #f1f5f9;
  --color-muted: #64748b;
  --glow-blue: 0 0 40px rgba(37, 99, 235, 0.4);
  --glow-green: 0 0 40px rgba(16, 185, 129, 0.4);
  --radius-card: 16px;
  --transition-base: 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
```

### TypeScript
- Cada section é uma função exportada: `export function initHero(): void`
- `main.ts` importa e chama todas no `DOMContentLoaded`
- Tipagem explícita — sem `any`
- Usar `querySelector` com cast: `document.querySelector<HTMLElement>('.hero')`
- Guardar referências DOM no topo da função, antes de qualquer lógica

### GSAP
- Registrar plugins uma única vez em `src/lib/gsap.ts`:
  ```ts
  import { gsap } from 'gsap'
  import { ScrollTrigger } from 'gsap/ScrollTrigger'
  gsap.registerPlugin(ScrollTrigger)
  export { gsap, ScrollTrigger }
  ```
- Importar sempre de `src/lib/gsap.ts`, nunca direto do pacote
- Todo ScrollTrigger usa `trigger` explícito — nunca confiar em defaults
- Animações de entrada usam `gsap.from`, não `gsap.to` com estado inicial no CSS
- Cleanup obrigatório: guardar instâncias ScrollTrigger para `kill()` se necessário

---

## Padrões de Animação

### Scroll Reveal padrão
```ts
gsap.from(elements, {
  y: 60,
  opacity: 0,
  duration: 0.8,
  stagger: 0.12,
  ease: 'power3.out',
  scrollTrigger: {
    trigger: container,
    start: 'top 80%',
  },
})
```

### Floating loop
```ts
gsap.to(element, {
  y: -12,
  duration: 2.5,
  ease: 'sine.inOut',
  yoyo: true,
  repeat: -1,
})
```

### CountUp
```ts
const obj = { val: 0 }
gsap.to(obj, {
  val: targetValue,
  duration: 2,
  ease: 'power2.out',
  onUpdate: () => {
    el.textContent = Math.round(obj.val).toLocaleString('pt-BR')
  },
  scrollTrigger: { trigger: el, start: 'top 85%' },
})
```

---

## Regras de Design

- Fundo sempre escuro: `#0a0a0f` base, `#111118` para superfícies
- Hierarquia de cor: azul primário → verde acento → branco texto
- Cards com `border: 1px solid rgba(255,255,255,0.06)` e `backdrop-filter: blur()`
- Glow apenas em elementos interativos e destaques — não abusar
- Espaçamento generoso: seções com `padding-block: clamp(80px, 12vw, 160px)`
- Tipografia: headlines em `Space Grotesk`, corpo em `Inter`

---

## O que NÃO fazer

- Não usar `!important`
- Não animar `width`, `height` ou `top/left` — usar `transform` e `opacity`
- Não criar animações sem ScrollTrigger fora da Hero
- Não usar `setTimeout` para timing de animações — usar GSAP delays
- Não duplicar seletores CSS entre arquivos de sections
- Não deixar `console.log` no código final
- Não usar `var` ou `let` onde `const` serve
- Não hardcodar strings de texto no TypeScript — deixar no HTML

---

## Checklist antes de marcar uma section como concluída

- [ ] Markup semântico correto
- [ ] CSS usa apenas tokens de `tokens.css`
- [ ] Responsivo em 375px, 768px e 1440px
- [ ] Animação GSAP com ScrollTrigger configurado
- [ ] `prefers-reduced-motion` respeitado:
  ```css
  @media (prefers-reduced-motion: reduce) {
    * { animation: none !important; transition: none !important; }
  }
  ```
- [ ] Sem erros no console
- [ ] TypeScript sem erros (`tsc --noEmit`)

---

## Ordem de Desenvolvimento

1. Setup: Vite + TypeScript + estrutura de pastas
2. `tokens.css` + `reset.css` + fontes
3. `src/lib/gsap.ts` com plugins registrados
4. Navbar
5. Hero (mais complexa — referência visual do projeto)
6. Como Funciona
7. Dashboard
8. Benefícios
9. Logos/Parceiros
10. Simulação QR Code
11. Estatísticas
12. Segurança
13. Depoimentos
14. Pricing
15. CTA Final
16. Footer
17. Polimento geral + Lighthouse
18. Deploy

---

## Comandos

```bash
npm create vite@latest . -- --template vanilla-ts
npm install gsap
npm install -D typescript

npm run dev      # dev server
npm run build    # build estático em /dist
npm run preview  # preview do build
```
