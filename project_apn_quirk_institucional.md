---
name: project_apn_quirk_institucional
description: "Deck institucional \"Apresentação de Negócios\" da Quirk Growth, high ticket genérico (sem nicho), com preços"
metadata: 
  node_type: memory
  type: project
  originSessionId: 798c9556-748c-4496-87e4-6ab001869e18
  modified: 2026-09-25T14:27:08.802Z
---

Apresentação de Negócios da Quirk Growth (versão institucional), adaptada da APN (`~/Desktop/APN - Assessoria de Performance.pdf`, 28 págs, image-based) pro estilo do deck de parceiros e **falas genéricas de high ticket** (removido todo nicho imobiliário, a pedido do Renan em set/2026).

- Pasta: `~/apn-quirk-growth/` — 24 slides HTML + `Apresentacao-Quirk-Growth.pdf`. Mesma identidade dark do [[project_metodo_cresce]] / deck de parceiros; render via [[gotcha_html_print_pdf_preview]] (Chrome headless 2x → PIL PDF 288dpi).
- Estrutura: capa → estágios ("Qual empresa descreve você hoje") → comparativo (gestor amador × agência × Quirk) → quem somos → time → localização → ecossistema → CRESCE (selo + 6 engrenagens) → frentes → movendo engrenagens → funil → cronograma → área de membros → relatórios → ferramentas → custo por lead → posicionamento → entregáveis → 4 planos (Scale R$15k / System R$8k / Foundation R$3k / Social R$4k) → obrigado.
- **Slides nichados adaptados**: "Custo por lead" agora por TICKET (baixo→luxo/enterprise), não tipo de imóvel; Ferramentas = "Guia de primeiro atendimento" (sem corretores) + "Calculadora de metas" (no lugar de VGV).
- **Prints reais** extraídos da APN via PyMuPDF get_images: área de membros (p15 xref 179) e relatórios (p16 xref 186, métricas Meta Ads genéricas) → `area-membros.png`/`relatorios.png`. Evitei rel_3 (tinha VGV/comissão, nichado).
- Assets reaproveitados do deck de parceiros: `time-foto.png` (selfie), `predio.png` (localização), `time-quirk.png` (grid 16 headshots do bio; APN real tinha 24 c/ cargos), antes-depois/novos-cases (posicionamento).
- Pendências oferecidas: time com 24 pessoas + cargos (falta headshots); trocar mockups por mais prints reais se quiser.
