---
name: gotcha-wa-me-nao-dispara-pixel
description: "Link wa.me pode não disparar o Pixel quando o gatilho do GTM casa por \"URL contém whatsapp\" — usar api.whatsapp.com"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 82379061-5612-4773-b98e-38e0debf380e
  modified: 2026-11-05T11:19:32.508Z
---

Em LP que converte por clique no WhatsApp, **conferir como o gatilho do GTM identifica
o clique antes de escolher o formato do link**.

É comum o container ter um gatilho do tipo `Click URL CONTÉM "whatsapp"`. Nesse caso:

- `https://api.whatsapp.com/send?phone=…&text=…` → **contém** "whatsapp", dispara ✅
- `https://wa.me/55…?text=…` → **não contém**, não dispara ❌

**Why:** achado em 05/11/2026 no container `GTM-TLXM73BN` da A2 Arquitetura (tag 58,
Meta Pixel, evento `Whatsapp`). O site do cliente usava os dois formatos em páginas
diferentes, então metade das conversões nunca foi registrada — e falha em silêncio:
o link funciona normalmente para o usuário, só a medição some.

**How to apply:** baixar o `gtm.js` publicado (`https://www.googletagmanager.com/gtm.js?id=<ID>`)
e ler as regras antes de escrever os links. Se o gatilho casa por "whatsapp", usar
`api.whatsapp.com/send?phone=`. Deixar comentário no código avisando, porque trocar
para `wa.me` parece uma simplificação inofensiva. Ver [[project-lp-a2-usucapiao]].

Na mesma auditoria: tags de conversão do Google Ads penduradas em `gtm.js` (todo
pageview) e em `gtm.linkClick` sem filtro de URL inflam conversão — vale checar isso
em qualquer container de cliente antes de usar custo por conversão como baseline.
