---
name: reference_onboarding_confirmacao_flow
description: "Fluxo de confirmação do onboarding Auto Ads (aguardando_confirmacao / em_revisao), bugs conhecidos e acesso ao banco"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-07-30T16:39:34.260Z
---

Rework do onboarding do [[project_quirk_auto_ads]] (workflow principal `fBUin1UPt5xJEp6g`): em vez de auto-ativar, agora **detecta e pede confirmação**.

**State machine (clientes.status):** `em_onboarding` → (msg com os 2 dados) → `trigger_revisao` seta **`em_revisao`** (trava anti-concorrência) → `revisao_meta` acha conta+página do cliente → devolve `precisa_confirmar:true` + `ad_candidate`/`page_candidate` → `if_precisa_confirmar` → `persist_candidatos` (seta `conversas.estado_json.etapa='aguardando_confirmacao'` + candidatos, e **reseta status→em_onboarding**) → `send_confirma_msg` ("Achei aqui... responde *SIM*"). Cliente responde SIM → `load_estado_onb`→`if_aguardando`(true)→`trigger_revisao`→`revisao_meta` bloco CONFIRMA-ATIVA → `update_cliente_ativo` (status=ativo). `switch_status` roteia por status; **`em_revisao`→`send_validando`** (beco sem saída de propósito, só pra msg durante processamento).

**2 bugs corrigidos em 30/jul (`scripts/e_10`):** (1) `persist_candidatos` tinha vírgula faltando entre `jsonb_set` → `type "jsonb_set" does not exist` → crash → confirmação nunca saía. (2) a ramificação de confirmação não resetava `status` de `em_revisao` → cliente travava (SIM caía em send_validando). Fix 2: `persist_candidatos` também reseta status. **Risco residual:** qualquer erro entre `trigger_revisao` e o reset deixa o cliente presto em `em_revisao` — considerar recovery de trava velha (timestamp) no futuro.

**Acesso ao banco (auto_ads):** ⚠️ as creds Supabase em `~/.config/n8n-quirk/` (projeto `gnqxetyrurdpjsnkuhli`) estão **STALE** — pooler responde "tenant/user not found". O banco vivo do auto_ads só é alcançável **via n8n** (credencial Postgres `NKHJwhesMp2Bo4Xw` "Quirk Auto Ads Postgres"). Não dá pra rodar SQL direto daqui. Pra destravar/consertar registro: editar conteúdo de nó (permitido) e reprocessar via webhook `POST https://n8n.quirkgrowth.online/webhook/quirk-auto-ads` com `{message:{type,text,id,from,sender_pn},chat:{phone}}`. Mudar CONEXÕES/roteamento do workflow é barrado pelo classifier (produção).
