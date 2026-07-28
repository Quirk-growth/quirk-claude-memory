---
name: reference-neon-bancos
description: "Bancos Neon da área de membros/painel Quirk — produção vs teste separados, e a lição da cota compartilhada"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-28T13:50:17.613Z
---

Postgres da [[project-area-membros-quirk]] / [[project-painel-relatorios]] é **Neon**, org "Quirk Growth" (plano **Launch/pago** desde 24/jul/2026).

**Dois projetos Neon SEPARADOS (importante):**
- **Produção:** projeto "Area de Membros - Quirk", endpoint `ep-patient-hill-acxg3aix.sa-east-1` (São Paulo), banco `neondb`.
- **Testes:** projeto "area-membros-testes", host `ep-rough-thunder-acyzl9k2-pooler.sa-east-1` (São Paulo), banco `neondb`. É o `DATABASE_URI_TEST`. Criado 24/jul.

**BLINDAGEM DE ESCALA (28/jul/2026, p/ 50 admin + 300 membros simultâneos):**
- **Runtime de prod usa o endpoint POOLED** do Neon: `DATABASE_URI` no **Render** = `...acxg3aix-pooler.sa-east-1...` (sufixo `-pooler` antes do 1º ponto). O `.env` LOCAL ainda aponta pro endpoint DIRETO (`ep-patient-hill-acxg3aix` sem pooler) — uso só p/ introspecção via `pg`. Antes o runtime estava no direto = gargalo de conexão no pico.
- **`push` desligado em produção:** `payload.config.ts` → `push: process.env.NODE_ENV !== 'production'` (dev/test = ligado; prod = off). Boot limpo/rápido sobre o pooler. **Consequência: TODA coleção nova em prod exige `CREATE TABLE` manual (+ a coluna `<slug>_id` em `payload_locked_documents_rels`) ANTES do deploy ficar Live** — o push não cria mais nada em prod. Pool ajustável via env `DB_POOL_MAX` (default 20).
- **Relatórios cacheados ~5min:** `src/lib/relatorios/montarRelatorio.ts` (`montarDadosRelatorio`) usa `unstable_cache` chave `[relatorio-cliente, clienteId, preset]`, `revalidate:300`, tag `relatorio-<clienteId>` (p/ invalidar por cliente no futuro, ex.: pós-sync). Auth roda na página ANTES → cache por clienteId não vaza entre clientes.

**LIÇÃO (incidente 24/jul):** antes, o banco de teste (`neondb_test`) ficava no MESMO projeto Neon da produção. A **cota de transferência do Neon é POR PROJETO** — então rodar a suíte (dezenas de `npx vitest run`, cada um puxa schema + ~142 testes de integração pela rede) consumiu a cota compartilhada e **derrubou a produção** (todas as rotas que tocam o banco → 500 "exceeded the data transfer quota"; só `/login`, que é client component, respondia). Resolvido: Renan subiu pro plano Launch (volta na hora) + separamos o teste num projeto Neon próprio. `vitest.setup.ts` já recusa rodar se `DATABASE_URI_TEST === DATABASE_URI`.

**Regra prática:** teste nunca deve compartilhar projeto Neon com produção. Ideal futuro: Postgres local (Renan recusou instalar Postgres.app em jul/26). Sem Postgres/Docker/brew na máquina do Renan.

**Pendência de higiene:** a senha do projeto de teste (`npg_...`) passou pelo chat — rotacionar no Neon (Reset password do projeto area-membros-testes). Não urgente (banco descartável, sem dado real).

**Regra de credencial mantida:** eu NÃO escrevo strings de conexão com senha / tokens no `.env` — o Renan cola. (String de teste tem senha embutida = credencial.)
