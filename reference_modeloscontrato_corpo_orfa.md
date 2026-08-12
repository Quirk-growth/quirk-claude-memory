---
name: reference-modeloscontrato-corpo-orfa
description: "GOTCHA — coluna órfã `corpo` em modelos_contrato trava npm run dev local com prompt de DATA LOSS"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-12T01:16:11.635Z
---

`src/collections/ModelosContrato.ts` não tem mais campo `corpo` desde o commit `4846dbf` (troca do fluxo de contrato de Puppeteer pra Google Docs), mas a coluna `corpo` ficou órfã no banco com 1 registro.

Como o dev local (`npm run dev`, `area-membros-quirk-4`) roda com `push: true` no Payload/Drizzle, TODO boot do `next dev` tenta reconciliar o schema e trava num prompt interativo:
```
Warnings detected during schema push:
· You're about to delete corpo column in modelos_contrato table with 1 items
DATA LOSS WARNING: Possible data loss detected if schema is pushed.
Accept warnings and push schema to database? (y/N)
```

O processo fica parado esperando stdin — qualquer `navigate`/`preview_start` no navegador trava (timeout), porque o Next nunca termina de subir.

**Nunca mandar `y`** — é apagar dado sem saber se ainda importa, decisão do Renan, não minha (nem com confirmação — apagar dado é ação proibida). Se acontecer de novo: matar o processo (`lsof -ti:3000 | xargs kill -9`), não confirmar nada, e avisar antes de tentar rodar o dev de novo.

**Como resolver de vez** (fica pro Renan decidir, ou eu faço se ele pedir explicitamente): ou apagar a coluna `corpo` de `modelos_contrato` no banco (dado antigo do fluxo Puppeteer, já não é lido em lugar nenhum do código), ou aceitar o prompt uma vez — mas isso é decisão de apagar dado, sempre com confirmação explícita antes.

Enquanto isso não for resolvido, verificação visual de mudanças de UI no dev local fica bloqueada — testes automatizados (Vitest, unit+int) continuam funcionando normalmente (não passam pelo push interativo).
