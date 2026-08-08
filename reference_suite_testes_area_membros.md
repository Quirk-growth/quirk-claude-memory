---
name: reference-suite-testes-area-membros
description: "Suíte de testes da área de membros — schema do banco de teste dessincroniza e derruba tudo (causa nº1), globalSetup que zera o banco, checagem de tipos dos testes, e os stubs de vitest"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-08-07T13:55:30.658Z
---

Como a suíte da [[project_area_membros_quirk]] se comporta. **Rodar com `npx vitest run` PURO (= `npm test`). NÃO usar `NODE_ENV=production`** — em 07/ago/2026 a suíte fecha **732/732** do jeito certo, e as duas falhas que eu achava serem "da home nova" eram artefato desse env. Antes de culpar sua mudança por qualquer falha, **rode o mesmo arquivo em `git switch --detach origin/main`** — foi assim que separei o que era meu do que já estava quebrado.

⚠️ **Worktree com `node_modules` por symlink: dependência nova de outro chat some.** Quando alguém adiciona pacote (ex.: `ical-expander` da agenda), o seu worktree simbolicamente ligado ao `node_modules` do principal falha com `Cannot find package '<x>' imported from ...` em VÁRIOS arquivos de uma vez. Não é código: rodar `npm install` no worktree PRINCIPAL resolve para todos.
⚠️ **Um banco de teste para N chats com schemas divergentes gera atrito.** O push do Drizzle alinha o banco ao branch que está rodando, então branch com coleção que os outros não têm faz o próximo rodar tentar DROPar constraint que já sumiu (`constraint "..." of relation "payload_locked_documents_rels" does not exist`). Costuma passar na 2ª tentativa (o push já acertou o estado). Se virar rotina, o caminho é um banco por worktree.

⚠️ **`NODE_ENV=production` na suíte QUEBRA DUAS COISAS — não use** (eu recomendei isso por um tempo e custou caro). Ele desliga o `push` do Drizzle e faz o Vite carregar o build de PRODUÇÃO do React:
1. **Schema dessincroniza** → quando outro chat sobe campo/coleção nova, o banco de teste fica pra trás e **dezenas de arquivos falham de uma vez**, com N erros IDÊNTICOS de `column "<algo>" of relation "<tabela>" does not exist` (07/ago: 56 arquivos por causa de `clientes.status_desde`). Parece catástrofe, é schema velho.
2. **`act` some** → no React 19 o `act` só existe no build de DEV; testes de componente com DOM morrem em `act is not a function` (`tests/unit/fila-de-acao.spec.tsx`).
Com `push` ligado (o padrão) o schema sincroniza sozinho a cada rodada e os dois sintomas desaparecem. O motivo original de desligar era o prompt interativo do Drizzle — mas ele só aparece em diff DESTRUTIVO (rename/drop); adicionar coluna passa em silêncio.

**Banco de teste: zera sozinho desde 07/ago (`tests/helpers/globalSetup.ts`).** `DATABASE_URI_TEST` = projeto Neon `ep-rough-thunder` (produção é `ep-patient-hill`; o `vitest.setup.ts` recusa rodar se forem iguais). Antes disso os testes não limpavam nada e o banco acumulava (chegou a **10.347 users + 6.171 clientes**); como `reconcile()` carrega TODOS os members (`pagination:false`, por design) e atualiza um a um, cada teste dele levava **940s** e morria. Agora o `globalSetup` faz `TRUNCATE` de todas as tabelas de `public` exceto `payload_migrations` (`RESTART IDENTITY CASCADE`) **no INÍCIO** da rodada — schema intacto, sem DDL nem re-seed, e a suíte fica determinística. Guardas com teste próprio (`tests/unit/globalSetupGuardas.spec.ts`) + prova por mutação: exige `DATABASE_URI_TEST`, recusa se == `DATABASE_URI`, recusa se estiver no MESMO host de produção; dummy `localhost` do CI é pulado.
⚠️ **Advisory lock SÓ funciona no endpoint DIRETO (sem `-pooler`).** É lock de sessão: pelo PgBouncer a sessão não é fixa, então a trava não segurava E o lock ficou ÓRFÃO numa conexão ociosa do pool, travando toda rodada seguinte pra sempre. Diagnóstico: `select l.pid,a.state from pg_locks l join pg_stat_activity a using(pid) where l.locktype='advisory' and l.objid=918273645`; cura: `pg_terminate_backend(pid)`. Por isso o globalSetup conecta com `-pooler` removido e a espera tem teto de 10 min (segue sem limpar + avisa, nunca trava).

**Checagem de tipos dos testes: `npm run typecheck:tests` (tsconfig.tests.json), desde 07/ago, também no CI.** `tests/` fica FORA do `tsconfig` principal (ruído travava o `next build`), então `tsc --noEmit` sozinho NÃO valida os testes — fixture desatualizado só aparecia como crash em runtime (foi assim que `cliquesLink` derrubou `secao-conta-midia`). Ligar revelou 58 erros represados: fixtures atrás do contrato (`cliquesLink` em 7 fakes de adapter; `LeadCard` do CRM 8 campos atrás — incl. `etiquetas`, que `filtrarRefino` lê com `.includes()`), `users.role`/`clientes.status` dependendo de `defaultValue`, e `CollectionConfig.endpoints` que pode ser `false`. **Ao corrigir erro de tipo em teste: adicionar o campo que falta, nunca `as any` nem enfraquecer asserção** (`toBe` → `toBeDefined` mascara).

**Stubs no `vitest.config.mts` (07/ago):** (1) alias `next/cache` → `tests/helpers/nextCacheStub.ts` (passthrough) — `unstable_cache` exige o runtime do Next e estoura `Invariant: incrementalCache missing` fora de uma request, o que tornava intestável qualquer função de produção cacheada (`percentualVertente`, relatórios, conteúdo); (2) `server.deps.inline: ['react-image-crop', /@payloadcms\//]` — o UI do Payload importa CSS de dentro do `node_modules` e o loader do Node morre com `Unknown file extension ".css"`, matando o arquivo de teste inteiro no import.

**Views do admin em teste:** renderizar `HubClienteView` & cia exige `initPageResult` COMPLETO (importMap/permissions/i18n). Quando o alvo do teste é a guarda (multi-tenant), **stubar o `TelaAdmin`** com `vi.mock` em vez de montar o chrome — senão quebra em `Cannot read properties of undefined (reading '/components/admin/NavQuirk#default')`.

**Rodar em worktree novo:** copiar o `.env` e apontar `node_modules` por symlink pro worktree principal. `npx tsc` pega o tsc errado — usar `./node_modules/.bin/tsc --noEmit`.

⚠️ **Rede:** processos longos contra o Neon caem com `getaddrinfo ENOTFOUND`/`EHOSTUNREACH` (já derrubou 43 arquivos de uma vez, mascarando o resultado real). Se a suíte falhar em massa com erro de conexão, é rede — re-rodar antes de investigar código.
