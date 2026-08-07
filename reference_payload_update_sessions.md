---
name: reference-payload-update-sessions
description: "GOTCHA crítico da área de membros: payload.update num doc de Users (useSessions:true) reescreve users_sessions e ZERA as sessões → desloga a pessoa. Salvar campo self-service (preferências) por write só-na-coluna, não payload.update."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-07T11:13:56.809Z
---

**Bug real (06/ago, prod, deslogou o Renan):** o `Users` tem `auth: { useSessions: true }` → as sessões vivem na tabela **`users_sessions`** (array field). Quando se faz `payload.update({ collection: 'users', id, data: {...} })` no documento do usuário — **mesmo com `overrideAccess: true` e `data` parcial** — o Payload reescreve o array `sessions`, e na prática **ZERA `users_sessions`**. Efeito: a sessão da pessoa morre → a próxima navegação vem sem sessão → erro 401 "Você deve estar logado" durante o SSR → o admin **desloga**.

Como apareceu: a feature de **Tema/Preferências** salvava a preferência (tema, visualização de tarefas) via `POST /api/users/preferencias` → `payload.update` no próprio user. Trocar o tema **deslogava a própria pessoa** toda vez. O 401 vinha em `.next/server/chunks/ssr/...` via `processTicksAndRejections` (não era rota de API — as rotas custom retornam 403, não 401).

**Fix (commit 7ce2b71 — NO AR e VERIFICADO pelo Renan em prod 06/ago: trocar tema não desloga mais):** gravar só a coluna, sem tocar nas sessões:
```ts
const pool = (req.payload.db as unknown as { pool: { query: (t: string, p: unknown[]) => Promise<unknown> } }).pool
await pool.query('UPDATE users SET preferencias = $1::jsonb, updated_at = now() WHERE id = $2', [JSON.stringify(valor), user.id])
```
(`payload.db.pool` é o pg Pool do adapter `@payloadcms/db-postgres`, criado em `connect.js`. `drizzle-orm` `sql` + `payload.db.drizzle.execute` também servem.)

**Regra geral:** qualquer update self-service num `users` (preferências, flags de UI, último-visto…) deve ser um **write direto na coluna** via `pool.query`/drizzle, NUNCA `payload.update` no doc — senão desloga. ⚠️ **Pendência:** há OUTRO `payload.update` em `users` (Users.ts ~L242, fluxo de vínculo de acesso, grava `cliente`) que tem o mesmo risco de derrubar a sessão do alvo — avaliar/migrar pro mesmo padrão.

**Meta-lição (multi-chat):** vários chats subindo pra MESMA prod em paralelo (inbox + reenviar-convite + tema/prefs) — o `origin/main` andou 4× durante um debug (`7a25578`→`bf2d901`→`d825176`→`9e2f313`→`7ce2b71`), bloqueou um rollback e confundiu a atribuição do bug. Ver [[feedback-memoria-multichat]]: conferir `origin/main` sempre; e **pausar deploys dos outros chats durante um incidente de prod**. Relacionado: [[project_area_membros_quirk]], [[reference_neon_bancos]].
