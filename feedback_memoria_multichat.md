---
name: feedback-memoria-multichat
description: "Protocolo pra memória compartilhada entre chats paralelos deste projeto — é UMA memória só (git + backup auto); re-ler antes de escrever, edições cirúrgicas, e fatos voláteis apontam COMO verificar em vez de cravar valor"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-07T17:10:11.042Z
---

Vários chats do Claude Code rodam **em paralelo neste projeto** e compartilham a **MESMA** pasta de memória (`~/.claude/projects/-Users-renanreal/memory/`), versionada em git com backup automático (hook `Stop` + hook `PostToolUse` por-escrita, ambos `git add -A && commit && push origin main`). **Não são "2 memórias" — é uma só.** O risco real não é falta de integração; é **clobber concorrente** (last-write-wins) + **visão de contexto defasada**.

**Why:** Em 06/ago um chat concluiu "Tarefas nunca mergeada" olhando a ref `main` **LOCAL** (defasada) e gravou isso na memória; outro chat corrigiu depois. Deploys vão via `git push origin HEAD:main`, que **NÃO move a ref local** → a `main` local mente sobre o que está no ar. Trocar por cima do arquivo do outro chat, ou concluir estado a partir de dado velho, é como se perde informação.

**How to apply:**
- **Re-ler antes de editar:** antes de mexer num arquivo de memória, reabrir ele com Read — outro chat pode tê-lo mudado desde que seu contexto carregou. Nunca editar a partir da cópia em contexto sem reconferir.
- **Edições cirúrgicas / append:** 1 fato por arquivo (padrão do sistema de memória); preferir Edit pontual/append a reescrever o arquivo inteiro → escritas concorrentes não se atropelam e o clobber vira, no pior caso, um conflito de git recuperável.
- **Fato volátil aponta COMO verificar, não o valor:** para o que está no ar / na main / SHAs / deploy, escrever o *método* de checagem (`git ls-remote origin main`; `git fetch` e olhar `origin/main`, nunca `main` local; probe em prod tipo `curl -s -o /dev/null -w '%{http_code}' -X POST .../api/<rota>` → 403 = vivo) em vez de cravar um número que envelhece.
- **Fonte da verdade = `origin/main` remoto + produção**, nunca a `main` local nem o worktree.
- **Cada chat no SEU worktree — nunca dois chats na mesma pasta.** Em 07/ago dois chats trabalhavam em `/Users/renanreal/area-membros-quirk`; o outro deu `git checkout` da branch dele (`feat/caixa-entrada-equipe`) por baixo do meu trabalho no meio de um `vitest`, e minhas alterações ficaram penduradas sobre a branch alheia. Recuperação sem perda: `git diff > patch` → `git checkout -- <arquivos>` (devolve a pasta do outro intacta, na branch dele) → `git worktree add ../area-membros-quirk-<tema> <minha-branch>` → `git apply patch`. Worktree novo precisa de `.env` copiado e `node_modules` por symlink pro worktree principal. Conferir com `git worktree list` + `git branch --show-current` ANTES de commitar — o cwd do shell também se perde entre chamadas e cai na home, então `cd` explícito.
- **Abrir chat novo SEMPRE na home `/Users/renanreal`, nunca dentro da pasta do worktree.** A pasta de memória é indexada pelo cwd (`-Users-renanreal`); abrir a sessão em `/Users/renanreal/area-membros-quirk-*` cria um projeto Claude novo e **a memória inteira não carrega** — sem erro, só silêncio. Os chats paralelos navegam até o worktree a partir da home. (Verificar com `ls ~/.claude/projects/` — deve existir só `-Users-renanreal`.)
- **PORTÃO DE DEPLOY (pre-push hook, instalado 07/ago em `.git/hooks/pre-push` do repo `area-membros-quirk` — vale pros 4 worktrees porque hooks vivem no git-common-dir; NÃO é versionado, re-clone precisa reinstalar).** `git push` que atualiza `main` é **BLOQUEADO** se: (a) o `main` remoto andou desde seu último fetch → faça `git fetch origin && git rebase origin/main` (+ re-rode os testes) antes — mata o clobber, inclusive `--force`; ou (b) há um **FREEZE** ativo. **Freeze pra parar TODOS os deploys num incidente/janela de review:** liga com `echo "motivo — quem — quando" > "$(git rev-parse --git-common-dir)/DEPLOY_FREEZE"`, desliga com `rm "$(git rev-parse --git-common-dir)/DEPLOY_FREEZE"`. Pushes de BRANCH passam livres. **Disciplina que o hook LEMBRA mas não força (o hook não julga código):** rodar `/code-review` na branch + testes verdes ANTES de subir pra `main` — é o que pega bug que compila (ex.: "trocar tema desloga", [[reference-payload-update-sessions]]). Contexto: 07/ago 4 chats colidiram, `origin/main` andou ~7× em uma noite e um deploy derrubou a prod.

Relacionado: [[project_tarefas]], [[reference_github_backup]], [[project_area_membros_quirk]], [[reference_neon_bancos]].
