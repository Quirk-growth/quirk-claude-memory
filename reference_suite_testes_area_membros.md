---
name: reference-suite-testes-area-membros
description: "Suíte de testes da área de membros — banco de teste compartilhado que incha e trava o reconcile, cegueira de tipos em tests/, e os stubs de vitest (next/cache, CSS do Payload)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-08-07T08:09:13.042Z
---

Como a suíte da [[project_area_membros_quirk]] se comporta (07/ago/2026: **164 arquivos / 591 testes verdes, ~4min30s**; conferir rodando `NODE_ENV=production npx vitest run` — o `NODE_ENV=production` desliga o `push` do Drizzle e evita o prompt interativo que trava a suíte quando o schema do banco de teste diverge).

**Banco de teste compartilhado INCHA e trava o `reconcile`.** `DATABASE_URI_TEST` = projeto Neon `ep-rough-thunder` (produção é `ep-patient-hill` — o `vitest.setup.ts` recusa rodar se forem iguais). Os testes **não limpam o que criam** e todos os chats paralelos escrevem no mesmo banco. Em 07/ago tinha **10.347 users + 6.171 clientes** acumulados; como `reconcile()` carrega TODOS os members (`pagination:false`, por design) e atualiza um a um, cada teste dele levava **940s** e morria de timeout/conexão. Sintoma típico: dezenas de `ERROR: Falha ao criar <email> ValidationError: email` + testes de reconcile estourando.
**Limpeza (aprovada pelo Renan em 07/ago):** `TRUNCATE` de todas as tabelas de `public` exceto `payload_migrations`, com `RESTART IDENTITY CASCADE`, via `pg` direto. Schema fica intacto → **não precisa DDL nem re-seed**. Guardas obrigatórias no script antes do TRUNCATE: URI de teste existe, ≠ `DATABASE_URI`, host ≠ host de produção, e host casa `/rough-thunder/`. Todos os emails do banco de teste são sintéticos (`teste.com`, `x.com`, `t.com`, `ex.com`, `quirk.test`, `quirk.local`) — se aparecer domínio real, PARAR e conferir.
⚠️ **Volta a inchar** — é paliativo. Correção de raiz (backlog): cada teste limpar os próprios registros.

**Cegueira de tipos:** `tests/` fica FORA do `include` do `tsconfig` (pra não travar o `next build`), então **`tsc --noEmit` NÃO valida os testes** — fixture desatualizado só aparece como crash em runtime. Foi assim que o KPI `cliquesLink` novo passou batido e derrubou `secao-conta-midia`.

**Stubs no `vitest.config.mts` (07/ago):** (1) alias `next/cache` → `tests/helpers/nextCacheStub.ts` (passthrough) — `unstable_cache` exige o runtime do Next e estoura `Invariant: incrementalCache missing` fora de uma request, o que tornava intestável qualquer função de produção cacheada (`percentualVertente`, relatórios, conteúdo); (2) `server.deps.inline: ['react-image-crop', /@payloadcms\//]` — o UI do Payload importa CSS de dentro do `node_modules` e o loader do Node morre com `Unknown file extension ".css"`, matando o arquivo de teste inteiro no import.

**Views do admin em teste:** renderizar `HubClienteView` & cia exige `initPageResult` COMPLETO (importMap/permissions/i18n). Quando o alvo do teste é a guarda (multi-tenant), **stubar o `TelaAdmin`** com `vi.mock` em vez de montar o chrome — senão quebra em `Cannot read properties of undefined (reading '/components/admin/NavQuirk#default')`.

**Rodar em worktree novo:** copiar o `.env` e apontar `node_modules` por symlink pro worktree principal. `npx tsc` pega o tsc errado — usar `./node_modules/.bin/tsc --noEmit`.

⚠️ **Rede:** processos longos contra o Neon caem com `getaddrinfo ENOTFOUND`/`EHOSTUNREACH` (já derrubou 43 arquivos de uma vez, mascarando o resultado real). Se a suíte falhar em massa com erro de conexão, é rede — re-rodar antes de investigar código.
