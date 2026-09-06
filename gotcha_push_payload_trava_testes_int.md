---
name: gotcha_push_payload_trava_testes_int
description: "Todos os testes de integração estouram 'Hook timed out in 30000ms': o push do Payload trava quando o banco de TESTE tem coluna que o config da branch não declara"
metadata:
  node_type: memory
  type: reference
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
