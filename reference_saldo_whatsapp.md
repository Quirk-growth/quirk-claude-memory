---
name: reference_saldo_whatsapp
description: "Feature SALDO do Quirk Auto Ads: cliente consulta/adiciona saldo da conta de anúncios pelo WhatsApp"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-13T17:00:57.152Z
---

No [[project_quirk_auto_ads]] (bot principal `fBUin1UPt5xJEp6g`), o cliente pede **"consultar saldo" / "incluir saldo" / "adicionar fundos"** pelo WhatsApp e recebe o saldo disponível ao vivo + o link pra adicionar fundos. Deploy 13/ago/2026 (`scripts/e_12_saldo_whatsapp.py`), testado end-to-end na linha Ignite (exec 26595, saldo ao vivo R$884,60, WhatsApp enviado).

**Arquitetura — branch determinística (padrão STATUS), NÃO passa pela IA** (determinismo importa pra link de pagamento): `classify_intent` (nova regra SALDO, regex `\bsaldo\b` + adicionar/incluir/colocar fundos|dinheiro|crédito|grana + recarregar) → `switch_intent` (saída nova, idx 10) → `load_token_saldo` (Postgres, `meta_access_token` de `auto_ads.config`) → `build_saldo` (code: fetch Meta + monta texto) → `send_saldo` (WhatsApp Cloud, cred `0tSLCLWrTWl9xSyO`, node dedicado pra não herdar a expressão `$('format_status_response')` do `send_gestao_msg`). 100% **aditivo** (não altera rota existente) → o classifier não barra (ao contrário de reroute).

**Gotchas de dados (Meta):**
- O saldo disponível **NÃO é o campo `balance`** (esse é valor devido, em centavos). É `funding_source_details.display_string` (ex.: "Saldo disponível (R$884,60 BRL)"), presente só em `is_prepay_account=True`. Regex extrai o `R$…`.
- Conta no **cartão** (`is_prepay_account=False`) não tem saldo pré-pago → mensagem explica isso e manda o link mesmo assim.
- `ad_account_id` é salvo **sem** `act_` (a URL faz `act_${id}`).
- **Link de adicionar fundos:** `https://www.facebook.com/ads/manager/account_settings/account_billing/?act={id}` (não há deep-link direto pro "Adicionar fundos"; a página de cobrança é onde fica o botão → mensagem instrui "Configurações de pagamento → Adicionar fundos").

**Alcance:** SALDO só é atingível quando NÃO está em fluxo de gestão de 10min — `if_em_gestao=false` → `classify_intent` (o caminho comum; `em_gestao=true` só durante um passo de gestão multi-turno recém-iniciado, aí vai pro `process_gestao_step`). Vale pra qualquer cliente ativo. Ver [[reference_gestao_lista_reconciliacao]] pro fluxo de gestão.
