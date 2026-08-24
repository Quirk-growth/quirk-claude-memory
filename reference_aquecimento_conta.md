---
name: reference_aquecimento_conta
description: "Auto Ads: aquecimento de conta nova (novato) com campanha de tráfego antes da 1ª campanha de mensagens"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-24T20:27:44.271Z
---

Feature do [[project_quirk_auto_ads]] (workflow `fBUin1UPt5xJEp6g`) pra proteger conta NOVA de novato contra bloqueio: roda uma campanha de **tráfego (visitas ao perfil/página)** por ~4 dias ANTES de liberar a 1ª campanha de mensagens. Deploy 24/ago/2026 (scripts `e_20`–`e_23`), spec+plano em `quirk_auto_ads/docs/`. Só ataca **anti-bloqueio** (decisão Renan) — não é CPM/verificação.

**Fluxo do novato (verificado end-to-end na Ignite):**
1. Onboarding confirmado → precheck ([[reference_onboarding_precheck_desfecho]]) verde → ativa → seta `aquecimento={status:'perguntando_idade'}` + pergunta "nova ou antiga?".
2. Intent **WARM_ONBOARD** (classify_intent early-return quando `aquecimento.status` é `perguntando_*`) → handler `warm_onboard_step`: antiga→`nao_precisa`; nova→`perguntando_cidade` (pede cidade); cidade→`aquecendo_pendente`+`city_nome` (pede criativo).
3. Criativo chega → rota de mídia: `warm_media_gate` detecta `aquecendo_pendente`+`city_nome` → `if_warm_media` → **warm_gather → …sub-fluxo…** (Task 3): `warm_d1_campaign`(OUTCOME_TRAFFIC) → `warm_d2_adset`(cidade, LINK_CLICKS, R$10/dia) → `warm_prep/upload/creative`(link ad pra fanpage, CTA LEARN_MORE) → `warm_d4_ad` → grava em `auto_ads.campanhas` ("Aquecimento — visitas ao perfil") → `aquecimento={status:'aquecendo',campaign_id,libera_em:+4d}` → avisa. Campanha sobe **ATIVA**; aparece no STATUS; cliente pausa se quiser.
4. Durante `aquecendo`: intent **WARM_ATIVO** barra CONFIRMAR/SUBIR_DENOVO/NOVA_CAMPANHA com contagem (STATUS/PAUSAR livres).
5. Dia 4: workflow agendado **"Quirk Auto Ads — Liberação Aquecimento"** `HboO7SGWY1iCJVXj` (Schedule 3h) → `aquecendo` com `libera_em<=NOW()` → avisa + `status='concluido'` (NÃO pausa a campanha).

**Estado:** coluna `auto_ads.clientes.aquecimento` jsonb — máquina: `perguntando_idade → perguntando_cidade → aquecendo_pendente → aquecendo → concluido` (ou `nao_precisa`).

**Gotchas validados:** cidade no targeting é `cities:[{key}]` **SEM raio** (raio dá subcode 1487110); geo resolve por mapa hardcoded (28 cidades) + `search?type=adgeolocation` fallback (Pomerode→264554); warm creative = meta_d3 trocando CTA `WHATSAPP_MESSAGE`→`LEARN_MORE`(link `facebook.com/{page_id}`); nó Postgres substitui o item (code lê dados por `$('no')`, não `$input`). Enriquecimento de intents é aditivo (padrão SALDO), não reroute → classifier não barra.
