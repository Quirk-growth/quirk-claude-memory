---
name: reference-conta-teste-demo
description: "renandmreal@gmail.com é conta de teste/demo protegida na área de membros — fora do sync, não desativa nem exclui"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-29T17:19:12.005Z
---

Na [[project-area-membros-quirk]], `renandmreal@gmail.com` (user id 518, role `member`, nome "Renan D.") é a **conta de demonstração** do Renan — ele usa pra apresentar o painel a clientes. NÃO é o admin dele (contato@quirkgrowth.com.br) nem o gestor "Renan DM." (id 519).

**Proteção** (29/jul/2026): campo `contaDeTeste` (checkbox, só admin edita) em `users`. Quando `true`:
- O **sync de membros** (`reconcile` em `src/lib/reconcile.ts`, alimentado pela Lista de Clientes do ClickUp via n8n → POST /api/sync/reconcile) **ignora** a conta: nunca desativa e não conta na trava anti-zeragem. Filtro é em JS (`.contaDeTeste !== true`) de propósito — `not_equals: true` no SQL excluiria os NULL (a maioria).
- `protegerExclusaoUsuario` **bloqueia a exclusão** (tem que desmarcar "Conta de teste" antes).

Coluna `conta_de_teste boolean` já em prod (DDL aplicado). Se essa conta aparecer como "inativa" de novo, é bug — ela deveria estar sempre ativa.
