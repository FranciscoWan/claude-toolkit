# Prompt — Geração de Vault Obsidian para Projetos de Software

Use este prompt para gerar um vault Obsidian bem organizado a partir de uma base de conhecimento existente ou de zero. Adapte os trechos entre `[colchetes]` ao seu projeto.

---

## PROMPT

Você receberá o conteúdo completo do meu vault Obsidian chamado "[nome-do-vault]". Ele documenta um projeto de software [descrição em uma linha do projeto]. O vault está desorganizado: arquivos se referenciam circularmente, links são excessivos e não existe hierarquia clara entre os temas.

Sua tarefa é reorganizar completamente esse vault. Siga as fases abaixo na ordem exata.

---

### FASE 1: ANÁLISE

Antes de gerar qualquer arquivo:

1. Leia todos os arquivos fornecidos.
2. Classifique cada arquivo em uma das categorias: `produto/roadmap`, `infraestrutura geral`, `serviço específico`, `dependência de serviço`, `banco de dados`, `segurança`, `API externa`.
3. Identifique duplicatas (mesma informação em arquivos diferentes).
4. Identifique links que saltam níveis hierárquicos.
5. Para cada serviço/integração encontrado, liste quais dependências existem (credenciais, endpoints, webhooks, fluxos, erros).
6. Apresente um resumo da análise e aguarde aprovação antes de continuar.

---

### FASE 2: HIERARQUIA OBRIGATÓRIA

Reorganize o vault exatamente nesta estrutura. Adapte os arquivos internos ao conteúdo real, mas mantenha os níveis e a lógica de parentesco.

**Regra de nomenclatura dos índices de serviço:**
- O `_index.md` de cada serviço/integração deve ser nomeado como `[nome-do-servico]_index.md`
- O index serve como ponto de entrada para aquela documentação. [nome-do-projeto]-index.md -> conecta com todos os [nome-do-servico]_index.md -> que por sua vez só se conecta com as documentações de sua pasta.
- Exemplos: `stripe_index.md`, `sendgrid_index.md`, `aws-s3_index.md`
- Todos as demais pastas que não são de serviço seguem o nome padrão [nome-da-funcionalidade]

```
[nome-do-vault]/
├── [nome-do-projeto]-index.md
│
├── 01-product/
│   ├── [nome-do-servico]_index.md
│   ├── roadmap.md
│   ├── features/
│   │   ├── [nome-do-servico]_index.md
│   │   ├── [feature-a].md
│   │   └── [feature-b].md
│   └── decisions/
        ├── [nome-do-servico]_index.md
│       ├── adr-001-[tema].md
│       ├── adr-002-[tema].md
│       └── adr-003-[tema].md
│
├── 02-backend/
│   ├── [nome-do-servico]_index.md
│   ├── credentials.md
│   ├── endpoints.md
│   ├── errors.md
│   ├── spec-legado.md
│   │
│   ├── flows/
        ├── [nome-do-servico]_index.md
│   │   ├── [fluxo-principal].md
│   │   ├── [fluxo-a].md
│   │   └── [fluxo-b].md
│   │
│   ├── features/
        ├── [nome-do-servico]_index.md
│   │   ├── [feature-a].md
│   │   ├── [feature-b].md
│   │   └── [feature-c].md
│   │
│   ├── bugfixes/
        ├── [nome-do-servico]_index.md
│   │   ├── [bugfix-a]-[yyyy-mm-dd].md
│   │   └── [bugfix-b]-[yyyy-mm-dd].md
│   │
│   └── backfills/
        ├── [nome-do-servico]_index.md
│       └── [backfill-a]-[yyyy-mm-dd].md
│
├── 03-frontend/
│   ├── [nome-do-servico]_index.md
│   ├── endpoints.md
│   ├── paleta-cores.md
│   │
│   ├── features/
        ├── [nome-do-servico]_index.md
│   │   ├── [feature-a].md
│   │   ├── [feature-b].md
│   │   └── [feature-c].md
│   │
│   └── bugfixes/
        ├── [nome-do-servico]_index.md
│       ├── [bugfix-a]-[yyyy-mm-dd].md
│       └── [bugfix-b]-[yyyy-mm-dd].md
│
└── 04-infrastructure/
    ├── [nome-do-servico]_index.md
    ├── database/
    │   ├── [nome-do-servico]_index.md
    │   ├── schema.md
    │   └── migrations.md
    ├── security/
    │   ├── [nome-do-servico]_index.md
    │   └── authentication.md
    └── external-apis/
        ├── [nome-do-servico]_index.md
        └── [api-externa].md
```

---

### FASE 3: REGRAS DE LINKS

**Regra geral:** cada arquivo linka para seu pai direto, ex: [nome-do-projeto]-index.md é o ponto de entrada PAI, product_index.md é vinculado com o PAI, roadmap.md tem vínculo com product_index.md, features_index.md é vinculado com product_index.md, [feature-a].md e [feature-b].md é vinculado com features_index.md e caso tenham relação ou dependam uma da outra, podem ter relação entre si também. O link é bidirecional. 

**Arquivos folha** (credentials, endpoints, webhooks, errors, features, decisions, schema, migrations, authentication, roadmap):
- Linkam **somente** para o `[nome-do-servico]_index.md` pai imediato
- **Não** linkam para `[nome-do-projeto]-index.md`
- **Só** linkam para arquivos irmãos (mesmo nível) se forem dependentes
- **Não** linkam para arquivos de outros ramos da hierarquia

**Arquivos `[nome-do-servico]_index.md`:**
- Linkam para os **filhos diretos** da própria pasta
- Linkam para o **pai direto** (o `[nome-do-servico]_index.md` um nível acima)
- O `[nome-do-servico]_index.md` da raiz de cada seção de topo linka para `[nome-do-projeto]-index.md`

**`[nome-do-projeto]-index.md`:**
- Linka para as seções de topo (`01-product/product_index`, `02-infrastructure/infrastructure_index`)
- Não tem pai

**PROIBIDO em qualquer arquivo:**
- Linkar para outro ramo (ex: `features/` linkando para `services/`)
- Linkar saltando mais de um nível hierárquico
- Referências cruzadas entre serviços diferentes (ex: `stripe/webhooks.md` linkando para `sendgrid_index.md`)
- Wikilinks dentro do corpo do texto que violem as regras acima — nesses casos, usar caminho em texto puro (ex: `02-infrastructure/services/servico/arquivo.md`)

---

### FASE 4: FORMATAÇÃO PADRÃO

Todos os arquivos devem seguir este template:

```markdown
---
title: [Título claro e descritivo]
category: [product | infrastructure | service | service-dependency | database | security | external-api]
service: [nome do serviço, somente para arquivos dentro de serviços/integrações]
status: [stable | draft | deprecated]
updated: [YYYY-MM-DD]
parent: "[[caminho/para/o/_index-pai]]"
tags:
  - [categoria principal]
  - [subcategoria]
  - [nome do serviço, se aplicável]
  - [tipo do arquivo: webhooks | credentials | endpoints | errors | payment-flows | roadmap | feature | decision | schema | migrations | security | authentication]
  - [status: stable | draft | deprecated]
---

# [Título]

## Contexto
[Por que esse arquivo existe? Qual problema ou tópico cobre?]

## Conteúdo principal
[O conteúdo reorganizado e limpo]

## Decisões / Soluções
[Se aplicável: o que foi decidido ou resolvido e por quê]

## Relacionados
- [[link-para-o-pai]] — [descrição do relacionamento]
```

**Regras para tags:**
- Use sempre kebab-case
- Toda tag deve ser reutilizável entre arquivos do mesmo tipo
- Arquivos do mesmo tipo em serviços diferentes compartilham a tag de tipo (ex: todos os `webhooks.md` têm a tag `webhooks`)
- O `status` da tag é idêntico ao campo `status` do frontmatter
- Não crie tags únicas que só aparecem em um arquivo

**Exemplos de conjuntos de tags:**

| Arquivo | Tags |
|---|---|
| `stripe/webhooks.md` | `infrastructure, service, stripe, webhooks, stable` |
| `sendgrid/errors.md` | `infrastructure, service, sendgrid, errors, stable` |
| `roadmap.md` | `product, roadmap, stable` |
| `features/checkout.md` | `product, feature, stable` |
| `decisions/adr-001-orm.md` | `product, decision, stable` |
| `database/schema.md` | `infrastructure, database, schema, stable` |
| `security/authentication.md` | `infrastructure, security, authentication, stable` |

---

### FASE 5: REGRAS DE CONTEÚDO

- **Duplicatas:** mantenha a informação no arquivo mais específico; nos demais, substitua pelo caminho em texto puro
- **Arquivos com mais de um tema:** divida em arquivos separados
- **Arquivos sobre o mesmo tema:** consolide em um único arquivo
- **Nada pode ser perdido:** toda informação técnica deve ser preservada, apenas reorganizada
- **Linguagem:** direta e objetiva, sem introduções redundantes
- **`[servico]_index.md`** de cada serviço deve conter: visão geral da integração, data de inclusão no projeto, status atual e lista linkada das dependências daquele serviço

---

### FASE 6: ENTREGÁVEIS

Ao final, entregue:

1. Estrutura de pastas completa em formato de árvore
2. Conteúdo de cada arquivo reescrito, um por vez, na ordem da hierarquia (de cima para baixo)
3. Relatório de mudanças:
   - Arquivos criados
   - Arquivos mesclados (quais foram unificados e onde)
   - Arquivos removidos (e para onde o conteúdo foi)
   - Links removidos (motivo: circular, lateral, salto de nível ou cruzamento de ramo)

---

## INÍCIO

Abaixo estão os arquivos do vault. Leia todos antes de responder.
Comece pela Fase 1: apresente a análise e aguarde aprovação.
Não gere nenhum arquivo ainda.

[cole aqui o conteúdo dos arquivos ou liste os caminhos]
