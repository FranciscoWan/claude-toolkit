# Refatoração estrutural de código NestJS — remover espaguetificação

## Parâmetros (preencha antes de rodar)
- **Escopo:** `[CAMINHO]` — uma pasta ou um único arquivo.
- **Contexto do domínio (opcional):** `[1–3 frases sobre o que esse código faz]`
- **Invariantes críticas (opcional):** `[o que não pode mudar em nada: formato de payload enviado a terceiros, ordem de campos, cálculo X, mensagens Y…]`

## Papel e objetivo
Você é um engenheiro sênior de NestJS/TypeScript fazendo uma refatoração **estrutural, sem mudança de comportamento**, no escopo acima. Os objetivos são três:
1. separar responsabilidades misturadas;
2. organizar os arquivos em pastas coesas por capacidade;
3. deixar explícito o que cada arquivo faz e o que ele não pode fazer.

Trabalhe em fases, com pontos de parada. Não pule a Fase 1. Se o escopo for um único arquivo, trate-o como ponto de partida: a proposta pode criar uma pasta no mesmo local e distribuir ali as partes extraídas.

## 1. Papéis permitidos
Todo arquivo do escopo deve ter exatamente um destes papéis. Um arquivo que cumpre mais de um precisa ser dividido.

| Papel | Faz | Não pode |
|---|---|---|
| Entrada (controller, resolver, consumer de fila, cron), se estiver no escopo | Recebe a chamada, valida o DTO, chama um orquestrador, formata a resposta, traduz erros para o protocolo (HTTP, RPC etc.) | Conter regra de negócio, acessar repositório/ORM ou serviço externo diretamente |
| Orquestrador (caso de uso) | Coordena um fluxo: obter dados → aplicar regras → montar → validar → persistir/enviar → interpretar retorno | Ter regra inline em `if`, montar payload, fazer query/ORM direto, conhecer HTTP/SDK externo, converter formatos |
| Regra (`*.rule.ts`) | Decide algo do negócio: cálculos, elegibilidade, transições de status, interpretação de códigos | Ter decorator do NestJS, fazer I/O, chamar `new Date()`/`Math.random()`/gerador de ID diretamente (recebe por parâmetro), depender de config |
| Parser (`*.parser.ts`) | Lê um formato externo de entrada (XML, CSV, planilha, JSON de terceiros) e devolve um objeto tipado interno | Decidir regra de negócio, montar saída |
| Builder (`*.builder.ts`) | Recebe dados internos prontos e monta uma saída (payload, XML, documento, mensagem) | Buscar dados, enviar, validar contra schema, aplicar regra |
| Validador (`*.validator.ts`) | Valida regras de dados que vão além do DTO (consistência entre campos, schema externo); devolve erros tipados | Corrigir dados silenciosamente, lançar exceção HTTP |
| Mapper (`*.mapper.ts`) | Converte entre formatos (registro do banco ↔ objeto interno, resposta externa ↔ objeto interno, objeto interno ↔ resposta) | Ter regra ou I/O |
| Adaptador de infra (`*.repository.ts`, `*.client.ts`, `*.gateway.ts`) | Fala com o mundo externo: banco/ORM, APIs, filas, storage, e-mail, criptografia; implementa um port | Conter regra; ser importado diretamente por orquestradores (eles dependem do port) |
| Suporte (`*.constants.ts`, `*.types.ts`, `*.error.ts`, `*.port.ts`, `*.dto.ts`) | Constantes, tipos, erros, interfaces de infra com token de injeção, DTOs | Conter lógica |

Use `@Injectable()` só onde há injeção de dependência real. Regras são sempre TypeScript puro: funções exportadas ou classes sem decorator. A validação de formato da requisição fica no DTO (`class-validator`), não no validador.

## 2. Regras de dependência
- A direção é única: entrada → orquestrador → regras, parsers, builders, validadores, mappers e ports.
- Builders, parsers, mappers e validadores podem usar regras, tipos e constantes.
- Regras não importam nada além de tipos e constantes.
- Nenhum arquivo importa uma entrada ou um orquestrador. A exceção é a entrada, que importa o orquestrador.
- Adaptadores de infra implementam um port e não importam nada de orquestração.
- Ciclos de import são proibidos.
- Cada pasta expõe sua API pública num `index.ts`, e as outras pastas importam só por ele. O que não está no index é interno. Se um barrel provocar import circular no Nest, importe o arquivo público diretamente e registre isso no relatório.
- O módulo NestJS exporta o mínimo necessário para quem está fora.

## 3. Fonte única de verdade
- Códigos de status, enums, números e strings mágicos, URLs e chaves de configuração ficam em constantes centralizadas por assunto.
- A mesma transformação (datas, decimais, documentos, normalização de texto) repetida em 2 ou mais arquivos deve virar uma função única.
- A mesma query repetida em 2 ou mais lugares deve virar um método do repositório.
- Se existe um mapa ou definição de campos, ele é a fonte única: builders e validadores consomem esse mapa, não duplicam.

## 4. Erros
- Fora da camada de entrada, lance erros tipados próprios (`*.error.ts`) ou retorne um resultado. Nunca lance `HttpException` ou suas subclasses; a tradução para o protocolo acontece na entrada ou num exception filter.
- Sem `catch` vazio ou que troque o erro por mensagem genérica. Erros de serviços externos preservam o código e a mensagem originais do provedor.

## 5. Proporcionalidade
- Crie interface (port) só para infraestrutura: banco, APIs, filas, e-mail e, se necessário para teste, relógio ou gerador de ID/aleatório. Não crie interface para regra pura com uma implementação.
- Não introduza entidades ricas, value objects, event bus ou CQRS, a menos que o projeto já use esses padrões fora do escopo.
- Trechos que são CRUD trivial, sem regra, não precisam da separação completa: entrada → orquestrador fino → repositório basta. Registre essa decisão.
- Service com vários métodos públicos (várias intenções):
  - divida em um orquestrador por intenção quando as intenções tiverem regras ou dependências diferentes;
  - quando compartilham o mesmo fluxo e só mudam os dados, mantenha um orquestrador e coloque a variação em mapper, builder ou estratégia por tipo.
- Não crie arquivo de poucas linhas só para cumprir padrão, a menos que seja uma regra com nome de negócio.

Sinais de alerta (indicam problema, não são leis):
- arquivo com mais de ~250 linhas;
- método com mais de ~40 linhas;
- construtor com mais de 4 dependências;
- parâmetro booleano que muda comportamento;
- dois arquivos que montam ou transformam a mesma coisa;
- service que injeta outro service do mesmo nível só para reaproveitar um método utilitário.

## 6. Organização em pastas
- Agrupe por capacidade (o que o código faz no negócio), não por tipo técnico. É proibido criar `utils/`, `helpers/`, `common/`, `mappers/` ou `services/` como pastas genéricas dentro do escopo.
- Um grupo faz sentido quando seus arquivos mudam juntos pelo mesmo motivo.
- Regras usadas por uma só capacidade ficam junto dela. Regras compartilhadas por várias podem ter pasta própria.
- Adaptadores de infra ficam agrupados pelo sistema externo com que falam.
- A spec fica sempre ao lado do arquivo testado, com o mesmo nome.
- Siga o idioma que o projeto já usa para termos de negócio. Sufixos técnicos ficam em inglês. Não renomeie arquivos só para trocar de idioma.

Exemplo ilustrativo (módulo de pedidos; não é um modelo a copiar):

```
Ruim (por tipo técnico)        Bom (por capacidade)
pedidos/                       pedidos/
├── services/                  ├── criacao/
├── mappers/                   ├── pagamento/
├── helpers/                   ├── expedicao/
└── utils/                     ├── regras/              (só as compartilhadas)
                               └── integracoes/
                                   ├── gateway-pagamento/
                                   └── transportadora/
```

## 7. Limites desta refatoração
- O comportamento deve ser idêntico:
  - mesmos retornos;
  - mesmos dados persistidos;
  - mesmos payloads enviados a sistemas externos;
  - mesmos efeitos colaterais, na mesma ordem;
  - mesmas mensagens de erro de negócio;
  - e as **invariantes críticas** dos Parâmetros.
- Nas specs existentes, só os imports podem mudar. Se uma assertion precisar mudar, isso é mudança de comportamento: pare e pergunte.
- Bugs encontrados não são corrigidos, só registrados no relatório.
- Nenhuma dependência nova no `package.json` e nenhuma atualização de versão.
- Fora do escopo, só podem mudar caminhos de import e o registro de providers nos módulos.
- Métodos públicos consumidos fora do escopo só são renomeados se isso constar na proposta aprovada.
- Trabalhe em passos pequenos: testes verdes ao fim de cada passo e um commit por passo.

## 8. Processo

**Fase 0 — Linha de base (sem alterar código)**
- Rode `git status`. Se houver arquivos modificados ou não rastreados no escopo, pare e me peça para commitar antes.
- Crie uma branch `refactor/<nome-do-escopo>`.
- Descubra o comando de testes no `package.json`, rode os testes do escopo e registre o resultado. Falhas pré-existentes são registradas, não consertadas.

**Fase 1 — Diagnóstico e proposta (somente leitura). Entregue:**
1. Tabela por arquivo com:
   - responsabilidade real hoje (uma frase);
   - papel ou papéis que cumpre (seção 1);
   - nº de linhas;
   - dependências importadas e injetadas;
   - quem consome o arquivo (dentro e fora do escopo);
   - sinais de espaguetificação, com número de linha.
2. Grafo de dependências entre os arquivos, destacando ciclos.
3. Lista de extrações: trecho → arquivo de destino → novo nome.
4. Árvore de pastas proposta e mapa antigo → novo (caminho e nome).
5. Três decisões, com justificativa:
   - qual sufixo usar nos orquestradores: `*.use-case.ts`, ou manter `.service.ts` se esse for o padrão do resto do projeto;
   - como dividir os services com várias intenções;
   - quais trechos ficam enxutos por serem CRUD trivial.
6. Mudanças necessárias fora do escopo.
7. Riscos e dúvidas.

**Pare aqui e aguarde minha aprovação.**

**Fase 2 — Rede de segurança**
Antes de mover qualquer coisa, crie testes de caracterização para cada fluxo público dos orquestradores do escopo:
- use fixtures fixas;
- congele relógio, aleatoriedade e geração de IDs via mock;
- grave em snapshot, a partir do código atual:
  - o retorno;
  - os argumentos das chamadas feitas aos adaptadores de infra (via spies/mocks);
  - os payloads montados.

Tudo deve estar verde antes de seguir.

**Fase 3 — Mover sem mudar lógica**
- Mova arquivos e specs para as pastas aprovadas com `git mv`.
- Aplique os novos nomes.
- Atualize os imports e os módulos.
- Crie os `index.ts`.

Testes verdes, commit.

**Fase 4 — Extrair responsabilidades, uma por vez**
Para cada item da lista de extrações: extrair → ajustar consumidores → testes → commit. Cada nova regra pura ganha spec própria.

**Fase 5 — Inverter dependências de infra**
- Para cada adaptador de infra consumido por orquestradores, crie um port com token (`Symbol`).
- Faça os orquestradores dependerem dos ports.
- Deixe o binding no módulo (`provide`/`useClass`).
- Nas specs dos orquestradores, use fakes dos ports.

**Fase 6 — Verificação**
Rode a suíte completa, `tsc --noEmit`, o lint e a checagem de ciclos (`npx madge --circular --extensions ts <escopo>`, se disponível). Confirme também que:
- não há `@nestjs` em nenhum `*.rule.ts`;
- não há `HttpException` fora da camada de entrada;
- não há import de ORM (`typeorm`, `@prisma/client`, `mongoose` etc.) fora de adaptadores de infra e seus mappers;
- nenhum import entra em outra pasta sem passar pelo `index.ts`;
- os snapshots de caracterização estão inalterados.

## 9. Pare e pergunte quando
- manter o comportamento exigir mudar uma assertion ou um contrato público;
- a responsabilidade de um arquivo for ambígua entre dois papéis;
- uma mudança fora do escopo for além de imports e registro de providers;
- resolver um ciclo exigir um redesenho maior que o aprovado;
- a suíte já falhar na linha de base em algo que bloqueie a verificação.

## 10. Entrega final
1. Árvore final do escopo.
2. Tabela de contrato de responsabilidade: arquivo | faz | não faz | depende de.
3. Um `README.md` curto na raiz do escopo, com essa tabela e as regras das seções 1 e 2, para guiar código novo.
4. Bugs e suspeitas encontrados e não corrigidos.
5. O que ficou de fora por proporcionalidade e vale tratar depois.
