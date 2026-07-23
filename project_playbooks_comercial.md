---
name: material-comercial-quirk-cultura-3-playbooks
description: Cultura Comercial + playbooks SDR/SS/Closer da Quirk em deck PDF — fontes HTML editáveis e como gerar o PDF
metadata: 
  node_type: memory
  type: project
  originSessionId: 654fb5d8-c7f7-464b-a130-7876c5612a0c
---

Material do departamento comercial da Quirk, em deck 16:9 (PDF), criado jun/2026. Mesma identidade visual (ver [[reference_quirk_brand]]): Sora+Poppins, azul #1D80FF, fundo #001D41, verde só em acentos sutis, logo oficial via `assets/quirk-icon.png`.

**3 papéis do comercial:** SS (Instagram, prospecção) → SDR (telefone, qualifica/agenda) → Closer (SPIN, fecha). Cada lead passa pela cadeia.

**Pasta:** `/Users/renanreal/manual-comercial-quirk/`
- `manual.html` → **Playbook do SDR** (19 págs). PDF: `Playbook_SDR_Quirk.pdf`
- `ss-playbook.html` → **Playbook do SS** (20 págs). PDF: `Playbook_SS_Quirk.pdf`
- `cultura-comercial.html` → **Cultura Comercial** (13 págs, 6 pilares que conectam os 3 papéis). PDF: `Cultura_Comercial_Quirk.pdf`
- Playbook do Closer = PDF externo `Downloads/SPIN - Closer Quirk (1).pdf` (não recriado; é a fonte do método SPIN/5 portas).
- `gf.css` + `fonts/` = fontes locais (não depende de rede). `assets/quirk-icon.png` = ícone oficial recortado.
- PDFs finais entregues em `/Users/renanreal/Desktop/`.

**Gerar PDF** (Chrome headless, fidelidade total):
`"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --no-pdf-header-footer --virtual-time-budget=10000 --print-to-pdf="OUT.pdf" "file://.../ARQUIVO.html"`

**Premissas-chave:** SDR = 200 contatos/dia, ligação primeiro, 4 pontos de contato no Dia 1, cadência Dia 1→7. SS = Instagram, 300–400 contatos/dia, sondagem com perguntas abertas, paciência (nunca agressivo), máx. 100 ações/perfil (senão bloqueia), handoff de lead qualificado. Closer = SPIN (situação/problema/implicação), 5 portas, "custo de continuar igual".

**Os 6 pilares da Cultura Comercial:** 1) Quem pergunta, conduz 2) Diagnóstico antes da solução 3) Venda é transferência de sentimentos 4) Disciplina vence talento 5) Entregamos crescimento, não tráfego 6) Um time, um funil. Norte central (do Closer): "fazer o cliente perceber o custo de continuar igual".
