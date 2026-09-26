---
name: project_vsl_quirk_imobiliario
description: "VSL da Quirk pro mercado imobiliário amplo (Método CRESCE), modelada na VSL da maquinadereunioes.com/lp4; roteiro v1 pronto com placeholders"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8be166d0-2761-4bc4-98e9-bbaa587f9f10
  modified: 2026-09-26T18:18:41.834Z
---

VSL da Quirk Growth pro mercado imobiliário, roteiro v1 escrito em 03/09/2026 em `/Users/renanreal/vsl-quirk-imobiliario/roteiro-vsl-v1.md` (~9–10 min).

**Decisões do Renan:**
- **Avatar: mercado imobiliário AMPLO** — profissionais, imobiliárias E incorporadoras. Nuance importante ao [[project_reposicionamento_incorporadoras]]: pra esta VSL ele escolheu explicitamente voltar a comunicar amplo ("quero ir para o mercado imobiliario de forma ampla, para conseguir comunicar com profissionais, imobs e incorps").
- **CTA:** formulário rápido de qualificação → call de diagnóstico com o time.
- **Ângulo central:** momento do mercado em 3 ondas — insatisfação com leads, anúncio encarecendo, corrida pra IA; reframe = "IA automatiza o desperdício se o posicionamento é fraco; o problema não é tráfego, é marketing"; solução = [[project_metodo_cresce]].

**Molde estrutural:** VSL da maquinadereunioes.com/lp4 (João Araújo), transcrita e analisada nesta sessão — player VTurb com teste A/B 50/50 (variante A 7min16s, B 9min47s). Estrutura de 8 etapas: gancho-pergunta → dores/alternativas → rediagnóstico (mecanismo do problema) → autoridade+categoria → mecanismo em 3 partes → cases por objeção → des/qualificação → CTA com escassez justificada. Transcrições completas salvas na sessão (scratchpad) e enviadas ao Renan como arquivos.
- CRESCE apresentado em **3 movimentos de 2 letras** (C+R ser visto/lembrado, E+S atenção→venda, C+E venda→escala) pra manter o ritmo "3 partes" do molde.

**Pendências antes de gravar:** preencher placeholders de autoridade (anos, nº clientes, verba gerida, VGV — NÃO inventar número); confirmar se "217 lotes em 17 dias" é case próprio; escolher depoimentos em vídeo (MP4 reais em `~/lp-quirk-tech/assets/video/`); depois derivar corte ~7min (variante A) pra teste A/B no estilo do concorrente.

**Fase LP (26/09/2026):** VSL gravada e editada ("VSL Quirk_Ver1.mp4" no Drive, 16min10s, 12,2GB 4K60 → transcodificada local p/ web: 1080p30 ~286MB + 720p30, h264_videotoolbox). LP construída em `~/quirk-lps/vsl/index.html` (monorepo LPs-quirk; mp4 no .gitignore): identidade Growth tech dark, player custom SEM controles (smart autoplay mudo + overlay "ativar som", barra de progresso falsa `x^0.5` — rápida no início, converge no fim, sem seek), reveals por currentTime persistidos em localStorage `quirk_vsl_v1`: botão Sala de Guerra aos 180s, formulário aos 300s. Pixel Quirk 905158130958739 c/ eventos VideoStart/Video3min/Video5min/Lead(eventID). Form multi-step 3 passos → webhook n8n `POST /webhook/lp-vsl-lead` (workflow **uuKWujYX3kRKiDsM** "Leads LP VSL [Comercial] → CRM", ATIVO e testado ponta a ponta): normaliza → POST intake CRM `membros.quirkgrowth.com.br/api/crm-leads/entrada/4Xk7F8oRAJLcZb5SUv9PkU9lKdhX_XQq` (mesmo token da LP Bio/Make 4896527; campanha="VSL"; respostas[] carrega empresa/instagram/VGV/nicho/investimento/UTMs) → Gmail interno (por ora SÓ contato@; add Yuri+Rodrigo no go-live). Preview local via launch.json "lp-vsl" porta 8098. GOTCHA: `python http.server` não suporta Range → seek de vídeo não funciona em preview local (não é bug da página). **Pendências:** link real da Sala de Guerra (placeholder `#SALA_DE_GUERRA_LINK` no CFG), DNS Cloudflare (proposto vsl.quirkgrowth.com.br; aguardando API token), upload cPanel (aguardando acesso FTP/SSH ou ZIP manual), copiar 720p pros assets quando encode terminar, APAGAR lead "TESTE Claude — apagar" do CRM.

Números usados vêm todos das mentorias (via [[project_metodo_cresce]] e skill tom-de-voz-quirk): 7·11·4, 80% pós-5º contato, 3% vs 1% Meta imob, João GO R$2.300→12 vendas, 500k→3M VGV, Gustavo R$108k/6k seguidores, Guarujá 1a8m, time-to-lead 5min vs 20 dias.
