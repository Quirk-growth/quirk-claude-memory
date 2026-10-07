---
name: project_apn_clinics
description: "Apresentação de Negócios da Quirk Clinics (estética/saúde) reconstruída em HTML claro, azul do logo"
metadata: 
  node_type: memory
  type: project
  originSessionId: 798c9556-748c-4496-87e4-6ab001869e18
  modified: 2026-10-07T02:28:09.122Z
---

APN da **Quirk Clinics** (unidade dedicada a clínicas de estética e saúde), reconstruída a partir do PDF image-based `~/Desktop/APN - Clinics.pdf` (20 págs) como deck HTML editável.

- Pasta: `~/apn-clinics-quirk/` — 18 slides HTML + `Apresentacao-Quirk-Clinics.pdf`. Render via Chrome headless 2x → PIL 288dpi (ver [[gotcha_html_print_pdf_preview]]).
- **Identidade LIGHT/clean** (não a dark do Growth): shared.css veio de `~/metodo-cresce-clinics/shared.css` (--qc-bg #FFF/#F5F7FB, texto #1F2430, Sora+Inter). Logo = `logo-clinics.png` (ícone) + wordmark "Quirk"(preto)+"Clinics"(azul).
- **Azul do logo = #4884E4** (amostrado do logo-clinics.png) — Renan pediu padronizar tudo nele; os azuis vívidos/neon originais (funil 3D) eram "muito diferentes do logo". --qc-blue setado p/ #4884E4, deep #2F6FC4, soft #E8F1FE.
- Mudanças do Renan (out/2026): funil comercial refeito DELICADO (degradê suave, sem neon); emojis trocados por ícones de linha clean em todo o deck; REMOVIDOS 3 slides (Cronograma de crescimento, Mentorias em grupo, Dados de mercado); NOVO slide "O que custaria montar esse time?" antes das ofertas (estilo print Syne: 5 profissionais = R$15k, "a partir de R$3.000/mês", economia R$12k/mês = R$144k/ano).
- Preços Clinics: Completo Multicanal R$15.000 · Multicanal+Social R$8.000 · Tráfego 1 plataforma R$3.000 · Social 1 plataforma R$3.000.
- Assets reaproveitados: `time-foto.png` (selfie) e `predio.png` do deck Growth; `equipe-grid.png` (24 pessoas c/ cargos) e `imersos-foto.png` (foto clínica) recortados do próprio PDF Clinics via PyMuPDF.
- Relacionado: [[project_apn_quirk_institucional]] (versão Growth high-ticket), [[project_metodo_cresce]].
