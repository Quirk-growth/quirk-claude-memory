---
name: reference-preview-start-worktree-errado
description: "GOTCHA — preview_start com name (ex. \"membros-teste\") pode abrir o dev server no worktree ERRADO quando há múltiplos checkouts irmãos (area-membros-quirk, -crm, -4, -blindagem)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-09T01:41:45.832Z
---

`mcp__Claude_Browser__preview_start` com `{name: "membros-teste"}` resolve o `.claude/launch.json` a partir de um cwd que **não é necessariamente** o diretório onde o Bash tool está operando. Com múltiplos worktrees irmãos do mesmo repo (`/Users/renanreal/area-membros-quirk`, `-crm`, `-4`, `-blindagem` — ver [[reference_github_backup]]), ele pode subir o servidor no worktree ERRADO (ex.: `area-membros-quirk` sem sufixo, numa branch antiga qualquer) mesmo estando trabalhando em `area-membros-quirk-crm`.

**Sintoma:** a UI mostra comportamento/menu ANTIGO mesmo com o código correto no disco do repo em que você está — `.next` limpo, hard reload, aba nova, nada resolve, porque o processo real está servindo outro diretório inteiro.

**Diagnóstico rápido:** `lsof -i :3000 -sTCP:LISTEN -P` pra achar o PID, depois `lsof -a -p <PID> -d cwd` (ou `ps -o pid,command -p <PID>` e conferir o path do `next dev`/`diag-dev-teste.mjs` no comando) — confirma de qual diretório o processo realmente está servindo.

**Fix:** matar o processo errado; subir o servidor manualmente via Bash com cwd explícito no diretório certo (ex. `node diag-dev-teste.mjs &` de dentro do repo certo, `disown`); anexar o Browser pane via `preview_start` com `{url: "http://localhost:3000/..."}` (não `{name: ...}`) pra evitar a resolução ambígua de novo.

**Como aplicar:** sempre que fizer verificação visual num repo que tem worktrees irmãos, confirmar o cwd do processo ANTES de confiar no que a tela mostra — principalmente se algo "não muda" depois de um fix que você tem certeza que está no código.
