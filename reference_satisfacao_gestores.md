---
name: reference-satisfacao-gestores
description: "Status de satisfação do cliente + % de aproveitamento do gestor no painel — modelo, fórmula e planilha-fonte"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-29T12:54:52.045Z
---

Feature do [[project-painel-relatorios]] (28/jul/2026): qualificação de cliente + nota do gestor, trazida da planilha "Dashboard — Squad de Tráfego" (Google Sheets `1-Vw5wdpxMp1bgyBr4IgDo6BoxmJy4bBI8KKDp6T8AHg`). No app: SÓ status + %, sem gratificação/cronograma (isso segue na planilha).

**Status de satisfação do cliente** (campo `clientes.statusSatisfacao`, enum, default `conta_nova`) e pontos — em `src/lib/hub/satisfacao.ts`:
excelencia 10 · satisfeito 8 · mediano 6 · basico 4 · conta_nova 0 · congelado 0 · em_risco 0 · churn_externo 0 · churn_interno −5.

**% de aproveitamento do gestor** (decifrado batendo os números da planilha): `soma dos pontos dos clientes ÷ (clientes ATIVOS × 10)`, onde ativos = total − churn(externo/interno) − congelado. Ex.: Gerson 104 pts / (13×10) = 80%.

**UI:** dropdown de status no lado direito da lista de clientes (`StatusSatisfacaoSelect`, grava PATCH /api/clientes); coluna "Aproveitamento" na lista de Gestores (`AproveitamentoGestor`, cores: verde ≥80, azul ≥60, âmbar ≥40, vermelho <40). Fórmula em `aproveitamentoDoGestor()`. Ajuste de pontos/cores é trivial nesse arquivo.

**Controle de performance** (29/jul/2026, `src/lib/hub/satisfacaoMensal.ts`): tela `/admin/historico-satisfacao` na aba **Tráfego** (só admin/supervisor) — junta o "Fechar mês" (`FecharMesButton` → POST /api/satisfacao/fechar-mes; default = mês passado) + comparação dos meses. Cada gestor é um `<details>` que abre a **lista de clientes+status do mês** (dropdown). TRÊS fontes, com precedência: (1) `satisfacao-mensal` — foto por cliente/período do "Fechar mês" no app (ganha se existir); senão (2) `aproveitamento-mensal` — % OFICIAL por gestor/mês (das linhas-resumo da planilha) + (3) `satisfacao-conta-mensal` — detalhe por conta (nome texto + status) pro dropdown. As três: FK SET NULL sobre NOT NULL → beforeDelete de cliente/usuário limpa antes.

**Import jun+jul/2026 da planilha** (`scripts/import-planilha-jun-jul.mjs` + dataset `scripts/data/squad-jun-jul-2026.json`): baixar o Google Sheet como **xlsx** via Drive MCP `download_file_content` (exportMimeType xlsx) e parsear com openpyxl célula a célula — o `read_file_content` (texto achatado) PERDE LINHAS, não serve. Layout das abas mensais: blocos de 5 colunas por gestor (r4=nome, r5=header, r6+=contas, depois linha-resumo "Contas:/Pontos:/% Qualidade"). **Ordem dos blocos ≠ ordem da Config** (varia por mês; validar por âncora, ex. Gerson). Gotchas: só entram gestores com usuário em prod (Anna/Phillipe/João Pedro saíram); "Conta Nova" É ativa; a fórmula da planilha ignora linhas de "overflow" (>15 vagas) então use o %/ativas das linhas-resumo, não recalcule. **App tem cliente↔gestor esparso (só ~15/96 vinculados)** — por isso "Fechar mês" no app não vai bater com a planilha até os vínculos serem preenchidos; a planilha é a fonte real do mapa gestor↔conta.
