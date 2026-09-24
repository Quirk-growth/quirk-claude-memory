---
name: reference_lp_clinics_crm
description: Fluxo de leads da clinics.quirkgrowth.com.br → Make 4727812; migrado pro CRM em 24/09/2026 (AC removido)
metadata: 
  node_type: memory
  type: reference
  originSessionId: cebfa691-b2ea-4769-8c2e-c5e72c588d06
  modified: 2026-09-24T22:00:17.831Z
---

**clinics.quirkgrowth.com.br** (WordPress+Elementor, GTM-TGBQHFJ9, pixel 1669763490913089) manda JSON com **chaves LIMPAS** (`nome, email, whatsapp, instagram, faturamento, especialidade, enviado_em, investimento`) pro webhook **`mrtt4p59sxfqw9epmz3beuok9ive0h3n`** = Make cenário **4727812** "Leads Clinics [Comercial Time] (UAZAPI - API)" (team 385200, hook 2712006).

**Fluxo (após fix de 24/09/2026):** Webhook → **CRM entrada** → Email interno (yuri/contato/rodrigo) → Email lead → Google Sheets (planilha `1WmzgFMidtIUMdg_Qg11AXlduPoY_qghrjaiQnlHxnu0`, aba "Leads Clinics") → ClickUp (list 900902474139, assignee Rodrigo 89341209) → Router → UAZAPI (msg do Rodrigo).

**CRM:** POST `membros.quirkgrowth.com.br/api/crm-leads/entrada/<token>`, token `4Xk7F8oRAJLcZb5SUv9PkU9lKdhX_XQq` = cliente **104 "Quirk — Comercial"** (7 vendedores, distribuição ativa), `campanha:"Clinics"` diferencia. `stopOnHttpError:false` (soluço do CRM não bloqueia email/planilha/whatsapp). Módulo id 100, dataStructure 288670.

**Fix (causa raiz):** o cenário NUNCA teve CRM — usava ActiveCampaign legado. Em 24/09 o AC começou a falhar (padrão 402) e o `Ignore` matava o resto do fluxo (execuções caíram de 8→2 ops, verdes por fora → nem email/planilha/whatsapp caíam). Removi os 2 módulos AC (upsertContact2024 + UpdateContactListStatus) e inseri o CRM em 2º. Ver [[reference_make_ignore_engole_falha]] e [[project_migracao_leads_clickup_crm]].

⚠️ Pipeline **"Quirk Clinics" (cliente id 91) existe mas está VAZIO** (0 vendedores, sem crmWebhookToken, crm_ativo=false) — por isso os leads de Clinics vão pro 104 com tag Clinics, não pro 91 (que deixaria o lead órfão).

**CRM entrada (contrato, `src/collections/CrmLeads.ts` + `src/lib/crm/entradaWebhook.ts`):** token = `crmWebhookToken` de um `clientes` com `crm_ativo=true` (o token É a auth). Payload aceito: `telefone` (obrigatório), `nome`, `email`, `campanha`, `valor`, `respostas[{pergunta,resposta}]`. `normalizarTelefone` limpa não-dígitos e prefixa 55; dedupe por cliente+telefone.
