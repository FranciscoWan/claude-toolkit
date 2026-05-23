# CLAUDE.md — Golang

## Comunicação
- Responda apenas o necessário. Sem introduções, confirmações ou explicações não solicitadas.
- Prefira código a prosa. Se autoexplicativo, omita o comentário.
- Nunca repita informação já presente no contexto.
- Sem "Claro!", "Ótimo!" ou qualquer preâmbulo.

---

## Stack
- **Linguagem:** Go (versão estável mais recente)
- **Tipagem:** Estática e explícita. Sem uso de `interface{}` / `any` sem justificativa.
- **Formatação:** `gofmt` obrigatório. Siga as convenções de Effective Go.

---

## Arquitetura

```
.
├── cmd/
│   └── [app]/
│       └── main.go          # Entrypoint — apenas inicialização
├── internal/                # Código privado da aplicação
│   └── [domain]/
│       ├── handler.go       # Entrada HTTP (equivalente ao controller)
│       ├── service.go       # Lógica de negócio
│       ├── repository.go    # Acesso a dados
│       └── model.go         # Structs de domínio
├── pkg/                     # Pacotes reutilizáveis e exportáveis
├── config/                  # Configurações de ambiente
└── migrations/              # Migrações de banco
```

---

## Regras

**Handlers**
- Apenas parsing de request e escrita de response. Zero lógica de negócio.
- Valide entrada antes de repassar ao service.

**Services**
- Toda lógica de negócio reside aqui.
- Sem acesso direto ao banco — delegue ao Repository.

**Repositories**
- Único ponto de acesso ao banco. Isola queries dos services.
- Defina a interface do repository no pacote do service (inversão de dependência).

**Models**
- Structs de domínio com tags explícitas (`json`, `db`) onde aplicável.
- Nunca exponha structs internas diretamente na resposta HTTP — use response structs.

---

## Ponteiros e Performance

- Use ponteiros (`*T`) para structs passadas entre funções quando mutação for necessária ou o valor for grande — evite cópias desnecessárias.
- Evite ponteiros para tipos primitivos (`int`, `bool`, `string`) salvo quando `nil` for semanticamente necessário.
- Prefira retornar ponteiro em construtores: `func NewUser(...) *User`.
- Nunca desreferencie um ponteiro sem verificar `nil` antes.

```go
// Correto
func (s *UserService) FindByID(id string) (*User, error) { ... }

// Evitar — copia a struct inteira desnecessariamente
func (s UserService) FindByID(id string) (User, error) { ... }
```

---

## Goroutines e Concorrência

- Use goroutines quando operações puderem ocorrer em paralelo e forem independentes.
- Sempre sincronize com `sync.WaitGroup`, channels ou `errgroup` — nunca dispare goroutines sem controle de ciclo de vida.
- Use `context.Context` para propagação de cancelamento e timeout em toda operação assíncrona ou I/O.
- Proteja estado compartilhado com `sync.Mutex` ou prefira comunicação via channels.
- Use `errgroup.WithContext` para goroutines que podem retornar erro.

```go
// Correto
g, ctx := errgroup.WithContext(ctx)
g.Go(func() error {
    return s.repo.Save(ctx, entity)
})
if err := g.Wait(); err != nil {
    slog.Error("erro ao salvar", "err", err)
}
```

---

## Tratamento de Erros e Logs

- Nunca ignore erros. Todo retorno de erro deve ser verificado e tratado.
- Erros são valores — não use `panic` para fluxo normal da aplicação.
- Envolva erros com contexto: `fmt.Errorf("operação X: %w", err)`.
- Use `log/slog` (Go 1.21+) para logs estruturados.
- Logue o erro no nível mais próximo da origem com contexto suficiente para diagnóstico.
- Nunca silencie um erro com `_` sem comentário justificando.

```go
// Correto
if err != nil {
    slog.Error("falha ao buscar usuário", "id", id, "err", err)
    return fmt.Errorf("UserService.FindByID: %w", err)
}

// Proibido
result, _ := repo.FindByID(id)
```

---

## SOLID (aplicado)

| | |
|---|---|
| **S** | Handler recebe, Service processa, Repository persiste |
| **O** | Estenda via interfaces, não modifique structs base |
| **L** | Implementações de interface intercambiáveis sem quebrar contratos |
| **I** | Interfaces pequenas e específicas por consumidor |
| **D** | Dependa de interfaces, não de implementações concretas |

---

## Interfaces

- Defina interfaces no pacote consumidor, não no pacote que implementa.
- Interfaces pequenas são preferidas (`io.Reader`, `io.Writer` como referência).
- Nomeie interfaces de um método com sufixo `-er`: `Storer`, `Finder`, `Sender`.

```go
// No pacote service — não no repository
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, user *User) error
}
```

---

## Nomenclatura

| Elemento | Padrão |
|---|---|
| Pacotes | `lowercase` sem underscores |
| Arquivos | `snake_case` |
| Funções / Métodos exportados | `PascalCase` |
| Funções / Métodos internos | `camelCase` |
| Interfaces | `PascalCase` com sufixo `-er` quando possível |
| Constantes exportadas | `PascalCase` |
| Variáveis de erro | prefixo `Err`: `ErrNotFound`, `ErrUnauthorized` |

---

## Proibido
- Ignorar erros com `_` sem justificativa · `panic` em fluxo normal
- `interface{}` / `any` sem necessidade · Goroutine sem controle de ciclo de vida
- Lógica de negócio no handler · Acesso ao banco fora do repository
- Ponteiro desreferenciado sem verificação de `nil` · Estado compartilhado sem sincronização
