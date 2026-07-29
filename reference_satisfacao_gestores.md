---
name: reference-satisfacao-gestores
description: "Status de satisfação do cliente + % de aproveitamento do gestor no painel — modelo, fórmula e planilha-fonte"
metadata:
  node_type: memory
  type: reference
---

Feature do [[project-painel-relatorios]] (28/jul/2026): qualificação de cliente + nota do gestor, trazida da planilha "Dashboard — Squad de Tráfego" (Google Sheets `1-Vw5wdpxMp1bgyBr4IgDo6BoxmJy4bBI8KKDp6T8AHg`). No app: SÓ status + %, sem gratificação/cronograma (isso segue na planilha).

**Status de satisfação do cliente** (campo `clientes.statusSatisfacao`, enum, default `conta_nova`) e pontos — em `src/lib/hub/satisfacao.ts`:
excelencia 10 · satisfeito 8 · mediano 6 · basico 4 · conta_nova 0 · congelado 0 · em_risco 0 · churn_externo 0 · churn_interno −5.

**% de aproveitamento do gestor** (decifrado batendo os números da planilha): `soma dos pontos dos clientes ÷ (clientes ATIVOS × 10)`, onde ativos = total − churn(externo/interno) − congelado. Ex.: Gerson 104 pts / (13×10) = 80%.

**UI:** dropdown de status no lado direito da lista de clientes (`StatusSatisfacaoSelect`, grava PATCH /api/clientes); coluna "Aproveitamento" na lista de Gestores (`AproveitamentoGestor`, cores: verde ≥80, azul ≥60, âmbar ≥40, vermelho <40). Fórmula em `aproveitamentoDoGestor()`. Ajuste de pontos/cores é trivial nesse arquivo.
