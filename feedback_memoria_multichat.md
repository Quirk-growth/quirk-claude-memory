---
name: feedback-memoria-multichat
description: "Protocolo pra memória compartilhada entre chats paralelos deste projeto — é UMA memória só (git + backup auto); re-ler antes de escrever, edições cirúrgicas, e fatos voláteis apontam COMO verificar em vez de cravar valor"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-06T22:35:51.094Z
---

Vários chats do Claude Code rodam **em paralelo neste projeto** e compartilham a **MESMA** pasta de memória (`~/.claude/projects/-Users-renanreal/memory/`), versionada em git com backup automático (hook `Stop` + hook `PostToolUse` por-escrita, ambos `git add -A && commit && push origin main`). **Não são "2 memórias" — é uma só.** O risco real não é falta de integração; é **clobber concorrente** (last-write-wins) + **visão de contexto defasada**.

**Why:** Em 06/ago um chat concluiu "Tarefas nunca mergeada" olhando a ref `main` **LOCAL** (defasada) e gravou isso na memória; outro chat corrigiu depois. Deploys vão via `git push origin HEAD:main`, que **NÃO move a ref local** → a `main` local mente sobre o que está no ar. Trocar por cima do arquivo do outro chat, ou concluir estado a partir de dado velho, é como se perde informação.

**How to apply:**
- **Re-ler antes de editar:** antes de mexer num arquivo de memória, reabrir ele com Read — outro chat pode tê-lo mudado desde que seu contexto carregou. Nunca editar a partir da cópia em contexto sem reconferir.
- **Edições cirúrgicas / append:** 1 fato por arquivo (padrão do sistema de memória); preferir Edit pontual/append a reescrever o arquivo inteiro → escritas concorrentes não se atropelam e o clobber vira, no pior caso, um conflito de git recuperável.
- **Fato volátil aponta COMO verificar, não o valor:** para o que está no ar / na main / SHAs / deploy, escrever o *método* de checagem (`git ls-remote origin main`; `git fetch` e olhar `origin/main`, nunca `main` local; probe em prod tipo `curl -s -o /dev/null -w '%{http_code}' -X POST .../api/<rota>` → 403 = vivo) em vez de cravar um número que envelhece.
- **Fonte da verdade = `origin/main` remoto + produção**, nunca a `main` local nem o worktree.

Relacionado: [[project_tarefas]], [[reference_github_backup]], [[project_area_membros_quirk]], [[reference_neon_bancos]].
