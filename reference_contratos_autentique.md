---
name: reference_contratos_autentique
description: "Integração de contratos Autentique na área de membros (Opção C); token no Render, coleção contratos, passos pendentes"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-31T03:01:07.416Z
---

Gestão de contratos via **Autentique** na área de membros ([[project_area_membros_quirk]]), escolhida a **Opção C** (completa: sync + webhook + painel), jul/2026.

**API Autentique:** GraphQL em `https://api.autentique.com.br/v2/graphql`, header `Authorization: Bearer <token>`, rate limit 60/min. Query `document(id){ name created_at files{original signed} signatures{ name email signed{created_at} rejected{created_at} viewed{created_at} } }`; `documents(limit,page){total data{...}}`. Webhook: POST com `event.type` + `event.data.object`, validado por HMAC-SHA256 no header `X-Autentique-Signature` (secret = do endpoint). A conta tem ~386 documentos; os nomes trazem o nome do cliente (dá pra casar por busca).

**Token:** `AUTENTIQUE_TOKEN` — Renan cola no Render (Environment). O token foi compartilhado no chat uma vez → **recomendei rotacionar** e usar um novo só no Render. Local (.env gitignored) só pra validação.

**Feito — Opção C completa:** (1/2, 7d621d9) cliente GraphQL `src/lib/autentique/api.ts` (buscarDocumento/listarDocumentos + `statusDe`); coleção `contratos` com endpoints `/vincular` `/buscar` `/atualizar` `/desvincular` (gate ehSupervisorOuAcima); seção "Contratos (Autentique)" na aba Cadastro. DDL `scripts/ddl/2026-07-30-contratos.sql` (com lock) aplicado. (3/4, 204060d) `syncContratos` em lote plugado no cron diário; webhook `src/app/api/autentique/webhook/route.ts` valida HMAC-SHA256. (5, ea61cb8) painel `/admin/contratos` (ListaContratosView, item no menu Administrativo).

**Env que Renan precisa pôr no Render:** `AUTENTIQUE_TOKEN` (API) e `AUTENTIQUE_WEBHOOK_SECRET` (validação do webhook) + configurar a callback URL `https://membros.quirkgrowth.com.br/api/autentique/webhook` no painel Autentique. Sem esses, vincular/sync/webhook ficam inertes (erro amigável), o resto funciona.
