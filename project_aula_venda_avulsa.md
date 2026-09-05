---
name: project_aula_venda_avulsa
description: "Funil de venda avulsa low-ticket da aula \"A Venda Invisível\" (R$19,90) — LP + checkout Asaas + entrega token-gated na área de membros + gatilho n8n"
metadata: 
  node_type: memory
  type: project
  originSessionId: d5b50112-4f32-4a16-b3c6-7f83ec1e8de2
  modified: 2026-09-05T01:21:59.386Z
---

Venda de uma aula de Marketing avulsa (low-ticket, R$19,90), **fora** da área do cliente/time — porta paralela sem login. No ar desde 04/09/2026 (commit `2b4cf68` em `area-membros-quirk`, branch de trabalho `feat/aula-venda-invisivel`).

**Aula:** Vimeo `1221207981` (aula "Persuasão digital…" já publicada na coleção `aulas`, módulo Marketing id 10). Renan precisa por **privacidade de domínio** no Vimeo (só embeddar em membros.quirkgrowth.com.br) pra ninguém compartilhar o player.

**LP:** `/Users/renanreal/Desktop/lp-aula-venda-invisivel.html` (self-contained; fonte em scratchpad aula19*). Sem preço na página (revela só no checkout). Topo/hero rolam pra `#oferta`; botão da oferta → checkout Asaas `https://www.asaas.com/c/mu34ah67kk94l6d3`. Hero com rei de xadrez glow (hero_concept.jpg). Capa do produto Asaas: `Desktop/capa-asaas-venda-invisivel.jpg`.

**Entrega (área de membros):**
- Coleção `acessos-aula` (`src/collections/AcessosAula.ts`): token único por compra, fail-closed no admin, SEM select (sem enum no DDL). DDL manual em `scripts/ddl/2026-09-04-acessos-aula.sql` (aplicada e verificada em prod; ver [[reference_payload_select_enum_ddl]] e skill payload-migracao-prod).
- Página pública `/assistir/[token]` (route group `(frontend)`, sem auth): valida token e libera o VideoVimeo.
- `POST /api/aula/liberar` (auth header `x-liberar-secret` == env `AULA_LIBERAR_SECRET`): idempotente por `asaasPaymentId`; resolve email/nome pelo id do cliente Asaas (`buscarClienteAsaas` em `src/lib/asaas/api.ts`) quando só vem `asaasCustomerId`; gera token e **dispara o e-mail** via `payload.sendEmail` (pipeline `MAIL_WEBHOOK_URL` que já existe). Defaults: vimeoId `1221207981`, título "A Venda Invisível".

**Gatilho (n8n):** workflow **`gEOf9yco2VPvMNe0`** "Aula A Venda Invisível — libera acesso pós-pagamento" (ativo). Webhook `https://n8n.quirkgrowth.online/webhook/aula-venda-invisivel-pago` → code identifica `PAYMENT_CONFIRMED`/`PAYMENT_RECEIVED` + valor 19,90 → IF → httpRequest POST no `/api/aula/liberar` (header com o secret) → responde 200. É um webhook Asaas SEPARADO do gateway do Auto Ads (`2ZnZqb4wFous4uEs`, não tocar) — ver [[reference_asaas_webhook_gateway]].

**Pendências manuais do Renan (04/09):** (1) setar `AULA_LIBERAR_SECRET` no Render — cuidado que a tela substitui a lista inteira ([[reference_render_memoria_oom]]/deploy-quirk); (2) adicionar o webhook no Asaas (URL acima, eventos confirmado/recebido) SEM mexer no do Auto Ads; (3) privacidade de domínio no Vimeo. Sem o secret, o endpoint responde 401 (Asaas re-tenta o webhook até passar). Falta ainda um teste de compra ponta a ponta.
