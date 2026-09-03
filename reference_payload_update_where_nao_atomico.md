---
name: reference-payload-update-where-nao-atomico
description: "GOTCHA Payload 3 + Postgres: payload.update({where}) NÃO é compare-and-swap atômico — faz SELECT depois UPDATE em dois passos; claim/lock condicional precisa de SQL cru via payload.db.drizzle.execute()"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-09-03T22:07:21.299Z
---

Descoberto em 02/set/2026, feature WhatsApp Oficial/Disparos ([[project_area_membros_quirk]]), corrigindo uma corrida entre invocações sobrepostas do cron de processamento de disparos.

**O erro**: para fechar uma corrida (duas invocações do mesmo endpoint pegando o mesmo item `pendente` e mandando a mesma mensagem de WhatsApp duas vezes), a primeira tentativa de fix usou `payload.update({ collection, where: { and: [{id},{status:'pendente'}] }, data: {status:'enviando'} })`, presumindo que isso seria um UPDATE condicional atômico (só troca o status se ainda estiver `pendente`) — um padrão de "claim" clássico. **Não é.** A implementação do `update`-por-`where` no `@payloadcms/drizzle` faz um `SELECT` primeiro (sem `FOR UPDATE`) pra achar os ids que batem, e SÓ DEPOIS roda o `UPDATE` por id. É um TOCTOU (time-of-check-to-time-of-use) clássico: duas invocações concorrentes podem AMBAS fazer o SELECT antes de qualquer uma commitar o UPDATE, e ambas "ganham" o claim.

**Como foi pego**: um revisor não confiou no relatório do fix e reproduziu de verdade — rodou 2 invocações concorrentes (`Promise.all`) contra o item real, com um mock de envio com delay artificial (50ms) pra alargar a janela da corrida, e mediu 2 chamadas reais à função de envio pro MESMO item (deveria ser 1). Também provou que o teste de "corrida fechada" que o fix trouxe passava IGUAL no código sem o fix — porque só testava que o `find` inicial não pegava um item já `enviando`, não simulava concorrência de verdade.

**A correção real**: `payload.db.drizzle` expõe a instância Drizzle crua (`NodePgDatabase`, `@payloadcms/db-postgres/dist/types.d.ts:79`) — dá pra rodar SQL parametrizado direto:

```ts
import { sql } from 'drizzle-orm'

const claim = await payload.db.drizzle.execute(sql`
  UPDATE crm_disparos_itens
  SET status = 'enviando', updated_at = now()
  WHERE id = ${candidato.id} AND status = 'pendente'
  RETURNING id
`)
const foiClaimado = claim.rows.length > 0
```

`claim` é o `pg.QueryResult` cru do driver `node-postgres` (`.rows` array simples, `.rowCount`, `.command`) — confirmado por sonda própria contra o banco de teste, não por suposição de doc. Um único `UPDATE...WHERE...RETURNING` é atômico de verdade (o lock de linha do próprio UPDATE resolve a corrida) — nunca existiu essa garantia via `payload.update({where})`.

**Quando isso importa**: qualquer "pegar e marcar como meu antes que outro processo pegue" (fila de processamento, cron com lote, worker) que dependa de `payload.update({where})` pra ser exclusivo está exposto a esse TOCTOU. Se o efeito colateral do double-processing for visível externamente (ex.: mensagem duplicada pro cliente, cobrança duplicada), vale a pena o SQL cru; se for só reprocessar um cálculo idempotente internamente, pode não valer a complexidade.

**Efeito colateral aceito, não resolvido só com o claim**: um "reclaim" de itens presos (ex.: processo morre entre claim e resolução, item fica `enviando` pra sempre) precisa de outra query — `UPDATE ... SET status='pendente' WHERE status='enviando' AND updated_at < now() - interval '5 minutes'` — que é idempotente por natureza (update de conjunto, sem `RETURNING`), mas troca "item travado pra sempre" por uma janela residual rara de duplicata se o crash acontecer EXATAMENTE entre o envio ter sucesso (ex.: WhatsApp já mandou) e o commit do novo status. Claim atômico fecha a corrida de LEITURA concorrente; não dá idempotência do lado da API externa (isso exigiria um idempotency key que a API não ofereça).
