---
name: reference_render_cloudflare_ip
description: "A área de membros fica atrás da Cloudflare pelo próprio Render (inclusive o .onrender.com) — cf-connecting-ip é confiável, X-Forwarded-For é forjável"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-09-20T17:35:23.490Z
---

Medido em 20/09/2026, com `curl -sSI`: **tanto `membros.quirkgrowth.com.br` quanto `area-membros-quirk.onrender.com`** respondem com `server: cloudflare` e `cf-ray` (`...-GRU`). Isso é a infra do **próprio Render** — o edge deles roda sobre Cloudflare para todo serviço hospedado lá, não é uma Cloudflare que a Quirk pôs na frente. Ou seja: **não existe "origem crua" alcançável** que um atacante possa usar para pular o edge.

Consequências práticas para qualquer coisa que dependa de IP (limite de requisição, log, geo, bloqueio):

- Use **`cf-connecting-ip`**. É escrito pelo edge e não dá para forjar de fora.
- **Nunca** use `X-Forwarded-For`: é cabeçalho de requisição comum, qualquer um manda o valor que quiser. Chave de limitador baseada em XFF é contornável com uma linha de `curl` — foi exatamente o achado crítico 2 da revisão do Lote 4.
- Quando `cf-connecting-ip` falta (dev local, chamada interna tipo Render Cron), **pule o limite por IP** e apoie-se no limite por token/chave, que não depende de cabeçalho nenhum. Não invente fallback para XFF.

Helper no projeto: `ipDaRequisicao` em `src/lib/limiteRequisicoes.ts` (lê só `cf-connecting-ip`, devolve `null` quando ausente).

Relacionado: [[reference_render_memoria_oom]], [[project_area_membros_quirk]].
