Você receberá o conteúdo completo do meu vault Obsidian chamado "brain". Ele contém arquivos sobre arquitetura, decisões técnicas, problemas e soluções de um projeto de software. Atualmente o vault está desorganizado: arquivos se referenciam circularmente, os links são excessivos e prejudicam a compreensão.

Sua tarefa é reorganizar completamente esse vault seguindo as regras abaixo.

─── FASE 1: ANÁLISE ───────────────────────────────────────────

Antes de gerar qualquer arquivo, leia todos os arquivos fornecidos e:

1. Identifique os temas centrais e agrupe-os em categorias lógicas.
2. Mapeie quais informações são únicas em cada arquivo e quais são repetidas.
3. Detecte referências circulares (A → B → A) e links excessivos.
4. Proponha uma estrutura hierárquica de pastas antes de gerar os arquivos.
5. Aguarde minha aprovação da estrutura antes de prosseguir para a Fase 2.

─── FASE 2: ESTRUTURA DE PASTAS ──────────────────────────────

Use a seguinte hierarquia como base (adapte conforme o conteúdo real):

brain/
├── 00-index.md              ← mapa geral do vault (único arquivo raiz)
├── 01-overview/             ← visão de alto nível do projeto
│   ├── project-summary.md
│   └── tech-stack.md
├── 02-architecture/         ← decisões estruturais e diagramas
│   ├── _index.md            ← índice da pasta
│   └── [arquivos específicos]
├── 03-features/             ← funcionalidades implementadas
│   ├── _index.md
│   └── [uma pasta ou arquivo por feature]
├── 04-decisions/            ← ADRs (Architecture Decision Records)
│   ├── _index.md
│   └── adr-001-[tema].md
├── 05-problems-solutions/   ← problemas encontrados e soluções
│   ├── _index.md
│   └── [arquivos por tema]
└── 06-reference/            ← referências externas, glossário, padrões

─── FASE 3: REGRAS DE REESCRITA ──────────────────────────────

Ao reescrever cada arquivo, siga obrigatoriamente:

LINKS:
- O arquivo 00-index.md é o único que pode ter links para todas as pastas.
- Cada _index.md pode linkar para os arquivos dentro de sua própria pasta.
- Arquivos filhos linkam apenas para o próprio _index.md da pasta e para 00-index.md.
- PROIBIDO: links circulares e links para arquivos de outras pastas sem passar pelo índice.

FORMATAÇÃO (padrão obrigatório para todos os arquivos):

---
title: [Título claro e descritivo]
category: [overview | architecture | feature | decision | problem | reference]
status: [stable | draft | deprecated]
updated: [YYYY-MM-DD]
parent: [[caminho/_index]] ou [[00-index]]
---

# [Título]

## Contexto
[Por que esse arquivo existe? Qual problema ou tópico cobre?]

## Conteúdo principal
[O conteúdo reorganizado e limpo]

## Decisões / Soluções
[Se aplicável: o que foi decidido ou resolvido e por quê]

## Relacionados
- [[link-1]] — [breve descrição do relacionamento]
- [[link-2]] — [breve descrição do relacionamento]

CONTEÚDO:
- Elimine duplicatas: se a mesma informação aparece em múltiplos arquivos, mantenha-a no arquivo mais específico e remova dos demais, adicionando apenas um link.
- Divida arquivos com mais de um tema em arquivos separados.
- Consolide arquivos que tratam do mesmo tema em um único arquivo.
- Use linguagem direta e objetiva, sem introduções redundantes.
- Preserve todas as informações técnicas — nada pode ser perdido, apenas reorganizado.

─── FASE 4: ENTREGÁVEIS ──────────────────────────────────────

Ao final, entregue:

1. A estrutura de pastas completa em formato de árvore.
2. O conteúdo de cada arquivo reescrito, um por vez, na ordem da hierarquia.
3. Um relatório de mudanças listando:
   - Arquivos criados
   - Arquivos mesclados (quais foram unificados)
   - Arquivos removidos (e para onde o conteúdo foi)
   - Links removidos (por serem circulares ou excessivos)

─── INÍCIO ───────────────────────────────────────────────────

Abaixo estão os arquivos do vault. Leia todos antes de responder.
Comece pela Fase 1: apresente sua análise e a estrutura de pastas proposta.
Não gere nenhum arquivo ainda — aguarde minha aprovação.

[COLE AQUI O CONTEÚDO DOS ARQUIVOS DO VAULT]
