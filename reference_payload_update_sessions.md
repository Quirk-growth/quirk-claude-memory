---
name: reference-payload-update-sessions
description: "RESOLVIDO (20/set): escrever em Users NÃO desloga mais. A causa do 'trocar o tema desloga' não era o payload.update — era o hook derrubarSessoesAoRedefinirSenha disparando em todo update, porque o Payload preenche hash/salt no data de QUALQUER escrita. Corrigido na raiz."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-09-20T00:00:00.000Z
---

**Bug real (06/ago, prod, deslogou o Renan):** trocar o tema (`POST /api/users/preferencias` → `payload.update` no próprio user) deslogava a pessoa: a sessão morria, a navegação seguinte vinha sem sessão e o SSR dava 401 "Você deve estar logado" (em `.next/server/chunks/ssr/…` via `processTicksAndRejections` — não era rota de API, que devolve 403). Contornado em `7ce2b71` gravando só a coluna por SQL cru.

**A causa que ficou registrada aqui era ERRADA** (20/set, achada investigando o item 3.7 do rem-3-5-3-7). Não é o `payload.update` reescrevendo `users_sessions`, nem o campo `array` do adapter: o Payload **preenche todo campo ausente do `data` com o valor atual do documento** antes dos hooks (`fields/hooks/beforeValidate/promise.js` → `getFallbackValue`), e lê esse doc com `showHiddenFields: true`. Logo **`hash` e `salt` chegam em `data` em TODA escrita** — até num PATCH que só manda `telefone`. O hook `derrubarSessoesAoRedefinirSenha` usava exatamente `'hash' in data` como sinal de "a senha está sendo regravada" e zerava `sessions` em todo update da coleção. O hook entrou em `12197f0d` (24/jul) e o bug do tema apareceu em 06/ago — o hook causou o bug.

**Fix na raiz (20/set):** o gatilho do hook passou a ser o hash **mudar** (`data.hash === originalDoc?.hash` → não faz nada). No update normal o `data.hash` é cópia do atual (hash de senha nova só é gerado depois, no beforeChange); no `resetPassword` o hash novo já está no objeto e o Payload chama os hooks sem `originalDoc`.

**Consequências práticas:**
- Editar QUALQUER campo de um usuário pelo painel (nome, telefone, papel, squad) **não desloga mais** a pessoa editada. Antes deslogava, em silêncio.
- Os writes por SQL cru de `/preferencias` e `/permissoes-extras` continuam lá — não são mais obrigatórios, só evitam acordar a cadeia de hooks. `payload.update` em `users` voltou a ser seguro.
- O que DEVE derrubar sessão continua derrubando: desativar conta (`revogarSessoesAoDesativar`), trocar/redefinir senha (o próprio Payload 3.90 zera ao gravar senha, mantendo só a sessão de quem pediu quando é a própria conta).
- Testes: `tests/int/usersPreservarSessoes.int.spec.ts` (5 casos, inclui as contraprovas) e `tests/unit/derrubarSessoesAoRedefinirSenha.spec.ts` (o de integração sozinho NÃO prova o hook, porque o Payload também zera no reset).

**Como conferir se está no ar:** `git log --oneline origin/main -- src/collections/hooks/derrubarSessoesAoRedefinirSenha.ts` — o fix só vale se o commit com o critério `data.hash === originalDoc?.hash` estiver em `origin/main` (nunca conferir pela main local, ver [[feedback-memoria-multichat]]).

**Lição que sobrou:** o contorno de 06/ago tratou o sintoma (não usar `payload.update`) sem confirmar o mecanismo, e a causa errada virou "regra geral" documentada por 6 semanas. Medir antes de generalizar.

**Meta-lição (multi-chat):** vários chats subindo pra MESMA prod em paralelo — o `origin/main` andou 4× durante aquele debug e confundiu a atribuição do bug. Ver [[feedback-memoria-multichat]]. Relacionado: [[project_area_membros_quirk]], [[reference_neon_bancos]].
