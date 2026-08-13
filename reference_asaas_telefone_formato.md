---
name: reference_asaas_telefone_formato
description: Asaas mobilePhone exige DDD+número sem DDI 55 — gotcha corrigido em 13/08/2026
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-13T15:48:00.589Z
---

O campo `mobilePhone` (e `phone`) do Asaas espera só **DDD+número** (10/11 dígitos) — **sem** o código de país 55, sem `+`. Confirmado contra o exemplo oficial da doc do Asaas: `"mobilePhone": "4799376637"`.

Isso conflita com a convenção interna do CRM, onde `normalizarTelefone` (tanto [[reference_quirk_docs]]-adjacente `src/lib/crm/telefone.ts` quanto `src/lib/onboarding/texto.ts`) sempre **adiciona** o 55 na frente (formato E.164-sem-mais usado pra WhatsApp/UAZAPI). O campo `clientes.telefone`/`dadosCadastro.celular` guarda esse formato com 55 — correto pro WhatsApp, errado se mandado cru pro Asaas.

**Sintoma:** Asaas manda e-mail "Número de telefone incorreto" ao criar/atualizar customer, e a criação de cobrança falha ou o telefone fica salvo errado (silencioso — só aparece quando o Asaas decide validar).

**Fix (13/08/2026):** `paraTelefoneAsaas()` em `src/lib/asaas/cobranca.ts` — remove `55` só quando o resultado já digits-only tem 12/13 dígitos E começa com 55 (não confunde com DDD 55 real de Santa Maria/RS, que tem 11 dígitos). Usado em `src/app/api/asaas/cobranca/route.ts` no `mobilePhone` passado a `criarClienteAsaas`.

**Ponto cego:** o customer só é criado UMA VEZ no Asaas (`criarClienteAsaas` só roda se `cli.asaasCustomerId` ainda não existe) — corrigir o código não conserta clientes que já têm customer criado com telefone errado. Levantamento em 13/08/2026 achou **19 clientes** nessa situação; corrigidos em lote via `PUT /customers/:id` direto na API do Asaas (script descartável, não mexeu no `clientes.telefone` do nosso banco — esse continua com 55, que é o formato certo pro nosso uso interno).

Se aparecer de novo (cliente novo com erro de telefone no Asaas): confirmar que passou pelo `paraTelefoneAsaas` antes de virar `mobilePhone`, e não confundir com [[reference_asaas_webhook_gateway]] (que é o webhook de ENTRADA de pagamento, fluxo totalmente diferente).
