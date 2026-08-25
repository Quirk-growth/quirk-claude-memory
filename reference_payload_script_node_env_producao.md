---
name: reference-payload-script-node-env-producao
description: "área de membros Quirk: script local via getPayload() precisa de NODE_ENV=production ao rodar contra o DATABASE_URI de produção, senão o push automático do Payload tenta alterar schema de prod"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-25T16:47:16.149Z
---

Na [[project-area-membros-quirk]], `postgresAdapter({ push: NODE_ENV !== 'production' })` em `payload.config.ts` — então QUALQUER script local que chama `getPayload({config})` (ex.: `npx tsx algum-script.ts`, mesmo um `diag-*.ts` de uma linha) dispara o **push automático de schema** se `NODE_ENV` não estiver setado como `'production'`, mesmo que o `.env` local aponte pro `DATABASE_URI` de PRODUÇÃO (que é o padrão deste repo — não há banco de dev separado).

**Quase incidente real (25/ago/2026):** rodei um script pra publicar uma aula nova (só um `payload.create`) e o Payload, antes de qualquer coisa, tentou fazer `pull schema from database` → `push schema` contra produção, com prompt interativo pedindo confirmação pra **dropar a tabela `guia_pastas` (4 itens) e 2 colunas de `guia_paginas` (21 itens)** — perda de dados real, quase aceita sem querer num processo em background. Matei o processo (`TaskStop`) sem nunca responder o prompt, confirmei via SQL direto que nada foi alterado, e re-rodei com `NODE_ENV=production npx tsx script.ts` — sem nenhum prompt, escreveu só o que era pra escrever.

**Regra:** todo script local de uso único que usa `getPayload()` contra o `DATABASE_URI` padrão (produção) deste repo precisa rodar com `NODE_ENV=production` na frente. Isso é diferente do gotcha já conhecido de "`npm run dev` aponta pra produção" — aqui o risco é justamente o OPOSTO de dev: é o push de schema achando que está em dev (por causa do NODE_ENV ausente) mesmo mirando o banco real.
