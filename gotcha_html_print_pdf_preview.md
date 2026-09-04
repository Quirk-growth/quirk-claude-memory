---
name: gotcha-html-print-pdf-preview
description: GOTCHA em HTML→PDF (Chrome headless) — screenshot headless mente pra layout @page; SVG inline com height:auto colapsa; conferir sempre o PDF rasterizado
metadata: 
  node_type: memory
  type: reference
  originSessionId: e3ef6d8d-357e-4fb8-996c-07c9c68c4351
  modified: 2026-09-04T14:38:54.892Z
---

Duas armadilhas do pipeline HTML → PDF com Chrome headless (decks, papelaria, manuais):

1. **Screenshot headless ≠ PDF.** `--screenshot` renderiza layout de TELA (viewport), não de página: elementos com `position:absolute; inset:0` resolvem contra o viewport, então a prévia sai deslocada/cortada mesmo quando o `--print-to-pdf` (que usa o `@page`) está perfeito. Já perdi tempo "consertando" peça que estava certa.
   **Como verificar**: rasterizar o PDF real — `python3 -c "import fitz; …get_pixmap(dpi=68)"` (PyMuPDF, instalado no Mac) ou `sips -s format png arquivo.pdf` (só 1ª página; sips NÃO tem opção de página).

2. **SVG inline com `height:auto` colapsa no Chrome** (não é replaced element; não herda proporção do viewBox). Sempre dar width E height explícitos no `<svg>` inline ou via CSS.

**Why:** essas duas juntas fazem parecer que "o layout quebrou" quando só a prévia mentiu — ou o inverso (prévia ok, peça sumida).
**How to apply:** em peça de impressão, validar SEMPRE pelo PDF rasterizado, nunca pelo screenshot da página; e nunca usar height:auto em SVG inline. Usado no [[project-cedeira-select-brand]].
