---
name: reference_quirk_docs
description: Onde estão os documentos, prompts e configs do projeto Quirk Auto Ads — repositório local em /Users/renanreal/quirk_auto_ads/ e referências externas (Make, Meta, claude.ai).
type: reference
originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
---
**Pasta local do projeto:** `/Users/renanreal/quirk_auto_ads/`
- `EXECUCAO_MULTI_CLIENTE.md` — documento de execução consolidado da arquitetura multi-cliente (D.1-D.4 + Data Store + validação + cadastro)
- `PUBLICOS_BM_QUIRK_2026-05-24.md` — snapshot dos ad sets ativos das 22 contas com spend > R$5k em 90d, gerado via Meta Ads MCP (sem targeting — só nomes + spend)

**Documentos anteriores (no Drive do Renan, formato .docx):**
- Prompt mestre Quirk v3.4 (cérebro do agente principal)
- Prompt classificador (CONFIRMADO/PENDENTE)
- Prompt extrator (JSON da campanha)
- Protocolo validação Fase 0 (régua de teste do prompt)
- Onboarding cliente v1 (Partnership BM)
- Pacote final Fase D v1 (versão anterior, mono-cliente)

**Recursos externos:**
- Make.com — cenário "Auto Ads - test (copy)", ID 4750002
- Meta for Developers — app "Quirk Auto Ads" (publicado, Live mode, Marketing API com Standard Access)
- Business Manager Meta — BM principal da agência: 803799600742642 ("Quirk - Growth Marketing"), gerencia ~219 contas de clientes; BM secundária 1612905538806887 ("BM - Quirk Auto Ads") usada no piloto Auto Ads
- Projeto no claude.ai — "Quirk Auto Ads — Fase 0" com custom instructions = prompt v3.4

**Migração de plataforma (2026-06-10):** A operação Quirk Auto Ads foi migrada por completo do Make.com para o **n8n**, rodando em produção. Não há conector n8n no registro MCP do Claude — a forma de o Claude executar no n8n é via API REST do n8n (`/api/v1/workflows`, `/activate`, `/executions?includeData=true`, header `X-N8N-API-KEY`) chamada por curl/Bash. Criar workflow = POST sem `active` (read-only); ativar = POST `/activate`; atualizar = PUT (substitui inteiro).

**Instância n8n:** `https://n8n.quirkgrowth.online/` — projeto pessoal "Renan Real <contato@quirkgrowth.com.br>". Endpoint `/api/v1/credentials` dá 403 (não lista credenciais); para achar IDs de credencial, ler o JSON de workflows existentes.

**Credenciais Google dentro do n8n (conta = contato@quirkgrowth.com.br, NÃO renan.reeal):**
- Google Sheets: cred id `0mCgIOcXR1YqK3nm` ("Google Sheets account"). Há também `PWAyQdmxBYpQzGgd` que aparece em workflows mas dá "credential does not exist" — não usar.
- Gmail: cred id `LhaU03ZBvdRKahBA` ("Gmail Quirk", usada 8x). Gmail node precisa `emailType:"html"` pro corpo HTML renderizar.
- ATENÇÃO conta cruzada: o conector Google Drive MCP do Claude é renan.reeal@gmail.com; a conta das credenciais n8n é contato@quirkgrowth.com.br. Planilha criada pelo Drive MCP (renan.reeal) NÃO é gravável pelo n8n (403 PERMISSION_DENIED) e o Drive MCP não tem ferramenta de compartilhar. Solução: criar a planilha PELO PRÓPRIO n8n (Google Sheets node, resource spreadsheet, operation create) — aí a conta n8n é dona e grava. Append com `mappingMode:autoMapInputData` numa aba vazia cria o cabeçalho a partir das chaves do item.

**Hospedagem das páginas estáticas do Auto Ads (jun/2026):** as páginas HTML do funil (`index.html`/vendas, `checkout.html`, `obrigado.html`, `onboarding.html` + `assets/`) estão no ar em **`https://autoads.quirkgrowth.com.br/`**, hospedadas no **cPanel** (conta `/home/renanrea`, mesma que serve `renanreal.com.br` como domínio primário e `lp.quirkgrowth.com.br`). DNS gerenciado pelo **Cloudflare** (não pelos nameservers do cPanel) — criar subdomínio só no cPanel não publica; precisa registro **A** no Cloudflare apontando pro mesmo IP do `lp` (todos os subdomínios da conta compartilham o IP do servidor; IP fica mascarado/proxied pelo Cloudflare). Document root do subdomínio = `public_html/autoads.quirkgrowth.com.br/` — os arquivos têm que ficar na RAIZ dessa pasta (não numa subpasta `autoradds/`, senão cai no site default `renanreal.com.br`). Renan tem acesso ao cPanel e ao Cloudflare da Quirk. O site principal da Quirk é Webflow (não dá subpasta), por isso o funil mora no cPanel do `lp`/`renanreal`.

**Conectores ativos no Claude Code (verificados em mai/2026):**
- Make MCP (Renan Real, renan.reeal@gmail.com) — ainda conectado, mas a operação já não roda no Make; consegue ler blueprint inteiro do cenário; NÃO consegue escrever blueprint completo de volta (181k chars excede limite)
- Meta Ads MCP — consegue listar contas/páginas e ler métricas (spend, CTR etc.); ~95 de 219 contas estão MCP-enabled (rollout gradual); NÃO expõe o campo `targeting` dos ad sets (sem interests/behaviors/demographics) — pra extrair targeting precisa Graph API direta com access token
