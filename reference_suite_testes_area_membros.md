---
name: reference-suite-testes-area-membros
description: "Suíte de testes da área de membros — banco de teste compartilhado que incha e trava o reconcile, cegueira de tipos em tests/, e os stubs de vitest (next/cache, CSS do Payload)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-08-07T13:35:02.611Z
---

Como a suíte da [[project_area_membros_quirk]] se comporta (07/ago/2026: **164 arquivos / 591 testes verdes, ~4min30s**; conferir rodando `NODE_ENV=production npx vitest run` — o `NODE_ENV=production` desliga o `push` do Drizzle e evita o prompt interativo que trava a suíte quando o schema do banco de teste diverge).

⚠️ **SCHEMA DO BANCO DE TESTE DESSINCRONIZA — causa nº1 de "a suíte quebrou inteira".** Rodar com `NODE_ENV=production` desliga o `push` do Drizzle, e é o `push` que sincroniza o schema do banco de teste com o código. Quando OUTRO chat sobe campo/coleção nova, o banco de teste fica pra trás e **dezenas de arquivos falham de uma vez**. Sintoma inconfundível: N erros IDÊNTICOS de `column "<algo>" of relation "<tabela>" does not exist` (07/ago: 56 arquivos / 60 erros por causa de `clientes.status_desde` da home nova). **NÃO é regressão — é schema velho.** Cura: rodar UMA vez um teste de integração SEM `NODE_ENV=production` (`npx vitest run tests/int/clientes.int.spec.ts`) e deixar o push sincronizar; adicionar coluna é aditivo e não abre prompt. Depois voltar ao `NODE_ENV=production`.

**Banco de teste: zera sozinho desde 07/ago (`tests/helpers/globalSetup.ts`).** `DATABASE_URI_TEST` = projeto Neon `ep-rough-thunder` (produção é `ep-patient-hill`; o `vitest.setup.ts` recusa rodar se forem iguais). Antes disso os testes não limpavam nada e o banco acumulava (chegou a **10.347 users + 6.171 clientes**); como `reconcile()` carrega TODOS os members (`pagination:false`, por design) e atualiza um a um, cada teste dele levava **940s** e morria. Agora o `globalSetup` faz `TRUNCATE` de todas as tabelas de `public` exceto `payload_migrations` (`RESTART IDENTITY CASCADE`) **no INÍCIO** da rodada — schema intacto, sem DDL nem re-seed, e a suíte fica determinística. Guardas com teste próprio (`tests/unit/globalSetupGuardas.spec.ts`) + prova por mutação: exige `DATABASE_URI_TEST`, recusa se == `DATABASE_URI`, recusa se estiver no MESMO host de produção; dummy `localhost` do CI é pulado.
⚠️ **Advisory lock SÓ funciona no endpoint DIRETO (sem `-pooler`).** É lock de sessão: pelo PgBouncer a sessão não é fixa, então a trava não segurava E o lock ficou ÓRFÃO numa conexão ociosa do pool, travando toda rodada seguinte pra sempre. Diagnóstico: `select l.pid,a.state from pg_locks l join pg_stat_activity a using(pid) where l.locktype='advisory' and l.objid=918273645`; cura: `pg_terminate_backend(pid)`. Por isso o globalSetup conecta com `-pooler` removido e a espera tem teto de 10 min (segue sem limpar + avisa, nunca trava).

**Checagem de tipos dos testes: `npm run typecheck:tests` (tsconfig.tests.json), desde 07/ago, também no CI.** `tests/` fica FORA do `tsconfig` principal (ruído travava o `next build`), então `tsc --noEmit` sozinho NÃO valida os testes — fixture desatualizado só aparecia como crash em runtime (foi assim que `cliquesLink` derrubou `secao-conta-midia`). Ligar revelou 58 erros represados: fixtures atrás do contrato (`cliquesLink` em 7 fakes de adapter; `LeadCard` do CRM 8 campos atrás — incl. `etiquetas`, que `filtrarRefino` lê com `.includes()`), `users.role`/`clientes.status` dependendo de `defaultValue`, e `CollectionConfig.endpoints` que pode ser `false`. **Ao corrigir erro de tipo em teste: adicionar o campo que falta, nunca `as any` nem enfraquecer asserção** (`toBe` → `toBeDefined` mascara).

**Stubs no `vitest.config.mts` (07/ago):** (1) alias `next/cache` → `tests/helpers/nextCacheStub.ts` (passthrough) — `unstable_cache` exige o runtime do Next e estoura `Invariant: incrementalCache missing` fora de uma request, o que tornava intestável qualquer função de produção cacheada (`percentualVertente`, relatórios, conteúdo); (2) `server.deps.inline: ['react-image-crop', /@payloadcms\//]` — o UI do Payload importa CSS de dentro do `node_modules` e o loader do Node morre com `Unknown file extension ".css"`, matando o arquivo de teste inteiro no import.

**Views do admin em teste:** renderizar `HubClienteView` & cia exige `initPageResult` COMPLETO (importMap/permissions/i18n). Quando o alvo do teste é a guarda (multi-tenant), **stubar o `TelaAdmin`** com `vi.mock` em vez de montar o chrome — senão quebra em `Cannot read properties of undefined (reading '/components/admin/NavQuirk#default')`.

**Rodar em worktree novo:** copiar o `.env` e apontar `node_modules` por symlink pro worktree principal. `npx tsc` pega o tsc errado — usar `./node_modules/.bin/tsc --noEmit`.

⚠️ **Rede:** processos longos contra o Neon caem com `getaddrinfo ENOTFOUND`/`EHOSTUNREACH` (já derrubou 43 arquivos de uma vez, mascarando o resultado real). Se a suíte falhar em massa com erro de conexão, é rede — re-rodar antes de investigar código.
