---
name: reference_lp_quirk_webhook_cors
description: "LP não pode postar direto no /api/crm-leads/entrada/<token>: exige JSON e o preflight volta sem Access-Control-Allow-Origin; sempre relay por n8n com form-urlencoded"
metadata:
  node_type: memory
  type: reference
  originSessionId: e503985f-17d1-44e8-8ea5-38b8e80d30b7
  modified: 2026-11-06T07:46:51.748Z
---

O webhook genérico de leads da área de membros — `POST /api/crm-leads/entrada/<token>` — **não serve para ser chamado direto do navegador de uma LP**, por dois motivos somados:

1. O handler faz `req.json()` e `validarPayloadEntrada` exige body JSON. Mandar `form-urlencoded` não passa.
2. Um POST `application/json` cross-origin dispara preflight. O preflight responde **200 com `access-control-allow-methods` e `-headers`, mas SEM `access-control-allow-origin`** — então o browser recusa. Verificado em 06/11/2026:
   `curl -i -X OPTIONS <url> -H "Origin: https://alttodesign.com.br" -H "Access-Control-Request-Method: POST"`.

**Como fazer, sempre:** a página manda `URLSearchParams` com `mode:"no-cors"` (requisição simples, sem preflight) para um webhook do **n8n**, e o n8n faz o fan-out server-side — HTTP Request com `specifyBody: "json"` para o CRM, e Google Sheets para a planilha. É a mesma forma usada em [[project_lp_altto_form]] e no *Leads LP VSL [Comercial] → CRM*.

Contrato aceito pelo endpoint: `telefone` (**obrigatório**), `nome`, `email`, `campanha`, `valor` (número), `respostas: [{pergunta, resposta}]` (máx. 50). O telefone pode ir **só com dígitos** — `lib/crm/telefone.ts` põe o DDI 55 sozinho (10/11 dígitos), e é isso que faz a dedupe bater com os leads que entram por Meta lead ads no mesmo cliente. Dedupe é `unique(cliente, telefone)`.

Relacionado: [[project_crm_quirk]], [[lp-quirk]].
