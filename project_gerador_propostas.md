---
name: gerador-de-propostas-area-membros
description: Gerador de Propostas comerciais no portal (comercial+admin) — skill proposta-quirk como cérebro, Claude API + PDF no servidor; NO AR desde 25/08
metadata:
  type: project
---

Feature do Caminho B (25/08/2026, commit 141b51c NO AR): página `/propostas` na área de membros gera propostas comerciais em PDF no padrão Quirk via API do Claude. Decisões do Renan: acesso **comercial + admin apenas**; **valores digitados livres**; **PDF sai direto** (sem aprovação).

**Cérebro:** skill `~/.claude/skills/proposta-quirk/` (processo, 3 modelos de oferta, regras de copy, exemplos reais das propostas Fiorucci/Zanardi/incorporadoras). O prompt do servidor (src/lib/propostas/gerarProposta.ts) embute as mesmas regras. Validada por teste A/B: baseline sem regras inventou preço âncora e usou 11 travessões.

**Arquitetura (repo area-membros-quirk):** coleção `propostas` (+ array `propostas_servicos`; DDL em scripts/ddl/2026-08-25-propostas.sql, aplicada em prod E teste) · `src/lib/propostas/` (template.ts com CSS+logo data-URI, gerarProposta.ts com validação anti-preço-inventado + 1 retry, renderizarPdf.ts puppeteer-core/@sparticuz concorrência 1 por causa do OOM do Render) · POST /api/propostas/gerar (gate por role, limite 10/dia, log de tokens, PDF → Media S3) · página (app)/propostas + item no menu Comercial.

**Gotchas desta entrega:** SDK Anthropic RECUSA create() não-streaming com max_tokens alto ("Streaming is required...") — usar messages.stream()+finalMessage() (fix f5ed9e4) · página de equipe NÃO pode morar no grupo (app), o layout expulsa papéis de equipe pro /admin — usar grupo (equipe) (fix 0d2d4e7) ·  push do Drizzle TRAVA (>10min) no banco de teste inchado quando há coleção nova — pré-criar as tabelas no teste com a mesma DDL destrava · setContent do puppeteer-core 25.9 não aceita networkidle0, usar 'load' + document.fonts.ready · custo por proposta ~R$ 0,50-2 (sonnet-5, ~30k tokens saída).

**Pendência/handoff:** outra sessão deixou o "Agente de Análise de Presença" commitado no worktree `area-membros-quirk-analise` (branch feat/analise-presenca, coleção `analises`, spec em ~/.claude/skills/proposta-quirk/references/agente-analise-presenca.md), aguardando rebase pra importar o renderizarPdf compartilhado; DDL da `analises` ainda não aplicada. Falha pré-existente no main: teste "Pessoas" do menuAdmin espera 'role' no href de time-quirk (frente de outra sessão).
