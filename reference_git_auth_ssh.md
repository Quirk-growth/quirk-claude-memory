---
name: reference-git-auth-ssh
description: origin do area-membros-quirk trocado de HTTPS pra SSH (credencial do osxkeychain expirou)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-12T01:16:20.592Z
---

Em 2026-08-11, `git fetch`/`push` no repo `area-membros-quirk` (worktree `area-membros-quirk-4`) começou a falhar com `fatal: could not read Username for 'https://github.com': Device not configured` — o remote usava HTTPS (`credential.helper=osxkeychain`), e o token salvo no Keychain sumiu/expirou (`git credential-osxkeychain get` voltava vazio, sem erro).

Acesso SSH pro org Quirk-growth já estava configurado e funcionando (`ssh -T git@github.com` autentica normal). Resolvido trocando o remote:
```bash
git remote set-url origin git@github.com:Quirk-growth/area-membros-quirk.git
```

Puramente config local, não mexe em credencial nenhuma. Se voltar a falhar (`could not read Username`), primeiro checar `git remote -v` — se voltou pra HTTPS ou se o SSH também parou de autenticar, investigar antes de tentar mexer em credencial diretamente (isso é ação que só o Renan faz).
