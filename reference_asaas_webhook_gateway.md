---
name: reference_asaas_webhook_gateway
description: Gotchas do webhook de pagamento Asaas → n8n no Quirk Auto Ads (gateway). Payload só traz ID do cliente; header de auth; valores e externalReference que marcam Auto Ads.
metadata: 
  node_type: memory
  type: reference
  originSessionId: ab66a44f-e962-4572-9a57-ed9c526973ed
---

Fluxo de ativação do Auto Ads: pagamento Asaas → webhook `https://n8n.quirkgrowth.online/webhook/quirk-auto-ads-payment` → workflow n8n **gateway** (ID `2ZnZqb4wFous4uEs`, nó `parse_payment`). Workflow de criar cobrança = ID `aXuUHCG2YN2IVMN2`.

**Gotcha principal (bug resolvido jun/2026):** no payload do webhook do Asaas, `payment.customer` vem como **ID string** (ex: `"cus_000183513262"`), NÃO objeto — então **não traz telefone/email/nome**. Tem que buscar via `GET https://api.asaas.com/v3/customers/{id}` com header `access_token: <asaas_api_key>`. O `parse_payment` faz essa busca (telefone vem em `mobilePhone`). Antes do fix, o fluxo morria em `telefone_invalido` e nenhuma ativação/email saía. Backup pré-fix: `/Users/renanreal/quirk_auto_ads/n8n_workflow/backup_gateway_pre_fix_telefone.json`.

**Auth do webhook:** Asaas manda o token no header **`asaas-access-token`** (formato `whsec_…`). O n8n **ainda NÃO valida** esse header (pendência de segurança). Plano: guardar segredo em `auto_ads.config` e checar no `parse_payment`. A chave de produção do Asaas (`$aact_prod_…`) hoje trafega em **texto plano** no output dos nós — endurecer depois (mover pra credential do n8n).

**Filtro "é Auto Ads" (parse_payment):** aceita se `valor==R$497` (mensalidade) OU `R$596` (1ª com bump) OU `externalReference` contém `auto-ads`/`dia1-com-bump` OU groupName == `Auto Ads - Imob`. Marcadores únicos confiáveis = externalReference (`lp-auto-ads`/`auto-ads-mensal`/`dia1-com-bump`); o match por valor é o único ponto que pode dar falso positivo se outra cobrança da conta for exatamente 497/596.

**Config no Postgres** (`auto_ads.config`, lido pelo nó `load_config_asaas`): chaves `asaas_api_key`, `asaas_group_name` (=`Auto Ads - Imob`), `asaas_product_value_cents` (=`49700`).

**Pendência operacional conhecida:** sessão WhatsApp da **UAZAPI** cai ("session is not reconnectable") e derruba o `send_welcome` — precisa reconectar (QR) no painel UAZAPI; código/pagamento ok, é a conexão do número. Push de boas-vindas também bate no **erro 463** (WhatsApp restringe início de conversa por número frio/qualidade) — risco inerente do modelo push via API não-oficial.

**DECISÃO (jun/2026): migrar pra WhatsApp Cloud API oficial.** Renan optou por **migrar o número atual** do Auto Ads + montar **direto na Cloud API da Meta** (sem BSP). Resolve queda de sessão (hospedado pela Meta, sem QR) e o 463 (push vira template aprovado). Playbook completo de cutover em `/Users/renanreal/quirk_auto_ads/docs/MIGRACAO_WHATSAPP_CLOUD_API.md` (template `ativacao_auto_ads` pronto, bodies Cloud API, parse de entrada, config keys `whatsapp_phone_number_id`/`whatsapp_token`/`whatsapp_verify_token`). Toca 15 nós de envio (2 gateway + 13 no bot `fBUin1UPt5xJEp6g`) + parse de entrada dos 2 workflows. Endpoint envio: `POST graph.facebook.com/v21.0/{phone_number_id}/messages`. Cutover ainda não feito — derruba o número, fazer em momento controlado. Base já pronta: BM, app "Quirk Auto Ads", system user QuirkOps.

**Como inspecionar execuções:** `cd /Users/renanreal/quirk_auto_ads/scripts && python3 n8n_api.py executions <wf_id>`; detalhe via `n8n_api._request("GET", f"/executions/{id}?includeData=true")` → `data.resultData.runData[<nó>][0]["data"]["main"][0][0]["json"]`. Pra reenviar um webhook que falhou: POST do `body` salvo na execução pro webhook URL com header `asaas-access-token`. Ver [[reference_quirk_docs]].
