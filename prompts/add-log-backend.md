# Task: Adicionar Logging Estruturado ao Backend

## Contexto
Projeto NestJS + TypeORM. Objetivo: instrumentar o backend com logging estruturado para rastrear erros e bugs em produção.

## Escopo

### 1. Instalar dependências
```bash
npm i nestjs-pino pino pino-http pino-pretty
npm i uuid
```

### 2. Logger global (`src/common/logger/logger.module.ts`)
Configurar `LoggerModule.forRoot` do `nestjs-pino`:
- `pino-pretty` em desenvolvimento, JSON puro em produção
- `level` via `process.env.LOG_LEVEL` (default: `info` em prod, `debug` em dev)
- `genReqId`: usar header `x-request-id` se existir, senão gerar UUID
- `redact`: `req.headers.authorization`, `req.headers.cookie`, `req.body.password`, `req.body.token`, `*.senha`, `*.cpf`, `*.creditCard`
- `autoLogging`: ignorar rotas `/health`, `/metrics`
- `customProps`: incluir `userId` do request quando autenticado

Importar em `AppModule` e trocar o logger padrão no `main.ts` com `app.useLogger(app.get(Logger))` + `bufferLogs: true`.

### 3. Correlation ID (`src/common/middleware/correlation-id.middleware.ts`)
Middleware que garante `x-request-id` em toda request e devolve no header da response. Aplicar globalmente no `AppModule`.

### 4. Interceptor de requests (`src/common/interceptors/logging.interceptor.ts`)
- Log de entrada: método, URL, params, query, body sanitizado
- Log de saída: status code + duração em ms
- Warn quando duração > 1000ms (endpoint lento)

### 5. Filtro global de exceções (`src/common/filters/all-exceptions.filter.ts`)
Implementar `ExceptionFilter` capturando tudo:
- `HttpException` com status < 500 → `warn`
- Status >= 500 e erros não tratados → `error` com stack trace completo
- `QueryFailedError` do TypeORM → logar `error.query`, `error.parameters`, `error.driverError.code`
- Response padronizada: `{ statusCode, message, timestamp, path, requestId }`
- Nunca vazar stack trace ou detalhe de SQL na response em produção

Registrar com `APP_FILTER` no `AppModule`.

### 6. Logging TypeORM
No `TypeOrmModule.forRoot`:
- `logging: ['error', 'warn', 'migration']` em prod; `['query', 'error', 'warn']` em dev
- `maxQueryExecutionTime: 1000` para capturar queries lentas
- Criar `src/common/logger/typeorm-logger.ts` implementando a interface `Logger` do TypeORM, delegando para o pino logger (para que query logs saiam no mesmo formato JSON)

### 7. Instrumentar services
Em cada service existente:
- Injetar `PinoLogger` com `@InjectPinoLogger(NomeDoService.name)`
- `debug` na entrada de métodos com os parâmetros relevantes
- `error` em blocos catch, sempre passando o objeto de erro como primeiro argumento: `this.logger.error({ err, contexto }, 'mensagem')`
- Nunca usar `console.log`

### 8. Health check
Endpoint `/health` retornando status da conexão com o banco, sem gerar log de acesso.

## Regras
- Não alterar lógica de negócio existente, apenas adicionar instrumentação
- Todo log estruturado (objeto), nunca string concatenada
- Rodar `npm run build` e os testes existentes ao final para garantir que nada quebrou
- Adicionar `LOG_LEVEL` ao `.env.example`

## Entregáveis
Arquivos criados/alterados listados no final, com resumo do que cada um faz.
