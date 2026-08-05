---
name: reference_reconcile_acesso
description: "Sync de reconciliação de acesso (área de membros): espelha a fonte por email e desativa quem não está nela; agora protege login de cliente vivo"
metadata:
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-05T13:59:53.457Z
---

O **sync de reconciliação** da [[project_area_membros_quirk]] (`src/lib/reconcile.ts` + `reconcileDiff.ts`, roda no cron diário) é um **espelho da fonte de clientes por EMAIL**: `computeDiff(atuais, desejados)` cria/reativa quem está na lista desejada e **desativa todo login `role=member` ATIVO cujo email não está na fonte**. Trava anti-zeragem: aborta se os desejados < 50% dos ativos atuais (`SYNC_MIN_RATIO`).

**Gotcha que mordeu (ago/2026):** um cliente convidado manualmente pelo CRM (ex.: jessica.schirato@vivaurban.com.br, cliente Urban Incorporadora) foi **desativado no dia seguinte ao convite** porque o email dela não estava na lista da fonte. Ao tentar definir a senha, o login dispara `recusarLoginInativo` → **"seu acesso está inativo. Fale com a Quirk"**. **Fix manual:** `UPDATE users SET ativo=true WHERE id=<id> AND role='member'` (ou reativar no admin).

**Fix definitivo (aplicado, escolha do Renan "nunca desativar cliente ativo"):** `computeDiff` ganhou um param `protegidos: Set<id>`; o reconcile monta esse set com os `member` ligados a um cliente com status **vivo** (`STATUS_ATIVOS` = tudo menos `churn`) e nunca os desativa, mesmo fora da fonte. **Contrapartida:** se um cliente sai mas ninguém marca o status dele como `churn`, o acesso persiste — então marcar churn no hub é o que corta o acesso agora. Contas de teste (`contaDeTeste=true`) já eram protegidas do mesmo jeito.
