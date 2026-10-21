---
name: gotcha_duplicate_copia_tokens_clientes
description: "Duplicar cliente (payload.duplicate) copiava tokens secretos; fix em branch fix/tokens-duplicar-cliente (commit b7d89f99, NÃO enviado); todo campo-token novo precisa de beforeDuplicate"
metadata:
  type: reference
---
`payload.duplicate` copia TODO campo, inclusive token secreto. Em `clientes`: `briefingToken`, `briefingSocialToken`, `briefingLandingToken` (+ as `...Em`) e `crmWebhookToken`. A cópia aceitava leads do CRM em nome do original e abria os links de briefing dele. Achado 20/10/2026.
**Fix (local, commit b7d89f99, branch fix/tokens-duplicar-cliente, worktree ~/area-membros-quirk-tokens-duplicar, sem push, sem schema):** `hooks: { beforeDuplicate: [() => null] }` nos de briefing; no `crmWebhookToken` `({ value }) => value ? gerarWebhookToken() : null` (quem tinha ganha token novo; quem não tinha continua sem, o token nasce ao ligar o CRM em /crm-ativar). Teste: `tests/int/clientes-duplicar-tokens.int.spec.ts`, 9 mutações, todas mortas. `resultadosSocialToken` já nasce com o hook na branch spec/relatorio-social-media.
**Produção (leitura, 20/10):** 0 valores repetidos entre clientes nos 4 tokens (146 clientes; 21 webhook, 14 briefing, 2 social, 1 landing). Nenhum dado corrigido.
**How to apply:** campo novo de token/segredo em coleção duplicável => beforeDuplicate no mesmo commit.
**Teste de integração em branch sem os campos das outras:** o banco de teste compartilhado guarda tabelas/colunas de outras branches (relatorios_social etc.) e o push trava ([[gotcha_push_payload_trava_testes_int]]). Sem dropar nada: merge descartável da sua branch sobre a branch dona do schema, rodar, voltar (git checkout -- src/payload-types.ts antes de trocar; o vitest o regenera).
