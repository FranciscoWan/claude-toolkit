# Adicionar a raiz do brain do projeto

---
title: Convenções de Vault de Documentação (template genérico)
category: product
status: stable
updated: 2026-08-03
tags:
  - product
  - convencoes
  - template
  - stable
---

# Convenções de Vault de Documentação — template genérico

> Template reutilizável para criar um vault de documentação técnica (Obsidian ou markdown puro) em **qualquer projeto**. Copie este arquivo para `brain/CONVENCOES.md` do novo projeto, faça as substituições da seção 0 e apague esta linha de aviso.
>
> A regra que mais se quebra sozinha com o tempo é a **hierarquia de links** (seção 3) — quando estiver com pressa, leia ao menos aquela.
>
> Este arquivo é **meta-documentação**: descreve o vault, não o projeto. Por isso fica na raiz, não tem `parent:` e nenhum outro arquivo linka para ele. Não é pai nem filho de ninguém.

---

## 0. Como adaptar este template

Substitua os placeholders abaixo em todo o arquivo. São os **únicos** pontos específicos de projeto:

| Placeholder | O que é | Exemplo |
|---|---|---|
| `<projeto>` | slug do projeto em kebab-case | `viaboleto`, `loja-api` |
| `<VAULT_PATH>` | caminho absoluto da pasta do vault | `C:\...\MeuProjeto\brain` |
| `<servico-a>`, `<servico-b>` | slugs dos serviços/apps | `ms-boleto`, `loja-fe` |

Decisões a tomar uma vez, no início:

1. **Quais seções de topo existem** (seção 1) — depende da forma do projeto, não há resposta única.
2. **O vocabulário de tags** (seção 4) — comece pequeno; tag nasce quando há 2+ arquivos para agrupar.
3. **Se haverá arquivo narrativo** `<projeto>-doc.md` além do índice raiz.

Tudo o mais (hierarquia, frontmatter, templates, checklist, validador) é igual em qualquer projeto.

---

## 1. Estrutura de pastas

A regra invariante é uma só:

> **Toda pasta tem exatamente um `_index.md`.** Criar pasta sem índice quebra a hierarquia — o conteúdo fica órfão.

As **seções de topo** (`01-`, `02-`, …) são as fatias naturais do projeto. Numere-as por prefixo para fixar a ordem de leitura. Três formatos que cobrem a maioria dos casos:

### A — Aplicação web (backend + frontend separados)

```
brain/
├── CONVENCOES.md              ← meta, fora da hierarquia
├── <projeto>-doc.md           ← documentação narrativa em prosa (opcional)
├── <projeto>-index.md         ← raiz do vault
│
├── 01-product/                product_index.md · roadmap.md
│   ├── features/              features_index.md + folhas
│   └── decisions/             decisions_index.md + adr-NNN-*.md
│
├── 02-backend/                <projeto>-be_index.md
│   │                          credentials.md · endpoints.md · errors.md
│   ├── flows/                 flows_index.md + folhas
│   ├── features/              features_index.md + folhas
│   ├── bugfixes/              bugfixes_index.md + folhas datadas
│   └── backfills/             backfills_index.md + folhas datadas
│
├── 03-frontend/               <projeto>-fe_index.md · paleta-cores.md
│   ├── features/              features_index.md + folhas
│   └── bugfixes/              bugfixes_index.md + folhas datadas
│
└── 04-infrastructure/         infrastructure_index.md
    ├── database/              database_index.md · schema.md · migrations.md
    ├── security/              security_index.md · authentication.md
    └── external-apis/         external-apis_index.md + folhas
```

### B — Monolito / CLI / biblioteca (sem separação back/front)

```
brain/
├── CONVENCOES.md
├── <projeto>-index.md
├── 01-product/                product_index.md · roadmap.md
│   ├── features/  ·  decisions/
├── 02-application/            application_index.md
│   │                          endpoints.md ou api-publica.md · errors.md
│   ├── flows/  ·  features/  ·  bugfixes/
└── 03-infrastructure/         infrastructure_index.md
    ├── database/  ·  security/  ·  external-apis/
```

### C — Monorepo com N serviços

```
brain/
├── CONVENCOES.md
├── <projeto>-index.md
├── 01-product/                product_index.md
│   ├── features/  ·  decisions/
├── 02-services/               services_index.md
│   ├── <servico-a>/           <servico-a>_index.md
│   │   └── features/  ·  bugfixes/
│   └── <servico-b>/           <servico-b>_index.md
│       └── features/  ·  bugfixes/
└── 03-infrastructure/         infrastructure_index.md
    └── database/  ·  security/  ·  external-apis/
```

> A profundidade é livre — o que não é livre é a regra do índice por pasta e a hierarquia da seção 3. O formato C mostra que **um `_index.md` pode ser pai de outro `_index.md`**: `services_index` é pai de `<servico-a>_index`, que é pai de `features_index`.

### Pastas transversais recomendadas

Independente do formato, estas se pagam rápido assim que o projeto tem histórico:

- `bugfixes/` — um arquivo por sessão de correção, datado. Evita que a causa raiz se perca no histórico do git.
- `decisions/` — ADRs. O "por que não fizemos do outro jeito" é a informação que mais falta seis meses depois.
- `backfills/` — só em projetos que fazem correção de dados em massa; caso contrário, omita.

---

## 2. Nomenclatura

| Tipo | Padrão | Exemplo |
|---|---|---|
| Índice de pasta | `<nome-da-pasta>_index.md` | `features_index.md`, `flows_index.md` |
| Índice de serviço | `<projeto>-<sufixo>_index.md` | `<projeto>-be_index.md`, `<projeto>-fe_index.md` |
| Índice de seção infra | `<nome-da-subpasta>_index.md` | `database_index.md`, `security_index.md` |
| Índice raiz | `<projeto>-index.md` | `<projeto>-index.md` |
| Bugfix | `<tema>-<yyyy-mm-dd>.md` | `race-carregando-2026-07-14.md` |
| Backfill | `<tema>-<yyyy-mm-dd>.md` | `comissao-2026-07-16.md` |
| ADR | `adr-<NNN>-<tema>.md` | `adr-003-cache-distribuido.md` |
| Demais | kebab-case descritivo | `historico-de-precos.md` |

Regras:

- **Sempre kebab-case minúsculo.** Nada de `MeuProjeto-doc.md`.
- **Bugfixes e backfills sempre terminam com a data** `-yyyy-mm-dd`. Sem data, viram um arquivo só que ninguém consegue datar depois.
- **O nome do arquivo deve bater com o `title:`.** Se o conteúdo mudar de assunto, renomeie o arquivo — não deixe o nome envelhecer.
- ADRs são **imutáveis no número**. Decisão revista não vira ADR novo com o mesmo número: registre a revisão dentro do próprio ADR, datada.
- **Índice raiz e índices de serviço usam o slug do projeto**; índices de pasta usam o nome da pasta. Misturar os dois gera o erro clássico de `parent:` apontando para arquivo inexistente.

---

## 3. Hierarquia de links — a regra central

### A árvore

Exemplo no formato A; adapte os nomes ao formato escolhido:

```
<projeto>-index.md                        (raiz — sem pai)
├── <projeto>-doc.md
├── 01-product/product_index.md
│   ├── roadmap.md
│   ├── features/features_index.md → folhas
│   └── decisions/decisions_index.md → adr-*.md
├── 02-backend/<projeto>-be_index.md
│   ├── credentials · endpoints · errors
│   ├── flows/flows_index.md → folhas
│   ├── features/features_index.md → folhas
│   ├── bugfixes/bugfixes_index.md → folhas
│   └── backfills/backfills_index.md → folhas
├── 03-frontend/<projeto>-fe_index.md
│   ├── paleta-cores.md
│   ├── features/features_index.md → folhas
│   └── bugfixes/bugfixes_index.md → folhas
└── 04-infrastructure/infrastructure_index.md
    ├── database/database_index.md → schema · migrations
    ├── security/security_index.md → authentication
    └── external-apis/external-apis_index.md → folhas
```

### As três únicas ligações permitidas

Um `[[wikilink]]` só é legal em três situações:

| # | Ligação | Direção |
|---|---|---|
| 1 | **Filho → pai imediato** | todo arquivo linka seu `_index.md` |
| 2 | **Pai → filho direto** | todo `_index.md` lista seus filhos |
| 3 | **Irmão ↔ irmão** | mesma pasta, e só havendo dependência real |

**Bidirecionalidade obrigatória:** se o pai lista o filho, o filho linka o pai. As duas pontas, sempre.

### Proibido

| Violação | Exemplo |
|---|---|
| **Salto de nível** | folha → `<projeto>-index` (pulando o `_index` da pasta) |
| **Cruzamento de ramo** | `02-backend/...` → `03-frontend/...` |
| **Cruzamento de seção** | `01-product/...` → `02-backend/...` |
| **Link para "tio"/"primo"** | `02-backend/endpoints` → `02-backend/features/alguma-feature` |

> `02-backend/endpoints.md` e `02-backend/features/x.md` **não são irmãos** — estão em pastas diferentes. O primeiro é filho do índice do serviço; o segundo, do índice de features. Ligá-los é salto de nível.

**Por que a restrição existe.** Sem ela, todo arquivo tende a linkar todo arquivo relacionado, e em poucos meses o grafo vira uma teia onde nada indica o que é pai de quê. A hierarquia estrita é o que mantém o grafo legível e o que permite validar a estrutura automaticamente (seção 8).

### Como referenciar algo fora da hierarquia

Use **caminho em texto puro entre crases** — nunca wikilink:

```markdown
✅ Detalhes em `02-backend/features/historico-de-precos.md`.
✅ Ver `04-infrastructure/database/schema.md` para o schema da tabela.

❌ Detalhes em [[02-backend/features/historico-de-precos]].
```

A informação de navegação continua lá para quem lê; o que não entra é a aresta no grafo. **É assim que se referencia entre backend e frontend, entre produto e implementação, entre serviço e infraestrutura.**

### Formato dos wikilinks

Sempre **caminho completo a partir da raiz do vault**, sem `.md`:

```markdown
✅ [[02-backend/features/features_index]]
❌ [[features_index]]           ← ambíguo: há um por seção
❌ [[02-backend/features/features_index.md]]
```

O mesmo vale para o `parent:` do frontmatter.

---

## 4. Frontmatter

Obrigatório em **todos** os arquivos:

```yaml
---
title: Título claro e descritivo
category: product | service | infrastructure
service: <servico-a> | <servico-b>       # só nas seções de serviço
status: stable | draft | deprecated
updated: YYYY-MM-DD
parent: "[[caminho/completo/do/_index-pai]]"   # omitir só no índice raiz
tags:
  - <categoria>       # product | service | infrastructure
  - <servico>         # <servico-a> | <servico-b>, se aplicável
  - <tipo>            # features | bugfixes | backfills | flows | decision | index | ...
  - <status>          # stable | draft | deprecated
---
```

Regras:

- `category` aceita **exatamente** `product`, `service` ou `infrastructure`. Não invente `feature`, `database`, `security` — esses são **tags**, não categorias. Três valores bastam: o que o produto faz, quem implementa, onde roda.
- `parent:` deve ser o **caminho completo** e bater com a árvore da seção 3.
- `updated:` é atualizado a cada edição de conteúdo. **Data absoluta, nunca relativa** ("ontem", "semana passada" são inúteis em documento que se lê meses depois).
- **Tags são kebab-case** e compartilhadas entre arquivos do mesmo tipo.
- **Não crie tag que só existirá em um arquivo.** Tag serve para agrupar; tag de uso único é ruído no grafo. Exceção: tags-de-tipo do padrão (`endpoints`, `errors`, `schema`, `migrations`, `authentication`, `roadmap`, `credentials`), únicas por definição.

**Vocabulário base** (comece com este e cresça sob demanda):

`product` · `service` · `infrastructure` · `index` · `features` · `bugfixes` · `backfills` · `flows` · `decision` · `security` · `database` · `external-api` · `stable` · `draft` · `deprecated`

Somem-se a esse: um slug por serviço (`<servico-a>`…) e um por domínio recorrente do negócio — que só você sabe quais são, e que só devem nascer quando houver **2+ arquivos** para agrupar.

---

## 5. Templates

### Folha (feature, bugfix, backfill, fluxo, ADR)

```markdown
# Título

## Contexto
Por que existe, qual problema cobre, data se relevante.

## Conteúdo principal
Informação técnica organizada.

## Decisões / Soluções
O que foi decidido ou resolvido e por quê — se aplicável.

## Relacionados
- [[caminho/do/_index-pai]] — índice de <tipo>
- [[caminho/do/irmao]] — descrição   ← só com dependência real
```

### `_index.md` de pasta

```markdown
# Seção — Índice

## Arquivos deste diretório
- [[caminho/do/arquivo-a]] — descrição de uma linha
- [[caminho/do/arquivo-b]] — descrição de uma linha

## Relacionados
- [[caminho/do/_index-pai]] — índice do serviço/seção
```

### `_index.md` de serviço / seção

```markdown
# Serviço — Índice do Serviço

## Contexto
O que é, stack, status atual.

## Arquivos do serviço
- [[.../credentials]] / [[.../endpoints]] / [[.../errors]] — descrição
- [[.../features/features_index]] — descrição
- [[.../bugfixes/bugfixes_index]] — descrição

## Relacionados
- [[<projeto>-index]] — raiz do vault
```

### Índice raiz (`<projeto>-index.md`)

```markdown
# <Projeto> — Índice do Vault

## Contexto
O que é o projeto, em 3–5 linhas.

## Stack
| Camada | Tecnologia |
|---|---|

## Seções do vault
- [[01-product/product_index]] — produto: roadmap, features, decisões
- [[02-backend/<projeto>-be_index]] — backend
- [[03-frontend/<projeto>-fe_index]] — frontend
- [[04-infrastructure/infrastructure_index]] — banco, segurança, APIs externas

## Relacionados
- (raiz — sem pai)
```

> O índice raiz **precisa listar todas as seções de topo**. É a checagem mais barata e a que mais escapa: um ramo não listado fica órfão a partir da raiz, mesmo com todos os links internos corretos.

---

## 6. Conteúdo

**Nada pode ser perdido.** Reorganizar é mover e referenciar, nunca apagar. Se um trecho sai de um arquivo, ele entra em outro — e o original ganha um ponteiro em texto puro.

**Duplicata: uma fonte da verdade.** A informação vive no arquivo mais específico; os demais referenciam por caminho em texto puro. Endpoint mora em `endpoints.md`; a feature descreve o comportamento e aponta para lá.

**Um arquivo, um tema.** Arquivo com dois assuntos vira dois arquivos. Arquivos sobre o mesmo assunto viram um.

**Contradição se resolve, não se acumula.** Achou dois arquivos afirmando coisas opostas? A versão mais recente/confirmada vence e a outra é corrigida — com a data da revisão registrada. Não deixe as duas versões convivendo.

**Registre o porquê, não só o quê.** O código já diz o que faz. O que o vault acrescenta é a alternativa descartada, a armadilha encontrada, o motivo de a solução óbvia não servir. Sem isso, a próxima pessoa "simplifica" e reintroduz o bug.

**Documente o que foi e o que não foi verificado.** "Compila e passa nos testes" e "testado em produção com dados reais" são afirmações diferentes; registrar qual delas vale evita confiança indevida.

**Narrativo vs. técnico:**
- Prosa, história, "por que o projeto é assim" → `<projeto>-doc.md`
- Referência técnica, specs, fluxos, tabelas → vault estruturado

`<projeto>-doc.md` linka **apenas** para `<projeto>-index.md` e não é pai de ninguém.

**Linguagem direta.** Sem introdução redundante, sem "neste documento veremos".

---

## 7. Checklist antes de commitar

Ao **criar** um arquivo:

- [ ] Nome em kebab-case; bugfix/backfill com sufixo `-yyyy-mm-dd`
- [ ] Está na pasta certa (produto ≠ serviço ≠ infra)
- [ ] Frontmatter completo, `category` dentro do enum
- [ ] `parent:` com caminho completo, apontando o `_index.md` da própria pasta
- [ ] Corpo tem `## Relacionados` linkando o pai
- [ ] **O `_index.md` pai foi atualizado com o link para o novo arquivo** ← o passo mais esquecido
- [ ] Referências fora da hierarquia estão em texto puro, não em wikilink

Ao **atualizar** um arquivo:

- [ ] `updated:` atualizado
- [ ] Nenhum wikilink novo cruzando ramo ou pulando nível
- [ ] Se renomeou: todas as referências ao nome antigo foram corrigidas
- [ ] Se o conteúdo mudou de assunto: o `title:` e o nome do arquivo acompanharam
- [ ] Informação que virou obsoleta foi corrigida nos outros arquivos que a repetiam

Ao **criar uma pasta**:

- [ ] Criou o `_index.md` dela junto
- [ ] O índice da pasta-mãe lista o novo `_index.md`
- [ ] O novo `_index.md` tem `parent:` apontando a pasta-mãe

---

## 8. Verificação automática

Vale rodar depois de mexer no vault. Detecta frontmatter faltante, `category` inválida, `parent:` errado, link morto, link ilegal e quebra de bidirecionalidade.

**É agnóstico de projeto:** descobre a raiz e os índices pela estrutura de pastas, sem nomes fixos. Ajuste apenas `VAULT` e, se necessário, `META` (arquivos fora da hierarquia) e `NARRATIVE_SUFFIX`.

```python
import os, re

VAULT = r"<VAULT_PATH>"          # ex.: r"C:\Users\voce\Projeto\brain"
META = {'CONVENCOES', 'CONVENCOES-TEMPLATE'}   # meta-docs: sem pai, ninguém linka
NARRATIVE_SUFFIX = '-doc'        # <projeto>-doc.md, filho direto da raiz ('' desativa)
CATEGORIES = ('product', 'service', 'infrastructure')

notes = {}
for root, dirs, files in os.walk(VAULT):
    dirs[:] = [d for d in dirs if d not in ('.obsidian', '.git', 'node_modules')]
    for f in files:
        if f.endswith('.md'):
            p = os.path.relpath(os.path.join(root, f), VAULT).replace('\\', '/')[:-3]
            notes[p] = open(os.path.join(root, f), encoding='utf-8').read()

folder = lambda p: os.path.dirname(p)

# Raiz = único .md na raiz do vault que não é meta nem narrativo.
roots = [p for p in notes if '/' not in p and p not in META
         and not (NARRATIVE_SUFFIX and p.endswith(NARRATIVE_SUFFIX))]
ROOT = roots[0] if len(roots) == 1 else None

def is_index(p):
    return os.path.basename(p).endswith('_index') or p == ROOT

def parent_of(p):
    if p in META or p == ROOT:
        return None
    if NARRATIVE_SUFFIX and p.endswith(NARRATIVE_SUFFIX) and '/' not in p:
        return ROOT
    d = folder(p)
    if is_index(p):
        if d and '/' not in d:
            return ROOT                      # índice de seção de topo
        pd = os.path.dirname(d)              # índice aninhado: índice da pasta-mãe
        return next((c for c in notes if folder(c) == pd and is_index(c)), None)
    return next((c for c in notes if folder(c) == d and is_index(c)), None)

WIKI, errs = re.compile(r'\[\[([^\]\|]+?)(\|[^\]]+)?\]\]'), []

if ROOT is None:
    errs.append(f"[RAIZ AMBIGUA] esperado 1 arquivo raiz, encontrados: {roots}")

for p, txt in sorted(notes.items()):
    if p in META:
        continue
    m = re.match(r'^(---\n.*?\n---\n)(.*)$', txt, re.S)
    if not m:
        errs.append(f"[SEM FRONTMATTER] {p}"); continue
    fm, body = m.groups()

    cat = re.search(r'^category: (.+)$', fm, re.M)
    if cat and cat.group(1).strip() not in CATEGORIES:
        errs.append(f"[CATEGORY INVALIDA] {p} = {cat.group(1)}")

    par, exp = re.search(r'^parent: "\[\[(.+?)\]\]"', fm, re.M), parent_of(p)
    if exp and not par:
        errs.append(f"[SEM PARENT] {p} (esperado {exp})")
    elif exp and par.group(1) != exp:
        errs.append(f"[PARENT ERRADO] {p}: {par.group(1)} != {exp}")
    if exp and f'[[{exp}]]' not in body:
        errs.append(f"[NAO LINKA PAI NO CORPO] {p} -> {exp}")

    for mo in WIKI.finditer(body):
        t = mo.group(1).strip()
        if t not in notes:
            errs.append(f"[LINK MORTO] {p} -> {t}")
        elif not (parent_of(p) == t or parent_of(t) == p
                  or (folder(p) == folder(t) and p != t)):
            errs.append(f"[LINK ILEGAL] {p} -> {t}")

for p in sorted(notes):
    par = parent_of(p)
    if par and f'[[{p}]]' not in notes[par]:
        errs.append(f"[PAI NAO LISTA FILHO] {par} nao linka {p}")

print(f"Notas: {len(notes)} | Problemas: {len(errs)}\n")
for e in errs:
    print("  " + e)
```

Estado esperado: **`Problemas: 0`**.

> Rode-o **antes** de povoar o vault, com só a raiz e os índices vazios criados. Estrutura errada com 5 arquivos custa minutos; com 60, custa uma tarde.

---

## 9. Armadilhas conhecidas

Padrões observados em vaults reais. Cada linha já custou retrabalho:

| Armadilha | Consequência |
|---|---|
| Índice raiz não lista uma das seções de topo | O ramo inteiro fica órfão a partir da raiz, mesmo com links internos corretos |
| `parent:` sem o prefixo de caminho (`[[x_index]]` em vez de `[[02-backend/x_index]]`) | Link não resolve, ou resolve para o arquivo errado quando há homônimos |
| Nome do índice de serviço divergindo do que os filhos referenciam | Dezenas de `parent:` apontando para arquivo inexistente — falha silenciosa no Obsidian |
| Wikilinks cruzando ramo/seção ou pulando nível | Grafo vira teia; a hierarquia deixa de ser legível e o validador perde utilidade |
| `## Relacionados` criado vazio, "pra preencher depois" | Bidirecionalidade quebrada silenciosamente |
| Nome de arquivo que envelheceu junto com o conteúdo | Arquivo mente sobre o que contém; ninguém acha a informação |
| Duas afirmações opostas sobre a mesma causa, em arquivos diferentes | Leitor sem como saber qual vale — pior que não ter documentação |
| Documentação descrevendo mecanismo já abandonado | Leva a decisão errada com confiança |
| Tags de uso único (`typeorm`, `rbac`, `layout`…) | Ruído: tag que não agrupa nada |
| Datas relativas ("semana passada") | Inúteis meses depois, que é justamente quando o arquivo é lido |
| Documentar o quê sem o porquê | A próxima pessoa "simplifica" e reintroduz o bug que a solução evitava |
