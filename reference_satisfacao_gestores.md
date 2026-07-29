---
name: reference-satisfacao-gestores
description: "Status de satisfação do cliente + % de aproveitamento do gestor no painel — modelo, fórmula e planilha-fonte"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-29T09:53:25.301Z
---

Feature do [[project-painel-relatorios]] (28/jul/2026): qualificação de cliente + nota do gestor, trazida da planilha "Dashboard — Squad de Tráfego" (Google Sheets `1-Vw5wdpxMp1bgyBr4IgDo6BoxmJy4bBI8KKDp6T8AHg`). No app: SÓ status + %, sem gratificação/cronograma (isso segue na planilha).

**Status de satisfação do cliente** (campo `clientes.statusSatisfacao`, enum, default `conta_nova`) e pontos — em `src/lib/hub/satisfacao.ts`:
excelencia 10 · satisfeito 8 · mediano 6 · basico 4 · conta_nova 0 · congelado 0 · em_risco 0 · churn_externo 0 · churn_interno −5.

**% de aproveitamento do gestor** (decifrado batendo os números da planilha): `soma dos pontos dos clientes ÷ (clientes ATIVOS × 10)`, onde ativos = total − churn(externo/interno) − congelado. Ex.: Gerson 104 pts / (13×10) = 80%.

**UI:** dropdown de status no lado direito da lista de clientes (`StatusSatisfacaoSelect`, grava PATCH /api/clientes); coluna "Aproveitamento" na lista de Gestores (`AproveitamentoGestor`, cores: verde ≥80, azul ≥60, âmbar ≥40, vermelho <40). Fórmula em `aproveitamentoDoGestor()`. Ajuste de pontos/cores é trivial nesse arquivo.

**Comparação mês a mês** (29/jul/2026, `src/lib/hub/satisfacaoMensal.ts`): duas fontes. (1) `satisfacao-mensal` — foto por cliente/período, criada pelo botão "Fechar mês" na lista de Clientes (`FecharMesButton` → POST /api/satisfacao/fechar-mes; default = mês passado). (2) `aproveitamento-mensal` — % já calculado por gestor/período, para meses históricos importados da planilha. Tela `/admin/historico-satisfacao` (`HistoricoSatisfacaoView`) mostra chips por mês + % por gestor + delta vs. mês anterior; a coluna compacta também mostra "atual · último mês". **Precedência:** se um período tem fechamento por cliente, ganha; senão usa o importado por gestor. Jun+jul/2026 já importados (`scripts/import-aproveitamento-planilha.mjs`); planilha e app NÃO têm chave comum (nomes de conta ≠ contas de anúncio) e o time de gestores mudou (Anna/Phillipe/João Pedro saíram) — por isso o histórico antigo é gestor-level, não por cliente. Ambas as coleções: FK SET NULL sobre NOT NULL → beforeDelete de cliente/usuário limpa os snapshots antes.
