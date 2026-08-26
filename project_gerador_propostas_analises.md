---
name: gerador-de-propostas-agente-de-an-lise-rea-de-membros
description: "Duas ferramentas comerciais na área de membros — divisão de frentes entre sessões, decisões travadas e estado das branches"
metadata: 
  node_type: memory
  type: project
  originSessionId: abd490d3-d546-47fd-9269-c2122b669bb6
  modified: 2026-08-26T01:19:54.787Z
---

Ferramentas do comercial na área de membros (ago/2026), nascidas da skill [[proposta-quirk]] (~/.claude/skills/proposta-quirk/ — processo, regras de copy do Renan, 3 exemplos reais, spec do agente de análise em references/agente-analise-presenca.md).

**Decisões do Renan (25/08):** acesso APENAS comercial + admin (fail-closed) · vendedor digita valores livremente · PDF sai direto sem aprovação · Caminho B escolhido (ferramenta no portal via API do Claude; ninguém do time precisa de assinatura Claude).

**Divisão de frentes entre sessões paralelas (coordenada por mensagem):**
- **renanreal-79**: Gerador de Propostas inteiro — branch `feat/gerador-propostas` no checkout principal (Propostas.ts, src/lib/propostas/*, /api/propostas, página (app)/propostas). ELA MERGEIA PRIMEIRO.
- **Esta sessão (renanreal-35)**: Agente de Análise de Presença — branch `feat/analise-presenca` no worktree `/Users/renanreal/area-membros-quirk-analise`. Commitada, SEM push (aguarda DDL manual + merge da 79 pra rebasear e reutilizar o renderizarPdf compartilhado dela).
- **renanreal-40**: materiais estáticos (proposta-incorporadora, manual-comercial, guia-perfil) — dona do proposta.html da incorporadora daqui pra frente.

**O Agente de Análise entrega:** avaliação sem preços de 3 pilares (Site auto-fetch · Instagram e Anúncios colados pelo vendedor na v1) com nota 0-10 → PDF identidade Quirk + e-mail personalizado (≤120 palavras) pronto pra copiar. Limite 10 análises/dia por usuário. Exporta `gerarObservacoes({site,instagramDados})` pro campo observacoes do gerador.

**Gotchas técnicos descobertos:** args do @sparticuz/chromium travam launch do Chrome desktop no Mac (teste local = CHROME_EXECUTABLE_PATH + args mínimos) · puppeteer-core 25.9 sem networkidle0 no setContent (usar 'load' + document.fonts.ready) · worktree novo não tem .env (fica no checkout principal) · ANTHROPIC_API_KEY só existe no Render, sem teste de API local.

**Antes do deploy da minha branch:** DDL manual (payload_locked_documents_rels + enum_analises_status) via skill payload-migracao-prod; conflito esperado e aceito em payload.config.ts no rebase.
