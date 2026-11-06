---
name: project_lp_altto_form
description: "LP de formulário da Altto Design (móveis planejados): HTML em ~/quirk-lps/altto-form/, n8n GK9uTneOYtxioXv0 → CRM cliente 68 + planilha; falta o dev deles publicar em /form e criar as tags no GTM-NK6X46FB"
metadata:
  node_type: memory
  type: project
  originSessionId: e503985f-17d1-44e8-8ea5-38b8e80d30b7
  modified: 2026-11-06T07:46:39.389Z
---

Formulário multi-step de captação da **Altto Design** (móveis planejados de alto padrão, SP — unidades Anália Franco e Lapa). Feito em 06/11/2026.

- **Página:** `~/quirk-lps/altto-form/index.html` — HTML autônomo, dark/dourado (Bodoni Moda + Inter, `#D4AF37` sobre `#0a0a0a`), estrutura copiada da LP de formulário da A2 (hero + card com barra de progresso + opções com letra + auto-avanço).
- **Destino:** `alttodesign.com.br/form` — site Next.js **do cliente**. Vai em `public/form/index.html` + rewrite `/form`. Não virar componente React (o GTM está dentro da página).
- **n8n:** workflow `GK9uTneOYtxioXv0`, webhook `/webhook/altto-form-lead`.
- **CRM:** token `n1AC30kO1mi66lMqEj7Ue6hP6wPED-_p`. **Planilha:** `1cntHwGmfX7ZYo7Bvy8gcHcvf27ukk1fJY7-eG4pyi5Y`.

⚠️ **Token errado entrega em silêncio (06/11/2026).** A integração subiu com o token de OUTRO cliente (id 68). O endpoint respondia `{"ok":true,"acao":"criado"}` e tudo parecia certo — os leads só entravam no CRM da empresa errada. Sintoma: "entra na planilha, não aparece no CRM". Um token válido mas de outro cliente **não dá erro nenhum**. Sempre conferir o token abrindo o CRM *daquele* cliente, seção Webhook, antes de ligar tráfego. O token mora no nó do n8n, nunca no HTML (a página é pública e o token é a autenticação).
- **GTM do cliente:** `GTM-NK6X46FB` (GA4 `G-7F4CS66JSH`, Pixel `675704084732207`). WhatsApp `5511916730777`.

**Por que tem n8n no meio (não é preguiça):** o endpoint `/api/crm-leads/entrada/<token>` exige body JSON, e o preflight cross-origin responde 200 **sem `Access-Control-Allow-Origin`** — POST direto do browser morre. Medido com `curl -X OPTIONS` em 06/11. Além disso navegador não escreve em Sheets. Ver [[reference_lp_quirk_webhook_cors]].

**Qualificação:** derivada da faixa de orçamento; `valor` no CRM é o **ponto médio** da faixa (não é valor declarado pelo lead — não tratar como receita prevista).

**Pendências (06/11/2026):** dev da Altto publicar; criar triggers/tags no GTM (hoje a página emite `generate_lead`, `form_step`, `whatsapp_click` e **nada no container escuta**); restringir compartilhamento da planilha, hoje "qualquer um com o link edita".

Relacionado: [[project_lp_a2_usucapiao]], [[gotcha_wa_me_nao_dispara_pixel]], [[feedback_teste_decorativo_mutacao]].
