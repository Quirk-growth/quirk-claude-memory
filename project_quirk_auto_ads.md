---
name: project_quirk_auto_ads
description: Projeto Quirk Auto Ads — automação n8n + Claude + Meta Marketing API que cria campanhas CTWA via WhatsApp. Status: NO AR como produto pago — 3 clientes a R$497/mês (ago/2026).
type: project
originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
modified: 2026-08-27T05:18:38.299Z
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

**Status comercial (ago/2026, contado na call com Marcelo/Sayne):** NO AR como produto pago — **3 clientes ativos a R$497/mês** (~R$2k MRR), vendidos quase sem esforço: quando o SDR identifica corretor sem estrutura ("cagado" no bench), oferece o Auto Ads numa call de 10 min; página própria com comparativo (gestor amador × Auto Ads × assessoria) e checkout que já abre cobrança no Asaas. API oficial do WhatsApp já embutida. Fluxo n8n com 150+ módulos. Conta do Renan: 100 clientes = R$50k MRR com 1 suporte + 1-2 gestores. Sinal de upsell planejado: quando o painel mostrar cliente investindo R$2-3k/mês em anúncio → oferecer assessoria. Marcelo chamou de "o killer da Quirk" (analogia: Acelera Aí do Allan Barros vs agência Pulse). Gargalo declarado: "tá parado, nem fui atrás" — sem push de marketing próprio ainda; plano de VSL + Instagram dedicado + tráfego pra demanda de corretor que o comercial descarta.

**Pendências conhecidas que valem lembrar:**
- Ads MCP bloqueado na conta de teste — Meta libera no rollout sem prazo
- Token EAA aparece em texto plano em vários lugares (Data Store por cliente é solução intermediária; centralizar em variável Make é Fase E)
- O Renan tem o documento `Quirk_onboarding_cliente_v1.docx` pro processo de Partnership com cliente novo
- Prompt-mestre atual: v3.4
