---
name: reference_google_ads_versao_api
description: "Google Ads API tem versão hardcoded no código (googleads.googleapis.com/vNN) — Google deprecia e bloqueia versões antigas sem aviso, quebra sync inteiro"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-13T22:12:09.451Z
---

A versão da Google Ads API está **hardcoded** em 3 lugares (não é lida de env var): `src/lib/sync/descobrirContasGoogle.ts` (descoberta de contas pra vincular no hub) e `src/lib/sync/googleAdapter.ts` (×2, sync diário de gasto + palavras-chave). Google descontinua versões e passa a **bloquear** requests com HTTP 400 `UNSUPPORTED_VERSION` — sem downtime gradual, é bloqueio duro.

**Sintoma:** contas Google Ads somem da lista de vínculo no Hub (mesmo já corretamente linkadas na MCC), ou o sync diário de gasto para de atualizar — ambos **silenciosos**, porque a rota `/api/sync/pendentes` usa `Promise.allSettled` e cai pra lista vazia sem erro visível na tela.

**Corrigido em 13/08/2026:** v21 → v25 (confirmado ao vivo: v22-v25 respondiam OK, v26+ ainda não existia, v21 retornava 400 bloqueado). Caso de gatilho: conta "Imobiliária UP" (9915123185) não aparecia pra vincular — investigação confirmou que ela JÁ estava corretamente linkada na MCC (status ENABLED, manager=false, link ACTIVE), o problema nunca foi a conta, era a versão da API pra TODAS as contas.

**Diagnóstico rápido pra próxima vez:** script disposable batendo em `googleAds:searchStream` com um `SELECT customer.id FROM customer LIMIT 1` contra a MCC (`GOOGLE_ADS_MCC_ID`), testando `vNN` incrementalmente até achar a faixa que responde 200 (não 400 nem 404). Usa as credenciais já no `.env` (GOOGLE_ADS_CLIENT_ID/SECRET/REFRESH_TOKEN/DEVELOPER_TOKEN/MCC_ID).

**Fica pendente:** não há alerta automático quando a Google descontinua a versão em uso — a única forma de descobrir é o sintoma (conta sumida/sync parado) ou testar manualmente. Vale considerar no futuro um healthcheck periódico batendo a API do Google Ads, similar ao `/api/health` já existente pro Render.
