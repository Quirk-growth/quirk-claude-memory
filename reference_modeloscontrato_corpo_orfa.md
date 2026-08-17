---
name: reference-modeloscontrato-corpo-orfa
description: "RESOLVIDO (14/08) — coluna órfã `corpo` em modelos_contrato dropada; DATABASE_URI local É produção, não dev separado"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-17T22:16:31.773Z
---

**Resolvido em 14/08/2026.** `src/collections/ModelosContrato.ts` não tem mais campo `corpo` desde o commit `4846dbf` (troca do fluxo de contrato de Puppeteer pra Google Docs — hoje usa `googleDocId`, ver [[reference_contratos_autentique]]), mas a coluna `corpo` tinha ficado órfã no banco com 1 registro.

**Achado importante nesse dia**: `DATABASE_URI` do `.env` local (em QUALQUER worktree, ex: `area-membros-quirk-crm`) aponta direto pro banco de **produção** (host `ep-patient-hill-acxg3aix`) — não existe um 3º projeto Neon de "dev" separado, só prod e teste (`ep-rough-thunder-acyzl9k2`, usado só pelo Vitest via `DATABASE_URI_TEST`). Confirmado cruzando com `tests/unit/globalSetupGuardas.spec.ts`, que usa esse mesmo host como o exemplo hardcoded de produção. Ou seja: **rodar `npm run dev` localmente conecta direto em prod** — qualquer prompt de schema push (`push: true` no Payload/Drizzle) que apareça no boot é uma mudança real de produção, nunca "só dev".

O que foi feito: antes de dropar, o conteúdo da coluna foi conferido (não era lixo — tinha o texto completo do "Contrato Principal — Tráfego", o modelo antigo pré-migração pro Google Docs) e salvo em backup local (`~/Desktop/backup_modelos_contrato_corpo_20260814.json`) via leitura direta com `pg` antes do `ALTER TABLE modelos_contrato DROP COLUMN corpo`. Coluna confirmada sem uso em nenhum código atual (substituída por `googleDocId`, required). `npm run dev` volta a subir normal depois disso.

**Se aparecer de novo outro prompt de DATA LOSS no boot do dev**: mesma regra de sempre — nunca mandar `y` sem entender o que a coluna contém e sem confirmação explícita do Renan, porque é sempre produção real, não um banco de brinquedo. Matar o processo (`lsof -ti:3000 | xargs kill -9`), checar o conteúdo da coluna antes de propor apagar, e SE for aprovado, fazer backup do conteúdo num arquivo antes do DROP.

**Reforço (17/08, sessão SDD do programador de posts):** um subagente implementador tentou `npm run dev` só pra verificação visual (não mexendo em schema de propósito) e o push automático quase dropou `crm_limite_whatsapps` (121 linhas em `clientes`) — matou o processo a tempo, sem aprovar nada. Conferido depois: coluna intacta, 121/121 clientes com valor, `/api/health` 200 — nenhum dado perdido. Confirma que o risco não é só em sessões que MEXEM em schema: qualquer `npm run dev` neste repo é perigoso por padrão, inclusive quando a intenção é só olhar a tela. Verificação visual segura = banco de TESTE (`diag-dev-teste.mjs`/`DATABASE_URI_TEST`) ou produção real via HTTPS (nunca `npm run dev` local pra "só ver").
