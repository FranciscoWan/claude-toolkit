# CLAUDE.md — Backend

## Comunicação
- Responda apenas o necessário. Sem introduções, confirmações ou explicações não solicitadas.
- Prefira código a prosa. Se autoexplicativo, omita o comentário.
- Nunca repita informação já presente no contexto.
- Sem "Claro!", "Ótimo!" ou qualquer preâmbulo.

---

## Stack
- **Framework:** NestJS + Fastify
- **ORM:** TypeORM
- **Banco:** PostgreSQL
- **Tipagem:** TypeScript strict, sem `any`, interfaces para contratos, classes para entidades

---

## Arquitetura

```
src/
├── modules/           # Um módulo por domínio
│   └── [domain]/
│       ├── dto/                  # Entrada e saída tipadas
│       ├── entities/             # Entidades TypeORM
│       ├── [domain].controller.ts
│       ├── [domain].service.ts
│       ├── [domain].repository.ts
│       └── [domain].module.ts
├── common/            # Decorators, guards, interceptors, pipes, filtros globais
├── config/            # Configurações de ambiente (TypeORM, Fastify, etc.)
├── database/          # Migrations, seeds
└── main.ts
```

---

## Camadas Obrigatórias

Toda feature **deve** implementar as três camadas abaixo. Nenhuma pode ser omitida.

| Camada | Responsabilidade | Onde vive |
|---|---|---|
| **DTO** | Contrato de entrada e saída da API. Validação com `class-validator`. | `[domain]/dto/` |
| **Entity** | Mapeamento da tabela. Decorators TypeORM. Nunca exposta na resposta. | `[domain]/entities/` |
| **Repository** | Único ponto de acesso ao banco. Isola queries do Service. | `[domain]/[domain].repository.ts` |

---

## Regras

**Controllers**
- Apenas roteamento e validação de entrada/saída. Zero lógica de negócio.
- Use DTOs com `class-validator` em toda entrada. Nunca aceite objetos sem tipagem.
- Retorne sempre DTOs de resposta, nunca entidades diretamente.

**Services**
- Toda lógica de negócio reside aqui.
- Sem acesso direto ao banco — delegue ao Repository.
- Lançe exceções do NestJS (`NotFoundException`, `BadRequestException` etc.).

**Repositories**
- Abstraia toda query do TypeORM. Services não usam `EntityManager` diretamente.
- Crie um repository customizado por entidade quando necessário.

**Entities**
- Use decorators TypeORM (`@Entity`, `@Column`, `@ManyToOne` etc.).
- Nunca exponha a entidade na resposta HTTP — mapeie para DTO.
- Defina `cascade`, `eager` e índices explicitamente.

**DTOs**
- `CreateXDto`, `UpdateXDto`, `ResponseXDto` — um por operação.
- `UpdateXDto` herda de `PartialType(CreateXDto)`.
- Valide com `@IsString()`, `@IsUUID()`, `@IsOptional()` etc.

**Migrations**
- Toda alteração de schema via migration. Nunca use `synchronize: true` em produção.
- Nomenclatura: `{timestamp}-{ação}-{tabela}` (ex: `1700000000000-add-status-to-orders`).

---

## SOLID (aplicado)

| | |
|---|---|
| **S** | Controller roteia, Service processa, Repository persiste |
| **O** | Estenda comportamentos via guards/interceptors, não modifique classes base |
| **L** | Repositórios e serviços implementáveis por interfaces intercambiáveis |
| **I** | Interfaces de repositório específicas por domínio |
| **D** | Injete dependências pelo construtor; dependa de abstrações |

---

## Módulos

- Cada domínio é um `@Module()` isolado.
- Exporte apenas o que outros módulos precisam consumir.
- `ConfigModule` e `TypeOrmModule` configurados no `AppModule` via `forRoot`.
- Features usam `TypeOrmModule.forFeature([Entidade])` no próprio módulo.

---

## Nomenclatura

| Elemento | Padrão |
|---|---|
| Arquivos | `kebab-case` |
| Classes / Interfaces / Enums | `PascalCase` |
| Variáveis / Funções | `camelCase` |
| Constantes globais | `UPPER_SNAKE_CASE` |
| Tabelas (banco) | `snake_case` plural (ex: `order_items`) |
| Colunas (banco) | `snake_case` |
| Rotas (URL) | `kebab-case` plural (ex: `/order-items`) |

---

## Logs de Erro
- Use o `Logger` do NestJS (`private readonly logger = new Logger(NomeDaClasse.name)`).
- Logue todo erro no Service antes de lançar a exceção, com contexto da operação.
- Nunca silencie um `catch` vazio. Sempre logue e trate ou relance.
- Erros inesperados devem incluir o stack trace: `this.logger.error(mensagem, error.stack)`.

---

## Proibido
- `any` · Lógica de negócio em controller · Acesso ao banco no service
- Expor entidade na resposta HTTP · `synchronize: true` em produção
- Query sem tipagem · Módulo importando outro módulo de feature diretamente
- Variáveis de ambiente acessadas fora de `config/` · Exceptions genéricas sem contexto
