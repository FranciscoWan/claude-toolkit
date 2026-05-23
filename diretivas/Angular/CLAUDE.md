# CLAUDE.md — Diretrizes do Projeto

## Comunicação
- Responda apenas o necessário. Sem introduções, confirmações, resumos ou explicações não solicitadas.
- Prefira código a prosa. Se o código for autoexplicativo, omita o comentário.
- Nunca repita informação já presente no contexto.
- Seja direto: sem "Claro!", "Ótimo!", "Certamente!" ou qualquer preâmbulo.

---

## Stack
- **Framework:** Angular 17+ (Standalone Components, signals, lazy loading)
- **Estilos:** Tailwind CSS — utilitários apenas, mobile-first, sem CSS inline
- **Tema:** arquivo `theme` → `tailwind.config` — fonte única de cor, fonte e espaçamento
- **Tipagem:** TypeScript strict, sem `any`, interfaces para objetos, enums para domínio

---

## Arquitetura

```
src/app/
├── core/          # Singletons: guards, interceptors, serviços globais
├── shared/        # Componentes, pipes e diretivas reutilizáveis
├── features/      # Um módulo por domínio (lazy-loaded)
│   └── [feature]/
│       ├── components/
│       ├── pages/
│       ├── services/
│       ├── models/
│       ├── [feature]-routing.module.ts
│       └── [feature].module.ts
└── layout/        # Header, sidebar, footer
```

---

## Regras

**Componentes**
- `OnPush` sempre. Lógica no serviço, não no componente.
- Estado local com `signal()` / `computed()`. Subscriptions com `takeUntilDestroyed()`.
- Sem `ngOnDestroy` manual. Sem `effect()` para atualização de layout.

**Serviços**
- HTTP exclusivamente em serviços (`providedIn: 'root'` para globais).
- Estado global em serviços dedicados (ex: `AuthStateService`).

**Formulários:** Reactive Forms. Validadores customizados em `shared/validators/`.

**Roteamento:** Resolvers para dados, guards para acesso. Features sempre lazy-loaded.

**Tema:** Nunca use `text-blue-500`, `bg-gray-100` etc. Use tokens semânticos do tema (`bg-primary`, `text-muted`, `border-border`).

---

## SOLID (resumo aplicado)

| | |
|---|---|
| **S** | 1 classe = 1 responsabilidade |
| **O** | Estenda por interfaces/DI, não modifique |
| **L** | Implementações intercambiáveis com seus tipos base |
| **I** | Interfaces pequenas e específicas |
| **D** | Dependa de abstrações, use `inject()` |

---

## Nomenclatura

| Elemento | Padrão |
|---|---|
| Arquivos | `kebab-case` |
| Classes / Interfaces / Enums | `PascalCase` |
| Variáveis / Funções | `camelCase` |
| Constantes globais | `UPPER_SNAKE_CASE` |
| Seletores | `app-[feature]-[nome]` |
| Rotas (URL) | `kebab-case` |

---

## Logs de Erro
- Todo erro capturado deve ser logado via `console.error` com contexto (componente/serviço + operação).
- Nunca silencie um `catch` vazio. Sempre logue e trate ou relance a exceção.
- Em serviços HTTP, logue o erro antes de relançar ou mapear para exceção de domínio.

---

## Proibido
- `any` · CSS inline · HTTP em componentes · `ngOnDestroy` manual
- Importar feature dentro de feature · Lógica de negócio em template
- Valores estéticos hardcoded fora do tema · `effect()` para layout
