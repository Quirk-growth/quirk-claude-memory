---
name: project-quirk-auto-creative
description: Skill quirk-auto-creative + conexão OpenAI gpt-image-1 pra gerar copy+banner imobiliário de tráfego pago
metadata: 
  node_type: memory
  type: project
  originSessionId: 9dff5fea-85e5-4ca9-8dd6-5833b089777d
---

Skill `quirk-auto-creative` em `~/.claude/skills/quirk-auto-creative/` gera criativos imobiliários (copy racional + banner) pro Meta Ads, a partir de briefing de 15 campos + referência visual. Fluxo de 8 fases, 3 checkpoints, regras invioláveis: ZERO CTA na arte, estrutura fixa H1+H2+3-4 tópicos, tom racional (sem emoção/emoji).

Motor de imagem: **OpenAI gpt-image-1** via `scripts/generate_creative.py` (urllib puro + Pillow). Chave em `/Users/renanreal/quirk-banner-designer/.env` (fonte única). Dois modos: `--mode estilo` (text-to-image, /v1/images/generations) e `--mode base` (edita render real do imóvel, /v1/images/edits, com `--ref`). Saídas em `output/`: master 1024×1024, Feed 1080×1080, Story 1080×1920.

Detalhe técnico chave: gpt-image-1 NÃO faz 9:16 nativo (só 1024×1024, 1024×1536, 1536×1024). O Story 1080×1920 é montado gerando o master quadrado (= safe zone central) e estendendo topo/base com gradiente derivado das cores de borda. Isso cumpre a lógica de safe zone do spec e sobrevive ao center-crop do Advantage+.

Status: MODO ESTILO validado live (ON Paulista) MAS Renan achou a qualidade da imagem da API ruim ("cara de banco de imagem + texto"). PREFERÊNCIA dele: dirigir o Custom GPT dele no navegador (Chrome MCP) — GPT "Banner Designer - Quirk" (chatgpt.com/g/g-6a146c97f424819183cf5a5be5238109), que entrega imagem melhor. Fluxo no browser: mandar briefing dos 15 campos → o GPT responde "gerando copy..." e PARA (o `APENAS` do prompt encerra o turno; precisa de nudge "segue, gera a copy") → entrega copy no checkpoint. Spec original do Custom GPT (DALL-E) portado pra skill local. Relacionado a [[project-quirk-auto-ads]].

Pendência de segurança: a chave OpenAI foi colada no chat ao montar — Renan deve gerar nova e revogar a antiga.
