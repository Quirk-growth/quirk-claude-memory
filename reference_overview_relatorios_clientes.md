---
name: reference-overview-relatorios-clientes
description: "Formato aprovado do overview de clientes na tela admin \"Relatórios dos clientes\" do painel"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-25T16:51:15.985Z
---

Design aprovado (25/jul/2026) para a tela **Relatórios dos clientes** (`/admin/relatorios-clientes`, `RelatoriosClientesView`) do [[project-painel-relatorios]]. Hoje ela só lista nome + status + botões; vai ganhar o overview de métricas de todos os clientes ativos.

**Layout escolhido (Opção 1 do brainstorm visual):**
- **Faixa de totais da agência no topo** (KPIs): Investido, Leads, Custo/lead, "clientes com veiculação (N de M)" — cada um com tendência ▲▼ vs período anterior.
- **Filtro de dias:** 1 / 7 / 14 / 30 / 90 / 365 (precisa CRIAR os presets `1d`, `14d`, `365d` em periodo.ts — hoje só existem 7/30/90/6m/12m/max).
- **Tabela densa, uma linha por cliente**, com as MÉTRICAS À MOSTRA: Cliente · **Tendência** (minilinha/sparkline SVG de leads/dia — verde se termina acima de onde começou, vermelha se abaixo) · Investido · Leads · Conversas · Custo/lead · CPM · Frequência · Status.
- **Os 4 botões de ação** (Ver relatório / Funil comercial / Gestores / Editar cadastro) **colapsam num menu "⋯"** por linha (não ficam soltos — a tela é pra VER número, ação é secundária).
- CPM = custo por mil impressões; Frequência = impressões/alcance. Todos computáveis do que já existe (metricas_diarias tem investido/impressoes/alcance/cadastros/conversas). Reusa `portfolioResumo` + `intervaloComparacao` (tendência) + flags.

Motivo: Renan gerencia muitos clientes e precisa comparar todos de um golpe de vista; tabela densa > cartões. Sparkline > porcentagem (mais claro visualmente).
