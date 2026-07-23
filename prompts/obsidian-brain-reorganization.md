# Prompt — Geração de Vault Obsidian para Projetos de Software

Use este prompt para gerar um vault Obsidian bem organizado a partir de uma base de conhecimento existente ou de zero. Adapte os trechos entre `[colchetes]` ao seu projeto.

---

## PROMPT

Você receberá o conteúdo completo do meu vault Obsidian para o projeto **[nome-do-projeto]**.
[Descrição em uma linha do projeto: o que é, stack principal, objetivo central.]

Sua tarefa é gerar um vault Obsidian completamente organizado seguindo a arquitetura abaixo.
Execute as fases na ordem exata. Não gere arquivos antes da aprovação da Fase 1.

---

### FASE 1 — ANÁLISE

Antes de qualquer geração:

1. Leia todos os arquivos fornecidos.
2. Identifique: nome do projeto, serviços (backend/frontend), APIs externas, features documentadas, bugs/bugfixes, fluxos técnicos, decisões arquiteturais (ADRs), schema de banco.
3. Identifique duplicatas (mesma informação em arquivos diferentes).
4. Identifique qual informação é "humana/narrativa" (vai para o doc) versus "técnica de navegação" (vai para o vault).
5. Apresente:
   - Lista de arquivos classificados por seção (product / backend / frontend / infrastructure)
   - Duplicatas encontradas e onde serão consolidadas
   - Estrutura de pastas proposta (árvore)
6. **Aguarde aprovação antes de continuar.**

---

### FASE 2 — ESTRUTURA OBRIGATÓRIA

Gere exatamente esta estrutura. Adapte apenas os arquivos folha ao conteúdo real do projeto.

```
[nome-do-projeto]/
│
├── [nome-do-projeto]-doc.md           ← documentação narrativa (visão geral, ADRs em prosa, arquitetura)
├── [nome-do-projeto]-index.md         ← índice do vault (raiz, sem pai)
│
├── 01-product/
│   ├── product_index.md
│   ├── roadmap.md
│   ├── features/
│   │   ├── features_index.md
│   │   ├── [feature-a].md
│   │   └── [feature-b].md
│   └── decisions/
│       ├── decisions_index.md
│       ├── adr-001-[tema].md
│       └── adr-002-[tema].md
│
├── 02-backend/
│   ├── [nome-do-projeto]-be_index.md  ← índice do serviço backend
│   ├── credentials.md
│   ├── endpoints.md
│   ├── errors.md
│   ├── spec-legado.md                 ← só se houver legado; omitir caso contrário
│   ├── flows/
│   │   ├── flows_index.md
│   │   ├── [fluxo-principal].md
│   │   └── [fluxo-b].md
│   ├── features/
│   │   ├── features_index.md
│   │   ├── [feature-a].md
│   │   └── [feature-b].md
│   ├── bugfixes/
│   │   ├── bugfixes_index.md
│   │   ├── [bugfix-a]-[yyyy-mm-dd].md
│   │   └── [bugfix-b]-[yyyy-mm-dd].md
│   └── backfills/
│       ├── backfills_index.md
│       └── [backfill-a]-[yyyy-mm-dd].md
│
├── 03-frontend/
│   ├── [nome-do-projeto]-fe_index.md  ← índice do serviço frontend
│   ├── paleta-cores.md
│   ├── features/
│   │   ├── features_index.md
│   │   ├── [feature-a].md
│   │   └── [feature-b].md
│   └── bugfixes/
│       ├── bugfixes_index.md
│       └── [bugfix-a]-[yyyy-mm-dd].md
│
└── 04-infrastructure/
    ├── infrastructure_index.md
    ├── database/
    │   ├── database_index.md
    │   ├── schema.md
    │   └── migrations.md
    ├── security/
    │   ├── security_index.md
    │   └── authentication.md
    └── external-apis/
        ├── external-apis_index.md
        └── [api-externa].md
```

**Regras de nomenclatura:**
- Índices de pasta usam o nome da pasta: `features_index.md`, `flows_index.md`, `bugfixes_index.md`, `backfills_index.md`, `decisions_index.md`
- Índices de serviço usam o nome do projeto + sufixo: `[projeto]-be_index.md`, `[projeto]-fe_index.md`
- Índices de infraestrutura usam o nome da subpasta: `database_index.md`, `security_index.md`, `external-apis_index.md`
- Bugfixes e backfills sempre terminam com `-[yyyy-mm-dd]`

---

### FASE 3 — HIERARQUIA DE LINKS

**Árvore de parentesco:**

```
[projeto]-index.md
├── 01-product/product_index.md
│   ├── roadmap.md
│   ├── features/features_index.md
│   │   ├── [feature-a].md
│   │   └── [feature-b].md
│   └── decisions/decisions_index.md
│       ├── adr-001-[tema].md
│       └── adr-002-[tema].md
├── 02-backend/[projeto]-be_index.md
│   ├── credentials.md / endpoints.md / errors.md / spec-legado.md
│   ├── flows/flows_index.md → [fluxo].md
│   ├── features/features_index.md → [feature].md
│   ├── bugfixes/bugfixes_index.md → [bugfix].md
│   └── backfills/backfills_index.md → [backfill].md
├── 03-frontend/[projeto]-fe_index.md
│   ├── paleta-cores.md
│   ├── features/features_index.md → [feature].md
│   └── bugfixes/bugfixes_index.md → [bugfix].md
└── 04-infrastructure/infrastructure_index.md
    ├── database/database_index.md → schema.md / migrations.md
    ├── security/security_index.md → authentication.md
    └── external-apis/external-apis_index.md → [api].md
```

**Regras:**

- Todo arquivo linka para o **pai imediato** e o pai linka de volta para os filhos diretos — bidirecional.
- **Arquivos folha** (credentials, endpoints, errors, roadmap, schema, migrations, authentication, features individuais, bugfixes, backfills, fluxos, ADRs): linkam **somente** para o `_index.md` pai imediato. Podem linkar para arquivos **irmãos** (mesma pasta) se houver dependência real.
- **`_index.md` de pasta** (features_index, flows_index etc.): linka para filhos diretos + para o `_index.md` do serviço pai.
- **`_index.md` de serviço** ([projeto]-be_index, [projeto]-fe_index, infrastructure_index): linka para `[projeto]-index.md` + para os `_index.md` das subpastas.
- **`[projeto]-index.md`**: linka para os quatro `_index.md` de seção. Não tem pai.
- **`[projeto]-doc.md`**: linka apenas para `[projeto]-index.md`. Não é pai de nenhum outro arquivo.

**PROIBIDO:**
- Saltar mais de um nível hierárquico (ex: feature linkando para `[projeto]-index.md`)
- Linkar entre ramos diferentes (ex: `02-backend/` linkando para `03-frontend/`)
- Wikilinks cruzando serviços ou seções — use caminho em texto puro quando precisar referenciar

---

### FASE 4 — FRONTMATTER PADRÃO

Todo arquivo deve ter este frontmatter:

```yaml
---
title: [Título claro e descritivo]
category: [product | service | infrastructure]
service: [nome-do-servico]   # apenas para arquivos dentro de 02-backend/ ou 03-frontend/
status: [stable | draft | deprecated]
updated: [YYYY-MM-DD]
parent: "[[caminho/para/_index-pai]]"   # omitir apenas no [projeto]-index.md
tags:
  - [categoria: product | infrastructure | service]
  - [nome-do-servico]        # se aplicável
  - [tipo: feature | bugfix | backfill | flows | credentials | endpoints | errors | schema | migrations | authentication | roadmap | decision | index]
  - [status: stable | draft | deprecated]
---
```

**Regras de tags:**
- Kebab-case sempre
- Arquivos do mesmo tipo compartilham a tag de tipo (`feature`, `bugfix`, `flows` etc.)
- Não criar tags únicas que só aparecem em um arquivo

---

### FASE 5 — TEMPLATES DE CONTEÚDO

**Arquivos folha** (features, bugfixes, backfills, fluxos, ADRs):

```markdown
# [Título]

## Contexto
[Por que existe? Qual problema cobre? Data se relevante.]

## Conteúdo principal
[Informação técnica reorganizada e limpa.]

## Decisões / Soluções
[O que foi decidido ou resolvido e por quê — se aplicável.]

## Relacionados
- [[_index-pai]] — [descrição do vínculo]
- [[arquivo-irmao]] — [descrição] ← só se houver dependência real
```

**Arquivos `_index.md` de pasta** (features_index, flows_index, bugfixes_index, backfills_index, decisions_index):

```markdown
# [Seção] — Índice

## Arquivos deste diretório
- [[arquivo-a]] — [descrição de uma linha]
- [[arquivo-b]] — [descrição de uma linha]

## Relacionados
- [[_index-do-servico-pai]] — índice do serviço
```

**Arquivos `_index.md` de serviço** ([projeto]-be_index, [projeto]-fe_index):

```markdown
# [Serviço] — Índice do Serviço

## Contexto
[O que é o serviço, stack, data de inclusão, status atual.]

## Arquivos do serviço
- [[credentials]] / [[endpoints]] / [[errors]] — [descrição]
- [[flows/flows_index]] — [descrição]
- [[features/features_index]] — [descrição]
- [[bugfixes/bugfixes_index]] — [descrição]
- [[backfills/backfills_index]] — [descrição]

## Relacionados
- [[nome-do-projeto-index]] — raiz do vault
```

**`infrastructure_index.md`:**

```markdown
# Infraestrutura — Índice

## Subseções
- [[database/database_index]] — schema, migrations
- [[security/security_index]] — autenticação, autorização
- [[external-apis/external-apis_index]] — APIs externas consumidas

## Relacionados
- [[nome-do-projeto-index]] — raiz do vault
- [[nome-do-projeto-be_index]] — serviço que consome esta infraestrutura
```

**`[projeto]-index.md`:**

```markdown
# [Projeto] — Índice do Vault

## Contexto
[O que é o projeto em 2-3 linhas. Stack resumida em tabela.]

| Camada | Tecnologia |
|---|---|
| Backend | [...] |
| Frontend | [...] |
| Auth | [...] |

## Documentação de leitura
- [[nome-do-projeto-doc]] — documentação narrativa do projeto

## Seções do vault
- [[01-product/product_index]] — produto: roadmap, features, decisões
- [[02-backend/nome-do-projeto-be_index]] — backend: serviço, flows, features, bugfixes
- [[03-frontend/nome-do-projeto-fe_index]] — frontend: features, paleta, bugfixes
- [[04-infrastructure/infrastructure_index]] — banco, segurança, APIs externas

## Relacionados
- (raiz — sem pai)
```

**`[projeto]-doc.md`:**

```markdown
# [Projeto] — Documentação do Projeto

> Para navegação técnica, use o [[[projeto]-index]].

---

[Documentação narrativa: o que é o projeto, as decisões principais em prosa,
arquitetura, contexto histórico. Sem links de navegação do vault — apenas
a referência ao index acima.]
```

---

### FASE 6 — REGRAS DE CONTEÚDO

- **Duplicatas:** mantenha no arquivo mais específico; substitua nos demais por referência em texto puro
- **Arquivo com mais de um tema:** divida em arquivos separados
- **Arquivos sobre o mesmo tema:** consolide em um só
- **Nada pode ser perdido:** toda informação técnica preservada, apenas reorganizada
- **Linguagem:** direta, sem introduções redundantes
- **Informação narrativa vs. técnica:** prosa de contexto/história → `[projeto]-doc.md`; referências técnicas, fluxos, specs → vault estruturado

---

### FASE 7 — ENTREGÁVEIS

1. Estrutura de pastas em árvore
2. Conteúdo de cada arquivo, um por vez, de cima para baixo na hierarquia
3. Relatório de mudanças:
   - Arquivos criados
   - Arquivos mesclados (quais unificados e onde)
   - Arquivos removidos (para onde o conteúdo foi)
   - Links removidos (motivo: circular, lateral, salto de nível, cruzamento de ramo)

---

## INÍCIO

Abaixo estão os arquivos do vault. Leia todos antes de responder.
Comece pela Fase 1: apresente a análise e aguarde aprovação.
Não gere nenhum arquivo ainda.

[cole aqui o conteúdo dos arquivos, um por vez com o caminho como cabeçalho]

