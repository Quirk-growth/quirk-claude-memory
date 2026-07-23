---
name: project_quirk_auto_ads
description: Projeto Quirk Auto Ads — automação Make.com que recebe briefing imobiliário no WhatsApp (UAZAPI), interpreta com Claude, cria campanhas CTWA via Meta Marketing API. Status: arquitetura multi-cliente em aplicação (mai/2026).
type: project
originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
---
**Quirk Auto Ads** — sistema que recebe briefing de cliente imobiliário pelo WhatsApp, usa Claude pra interpretar e estruturar a campanha, e cria via Meta Marketing API (Click-to-WhatsApp). Multi-cliente: cada cliente tem ad_account/page/token próprios no Data Store.

**Stack:**
- WhatsApp: UAZAPI (não-oficial, multi-device — escolha consciente, sabe do tradeoff)
- Orquestração: **n8n** — a operação inteira da Quirk Auto Ads foi migrada do Make.com para o n8n (confirmado 2026-06-10, rodando em produção). Histórico: antes era Make (cenário ID 4750002). Não oferecer Make como plataforma de automação; a stack atual é n8n.
- IA: 3 chamadas Claude — agente principal (prompt v3.4), classificador de confirmação, extrator de JSON
- Execução: Meta Marketing API via HTTP no Make
- Memória: Data Store do Make (`Memoria Conversas Quirk`), Key = telefone

**Identificadores principais:**
- Meta App: "Quirk Auto Ads" (publicado, modo Live, Standard Access)
- Business Manager principal: 1612905538806887
- Conta de teste: act_3771507593117364 (Ads MCP bloqueado pela Meta no rollout — só funciona via Make)
- Página principal: 687786881077238 ("Renan Real - Marketing Imobiliário")
- System User: QuirkOps (com escopos ads_management, ads_read, business_management, pages_manage_ads, pages_show_list, leads_retrieval)
- Telefone de teste do Renan: 5511980838409

**Why:** A matriz de públicos Quirk (Pub 0-7, Invest, Profissões, Corretores) é o IP da agência. Esse sistema codifica essa expertise pra escalar criação de campanhas sem perder qualidade.

**How to apply:** Antes de qualquer mudança técnica, lembrar que o sistema é multi-cliente — toda hardcode de ad_account/page/token quebra o multi-tenancy. Manter dados por cliente no Data Store via Key = telefone.

**Fase atual (mai/2026):** Multi-cliente em aplicação. Documento de execução: /Users/renanreal/quirk_auto_ads/EXECUCAO_MULTI_CLIENTE.md. Falta: aplicar bodies novos no Make + estender Data Store + cadastrar primeiro cliente de teste + testar end-to-end. Depois disso: Fase E (auditoria/logs) e Fase F (relatórios sob demanda).

**Pendências conhecidas que valem lembrar:**
- Ads MCP bloqueado na conta de teste — Meta libera no rollout sem prazo
- Token EAA aparece em texto plano em vários lugares (Data Store por cliente é solução intermediária; centralizar em variável Make é Fase E)
- O Renan tem o documento `Quirk_onboarding_cliente_v1.docx` pro processo de Partnership com cliente novo
- Prompt-mestre atual: v3.4
