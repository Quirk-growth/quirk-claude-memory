---
name: gotcha_pool_pg_error_mata_script_longo
description: "Script Node longo contra o Neon morre com EADDRNOTAVAIL: o pool do pg emite 'error' e sem listener o Node aborta o processo"
metadata:
  node_type: memory
  type: reference
---

Script local de horas usando o Payload contra o Neon **morre no meio** com `Error: read EADDRNOTAVAIL` vindo de `pg-pool` (`Client.idleListener`). Aconteceu no job de importação do ClickUp ([[reference_migracao_historico_clickup_crm]]) depois de 2.050 cards.

**Why:** o erro de socket ocioso é recuperável — o `pg` descarta o cliente ruim e abre outro. Quem mata o processo é o Node: evento `error` sem NENHUM listener é exceção fatal. E o pool do Payload vem com **0 listeners de `error`** (medido via `pool.listenerCount('error')`).

**How to apply:** em qualquer script que rode por muito tempo, logo depois do `getPayload`:

```ts
const pool = (payload.db as { pool?: { on?: (e: string, cb: (err: Error) => void) => void } }).pool
if (typeof pool?.on === 'function') pool.on('error', (err) => console.error(`[pool] ${err.message} — segue`))
else console.warn('AVISO: não achei o pool para proteger')
```

Não engole falha: a query em andamento ainda estoura e cai no try/catch do laço. O `else` importa — sem ele, o dia em que o adapter parar de expor `.pool` você segue achando que está protegido. Ter o job **retomável** continua sendo obrigatório; isso só evita ter que retomar toda hora.
