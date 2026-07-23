---
name: feedback_isolamento_dados_clientes
description: "Requisito duro do Quirk Auto Ads: IA/backend NUNCA pode vazar dados entre clientes (multi-tenant)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
---

No [[project_quirk_auto_ads]] (multi-tenant: uma BM Quirk `1612905538806887` gerencia contas/páginas de VÁRIOS clientes), o Renan tratou como **erro extremamente grave** qualquer vazamento de dados entre clientes. Regra dura: o cliente só pode saber da **própria conta** — nunca sobre outros clientes, outras contas de anúncio/Páginas, nem o funcionamento interno/infra/IDs/tokens da Quirk.

**Why:** confiança e proteção legal. Um cliente ver a lista de Fanpages/contas de outros é quebra de confidencialidade inaceitável.

**How to apply:**
- Qualquer mensagem enviada ao cliente que venha de uma consulta à Meta (`client_pages`, `owned_pages`, `client_ad_accounts`, etc.) NUNCA pode enumerar a lista completa — só pode referenciar o dado que o próprio cliente informou. (Bug real corrigido em `scripts/e_08_data_isolation.py`: `revisao_meta` fazia `pages.slice(0,8).map(...name)` na msg de falha → vazava páginas de todos.)
- Todo prompt de IA client-facing (`build_onboarding_body`, `build_agente_body`) tem bloco "CONFIDENCIALIDADE E ISOLAMENTO DE DADOS" (regra inviolável): recusar pedidos de listar contas/páginas/outros clientes/como funciona por dentro.
- Toda query que serve dados ao cliente DEVE filtrar por `telefone` (e `ad_account_id` do próprio cliente) — padrão já usado em `list_campanhas`/`select_cliente`. Nunca rodar SELECT sem escopo do cliente.
- Mensagem de falha vazada entra no `historico_onboarding` e a IA relê → corrigir na raiz (não emitir o dado), não só no prompt.
