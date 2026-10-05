# CLAUDE.md — Diretrizes do Projeto

## Comunicação

- Responda apenas o necessário. Sem introduções, confirmações, resumos ou explicações não solicitadas.
- Prefira código a prosa. Se o código for autoexplicativo, omita o comentário.
- Nunca repita informação já presente no contexto.
- Seja direto: sem "Claro!", "Ótimo!", "Certamente!" ou qualquer preâmbulo.

## Stack

- **Framework:** Next.js 16+ (App Router, React Server Components, Turbopack)
- **Build:** export estático (`output: 'export'`), sem servidor Node em produção
- **Estilos:** Tailwind CSS v4: utilitários apenas, mobile-first, sem `style={}` inline
- **Tema:** `src/styles/globals.css` → bloco `@theme`: fonte única de cor, fonte e espaçamento
- **Tipagem:** TypeScript strict, sem `any`, `interface` para objetos e props, union types ou objetos `as const` para domínio
- **Fontes / imagens:** `next/font` e `next/image`
- **Lint:** ESLint (flat config) + Prettier com `prettier-plugin-tailwindcss`

## Arquitetura

```
src/
├── app/                    # Somente roteamento: page, layout, error, not-found, metadata
│   ├── layout.tsx          # Root layout: fontes, metadata global, <Header/>, <Footer/>
│   ├── page.tsx            # Compõe seções vindas de features/
│   ├── not-found.tsx
│   └── global-error.tsx
├── features/               # Um diretório por domínio (hero, projects, experience, contact…)
│   └── [feature]/
│       ├── components/     # Componentes exclusivos da feature
│       ├── hooks/          # Hooks exclusivos da feature
│       ├── services/       # Leitura/transformação de dados da feature
│       ├── data/           # Conteúdo estático tipado (projetos, experiências)
│       ├── types/          # Interfaces e tipos de domínio
│       └── index.ts        # API pública da feature (barrel)
├── shared/                 # Reutilizáveis sem conhecimento de domínio
│   ├── components/ui/      # Button, Card, Section, Badge…
│   ├── hooks/              # use-media-query, use-in-view…
│   ├── lib/                # Utilitários puros (cn, formatters)
│   └── validators/         # Schemas Zod reutilizáveis
├── core/                   # Singletons e configuração global
│   ├── config/             # site.config.ts (nome, links, SEO), constantes
│   ├── providers/          # Context providers client-side (theme, motion)
│   └── logger/             # Wrapper de log com contexto
├── layout/                 # Header, Footer, Nav
└── styles/
    └── globals.css         # @import "tailwindcss" + @theme
public/                     # Assets estáticos (imagens, CV, favicon)
```

- `app/` não contém lógica nem componentes de UI próprios: apenas importa de `features/` e `layout/`.
- Uma feature importa de `shared/` e `core/`, nunca de outra feature. Importações sempre via `index.ts`.
- Alias de import: `@/*` → `src/*`.

## Regras

### Componentes

- **Server Component por padrão.** `'use client'` apenas no menor componente folha que precisa de estado, efeitos, eventos ou APIs do browser.
- Nunca marque `page.tsx` ou `layout.tsx` como `'use client'`.
- Lógica em hooks ou `services/`, não no JSX. Componente só compõe e renderiza.
- Estado local com `useState`/`useReducer`. Valores derivados são calculados no render, não sincronizados com `useEffect`.
- `useEffect` apenas para sincronizar com sistemas externos (DOM, observers, bibliotecas de animação), sempre com função de cleanup.
- React Compiler ativo (`reactCompiler: true`): sem `useMemo`/`useCallback` manuais, salvo necessidade medida.
- Props tipadas com `interface [Nome]Props`. Sem `React.FC`.

### Dados e serviços

- Projeto sem backend: conteúdo vive em `features/[feature]/data/` como objetos tipados.
- Leitura e transformação de dados em `services/` (funções puras), consumidas por Server Components em build time.
- `fetch` externo, se houver, apenas em Server Components ou `services/`, executado no build.
- Estado global client-side apenas via Context dedicado em `core/providers/` (ex: `ThemeProvider`). Sem bibliotecas de estado global.

### Formulários

- React Hook Form + Zod (`@hookform/resolvers/zod`). Schemas reutilizáveis em `shared/validators/`.
- Envio via serviço externo (ex: Formspree/EmailJS) encapsulado em `features/contact/services/`.

### Roteamento e carregamento

- App Router com convenções de arquivo (`page`, `layout`, `error`, `not-found`, `loading`).
- SEO via Metadata API (`export const metadata` / `generateMetadata`), `sitemap.ts`, `robots.ts` e `opengraph-image`.
- Bibliotecas pesadas de efeito (GSAP, Three.js, Lottie…) carregadas com `next/dynamic` e `ssr: false` dentro de um Client Component.
- Âncoras internas da SPA com `<Link href="#secao">`. Navegação programática com `useRouter` de `next/navigation`.

### Restrições do export estático

- Proibido: Server Actions, Route Handlers dinâmicos, `proxy.ts`, `cookies()`, `headers()`, ISR, rotas dinâmicas sem `generateStaticParams`.
- `next/image` com `images.unoptimized: true` (ou loader customizado).

### Tema

- Nunca use cores da paleta padrão (`text-blue-500`, `bg-gray-100` etc.).
- Use apenas tokens semânticos definidos em `@theme` (`bg-primary`, `text-muted`, `border-border`, `font-heading`).
- Dark mode via variáveis CSS sobrescritas no seletor `.dark` (ou `prefers-color-scheme`), nunca com classes `dark:` espalhadas por cor.
- Combinação condicional de classes com `cn()` (`clsx` + `tailwind-merge`) em `shared/lib/cn.ts`.

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.62 0.19 260);
  --color-background: oklch(0.99 0 0);
  --color-foreground: oklch(0.2 0 0);
  --color-muted: oklch(0.55 0 0);
  --color-border: oklch(0.9 0 0);
  --font-heading: var(--font-geist-sans);
}
```

## SOLID (resumo aplicado)

| | |
|---|---|
| **S** | 1 componente/hook/função = 1 responsabilidade |
| **O** | Estenda por composição (`children`, props de render, variantes), não modifique |
| **L** | Componentes que estendem outro aceitam suas props base (`ComponentProps<'button'>`) |
| **I** | Props pequenas e específicas; sem objetos "faz-tudo" |
| **D** | Dependa de abstrações: receba dados e funções por props/parâmetros ou Context, não importe implementações concretas no componente |

## Nomenclatura

| Elemento | Padrão |
|---|---|
| Arquivos e pastas | kebab-case (`project-card.tsx`) |
| Arquivos de convenção Next | nome reservado (`page.tsx`, `layout.tsx`, `error.tsx`) |
| Componentes / Interfaces / Tipos | PascalCase (`ProjectCard`, `ProjectCardProps`) |
| Hooks | `use-[nome].ts` exportando `use[Nome]` |
| Variáveis / Funções | camelCase |
| Constantes globais | UPPER_SNAKE_CASE |
| Componentes de feature | `[Feature][Nome]` (`ProjectsGrid`, `HeroTitle`) |
| Rotas (URL) e IDs de seção | kebab-case |
| Exports | nomeados; `default` apenas onde o Next exige |

## Logs de Erro

- Todo erro capturado deve ser logado via `console.error` (ou `core/logger`) com contexto: `[componente/serviço] operação`.
- Nunca silencie um `catch` vazio. Sempre logue e trate ou relance a exceção.
- Em serviços que fazem requisição, logue o erro antes de relançar ou mapear para erro de domínio.
- Erros de renderização tratados por `error.tsx` (por segmento) e `global-error.tsx`, que também logam o erro recebido.
