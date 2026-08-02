---
name: reference_gestao_lista_reconciliacao
description: "Gestão de campanhas Auto Ads: lista vem do banco + reconcilia com Meta ao vivo; drift quando cliente edita direto no Gerenciador"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-02T03:29:54.840Z
---

Gestão de campanhas do [[project_quirk_auto_ads]] (STATUS/PAUSAR/REATIVAR/ENCERRAR/ALTERAR_*): a lista é montada por `list_campanhas` (Postgres, lê `auto_ads.campanhas` filtrando por telefone+ad_account_id+status, `ORDER BY criada_em DESC LIMIT 50`) → `init_gestao` (monta lista_candidatas) → `build_gestao_response`.

**Problema resolvido (01/ago):** o banco local **drifta do Meta**. Quando o cliente deleta/arquiva campanha DIRETO no Gerenciador de Anúncios, o banco não sabe → lista defasada (mostrava 12, no Meta só 4). Fix (`bda80de`): `list_campanhas` leva o `meta_token` junto (subquery em `auto_ads.config` chave `meta_access_token`); `init_gestao` virou async e chama `GET /act_{ad_account_id}/campaigns?fields=id,effective_status&limit=500` e **filtra** as que não existem mais no Meta ou estão DELETED/ARCHIVED. Com try/catch: se o Meta cair, mostra a lista do banco (não quebra). Validado: 12 no banco → 4 vivas.

**Follow-ups RESOLVIDOS (02/ago):** (1) **status ao vivo** — `init_gestao` agora sobrescreve o status exibido com o `effective_status` do Meta (some o CREATED_ACTIVE defasado). (2) **Duplicatas** — não era bug de criação (cada CONFIRMADO = 1 campanha nova, correto); as dupes vinham dos **retries da Meta** (já barrados pelo dedup de wamid no `normalize_phone`) e as fantasmas somem pela reconciliação. (3) **Sync periódico** — criado workflow ISOLADO `Quirk Auto Ads — Sync Campanhas (Meta<->DB)` id **`U2CYzx24OLWpW8Yo`** (Schedule 15min → SELECT campanhas de clientes ativos + token → Code busca Meta por conta → Postgres UPDATE **per-item** status/DELETED). ⚠️ n8n Postgres roda a query 1×/item (per-item OK), mas **multi-statement e VALUES-em-lote não** — usar per-item. Guarda: se Meta retorna 0 campanhas, não marca nada (evita falso-delete). Validado: 34 mudanças aplicadas → run seguinte 0.

**Layout do workflow principal** reorganizado (02/ago) em **6 bandas funcionais** com sticky notes (Entrada/Onboarding/Criação/Gestão/Status/Mídia); backup de posições em scratchpad `positions_backup.json`. Só mudou position + notas (execução idêntica). `update_workflow` da API pública NÃO desativa o workflow nem desregistra webhook em mudanças de conteúdo/posição (só mudança de webhook exige deactivate+activate).
