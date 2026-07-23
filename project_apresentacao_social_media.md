---
name: project_apresentacao_social_media
description: "Deck comercial de Social Media da Quirk (HTML 16:9 navegável + PDF), fundido a partir da LP de social media e da APN de performance"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7e267ea1-9bd0-4c4e-a15e-4d8e881a80fe
---

**Apresentação de Negócios — Social Media** (jul/2026). Deck de 23 slides em `/Users/renanreal/apresentacao-social-quirk/` — `index.html` (16:9, setas/clique, escala pra qualquer tela) + PDF exportado + `assets/`.

Nasceu da fusão de duas fontes: a LP de social media (`~/Desktop/Quirk - Apresentacao Comercial.html`, de onde vieram os dados do Instagram 72/73/83/90%, o funil desconhecido→cliente, o feed antes/depois e os 8 cases) e a APN de performance (`~/Desktop/APN - Assessoria de Performance (2).pdf`, de onde vieram perfil do cliente, tipos de agência, time, localização, ecossistema, CRESCE + engrenagens, poder dos 5%, mentorias e os preços).

**Decisões travadas:** multi-nicho — ZERO menção a mercado imobiliário no posicionamento da Quirk (o card do slide 3 virou "a melhor agência de conteúdo, marca e performance para o seu negócio"). **Três planos, nesta ordem (âncora alta primeiro):** R$ 8.000 (Multicanal) → R$ 4.000 (Social Media & Captação, com captação audiovisual de 4h 1x/mês) → R$ 3.000 (Social media). Mentorias existem só como linha do slide de Entregáveis. Ficaram FORA: calculadora de VGV, CPL, dados de follow-up, planos de tráfego (3k/8k/15k), inbound/Base Engine, o slide de mentorias (tinha foto do guia p/ corretores), o slide de posicionamento estratégico (redundante com os cases) e a frase "agências de tráfego param na primeira engrenagem".

**Como foi feito (reaproveitável):** 15 slides da APN são renders PNG 1920×1080 das páginas do PDF (via pymupdf, `Matrix(1920/1440)`); 8 slides são HTML/CSS construídos do zero. Logo e selo CRESCE foram recortados do PDF e tiveram o fundo escuro removido por keying (alpha = max(r,g,b), limiar 60) — sem isso aparece uma caixa escura em volta. Export do PDF: Chrome headless `--print-to-pdf` com `@page{size:1920px 1080px}`; no `@media print` é obrigatório zerar `position/left/top` do `#stage`, não só o `transform`, senão os slides saem deslocados. Servidor local: `deck-social` na porta 8102 (`~/.claude/launch.json`).

Relacionado: [[project_metodo_cresce]], [[reference_quirk_brand]], [[project_playbooks_comercial]]
