---
name: gotcha_drizzle_push_interativo_trava
description: "npm test não-interativo trava pra sempre num prompt y/N do Drizzle quando o schema do banco de teste diverge do código; auto-aceitar com `yes` sem olhar pode derrubar coluna que ainda é necessária"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-09-22T16:14:22.630Z
---

Quando o schema declarado no código (CollectionConfig do Payload) diverge do schema real do banco de TESTE (`DATABASE_URI_TEST`, push:true), o `push` do Drizzle às vezes pede confirmação interativa — "Accept warnings and push schema to database? (y/N)" — antes de aplicar uma mudança que ele classifica como destrutiva (drop de coluna/tabela com dados). Rodando `npm test` em background/não-interativo, esse prompt nunca recebe resposta e o processo fica preso pra sempre (CPU quase zero, não é crash — parece um hang comum, mas na real está esperando stdin). Sintoma: subagentes/monitors ficam "esperando a suíte terminar" indefinidamente sem nunca receber a notificação de conclusão.

**Como diagnosticar:** ler as últimas linhas do log da suíte (não confiar só no exit code de um pipe truncado — `comando | tail -N` faz o exit code refletir o `tail`, não o comando real). Se aparecer "Warnings detected during schema push" seguido de uma lista "You're about to delete X column" e depois "(y/N)" sem nenhuma linha depois, é isso.

**Como destravar:** `yes | npm test` (ou `yes | npx vitest run <arquivo>`) auto-aceita o prompt. **MAS: antes de fazer isso, olhe a lista de colunas que serão dropadas** — nem toda coluna "extra" é lixo órfão. Aconteceu de verdade (22/set): o prompt listava `users.reset_password_requested_at` e `media._objectkey` entre as colunas "a deletar" — só que essas são colunas RECENTES (bump Payload 3.90, 18/set) que o schema atual EXIGE, não sobra de campo removido. Aceitar sem olhar derrubou as duas no banco de teste, quebrando um teste-canário (`verificarRestauracaoSchema.int.spec.ts`) que existe justamente pra pegar esse tipo de regressão. DDL correta pra restaurar (aditiva, idempotente): `scripts/ddl/2026-09-18-payload-390.sql`.

**Descoberta mais séria:** mesmo restaurando as colunas via SQL direto, elas voltam a sumir na PRÓXIMA execução da suíte — o `push` do Drizzle as derruba de novo sozinho, sem sequer mostrar o prompt (aceita "silenciosamente" ou trata como não-destrutivo dessa vez). Ou seja, o teste-canário `verificarRestauracaoSchema.int.spec.ts` falha hoje pra QUALQUER sessão que rode a suíte completa não-interativamente, independente de branch — é um problema sistêmico pré-existente, não causado por uma feature específica. Ainda não investigado a fundo (por que o push considera essas colunas órfãs se o código/Payload 3.90 as exige) nem reportado como issue formal — vale abrir isso com quem escreveu `verificarRestauracaoSchema.int.spec.ts` (commit `750a52a3`, 18/set).

**Como aplicar:** se a suíte travar em background sem nunca notificar conclusão, suspeitar deste prompt ANTES de qualquer outra teoria (rede, Mac dormindo). Ver as últimas linhas do log primeiro. Nunca `yes |` às cegas — ler a lista de colunas a deletar; se alguma parecer recém-adicionada (não removida de um CollectionConfig), parar e investigar em vez de aceitar. Relacionado: [[gotcha_push_pg_error_mata_script_longo]] descreve um sintoma de drift diferente (timeout de hook, não prompt travado).
