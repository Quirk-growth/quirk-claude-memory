---
name: gerador-de-propostas-agente-de-an-lise-rea-de-membros
description: "Duas ferramentas comerciais na área de membros — divisão de frentes entre sessões, decisões travadas e estado das branches"
metadata: 
  node_type: memory
  type: project
  originSessionId: abd490d3-d546-47fd-9269-c2122b669bb6
  modified: 2026-08-26T21:10:13.372Z
---

Ferramentas do comercial na área de membros (ago/2026), nascidas da skill [[proposta-quirk]] (~/.claude/skills/proposta-quirk/ — processo, regras de copy do Renan, 3 exemplos reais, spec do agente de análise em references/agente-analise-presenca.md).

**Decisões do Renan (25/08):** acesso APENAS comercial + admin (fail-closed) · vendedor digita valores livremente · PDF sai direto sem aprovação · Caminho B escolhido (ferramenta no portal via API do Claude; ninguém do time precisa de assinatura Claude).

**Divisão de frentes entre sessões paralelas (coordenada por mensagem):**
- **renanreal-79**: Gerador de Propostas inteiro — branch `feat/gerador-propostas` no checkout principal (Propostas.ts, src/lib/propostas/*, /api/propostas, página (app)/propostas). ELA MERGEIA PRIMEIRO.
- **Esta sessão (renanreal-35)**: Agente de Análise de Presença — branch `feat/analise-presenca` no worktree `/Users/renanreal/area-membros-quirk-analise`. Commitada, SEM push (aguarda DDL manual + merge da 79 pra rebasear e reutilizar o renderizarPdf compartilhado dela).
- **renanreal-40**: materiais estáticos (proposta-incorporadora, manual-comercial, guia-perfil) — dona do proposta.html da incorporadora daqui pra frente.

**O Agente de Análise entrega:** avaliação sem preços de 3 pilares (Site auto-fetch · Instagram e Anúncios colados pelo vendedor na v1) com nota 0-10 → PDF identidade Quirk + e-mail personalizado (≤120 palavras) pronto pra copiar. Limite 10 análises/dia por usuário. Exporta `gerarObservacoes({site,instagramDados})` pro campo observacoes do gerador.

**Gotchas técnicos descobertos:** args do @sparticuz/chromium travam launch do Chrome desktop no Mac (teste local = CHROME_EXECUTABLE_PATH + args mínimos) · puppeteer-core 25.9 sem networkidle0 no setContent (usar 'load' + document.fonts.ready) · worktree novo não tem .env (fica no checkout principal) · ANTHROPIC_API_KEY só existe no Render, sem teste de API local.

**NO AR (26/08):** Análise de Presença deployado — main em `55fe6b9`, smoke test verde (POST /api/analises/gerar = 403; página /analises redireciona pro login). DDL de prod aplicada antes do push. Renderizador DELEGA pro renderizarPdf compartilhado da 79 (fila única de Chromium). Página fica no grupo `(equipe)` — o grupo `(app)` tem gate da área do cliente que barra o papel comercial (fix espelhado do 0d2d4e7 do gerador). Chamada ao Claude em **streaming** (stream()+finalMessage()) — SDK recusa create() não-streaming com max_tokens alto (fix espelhado do f5ed9e4). Runner de DDL: scripts/aplicar-ddl.mts (bloqueia -pooler). Falta validar geração real com usuário logado (ANTHROPIC_API_KEY só no Render).

**GOTCHA novo do banco de teste:** virou terra disputada — sessões paralelas rodando suíte/push com código da main DROPAM tabelas que não estão no schema delas (minha analises sumiu 2x). Validação atômica = aplicar DDL + introspectar NA MESMA CONEXÃO. E o drizzle push interativo trava em prompt "create or rename" por causa da tabela órfã permissoes_padrao no teste (não existe em nenhuma collection do código).

**Pré-existente na main (não meu, verificado no checkout principal em 0d2d4e7):** typecheck:tests com 98 erros (tipos gerados dessincronizados por outra sessão) e 4 testes vermelhos (tarefa-templates-aplicar, tarefas-recorrencia, agenda-conectar-form, menuAdmin "Pessoas…role" — esse último é da feature de permissões que está no checkout -crm sem push).
