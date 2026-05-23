# Prompt: Converter HTML de Documentação de API para OpenAPI 3.1.0 JSON

## Contexto

Você receberá o conteúdo bruto de HTML de uma documentação de API. Sua tarefa é fazer engenharia reversa desse HTML e gerar uma documentação no formato **OpenAPI 3.1.0** em JSON, seguindo exatamente a estrutura dos exemplos fornecidos abaixo.

---

## Estrutura Esperada do JSON

O JSON de saída deve seguir o padrão OpenAPI 3.1.0 com esta estrutura de topo:

```json
{
    "openapi": "3.1.0",
    "info": {
        "title": "<nome da API>",
        "description": "<descrição da API>",
        "contact": {
            "name": "<nome do contato>",
            "url": "<url do portal>"
        },
        "version": "<versão>"
    },
    "externalDocs": {
        "description": "<descrição da doc externa>",
        "url": "<url>"
    },
    "servers": [
        { "url": "<url de teste>", "description": "Teste" },
        { "url": "<url de produção>", "description": "Produção" }
    ],
    "paths": { ... },
    "components": {
        "schemas": { ... },
        "securitySchemes": { ... }
    }
}
```

---

## Regras de Mapeamento do HTML → OpenAPI JSON

### 1. `paths` — Endpoints

Cada endpoint encontrado no HTML deve virar uma chave em `paths`. O padrão é:

```json
"/caminho/{parametro}": {
    "get": { ... },
    "post": { ... },
    "put": { ... },
    "patch": { ... },
    "delete": { ... }
}
```

Cada operação HTTP deve conter:

#### `tags`
Array com o nome do grupo/tag ao qual o endpoint pertence.
```json
"tags": ["NomeDoGrupo"]
```

#### `summary`
Título curto da operação extraído do HTML (geralmente o título do endpoint).
```json
"summary": "Consultar cobrança imediata - COB"
```

#### `description`
Descrição completa em markdown. **Inclua obrigatoriamente**:
- Texto descritivo do endpoint
- Exemplos de respostas de erro em blocos de código markdown, no seguinte formato:

```
"description": "Texto descritivo do endpoint.\n<br><br>**Responses**<br>\n<br>\n\n**400**<br>\n```\n{ \"type\": \"...\", \"title\": \"...\", \"status\": 400, \"detail\": \"...\" }\n```\n\n**403**<br>\n```\n{ ... }\n```\n\n**404**<br>\n```\n{ ... }\n```\n\n**503**<br>\n```\n{ ... }\n```"
```

#### `operationId`
ID único numérico ou string da operação (extrair do HTML se disponível).
```json
"operationId": "12345678"
```

#### `parameters`
Array de parâmetros (query, path, header). Para cada parâmetro:
```json
{
    "name": "txid",
    "in": "query",           // ou "path", "header"
    "description": "Descrição do parâmetro",
    "required": true,        // true para path params, false para query opcionais
    "example": "abc123",
    "schema": {
        "type": "string",
        "maxLength": 35
    }
}
```

Para parâmetros de paginação, usar o padrão:
```json
{
    "name": "paginacao.paginaAtual",
    "in": "query",
    "description": "paginacao.paginaAtual",
    "required": false,
    "example": "0",
    "schema": {
        "format": "int32",
        "pattern": "Default value : 0",
        "type": "integer"
    }
}
```

#### `requestBody`
Quando o endpoint recebe body:
```json
"requestBody": {
    "content": {
        "application/json": {
            "schema": {
                "type": "object",
                "properties": { ... },
                "required": ["campo1", "campo2"]
            }
        }
    }
}
```

#### `responses`
Sempre incluir ao menos os status codes documentados. Erros de negócio e sistema devem referenciar schemas reutilizáveis:
```json
"responses": {
    "200": {
        "description": "OK",
        "content": {
            "application/json": {
                "schema": { ... }
            }
        }
    },
    "422": {
        "description": "Erro Negocial",
        "content": {
            "application/json": {
                "schema": { "$ref": "#/components/schemas/erroMsg" }
            }
        }
    },
    "500": {
        "description": "Erro Sistema",
        "content": {
            "application/json": {
                "schema": { "$ref": "#/components/schemas/erroMsg" }
            }
        }
    }
}
```

Quando o response é vazio (ex: DELETE bem-sucedido):
```json
"204": { "description": "OK" }
```

#### `security`
Sempre incluir os 4 schemes de segurança com os escopos corretos:
```json
"security": [
    { "apikey": [] },
    { "mutualTLS": ["<recurso>.<permissao>"] },
    { "OAuth2_Teste": ["<recurso>.<permissao>"] },
    { "OAuth2_Producao": ["<recurso>.<permissao>"] }
]
```
Exemplos de escopos: `"cob.read"`, `"cob.write"`, `"pix.read"`, `"webhook.write"`.

---

### 2. Propriedades de Schema

Para cada campo/propriedade extraída do HTML, usar:

```json
"nomeCampo": {
    "type": "string",              // string, integer, boolean, object, array, number
    "format": "int32",             // opcional: int32, int64, date-time, etc.
    "description": "Descrição detalhada do campo.",
    "pattern": "[a-zA-Z0-9]{1,35}",  // regex de validação se houver
    "maxLength": 35,
    "minLength": 1,
    "examples": ["valorExemplo"]   // sempre array, mesmo com um único exemplo
}
```

Para campos `required`, listar em array separado no nível do objeto pai:
```json
"required": ["campo1", "campo2"]
```

Para campos do tipo `array`:
```json
"nomeLista": {
    "type": "array",
    "description": "Descrição",
    "maxItems": 50,
    "items": {
        "type": "object",
        "properties": { ... },
        "required": [...]
    }
}
```

Para campos com enum:
```json
"status": {
    "type": "string",
    "description": "Status da cobrança. Enum: ATIVA, CONCLUIDA, REMOVIDA_PELO_USUARIO_RECEBEDOR, REMOVIDA_PELO_PSP",
    "maxLength": 40,
    "examples": ["ATIVA"]
}
```
> Nota: enums são documentados dentro da `description` como texto, não como campo `enum` separado, seguindo o padrão observado.

---

### 3. `components.schemas`

Definir schemas reutilizáveis. No mínimo incluir:

```json
"components": {
    "schemas": {
        "erroMsg": {
            "type": "object",
            "description": "Mensagem de erro",
            "properties": {
                "type": {
                    "type": "string",
                    "description": "URI que identifica o tipo do problema.",
                    "examples": ["https://api.exemplo.com.br/error/TipoDoErro"]
                },
                "title": {
                    "type": "string",
                    "description": "Título do problema.",
                    "examples": ["Título do erro"]
                },
                "status": {
                    "type": "integer",
                    "format": "int32",
                    "description": "Código HTTP do status.",
                    "examples": [422]
                },
                "detail": {
                    "type": "string",
                    "description": "Detalhamento do erro.",
                    "examples": ["Descrição do problema."]
                },
                "violacoes": {
                    "type": "array",
                    "description": "Lista de violações.",
                    "items": {
                        "type": "object",
                        "properties": {
                            "razao": {
                                "type": "string",
                                "description": "Razão da violação."
                            },
                            "propriedade": {
                                "type": "string",
                                "description": "Propriedade que gerou a violação."
                            }
                        }
                    }
                }
            }
        }
    },
    "securitySchemes": {
        "apikey": {
            "type": "apiKey",
            "name": "Authorization",
            "in": "header"
        },
        "mutualTLS": {
            "type": "mutualTLS"
        },
        "OAuth2_Teste": {
            "type": "oauth2",
            "flows": {
                "clientCredentials": {
                    "tokenUrl": "<url-de-token-de-teste>",
                    "scopes": {
                        "<recurso>.read": "Permissão de leitura",
                        "<recurso>.write": "Permissão de escrita"
                    }
                }
            }
        },
        "OAuth2_Producao": {
            "type": "oauth2",
            "flows": {
                "clientCredentials": {
                    "tokenUrl": "<url-de-token-de-producao>",
                    "scopes": {
                        "<recurso>.read": "Permissão de leitura",
                        "<recurso>.write": "Permissão de escrita"
                    }
                }
            }
        }
    }
}
```

---

## Divisão em Múltiplos Arquivos

O JSON final deve ser dividido em arquivos menores para que cada arquivo possa ser lido independentemente pelo Claude (máximo ~1500 linhas por arquivo). Seguir a convenção:

```
<nome-api>_part1.json   → info + servers + primeiros paths
<nome-api>_part2.json   → paths (continuação)
...
<nome-api>_partN.json   → últimos paths + components (schemas + securitySchemes)
```

**Regras de divisão:**
- Cada arquivo deve ser um JSON válido e completo (não fragmentado no meio de um objeto)
- Divida sempre nos limites de endpoints (nunca corte um path ao meio)
- O arquivo final (partN) deve conter o fechamento de `paths`, e a seção `components`
- Os arquivos intermediários contêm apenas fragmentos do objeto `paths`
- Inclua um comentário de cabeçalho em cada arquivo indicando o conteúdo:

```json
{
  "_meta": {
    "part": 1,
    "totalParts": 5,
    "description": "OpenAPI 3.1.0 - Parte 1/5: info, servers, paths /webhook e /pix"
  },
  "openapi": "3.1.0",
  ...
}
```

> **Nota:** O campo `_meta` é apenas para orientação — remova-o se for fazer parse automático do OpenAPI.

---

## Instruções de Execução

1. **Leia o HTML completamente** antes de começar a gerar o JSON.
2. **Identifique todos os endpoints** listados (método HTTP + caminho).
3. **Para cada endpoint**, extraia: descrição, parâmetros, request body, responses, segurança.
4. **Preserve textos originais** nas `descriptions` (não traduza nem resuma).
5. **Inclua todos os exemplos** presentes na documentação HTML.
6. **Gere os arquivos em sequência**, confirmando ao final de cada parte antes de prosseguir.
7. **Valide mentalmente** se o JSON seria válido OpenAPI 3.1.0.

---

## HTML de Entrada

Cole o conteúdo HTML bruto abaixo:

```
<COLE AQUI O CONTEÚDO HTML DA DOCUMENTAÇÃO>
```

---

## Saída Esperada

Gere os arquivos JSON em sequência, um por vez, aguardando confirmação antes de prosseguir para o próximo. Para cada arquivo, informe:

- Nome do arquivo (ex: `minha_api_part1.json`)
- Quais endpoints estão contidos
- O JSON completo do arquivo
