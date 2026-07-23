---
name: projeto-lp-quirk-tech-rebrand-dark-premium
description: Nova variação da LP institucional/comercial da Quirk com branding dark premium tech; síntese da LP validada + posicionamento do site
metadata: 
  node_type: memory
  type: project
  originSessionId: cebfa691-b2ea-4769-8c2e-c5e72c588d06
---

Nova variação da landing page principal da Quirk Growth, criada em 2026-06-25, em `/Users/renanreal/lp-quirk-tech/index.html` (HTML standalone único, CSS+JS inline; logo em `assets/img/logo-quirk.png`; serve via launch.json "lp-quirk-tech" porta 8095).

**Tese:** combina a estrutura/copy de conversão validada de [[project_lp_iscas]]-style da `lp.quirkgrowth.com.br` (hero com form de qualificação VGV+nicho+investimento, 4 pilares, para-quem-é, 6 vantagens, quem somos MVV, logos, FAQ com cronograma 90 dias) + o posicionamento maduro "Mais que tráfego" do site `www.quirkgrowth.com.br` + **branding dark premium tech** (fundo #001D41/#00132d, azul #1D80FF com glow, grid de fundo, números em JetBrains Mono, gradiente nos headings) — o DNA visual de [[reference_quirk_brand]] que o site atual quase não usa.

**Decisões travadas:** headline mantém "Nº 1 em gestão de tráfego para serviços High Ticket" (da LP validada).

**v3 (2026-06-29) — assets reais + dinamismo tech:** canvas base mudou p/ quase-preto #05080f. Usa fundos gráficos reais da Quirk (bg-arrow=seta crescimento no hero, bg-ribbons em "quem somos", bg-sphere no CTA), ícones glassmorphic do site (icon_1-4), efeito radial, números SVG. Seção nova de **resultados/depoimentos** com fotos reais de cases (case-15-27mm, case-26mm, treino-vitacon). **Logos reais** (28, em `assets/img/logos/`, baixados da pasta Drive 1-NJIL4T4ubFDofXSllzaO4znjYy_XIw8) num **marquee infinito** de 2 fileiras (chips claros). **Seção de time** (17 fotos em `assets/img/team/sq/`, 440px quadradas, da pasta Drive 15KjI-vEhWdHREfMKnuoiXdY3AsTyMHZV; HEIC convertido via sips). Dinamismo: rede de partículas em canvas no hero, spotlight no mouse, parallax na seta, bordas com conic-gradient girando (.ring), pulse no CTA, hover grayscale→cor no time.

**Gotcha resolvido:** `overflow-x:hidden` no body criava scroll-container e travava o scroll da janela quando surgiu overflow horizontal (marquee/orbs). Corrigido com `overflow-x:clip` em html+body.

**Ajustes pós-feedback (2026-06-29):** logos do marquee voltaram aos ORIGINAIS com fundo (não-removido — a versão sem fundo foi rejeitada), em chips QUADRADOS 150×150 (mobile 128) com object-fit:contain. Vídeos de depoimento agora FUNCIONAM: baixei os MP4 reais do site (Webflow CDN `cdn.prod.website-files.com/.../<hash>_<nome>-transcode.mp4`, e originais em s3.amazonaws.com/webflow-prod-assets/) p/ `assets/video/` e abrem em **lightbox** (controls+autoplay) ao clicar no card — IDs "YouTube" nos nomes eram falsos, é tudo MP4 Webflow. Bônus: gerei versões SEM FUNDO dos 41 logos (flood-fill PIL das bordas) em `/Users/renanreal/quirk-logos-sem-fundo/` + pasta Drive criada `13z2NPlfH9G63pwRAZpfHjFuJAXYyU6Tb` (upload manual: conector Drive não sobe binário em lote).

**Meta Pixel (2026-06-29):** Pixel Quirk ID `905158130958739` no `<head>`, com AUTO-DETECÇÃO anti-duplicidade — checa `fbq.getState()` e só faz `init`+`PageView` se o pixel ainda não existir na página. Confirmado pelo Renan que o pixel JÁ está instalado no Elementor (embed mesmo-documento), então a LP NÃO refaz init/PageView (cede ao existente). Evento `Lead` no submit do form via `trackSingle(pixelId,'Lead',...)` + `eventID` (dedup CAPI) + trava `__quirkLeadSent` (1x só, testado com submit duplo). Atenção: se um dia embutir via IFRAME, o fbq do pai não é visível no iframe → ajustar.

**Webhook comercial (2026-06-29):** form ligado ao cenário Make **4805666** (team 385200) "Leads Quirk [Comercial Time] (UAZAPI-API) (NOVA LP)" — webhook `https://hook.us1.make.com/d3c8trdwxtymm51yxhjqt9sap6kpt1u9` (hookId 2017774). Fluxo: ActiveCampaign (contato+lista 7) → Google Sheets "Leads Imob" → ClickUp (list 900902474139) → e-mail interno (yuri/contato/rodrigo) → e-mail lead → WhatsApp UAZAPI. CRÍTICO: o cenário foi clonado de um form **Elementor** e as chaves do payload são `No Label name`, `No Label email`, `No Label field_0731945`(empresa), `No Label field_be27370`(telefone), `No Label field_f11eb72`(instagram), `Qual é o VGV mensal da sua empresa?`, `Selecione seu nicho de atuação:`, `Quanto pretende investir mensalmente em anúncios?`, `Date` — o form da LP envia exatamente essas chaves + `origem:"LP Nova (Quirk Tech)"` + `fb_event_id`. Adicionei campos **Email (obrigatório) e Empresa** no form (fluxo depende de email p/ AC). Cenário **ATIVADO** e testado (execução status 1, 8 ops OK). Pendente: APAGAR contato AC + tarefa ClickUp de teste ("TESTE — LP Nova"); surfacing de `origem` no ClickUp/e-mail/planilha ainda NÃO aplicado (entreguei como edição manual — re-upload do blueprint 32KB via API foi considerado arriscado p/ fluxo de produção).

**Pendências p/ produção:** confirmar links Instagram/LinkedIn no rodapé; papéis/cargos reais do time (hoje todos "Time Quirk"); aplicar origem nos 3 módulos do Make (manual); apagar lead de teste; mais vídeos de case no site se quiser ampliar a seção.
