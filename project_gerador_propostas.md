---
name: gerador-de-propostas-area-membros
description: "Gerador de Propostas comerciais no portal (comercial+admin) — skill proposta-quirk como cérebro, Claude API + PDF no servidor; NO AR desde 25/08"
metadata: 
  node_type: memory
  type: project
  originSessionId: 654fb5d8-c7f7-464b-a130-7876c5612a0c
  modified: 2026-08-27T05:36:09.343Z
---

Feature do Caminho B (25/08/2026, commit 141b51c NO AR): página `/propostas` na área de membros gera propostas comerciais em PDF no padrão Quirk via API do Claude. Decisões do Renan: acesso **comercial + admin apenas**; **valores digitados livres**; **PDF sai direto** (sem aprovação).

**Cérebro:** skill `~/.claude/skills/proposta-quirk/` (processo, 3 modelos de oferta, regras de copy, exemplos reais das propostas Fiorucci/Zanardi/incorporadoras). O prompt do servidor (src/lib/propostas/gerarProposta.ts) embute as mesmas regras. Validada por teste A/B: baseline sem regras inventou preço âncora e usou 11 travessões.

**Arquitetura (repo area-membros-quirk):** coleção `propostas` (+ array `propostas_servicos`; DDL em scripts/ddl/2026-08-25-propostas.sql, aplicada em prod E teste) · `src/lib/propostas/` (template.ts com CSS+logo data-URI, gerarProposta.ts com validação anti-preço-inventado + 1 retry, renderizarPdf.ts puppeteer-core/@sparticuz concorrência 1 por causa do OOM do Render) · POST /api/propostas/gerar (gate por role, limite 10/dia, log de tokens, PDF → Media S3) · página (app)/propostas + item no menu Comercial.

**Gotchas desta entrega:** SDK Anthropic RECUSA create() não-streaming com max_tokens alto ("Streaming is required...") — usar messages.stream()+finalMessage() (fix f5ed9e4) · Payload aplica defaultMaxTextLength=40000 em text/textarea SEM maxLength explícito — o htmlFonte do deck (60-100k chars) caiu com "O campo a seguir está inválido: HTML fonte"; fix = maxLength grande no campo (f6d1c8c, 27/08; validação apenas, varchar já é ilimitado, sem DDL). Nessa falha o PDF já tinha subido pro Media — dá pra recuperar religando pdf_id no registro de erro (feito pra Syne, media 40 → proposta 2) · página de equipe NÃO pode morar no grupo (app), o layout expulsa papéis de equipe pro /admin — usar grupo (equipe) (fix 0d2d4e7) ·  push do Drizzle TRAVA (>10min) no banco de teste inchado quando há coleção nova — pré-criar as tabelas no teste com a mesma DDL destrava · setContent do puppeteer-core 25.9 não aceita networkidle0, usar 'load' + document.fonts.ready · custo por proposta ~R$ 0,50-2 (sonnet-5, ~30k tokens saída).

**Fotos institucionais (27/08, commit 97847c0):** src/lib/propostas/fotos.ts guarda 4 data URIs JPEG extraídos da APN oficial (equipe na sede, sede aérea Perdizes/Allianz, retratos Renan e Chaiene); modelo escreve placeholders FOTO_EQUIPE/FOTO_SEDE/FOTO_RENAN/FOTO_CHAIENE e o servidor substitui — LOGO É SUBSTITUÍDO ANTES das fotos porque "LOGO" pode ocorrer dentro do base64 delas (FOTO_* tem underscore, imune). Duas páginas obrigatórias: Quem somos (foto equipe + institucionais) e Liderança e sede (3 colunas pcard). Fix da barra no logo: .rule ganhou margin:22px 0 + prompt proíbe rule encostada no logo (era o bug "barra sobrepondo o logo" da capa/fechamento). Fonte das fotos: PDF "APN - Assessoria de Performance" no Desktop (pymupdf extrai).

**Pendência/handoff:** outra sessão deixou o "Agente de Análise de Presença" commitado no worktree `area-membros-quirk-analise` (branch feat/analise-presenca, coleção `analises`, spec em ~/.claude/skills/proposta-quirk/references/agente-analise-presenca.md), aguardando rebase pra importar o renderizarPdf compartilhado; DDL da `analises` ainda não aplicada. Falha pré-existente no main: teste "Pessoas" do menuAdmin espera 'role' no href de time-quirk (frente de outra sessão).
