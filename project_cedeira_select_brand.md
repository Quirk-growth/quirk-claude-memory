---
name: project-cedeira-select-brand
description: "Kit de marca do cliente Cedeira Select (curadoria de imóveis) — vetores recriados de logo AI, fontes, paleta, manual e papelaria em /Users/renanreal/cedeira-select-brand/"
metadata: 
  node_type: memory
  type: project
  originSessionId: e3ef6d8d-357e-4fb8-996c-07c9c68c4351
  modified: 2026-09-04T15:12:06.670Z
---

Kit de marca completo do cliente **Cedeira Select** (imobiliária de curadoria/seleção), entregue em 04/09/2026 em `/Users/renanreal/cedeira-select-brand/` (ZIP `cedeira-select-kit-marca-v1.zip`).

- Logo original era imagem AI-gerada; recriado 100% em vetor: símbolo redesenhado em grid + texto convertido em curvas via fontTools (script `_build/make_logos.py` — regenera tudo).
- **Fontes (decisão do Renan)**: CEDEIRA = Bodoni Moda Regular **no corte display (opsz 96, instanciado da variável — o corte de texto ficou "grosseiro", feedback 04/09)**; SELECT e descritor = Montserrat Light/Regular (Google Fonts, OFL). Arquivo: `BodoniModa-Display-Regular.ttf`.
- **Símbolo — spec fina (feedback do Renan)**: as linhas NÃO se tocam — dourado horizontal termina com respiro de ~10 unidades antes da vertical preta/grafite, que é contínua; vertical curta e bracket também têm respiro. Vertical direita = preta/azul-escuro (#1B1B19), nunca dourada.
- **Slogan trocado em produção**: "Imóveis Selecionados" → **"CURADORIA DE IMÓVEIS"** (pedido mid-task; é o descritor oficial).
- **Paleta**: Marfim #F4EFE7, Preto Grafite #1B1B19, Dourado Select #B08D5B (Pantone 4515 C / 871 C metálico premium), apoios Areia #E5DCCB e Bronze #8C6F45. `.ase` incluso.
- 11 variações de logo (SVG+PDF+PNG 300dpi), manual 10 págs, cartão 96×56 c/ sangria, timbrado A4, assinatura e-mail. Papelaria com placeholders (Nome/CRECI/telefone) — fontes editáveis em `05-papelaria/fontes-editaveis/`.
- Pipeline de export: Chrome headless print-to-pdf (mesmo padrão dos decks) — ver [[gotcha-html-print-pdf-preview]].
- Pasta ainda NÃO está no GitHub ([[reference-github-backup]] — pendência de blindagem).
