---
name: reference_onboarding_confirmacao_flow
description: "Fluxo de confirmação do onboarding Auto Ads (aguardando_confirmacao / em_revisao), bugs conhecidos e acesso ao banco"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-07-30T16:58:16.810Z
---

Rework do onboarding do [[project_quirk_auto_ads]] (workflow principal `fBUin1UPt5xJEp6g`): em vez de auto-ativar, agora **detecta e pede confirmação**.

**State machine (clientes.status):** `em_onboarding` → (msg com os 2 dados) → `trigger_revisao` seta **`em_revisao`** (trava anti-concorrência) → `revisao_meta` acha conta+página do cliente → devolve `precisa_confirmar:true` + `ad_candidate`/`page_candidate` → `if_precisa_confirmar` → `persist_candidatos` (seta `conversas.estado_json.etapa='aguardando_confirmacao'` + candidatos, e **reseta status→em_onboarding**) → `send_confirma_msg` ("Achei aqui... responde *SIM*"). Cliente responde SIM → `load_estado_onb`→`if_aguardando`(true)→`trigger_revisao`→`revisao_meta` bloco CONFIRMA-ATIVA → `update_cliente_ativo` (status=ativo). `switch_status` roteia por status; **`em_revisao`→`send_validando`** (beco sem saída de propósito, só pra msg durante processamento).

**Bugs corrigidos em 30/jul (`scripts/e_10` e principalmente `e_11`):** (1) vírgula faltando nos `jsonb_set` do `persist_candidatos` → crash `type "jsonb_set" does not exist`. (2) **CAUSA RAIZ do loop de 3× repetições:** `persist_candidatos` fazia UPDATE em `conversas`, mas cliente de onboarding NÃO tem linha em conversas (só criada no chat principal via `upsert_conversa`) → UPDATE 0 linhas → `etapa='aguardando_confirmacao'` nunca gravava → `if_aguardando` sempre FALSE → SIM re-detectava e re-enviava confirmação em loop (o `success:true` do nó mascarava o 0-rows). **Fix e_11:** persist_candidatos virou **UPSERT** (INSERT…ON CONFLICT(telefone) DO UPDATE…RETURNING); removido o lock `status='em_revisao'` do `trigger_revisao` (era fonte de travas + forçava multi-statement que o n8n Postgres não roda direito); **dedup por wamid** no `normalize_phone` via `$getWorkflowStaticData('global')` (retorna `[]` se msg repetida — mata os retries da Meta). ⚠️ **Multi-statement (`;`) NO NÓ POSTGRES do n8n NÃO executa os 2 statements** — só um. Nunca use; um statement por nó.

**Acesso ao banco (auto_ads):** ⚠️ as creds Supabase em `~/.config/n8n-quirk/` (projeto `gnqxetyrurdpjsnkuhli`) estão **STALE** — pooler responde "tenant/user not found". O banco vivo do auto_ads só é alcançável **via n8n** (credencial Postgres `NKHJwhesMp2Bo4Xw` "Quirk Auto Ads Postgres"). Não dá pra rodar SQL direto daqui. Pra destravar/consertar registro: editar conteúdo de nó (permitido) e reprocessar via webhook `POST https://n8n.quirkgrowth.online/webhook/quirk-auto-ads` com `{message:{type,text,id,from,sender_pn},chat:{phone}}`. Mudar CONEXÕES/roteamento do workflow é barrado pelo classifier (produção).
