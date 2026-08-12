---
name: project_migracao_leads_clickup_crm
description: "Migração dos 11 cenários Make.com de captação de leads (ClickUp) para o CRM interno — módulo aditivo, concluída e verificada em 12/ago/2026"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-08-12T21:12:14.056Z
---

Migração de todos os fluxos de captação de lead do Make.com (pasta 220449, time 385200) do ClickUp para o [[project_crm_quirk]] interno, mantendo o ClickUp rodando em paralelo (decisão explícita do Renan: "não podemos parar oq esta rodando"). Spec/plano em `docs/superpowers/{specs,plans}/2026-08-11-migracao-leads-clickup-crm*.md` no repo `area-membros-quirk-4`, branch `feat/migracao-leads-clickup-crm`.

**Padrão aplicado nos 11 cenários:** módulo novo `http:MakeRequest` (id 100) inserido logo após o gatilho, POST pra `/api/crm-leads/entrada/<token>` com `onerror: Ignore` isolado (falha no CRM nunca derruba o resto da cadeia). Campo `campanha` = nome legível da origem (ex. "Comercial Time - Google Ads", "TikTok Ads") — é o único identificador de origem, sem etiqueta separada (confirmado com o Renan). `respostas` opcional, array `{pergunta,resposta}`.

**Why:** ClickUp vinha sendo o destino único dos leads; unificar no CRM interno permite fila/roleta/SLA que o ClickUp não tem. Migração é aditiva-primeiro (valida em paralelo, remove ClickUp só depois de confirmado — ainda não removido).

**How to apply:** Se for mexer em qualquer um desses 11 cenários de novo, o padrão (módulo 100 + campanha) já está lá — só ajustar/estender, não recriar. Cenário `quirk-comercial` (cliente interno id 104) é o destino de todos.

**GOTCHA real, recorrente — aba "Página1" quebrada no Sheets:** dois cenários (Treinamentos, TikTok) tinham o módulo `google-sheets:addRow` apontando pra aba `Página1` que não existe mais na planilha `1WmzgFMidtIUMdg_Qg11AXlduPoY_qghrjaiQnlHxnu0` — erro `Unable to parse range: 'Página1'!A1`, silencioso há semanas (leads reais não caíam na planilha, só percebido ao testar). Corrigido repontando pra abas reais (`Leads Treinamentos`, `Leads Tiktok` — nomes confirmados pelo Renan, não adivinhados). **Se outro cenário dessa planilha der o mesmo erro, é o mesmo bug — perguntar o nome certo da aba, não assumir.**

**GOTCHA Make — scenario auto-desativa em erro:** com `maxErrors:1`, uma execução que falha (ex. o bug acima) derruba `isActive` pra `false` e `isinvalid` pra `true` — mesmo depois de corrigir o blueprint via `scenarios_update`, o cenário continua inativo até chamar `scenarios_activate` explicitamente. A reativação pode disparar um retry automático da execução falha que ficou na fila.

**GOTCHA Make — `scenarios_update` é substituição total, cuidado com `metadata.designer.orphans`:** ao montar o blueprint manualmente (Python + colar no tool call) pra inserir o módulo novo, um módulo órfão desconectado (WhatsApp antigo com API key exposta, presente em vários cenários) foi apagado sem querer numa das edições — o array `orphans` ficou `[]` em vez do conteúdo original. Detectado comparando `scenarios_get` antes/depois; restaurado a partir do snapshot salvo em `tool-results/` antes da edição. **Sempre confirmar via `scenarios_get` pós-update que `orphans` bate com o esperado, especialmente quando o blueprint é grande demais pra ler de uma vez.**

**GOTCHA — gatilho nativo Meta Lead Ads não é testável com payload sintético:** Tasks 6 (Form Nativo) e 7 (RMKT Form Nativo) usam `facebook-lead-ads:NewLeadMultiple`, não um webhook custom. `scenarios_run` com `data` não injeta lead fake (só faz polling real, `operations:1` sem achar nada). Essas duas foram validadas só estruturalmente (revisão cuidadosa de sintaxe de campo, copiada de módulos já funcionando no mesmo cenário) — decisão do Renan: "Seguir sem teste ao vivo (recomendado)".

**Metodologia de verificação usada (webhooks custom):** `hooks_get` → `curl POST` com payload realista (nome "TESTE MIGRACAO ...") → query direta em `crm_leads` via script descartável `scripts/ddl/_tmp-check*.mjs` → conferir `executions_list` bate com a contagem esperada de módulos. ~18 leads de teste ficaram no CRM (não apagados — não é decisão minha apagar dado).

**Pendências explicitamente fora de escopo (não iniciar sem o Renan reabrir):** remoção do ClickUp, projeto de Meta CAPI (eventos de volta pro Meta conforme funil avança), UTM estruturada (campanha/conjunto/criativo), redesign dos e-mails automáticos, consolidação de cenários duplicados, limpeza do módulo órfão com a API key exposta (`evolutionapi.quirkgrowth.online`).
