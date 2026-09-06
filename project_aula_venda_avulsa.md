---
name: project_aula_venda_avulsa
description: "Funil de venda avulsa low-ticket da aula \"A Venda Invisível\" (R$19,90) — LP + checkout/entrega na Greenn (nativo). Infra Asaas+token na área de membros existe mas virou plano B"
metadata: 
  node_type: memory
  type: project
  originSessionId: d5b50112-4f32-4a16-b3c6-7f83ec1e8de2
  modified: 2026-09-06T23:24:20.774Z
---

Venda de uma aula de Marketing avulsa (low-ticket, R$19,90), **fora** da área do cliente/time — porta paralela sem login. No ar desde 04/09/2026 (`area-membros-quirk`, branch `feat/aula-venda-invisivel`): commit `2b4cf68` (feature) + `526b4fd` (fix do host). **Validado ponta a ponta em prod 05/09** (token→página com Vimeo→e-mail com link certo→Renan clicou e abriu; registros de teste apagados).

**GOTCHA do host no Render:** `new URL(req.url).origin` num route handler resolve pra `https://localhost:10000` (app roda atrás de proxy) — o link do e-mail saía quebrado. Fix: derivar de `NEXT_PUBLIC_SITE_URL` → `x-forwarded-host` → `host` → req.url. Vale pra qualquer rota que monte URL absoluta pra e-mail/redirect.

## PIVOT 06/09: checkout + entrega migraram pra Greenn (order bump/upsell + controle de vendas)
Decisão do Renan: em vez do Asaas (só gateway, sem upsell), usar **Greenn** (checkout + área de membros nativa "Greenn Club", já usada no kit R$27,90).
- **Produto Greenn:** "A Venda Invisível", R$19,90, cobrança única, Pix+cartão. **Checkout da oferta: `https://payfast.greenn.com.br/gq7g6a2/offer/eOGbGJ`** (o base é `.../gq7g6a2`).
- **Entrega = Greenn Club nativo:** o Vimeo `1221207981` foi **incorporado numa aula dentro da área de membros da Greenn**. Comprou → Greenn libera o acesso sozinha (login por e-mail). NÃO usa nossa página de token nem o gatilho n8n pra esse produto.
- **Vimeo:** privacidade de incorporação agora precisa permitir o domínio da Greenn (`greenn.com.br` / `club.greenn.com.br`) — feito, o embed funcionou.
- **LP:** `Desktop/lp-aula-venda-invisivel.html` e o deploy `Desktop/index.html` (self-contained, pronto pro cPanel). Os **3 botões vão direto pro checkout da oferta Greenn**. Capa: `Desktop/capa-asaas-venda-invisivel.jpg` (serve na Greenn também).
- **Pendências:** (a) publicar a LP no subdomínio (cPanel + registro A no Cloudflare + purgar cache; ver skill lp-quirk — arquivo na RAIZ do document root); (b) finalizar o checklist do produto na Greenn (Entrega estava pendente, resolve com o vídeo adicionado); (c) compra real de R$19,90 de teste.

---
## Infra Asaas + token (PLANO B — construída e validada, mas NÃO em uso pró produto)
**Aula:** Vimeo `1221207981` (aula "Persuasão digital…" já publicada na coleção `aulas`, módulo Marketing id 10).

Asaas hosted checkout `https://www.asaas.com/c/mu34ah67kk94l6d3` (produto "Aula Venda invisível", só Pix/Boleto — faltava cartão). Entrega token-gated na nossa área de membros:
- Coleção `acessos-aula` (`src/collections/AcessosAula.ts`): token único por compra, fail-closed no admin, SEM select (sem enum no DDL). DDL manual em `scripts/ddl/2026-09-04-acessos-aula.sql` (aplicada e verificada em prod; ver [[reference_payload_select_enum_ddl]] e skill payload-migracao-prod).
- Página pública `/assistir/[token]` (route group `(frontend)`, sem auth): valida token e libera o VideoVimeo.
- `POST /api/aula/liberar` (auth header `x-liberar-secret` == env `AULA_LIBERAR_SECRET`): idempotente por `asaasPaymentId`; resolve email/nome pelo id do cliente Asaas (`buscarClienteAsaas` em `src/lib/asaas/api.ts`) quando só vem `asaasCustomerId`; gera token e **dispara o e-mail** via `payload.sendEmail` (pipeline `MAIL_WEBHOOK_URL` que já existe). Defaults: vimeoId `1221207981`, título "A Venda Invisível".

`AULA_LIBERAR_SECRET` no Render = `86193e639fae0e8b74ce6051bf1cf544f9275c5c60b034a2` (setado 05/09, validado). ASAAS_API_KEY já existia no app.

**Gatilhos n8n (2, ambos ativos mas OCIOSOS no fluxo Greenn):**
- `gEOf9yco2VPvMNe0` (Asaas): webhook `.../webhook/aula-venda-invisivel-pago`, filtra PAYMENT_CONFIRMED/RECEIVED + R$19,90 → POST /api/aula/liberar. Webhook Asaas nunca foi configurado no painel (paramos antes, ao migrar pra Greenn).
- `hJGkE0CN63JhSU3B` (Greenn): webhook `.../webhook/aula-venda-invisivel-green`, parse tolerante do postback da Greenn (email/nome direto) → POST /api/aula/liberar. Criado caso um dia se queira entrega sem-login pela Greenn; hoje NÃO usado (entrega é nativa na Greenn Club). Se for usar, travar o parse contra o 1º postback real.
- Ambos separados do gateway Auto Ads (`2ZnZqb4wFous4uEs`, não tocar) — ver [[reference_asaas_webhook_gateway]].
