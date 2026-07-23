---
name: Projeto LP Calculadora de VGV Perdido
description: LP isca "Calculadora de VGV Perdido + Raio-X Comercial" da Quirk; código local + cenário Make clonado da isca KPI
type: project
originSessionId: a25caae7-e2a4-4c33-a0c8-b30369b0ecac
---
LP lead-magnet **"Calculadora de VGV Perdido + Raio-X Comercial"** — vanilla JS (HTML/CSS/ESM, sem deps), testes via `node --test`.

- **Código:** `/Users/renanreal/lp-vgv-quirk/` (engine.js calc puro, form.js multi-step + POST webhook, result.js tela de resultado). Preview via `~/.claude/launch.json` (python http.server :8090).
- **Pipeline Make:** cenário **4779375 "Calculadora VGV (UAZAPI - API)"** (team 385200), **clonado** do cenário isca 4755230 — mesmo fluxo (webhook → ActiveCampaign → e-mails → Google Sheets → ClickUp → WhatsApp UAZAPI), **mesma planilha** (`1WmzgFMidtIUMdg_Qg11AXlduPoY_qghrjaiQnlHxnu0`) mas aba própria **"Leads VGV"** (18 colunas). Hook 2743474, URL `https://hook.us1.make.com/s6a9j5wwgbkplsu81yfv0ncqiy6a8hkc`.
- Comunicação (e-mail lead/interno, ClickUp, msg WhatsApp) adaptada pro contexto VGV: Índice de Eficiência, gargalo, VGV na mesa/ano.
- CTA WhatsApp em result.js → `wa.me/5511965951288` (número Quirk/UAZAPI).

**Why:** nova isca da Quirk, reaproveita infra comprovada da isca KPI pra captar/qualificar leads imobiliários.
**How to apply:** mexer no fluxo Make = clonar/ajustar a partir do 4779375; envio real dispara e-mail+WhatsApp, cuidado ao testar com submissão de form (cria lead real).
