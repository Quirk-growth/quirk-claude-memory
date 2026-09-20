---
name: gotcha_afterlogin_transacao_pendura
description: "afterLogin do Payload roda DENTRO da transação do login — await numa escrita que referencia users(id) pendura o login inteiro, e repro em SQL cru não reproduz"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-09-20T20:25:08.843Z
---

Na área de membros, o hook `afterLogin` do Payload roda **dentro da transação do login**, que ainda não commitou. Consequência: se o hook fizer `await` numa escrita que referencia `users(id)` — por exemplo `payload.create` numa coleção com FK `ator_id → users(id)` — o INSERT espera uma transação que só termina depois do hook retornar. **Deadlock lógico: o login pendura** (medido: ~30s até o timeout).

A solução é `void` (sem `await`), com `try/catch` interno na função chamada para não deixar promessa órfã. É o que `registrarLoginNaAuditoria` faz.

**A armadilha ao investigar isto** (custou duas rodadas de revisão em 20/09/2026): **repro em SQL cru NÃO reproduz**. Nenhuma destas bloqueia, testadas uma a uma contra o banco de teste:

- `INSERT INTO users_sessions` sozinho;
- `UPDATE users SET updated_at` sozinho;
- `UPDATE users` reescrevendo o e-mail (índice único);
- as duas juntas.

Replicar os *comandos* do login não replica o *lock* do login. A única sonda que reproduz: subir o Payload, injetar um hook que segura a promise no fim de `afterLogin` (a transação fica aberta), disparar `payload.login()` de verdade e, de outra conexão, tentar `SELECT ... FROM users WHERE id=<quem logou> FOR KEY SHARE` — que bloqueia, com `pg_blocking_pids` apontando o backend do login (`idle in transaction`, query bloqueadora `insert into "users_sessions"`).

O que está **medido** é a espera. O modo do lock (`FOR UPDATE` na linha) é *dedução* pela regra de compatibilidade — `pg_locks` do bloqueador não mostra o modo da tupla, porque lock de linha sem disputa não é registrado lá. Não confunda a inferência com a observação ao escrever comentário no código.

Relacionado: [[reference_payload_update_sessions]], [[project_area_membros_quirk]].
