---
name: project_auto_ads_inadimplencia
description: "Inadimplência automática no Auto Ads — pausa campanhas + bot cobra com link + reativa ao pagar; estado das fases e o que falta"
metadata:
  node_type: memory
  type: project
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-09-01T18:13:00.292Z
---

Automação de inadimplência do [[project_quirk_auto_ads]] (pedido do Renan 01/09): quando um cliente atrasa a mensalidade → **pausar as campanhas** dele na Meta + o **bot cobrar com o link de pagamento** quando ele falar; ao **regularizar** → status volta pra `ativo` e **reativa só as campanhas que o sistema pausou**; e **avisar o time** (WhatsApp suporte **5511936203693**). Bônus: o cancelamento (`inativo`), que hoje é um stub que não pausa, passa a pausar de verdade.

Antes disso o sistema **ignorava `PAYMENT_OVERDUE`** (parse_payment só tratava confirmado→pago e refund/cancel→inativo) e o `pause_campaigns` do gateway era um **TODO stub**.

**Decisões (brainstorming):** status próprio `inadimplente`; reativar automático só as pausadas pelo sistema (lista guardada); avisar o time. Abordagem da pausa: **endpoint interno reusando a lógica testada** (não reimplementar o laço Meta no n8n).

Spec/plano: `quirk_auto_ads/docs/{specs,plans}/2026-08-31-inadimplencia-automatica*`.

## Estado das fases (01/09)
- ✅ **Fase 1 (DDL prod):** status `inadimplente` na constraint + colunas `link_pagamento`, `campanhas_pausadas_inadimplencia` jsonb. Registrada em `quirk_auto_ads/scripts/e_14_inadimplencia_schema.py`.
- ✅ **Fase 2 (bot cobra) — NO AR:** bot `fBUin1UPt5xJEp6g` ganhou `if_inadimplente`+`send_cobranca` (e `if_media_inadimplente`), no mesmo padrão do ramo `removido`. `status=='inadimplente'` → manda cobrança com `{{ $json.link_pagamento }}` e para. 184 nós, ativo.
- 🟡 **Fase 3A (endpoint):** `POST /api/autoads/servico-campanhas` (área de membros, auth `SYNC_SECRET`/`x-sync-secret`), body `{telefone, acao:'pausar'|'reativar'}`. Reusa `pausarCampanhasCliente` + `reativarCampanhasCliente` (nova) + mutations `salvar/ler/limpar PausadasInadimplencia`. **PR #7** (`feat/auto-ads-inadimplencia`), 253 testes verdes. Review adversarial pegou Critical de idempotência (2ª pausa com ids=[] zerava a lista) → **corrigido por união**. BLOQUEADO: CI da main vermelha por teste alheio de relatórios (`topCampanhas`) → fix em **`fix/ci-topcampanhas-objetivo`** (PR a criar/mergear).
- 🟢 **Fase 3B (gateway) — CONSTRUÍDA, NÃO APLICADA:** gateway `2ZnZqb4wFous4uEs` (18→22 nós): parse_payment trata `PAYMENT_OVERDUE`→inadimplente+captura invoiceUrl; upsert com transições (inadimplente + regularização inadimplente→ativo + link) referenciando `$('parse_payment')`; `load_status_atual` (status anterior); switch_action reescrito (welcome/pausar/reativar/noop); switch_router +rota reativar; `http_pausar_endpoint`/`http_reativar_endpoint` chamam o endpoint (cred sync `1cWXZL2pvhbCOyA8`); `if_inadimplencia`→`send_alerta_time` (WhatsApp suporte); stub `pause_campaigns` removido. Artefatos + backup + restore + como-aplicar em `quirk_auto_ads/scripts/inadimplencia_f3b/`.

## O que falta (ordem)
1. Mergear `fix/ci-topcampanhas-objetivo` → main verde.
2. Mergear **PR #7** (após `git merge origin/main` na branch pra herdar o fix) → deploy do endpoint no Render.
3. **Aplicar a Fase 3B** (PUT do `gateway_modificado_f3b.json`) — SÓ depois do endpoint no ar. Backup+verificação+restore prontos.
4. Smoke-test end-to-end (simular OVERDUE→pausa+alerta; RECEIVED→reativa).
5. **Verificar no Asaas** que o evento `PAYMENT_OVERDUE` está habilitado no webhook (hoje só chegam CONFIRMED/RECEIVED — senão o gatilho nunca dispara).

⚠️ Gateway = pipeline de dinheiro ao vivo: aplicar 3B com backup + verificação estrutural + smoke-test (padrão do [[n8n-quirk]]). Ver [[project_painel_auto_ads]] (feature "remover do serviço" reusa `pausarCampanhasCliente`).
