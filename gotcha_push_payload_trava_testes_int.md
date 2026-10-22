---
name: gotcha_push_payload_trava_testes_int
description: "Testes de integração falham no push do Drizzle contra o banco de TESTE compartilhado — 'Hook timed out' (schema órfão de outra branch) OU 'constraint does not exist' (corrida entre sessões rodando ao mesmo tempo)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-10-22T00:06:47.844Z
---

Sintoma: **todos** os arquivos de `tests/int/` falham com `Error: Hook timed out in 30000ms` no `beforeAll`, em `getTestPayload()`. Aconteceu em 05/09/2026 — 191 de 536 arquivos, e a suíte inteira levou 1h43 só de timeouts.

**Why:** nos testes `NODE_ENV` é `test`, então o `push` do Drizzle roda. Se o banco de TESTE tem uma coluna que o config da branch atual **não declara**, o push quer DERRUBAR a coluna — mudança destrutiva — e fica esperando confirmação que nunca chega. Não é lentidão: `getTestPayload()` pendura para sempre (medido: >5min sem completar).

Como isso nasce: uma branch com campo novo roda os testes e o push cria a coluna no banco de teste. Depois você volta pra `main` (ou pra uma branch sem o campo) e **toda** a integração trava. Foi exatamente o caso do `clickup_task_id` ([[reference_migracao_historico_clickup_crm]]).

**Diagnóstico rápido — o que NÃO é:** o banco de teste responde normal (conectar ~300ms, introspecção ~50ms) e as conexões estão longe do limite (36 de 901). Não perca tempo aí.

**How to apply:** comparar as colunas do banco de teste com o que a branch declara e dropar a órfã — o banco de teste é descartável.

```bash
node -e "... ALTER TABLE <tabela> DROP COLUMN IF EXISTS <coluna> ..."
```

Sempre com trava tripla no script: exigir `DATABASE_URI_TEST`, recusar se for igual a `DATABASE_URI`, e recusar se a URI não casar com /test/i. Ver [[reference_neon_bancos]].

**Atualização 15/10/2026 (2ª causa e atalho):** o travamento também vem de **TIPO ENUM órfão**, não só de coluna. Caso: uma branch do robô LinkedIn criou 5 colunas em `linkedin_robo` + o enum `enum_linkedin_robo_comando_programa` no banco de teste; dropar as colunas NÃO bastou (o enum ficou e o push seguiu esperando confirmação). Como achar tudo de uma vez: comparar TESTE × PRODUÇÃO (somente leitura nos dois) em colunas, enums (`pg_type`+`pg_enum`), índices, restrições e tabelas — o que existe só no teste e não na prod é órfão de outra branch; o que existe só na prod e não no teste é o que a SUA branch declara e outra sessão dropou (o push recria sem prompt). Sintoma de diagnóstico: `pg_stat_activity` mostra dezenas de sessões `idle` rodando as consultas de introspecção do Drizzle (constraints/índices) = o push terminou de ler e espera o "sim" que nunca vem. Depois de dropar coluna + enum órfãos (`DROP TYPE IF EXISTS`, só no banco de TESTE, com trava `DATABASE_URI_TEST !== DATABASE_URI`), a integração voltou (6 arquivos / 96 testes). **Atalho que evita o problema:** para mudança pequena de webhook/lógica pura, escreva o teste como UNITÁRIO com `vi.mock` do módulo que acessa o banco (ex.: `resolverColaboradorPorChatId`) — roda isolado em segundos, sem tocar o banco compartilhado.

**Atualização 21/10/2026 (3ª causa — concorrência pura, não schema desalinhado):** sintoma diferente: não trava, FALHA com
`error: constraint "payload_locked_documents_rels_relatorios_social_fk" of relation "payload_locked_documents_rels" does not exist`
no `ALTER TABLE ... DROP CONSTRAINT` do `pushDevSchema`. Investiguei achando que era uma branch não-mergeada com a coleção `relatorios-social` — **não era**: `git merge-base --is-ancestor` confirmou que a coleção já está em `origin/main`, ou seja, toda sessão declara o MESMO schema. A causa real é concorrência de verdade: duas (ou mais) sessões em worktrees diferentes rodando `vitest` ao mesmo tempo contra o MESMO `DATABASE_URI_TEST` cada uma dispara seu próprio `pushDevSchema`; os dois calculam o mesmo diff (`DROP CONSTRAINT X`), mas um executa primeiro e o segundo tenta dropar algo que já sumiu — corrida pura, sem nenhum problema de código. Confirmado reproduzindo em worktree isolado (branch limpa a partir de `origin/main`, sem o trabalho em andamento de outra sessão): a mesma suíte que falhava intermitente numa execução concorrente passou limpo rodando sozinha. **Não há correção de código** — é limite operacional de testar em paralelo contra banco compartilhado. **Mitigação:** se a suíte falhar com "constraint/column does not exist" (não "Hook timed out"), é sinal de corrida, não de schema órfão — só rodar de novo (ou isolar em worktree temporário via `git worktree add --detach <path> origin/main` + symlink de `node_modules`, like já fiz, se quiser 100% de certeza antes de decidir se é causa de código).
