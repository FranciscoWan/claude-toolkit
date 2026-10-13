# Guia de Organização de Arquivos e Pastas no NestJS (DDD + Clean Architecture + SRP)

Este documento estabelece a estrutura e a organização de arquivos e pastas recomendada para projetos **NestJS**, fundamentada nos princípios de **Domain-Driven Design (DDD)**, **Clean Architecture** (Arquitetura Limpa) e **Single Responsibility Principle (SRP)**, com base na síntese de múltiplos estudos e práticas de mercado.

---

## 1. Visão Geral e Princípios Fundamentais

A estrutura padrão gerada pelo NestJS CLI convida ao desenvolvimento rápido, mas frequentemente evolui para o antipadrão de **"God Services"** (serviços com centenas ou milhares de linhas concentrando regras de negócio, chamadas de banco de dados e envio de e-mails) e **código espaguete** sem fronteiras claras.

Para garantir que a aplicação permaneça manutenível, testável e escalável à medida que a equipe e o produto crescem, adotamos os seguintes princípios centrais:

1. **Camadas Concêntricas (Clean Architecture):** As dependências de código apontam **estritamente para dentro**, em direção ao Domínio.
2. **Independência de Framework no Domínio:** O coração do negócio (camada de Domínio) é TypeScript puro, isento de decorators do NestJS (`@Injectable()`, `@Controller()`), bibliotecas HTTP ou ORMs (TypeORM, Prisma, Mongoose).
3. **Mapeamento de Bounded Contexts em Módulos NestJS:** Cada Bounded Context do DDD representa um módulo ou diretório coeso, garantindo coesão conceitual e linguagem ubíqua.
4. **Inversão de Dependências (Ports & Adapters):** O Domínio define interfaces (*ports*) para persistência e serviços externos; a Infraestrutura implementa essas interfaces (*adapters*).
5. **Responsabilidade Única (SRP):** Casos de uso (*Use Cases*) substituem serviços omnipotentes. Cada ação de negócio possui sua própria classe e responsabilidade isolada.

---

## 2. Estrutura Recomendada: Modular por Bounded Contexts

A abordagem mais robusta para projetos médios e grandes organiza a aplicação sob a raiz `src/` em **módulos/contextos delimitados (Bounded Contexts)**, onde cada módulo contém internamente as 4 camadas da Clean Architecture.

```
src/
├── contexts/                         # Bounded Contexts / Módulos de Domínio
│   ├── users/                        # Exemplo de Bounded Context: Usuários
│   │   ├── domain/                   # Camada 1: Domínio (Negócio Puro)
│   │   │   ├── entities/             # Entidades de Domínio com comportamento
│   │   │   │   └── user.entity.ts
│   │   │   ├── value-objects/        # Objetos de Valor imutáveis
│   │   │   │   └── email.vo.ts
│   │   │   ├── events/               # Eventos de Domínio (fatos do negócio)
│   │   │   │   └── user-registered.event.ts
│   │   │   ├── repositories/         # Interfaces / Ports de Repositório
│   │   │   │   └── user.repository.ts
│   │   │   └── errors/               # Erros específicos do Domínio
│   │   │       └── invalid-email.error.ts
│   │   │
│   │   ├── application/              # Camada 2: Aplicação (Orquestração / Use Cases)
│   │   │   ├── use-cases/            # Casos de uso (um arquivo/classe por ação)
│   │   │   │   ├── register-user.use-case.ts
│   │   │   │   └── update-password.use-case.ts
│   │   │   ├── dto/                  # DTOs de entrada e saída dos casos de uso
│   │   │   │   └── register-user-input.dto.ts
│   │   │   └── handlers/             # Event Handlers / Subscriber de eventos
│   │   │       └── on-user-registered.handler.ts
│   │   │
│   │   ├── infrastructure/           # Camada 3: Infraestrutura (Detalhes Técnicos)
│   │   │   ├── persistence/          # Implementação de Repositórios & Mapeamento
│   │   │   │   ├── prisma/           # Adaptação Prisma / TypeORM / Mongoose
│   │   │   │   │   ├── user.prisma-repository.ts
│   │   │   │   │   └── user.schema.prisma
│   │   │   │   └── mappers/          # Conversores entre DB Row <-> Domain Entity
│   │   │   │       └── user.mapper.ts
│   │   │   └── adapters/             # Clientes de APIs externas, gateways, etc.
│   │   │
│   │   ├── presentation/             # Camada 4: Apresentação (Interface / Entrega)
│   │   │   ├── controllers/          # Controllers REST / GraphQL / gRPC
│   │   │   │   └── user.controller.ts
│   │   │   ├── dtos/                 # DTOs de requisição HTTP (com class-validator)
│   │   │   │   └── create-user-request.dto.ts
│   │   │   └── presenters/           # Formatadores de resposta HTTP / View Models
│   │   │       └── user.presenter.ts
│   │   │
│   │   └── user.module.ts            # Módulo NestJS do Bounded Context
│   │
│   └── orders/                       # Outro Bounded Context: Pedidos
│       ├── domain/
│       ├── application/
│       ├── infrastructure/
│       ├── presentation/
│       └── order.module.ts
│
├── shared/                           # Shared Kernel & Utilitários Globais
│   ├── domain/                       # Classes base do domínio (BaseEntity, ValueObject)
│   │   ├── base.entity.ts
│   │   └── base-repository.interface.ts
│   ├── infrastructure/               # Módulos compartilhados de infra (Database, EventBus)
│   │   ├── database/
│   │   │   └── prisma.service.ts
│   │   └── event-bus/
│   │       └── event-bus.module.ts
│   └── utils/                        # Geradores de ID (Snowflake/UUID), utilitários
│       └── id-generator.util.ts
│
├── app.module.ts                     # Módulo Raiz da Aplicação
└── main.ts                           # Ponto de entrada do NestJS
```

---

## 3. Detalhamento e Responsabilidades das Camadas

### 3.1. Domain Layer (`domain/`)
- **Responsabilidade:** Contém o coração do sistema: regras de negócio, invariantes, entidades e objetos de valor.
- **Regra de Ouro:** **TypeScript Puro.** É proibido importar decoradores do NestJS (`@Injectable()`, `@Controller()`), pacotes de ORM (TypeORM, Prisma) ou exceções HTTP.
- **Componentes:**
  - **Entities / Aggregates:** Encapilam estado e comportamento (métodos mutadores com validações).
  - **Value Objects (VOs):** Garantem validação de tipos e imutabilidade (ex: `Email`, `CPF`, `Money`).
  - **Domain Events:** Fatos ocorridos no passado expressos em linguagem ubíqua (ex: `OrderPlacedEvent`).
  - **Repository Interfaces (Ports):** Contratos que definem como o domínio interage com a persistência sem conhecer o banco de dados.

### 3.2. Application Layer (`application/`)
- **Responsabilidade:** Orquestra o fluxo da aplicação e executa a lógica de negócio invocando os objetos de domínio e os repositórios.
- **Princípio SRP:** Em vez de serviços gordos, cada ação de negócio é isolada em um **Use Case** individual (ex: `RegisterUserUseCase`).
- **Componentes:**
  - **Use Cases / Command Handlers:** Executam uma única intenção do usuário.
  - **Application DTOs:** Contratos simples de dados para entrada e saída dos casos de uso.
  - **Event Handlers:** Listeners que reagem a eventos publicados por outros contextos.

### 3.3. Infrastructure Layer (`infrastructure/`)
- **Responsabilidade:** Implementa os detalhes técnicos de persistência, comunicação externa e frameworks.
- **Componentes:**
  - **Concrete Repositories (Adapters):** Implementam as interfaces definidas no Domínio usando ferramentas como Prisma, TypeORM ou Mongoose.
  - **Mappers:** Convertem registros do banco de dados (*Database Rows/Schemas*) em Entidades de Domínio puras e vice-versa.
  - **External Adapters:** Serviços de e-mail (SendGrid/SES), gateways de pagamento (Stripe), armazenamento (S3).

### 3.4. Presentation Layer (`presentation/` ou `interface/`)
- **Responsabilidade:** Ponto de entrada da aplicação (HTTP REST, GraphQL, CLI, WebSockets).
- **Componentes:**
  - **Controllers:** Controllers do NestJS finos. Apenas recebem a requisição, validam entrada, invocam o Use Case e retornam o resultado formatado.
  - **Request DTOs:** Classes com decoradores do `class-validator` e `class-transformer`.
  - **Presenters / View Models:** Formatação e projeção do resultado retornado para o cliente.

---

## 4. Exemplo Prático de Implementação

### 1. Port do Repositório (`domain/repositories/user.repository.ts`)
```typescript
import { User } from '../entities/user.entity';

export const USER_REPOSITORY = Symbol('USER_REPOSITORY');

export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```

### 2. Caso de Uso (`application/use-cases/register-user.use-case.ts`)
```typescript
import { Inject, Injectable } from '@nestjs/common';
import { USER_REPOSITORY, UserRepository } from '../../domain/repositories/user.repository';
import { User } from '../../domain/entities/user.entity';

@Injectable()
export class RegisterUserUseCase {
  constructor(
    @Inject(USER_REPOSITORY)
    private readonly userRepository: UserRepository,
  ) {}

  async execute(input: { name: string; email: string }): Promise<User> {
    const existing = await this.userRepository.findByEmail(input.email);
    if (existing) {
      throw new Error('E-mail já cadastrado.');
    }

    const user = User.create(input.name, input.email);
    await this.userRepository.save(user);

    return user;
  }
}
```

### 3. Registro e Injeção de Dependência no Módulo (`user.module.ts`)
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './presentation/controllers/user.controller';
import { RegisterUserUseCase } from './application/use-cases/register-user.use-case';
import { USER_REPOSITORY } from './domain/repositories/user.repository';
import { PrismaUserRepository } from './infrastructure/persistence/prisma/user.prisma-repository';

@Module({
  controllers: [UserController],
  providers: [
    RegisterUserUseCase,
    {
      provide: USER_REPOSITORY,
      useClass: PrismaUserRepository, // Inversão de dependência ligando o Port ao Adapter
    },
  ],
  exports: [RegisterUserUseCase],
})
export class UserModule {}
```

---

## 5. Proporcionalidade: Quando Usar e Quando Evitar

A adoção do DDD e Clean Architecture traz imenso valor, mas possui um custo de *boilerplate*. Aplique o **princípio da proporcionalidade**:

| Cenário de Aplicação | Abordagem Recomendada |
| :--- | :--- |
| **Sistemas com Regras de Negócio Complexas / Múltiplas Equipes** | **DDD + Clean Architecture Completo** (Módulos por Bounded Context, Use Cases, Mappers, VOs e Repositories isolados). |
| **Módulos / CRUDs Triviais (Apenas leitura/escrita simples)** | **Abordagem Lean / Anêmica** (Controller + Service + Repository ORM direto), evitando excesso de arquivos sem regras de negócio para proteger. |
| **Protótipos ou PoCs Descartáveis** | **Abordagem padrão NestJS CLI**, focando em velocidade máxima de entrega inicial. |

---

## 6. Checklist de Boas Práticas

- [ ] **Camada de Domínio sem NestJS:** Verifique se o diretório `domain/` não importa nada de `@nestjs/*` ou ORMs.
- [ ] **Orientação a Casos de Uso (SRP):** Evite criar `UsersService` com múltiplos métodos desalinhados. Cada caso de uso representa uma única intenção.
- [ ] **Mapeadores Obrigatórios:** Garanta que repositórios de infraestrutura usem *Mappers* para converter entre o modelo do banco de dados e as entidades puras do domínio.
- [ ] **Módulos com Fronteiras Fechadas:** Mantenha a chave `exports` do módulo a mais estreita possível. Exporte apenas os casos de uso necessárias para consumo por outros módulos.
- [ ] **Comunicação por Eventos:** Para comunicação assíncrona ou efeitos colaterais entre Bounded Contexts, utilize Eventos de Domínio (`DomainEvents`) via barra de eventos (`EventBus`).
