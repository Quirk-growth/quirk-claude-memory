---
name: feedback-push-squash-base-velha
description: Squash via commit-tree montado sobre base velha e empurrado com a main já adiantada DESFAZ o trabalho de outras sessões; sempre rebasear e conferir `diff base..squash` antes do push
metadata:
  node_type: memory
  type: feedback
---

**Incidente 19/10/2026:** montei o commit único (`git commit-tree HEAD^{tree} -p origin/main`) dos ajustes das copys com a árvore do branch (base `70483ca4`) mas com PAI = `origin/main` que, depois de um `git fetch` novo, já estava em `0aa3e7c5` (commit do CRM de outra sessão). O push (fast-forward, então o git não reclamou) trocou a árvore da main pela minha e DESFEZ o commit do CRM (97 arquivos no diff do squash). Percebi pelo `--stat` (97 files, 1703 deletions) logo depois; corrigi SEM force push com um commit cuja árvore = rebase do meu branch sobre `0aa3e7c5` (parent = main ruim), conferi `git diff 0aa3e7c5 origin/main` = só os meus arquivos e o Render ficou Live. A main ficou ~10 min com o CRM revertido (deploy 2f31e52 live).

**Why:** `commit-tree` com árvore do branch + pai diferente da base do branch = "reverter tudo o que entrou entre a base e o pai". Meu portão (`if origin/main != ...`) era lixo e o `fetch` veio DEPOIS da conferência.

**How to apply:** (1) `git fetch origin main` e `git rebase origin/main` NO BRANCH antes de montar o squash; (2) só então `commit-tree HEAD^{tree} -p origin/main`; (3) antes do push, `git diff --stat origin/main..$SQUASH` tem de listar SÓ os arquivos da minha mudança (número de arquivos ≈ esperado, deleções baixas) — se aparecer arquivo de outra frente, PARE; (4) rodar tsc + testes DEPOIS do rebase, não antes; (5) fetch de novo logo antes do push e abortar se `origin/main` ≠ pai do squash. Relacionado: [[feedback-git-repo-compartilhado-sessoes]], [[feedback-acompanhar-deploy-ate-verde]].
