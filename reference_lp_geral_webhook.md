---
name: reference_lp_geral_webhook
description: LP neutra (lp.quirkgrowth.com.br) → Make 4896527; contrato de chaves limpas e correções aplicadas
metadata: 
  node_type: memory
  type: reference
  originSessionId: cebfa691-b2ea-4769-8c2e-c5e72c588d06
  modified: 2026-08-31T01:00:00.462Z
---

LP "geral"/neutra publicada em **https://lp.quirkgrowth.com.br/** (arquivo self-contained em `~/lp-quirk-geral/index.html`, imagens em base64) manda **JSON** (`Content-Type: application/json`) pro webhook **18bbmmonyamjnbk9gtp7iaoll3nje1vw** = Make **cenário 4896527** "Leads Quirk [Comercial Time] LP Genérica" (team 385200, hook 2809412).

**Contrato de chaves LIMPAS** (diferente do legado Elementor das outras LPs — ver [[reference_lp_social_media]] e a skill lp-quirk): `nome, email, empresa, whatsapp, instagram, faturamento, nicho, investimento, data, origem, fb_event_id`.

Fluxo do cenário: Webhook → HTTP CRM (`membros.../api/crm-leads/entrada/…`) → e-mail interno (yuri+contato+rodrigo) → e-mail pro lead → Google Sheets (aba "Leads LP genérica") → ClickUp (list 900902474139, assignee Rodrigo) → Router → UAZAPI WhatsApp (msg do Rodrigo).

**Correções feitas em 30/08/2026** (o cenário fora clonado do Tech 4805666 e lia chaves legadas `No Label …` → caía tudo vazio): remapeei os 7 módulos pras chaves limpas + **sanitizei o telefone** (`{{replace(2.whatsapp; "/[^0-9]/g"; "")}}`) no CRM e no UAZAPI (`55` + sanitizado), porque a LP NÃO sanitiza telefone no submit (bug clássico da skill). Testado: execução SUCCESS, todos os campos corretos na planilha. Cenário ATIVO.

⚠️ Edge case: se o lead digitar o número já com 55/+55, o UAZAPI dobra (`5555…`). Placeholder guia sem DDI, risco baixo. Se quiser blindar, sanitizar na própria LP.
