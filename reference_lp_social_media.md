---
name: reference_lp_social_media
description: "LP Social Media Estratégica Imobiliária + cenário Make 4493780 (webhook, contrato de campos, mapeamento planilha/ClickUp/WhatsApp)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8056b727-932f-4614-a7ae-145c166b8320
---

LP "Social Media Estratégica Imobiliária" (Quirk) — página HTML custom (dc-runtime/bundler) que substitui a antiga LP Elementor. Publicada via cPanel; imagens puxadas de `https://lp.quirkgrowth.com.br/wp-content/uploads/2026/07/` (pasta `assets-para-wordpress/` no zip). Pixel Meta `1584493379956270` embutido no `<head>` (jul/2026).

**Formulário → webhook:** o `onSubmit` faz `FormData → URLSearchParams POST` (mode no-cors) para `https://hook.us1.make.com/iu4xnbrmz6fgwsx5cjuvnsrskpetp527` (hookId 2572028, sem auth). Campos do form (name=): `nome, email, telefone, instagram, nicho, faturamento` (+ extras ignorados `destino`, `origem`). `nicho` agora é input de texto aberto ("Seu segmento"), `faturamento` segue select. ⚠️ telefone não é sanitizado — se vier com máscara, quebra o WhatsApp (`55{telefone}`).

**Deploy/reposicionamento (jul/2026):** hospedada em cPanel próprio no subdomínio `socialmedia.quirkgrowth.com.br` (atrás de Cloudflare — purgar cache ao trocar). O arquivo publicado é o **bundle self-contained** (~23MB, `index.html`) — NÃO a versão leve `Social-Media-Imobiliaria.html` (essa é export de *preview*: depende de React+Babel do unpkg e transpila no browser → tela preta em produção). Tela de carregamento do bundle foi limpa (spinner azul no lugar do "Q"/"Unpacking"). Copy foi **reposicionada de "órbita imobiliária" para "serviços High Ticket"** genérico (removidas menções a imobiliária/incorporadora/arquiteto).

**Tracking (jul/2026):** o pixel Meta direto foi REMOVIDO; agora a página usa **Google Tag Manager `GTM-TGBQHFJ9`** (snippet no head + noscript no body, dentro do template pra sobreviver ao `documentElement.replaceWith`). No submit dispara `dataLayer.push({event:'generate_lead', event_id, lead_email, lead_telefone, lead_nicho, ...})`; o `event_id` também vai pro Make (`data.event_id`) pra dedup futura com CAPI. Vídeos YouTube com `enablejsapi=1` (gatilho de vídeo GTM). As tags (Meta Pixel base All Pages + Meta Lead no evento generate_lead com Advanced Matching e eventID) precisam ser criadas no painel do GTM. CAPI é per-dataset (pixel 1584493379956270), não por subdomínio; subdomínio herda verificação do domínio raiz.

**Cenário Make 4493780** "Leads Quirk [Social Media] (UAZAPI - API)" (team 385200): webhook → Email (contato@quirkgrowth + yuri + rodrigo), Google Sheets addRow (planilha id 1WmzgFMidtIUMdg_Qg11AXlduPoY_qghrjaiQnlHxnu0 "Leads Growth", aba **"Lead SM"** — movido da Página1 em jul/2026 pra isolar leads de Social Media; A=nome, C=telefone, D=instagram, E=email, F=faturamento, G=nicho, H=data; B/Empresa fica vazia; aba precisa ter cabeçalho na linha 1), ClickUp createTaskInList (list 900902474139, tag "social media"), Router→HTTP UAZAPI `quirkgrowth.uazapi.com/send/text`. Data usa `{{formatDate(now; "DD/MM/YYYY HH:mm")}}`. A LP antiga era Elementor com campos `field_0731945/be27370/f11eb72` + "Empresa"+"Qual é tamanho da sua equipe?" — descontinuados.

Relacionado: [[project_quirk_auto_ads]], [[reference_whatsapp_cloud_api]], [[reference_quirk_docs]].
