---
name: reference_olho_senha
description: "Olho de mostrar/ocultar senha no CRM, Hub (/admin) e telas públicas; como funciona e onde mexer"
metadata:
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-21T16:09:20.430Z
---

Olho de ver/ocultar senha NO AR desde 21/10/2026 (commit `97ee7d03`, squash sobre `0bc067e2`, sem DDL).

- **Telas do site/CRM** (login, definir-senha, conta): componente `src/components/comum/CampoSenha.tsx` (+ `iconesOlhoSenha.ts`). Campo de senha novo no site → usar `CampoSenha`, não `<input type="password">` cru.
- **Hub (`/admin`)**: o input de senha do Payload NÃO é sobrescrevível. Solução = provider `src/components/admin/OlhoSenhaAdmin.tsx` (registrado em `admin.components.providers` no `payload.config.ts` + `importMap.js`), que com MutationObserver injeta um `<button class="olho-senha-admin">` ao lado de todo `input[type=password]`. Espera a hidratação do React (chaves `__reactProps$`/`__reactFiber$`) antes de mexer, senão dá erro de hidratação.
- CSS: `custom.scss` (admin, `[data-olho-senha-alvo]`) e `globals-quirk.css` (site).
- Testes: `tests/unit/campoSenha.spec.tsx`, `tests/unit/olhoSenhaAdmin.spec.tsx`.

**Why:** Renan pediu como "detalhe simples mais importante" — usuário erra senha digitando às cegas.
**How to apply:** o `custom.scss` é ponto de conflito frequente em rebase (outros chats mexem nele); depois de resolver conflito, sempre `git grep '^<<<<<<<'` no HEAD antes de empurrar — [[feedback_push_squash_base_velha]].
