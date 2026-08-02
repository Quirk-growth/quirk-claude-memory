---
name: reference_gestao_lista_reconciliacao
description: "Gestão de campanhas Auto Ads: lista vem do banco + reconcilia com Meta ao vivo; drift quando cliente edita direto no Gerenciador"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-02T02:47:24.285Z
---

Gestão de campanhas do [[project_quirk_auto_ads]] (STATUS/PAUSAR/REATIVAR/ENCERRAR/ALTERAR_*): a lista é montada por `list_campanhas` (Postgres, lê `auto_ads.campanhas` filtrando por telefone+ad_account_id+status, `ORDER BY criada_em DESC LIMIT 50`) → `init_gestao` (monta lista_candidatas) → `build_gestao_response`.

**Problema resolvido (01/ago):** o banco local **drifta do Meta**. Quando o cliente deleta/arquiva campanha DIRETO no Gerenciador de Anúncios, o banco não sabe → lista defasada (mostrava 12, no Meta só 4). Fix (`bda80de`): `list_campanhas` leva o `meta_token` junto (subquery em `auto_ads.config` chave `meta_access_token`); `init_gestao` virou async e chama `GET /act_{ad_account_id}/campaigns?fields=id,effective_status&limit=500` e **filtra** as que não existem mais no Meta ou estão DELETED/ARCHIVED. Com try/catch: se o Meta cair, mostra a lista do banco (não quebra). Validado: 12 no banco → 4 vivas.

**Ainda em aberto (follow-ups):** (1) o **status exibido** ainda é o do banco (pode mostrar CREATED_ACTIVE quando o Meta já está PAUSED) — e o FILTRO de status do `list_campanhas` também usa o banco, então REATIVAR pode não pegar uma pausada-no-Meta. Fix real = sync do status Meta→banco (precisa de write, hoje só via nó Postgres). (2) **Duplicatas** no banco (mesma campanha 2-3×, ex "Casa Mooca") — algo cria linha nova em vez de reusar (provável no "subir denovo"/NOVA_CAMPANHA); inflam a lista. (3) O ideal seria um **workflow de sync periódico** (isolado) que reconcilia banco↔Meta (marca deletadas, atualiza status) — resolveria 1 e 2 de vez. Anexar em `~/.config/n8n-quirk/`: token de teste local funciona pra listar campanhas do Meta (mesmo System User).
