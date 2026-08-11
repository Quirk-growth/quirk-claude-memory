---
name: feedback_acompanhar_deploy_ate_verde
description: "Em deploy de prod, acompanhar o Render até o health check verde antes de dizer \"no ar\" — não declarar cedo"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-11T00:54:59.691Z
---

Num deploy de produção da área de membros, dei `git push` e declarei "no ar" logo depois, sem acompanhar o Render. O deploy subiu quebrado (OOM → 502) e quem descobriu foi o Renan, não eu.

**Why:** "push feito" ≠ "no ar e saudável". O Render ainda builda + faz swap + o processo pode crashar no boot/runtime (OOM, env, etc.). Declarar cedo transfere a descoberta do incidente pro cliente — o pior lugar.

**How to apply:** depois de todo push pra `main` (deploy de prod), acompanhar até: (1) o deploy ficar **live/verde** nos Events do Render, E (2) o site manter **200 estável** por alguns minutos após o swap (curl em loop pega 502/OOM que volta). Só então dizer que está pronto. Se aparecer 502, rollback na hora ([[reference_render_memoria_oom]]). Vale pro padrão geral de deploy da Quirk ([[reference_github_backup]]).
