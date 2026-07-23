---
name: Projeto LP Iscas KPI + GPA
description: Landing page de captura standalone (HTML/CSS/JS vanilla) com formulário multi-step embedado, pra qualificar leads imobiliários da Quirk Growth
type: project
originSessionId: 27ea46d4-f02a-44c6-9485-06c9ecc1009e
---
## Resumo
Hospedada em `lp.quirkgrowth.com.br/presentekpi/`. **PIVOTOU (jun/2026) de presente grátis → ISCA PAGA "Kit Quirk" R$ 27,90** (preço-filtro: só quem paga é sério, e custeia a mídia). A versão grátis de 13 perguntas atraía agências demais.

**Entregáveis do kit (3):** Planilha de KPIs + GPA (Guia de Primeiro Atendimento) + **NOVO: Calculadora de CPL × Custo por Venda** (mesma da [[project_curso_cpl]]; tese "pare de comparar CPL, o que importa é custo por venda"). Print dela embutido na copy — tem bug de "RR$" (R$ duplicado), não corrigido.

**Fluxo atual:** form curto (só **nome/email/celular**) → captura no Make → **redireciona pro checkout da Green** (`payfast.greenn.com.br/tvj52m6/offer/KwxKhg`). A entrega dos materiais é pós-pagamento via Green (NÃO entregar grátis antes de pagar).

**Por quê:** receita + filtro de leads + caixa pra subsidiar anúncios.

## Localização
- Código: `/Users/renanreal/lp-iscas-quirk/`
- SPEC: `lp-iscas-quirk/SPEC.md`

## Stack
- HTML + CSS + JS vanilla (zero deps)
- Sora + Poppins via Google Fonts
- Form nativo multi-step com lógica condicional (NÃO usa Typeform)
- Hospedagem sugerida: Vercel ou subdomínio próprio (`materiais.quirkgrowth.com.br`)

## Integração Make
- **Cenário antigo (grátis): 4755230 "Isca Digital"** — fluxo completo com e-mail de ENTREGA dos materiais grátis + WhatsApp. NÃO reusar direto na versão paga (entregaria o material antes do pagamento).
- **Cenário NOVO (pago): 4802778 "Kit Pago R$27,90 - Captura"** (ativo, hook `qmyn1t8o7a9e1h3uvj3friza5ow3sxso`). Só captura, sem entrega grátis: webhook → ActiveCampaign (contato + lista 7) → Google Sheets aba **"Leads Isca KPI's"** (planilha `1WmzgFMidtIUMdg...`, leads pagos vão na MESMA aba dos grátis) → e-mail interno pro time → tarefa ClickUp (lista 900902474139). Conexões: AC 4684100, Sheets 1636819, Gmail 3698744, ClickUp 1770565. Payload do form: nome/email/whatsapp/origem/produto/submitted_at/utm_*.

## Estado (2026-06-26)
- **Entregável final: `lp-iscas-quirk/presentekpi-final.html`** — arquivo único autônomo (CSS/JS/imagens base64 inline), pronto pra subir. Captura plugada + checkout Green + DEBUG_MODE false.
- **Pixel Meta: Quirk Academy `1481911676389647`** (o antigo `905158130958739` foi removido). Purchase real acontece no checkout da Green (configurar o pixel lá pra fechar o funil).
- Acesso a arquivos no iCloud (`~/Library/Mobile Documents`) é bloqueado pelo macOS/TCC — pedir prints salvos no Desktop/Downloads.
- Pendências: limpar leads de teste ("Teste Kit Pago"), corrigir "RR$" da planilha CPL se quiser print limpo, decidir e-mail pro lead.
