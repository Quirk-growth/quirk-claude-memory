---
name: project-relatorio-social-media
description: "Relatório próprio de resultados de social media (página da equipe, link do cliente, miniaturas, análise por IA): PRONTO em branch local spec/relatorio-social-media, NÃO publicado; bloqueio de envio por dia parcial do sync; 3 DDLs antes do deploy"
metadata:
  node_type: memory
  type: project
---

**Pedido do Renan (19/10/2026):** relatório próprio de social media (não seção do relatório de anúncios) com miniaturas dos conteúdos que mais performaram, crescimento, alcance, novos seguidores e cidades; PDF e link para o cliente; resumo por IA como RASCUNHO que a social media edita e aprova. Frequência do plano 2/3/5 por semana é outro assunto (agente).

**Estado (21/10/2026):** worktree `~/area-membros-quirk-relatorio-social`, branch `spec/relatorio-social-media` (HEAD a62cda6f, 49 commits sobre origin/main 16caa8a5, rebaseada; backup `backup/relatorio-social-pre-rebase-20261021`), NÃO enviada. Spec `docs/superpowers/specs/2026-10-20-relatorio-resultados-social-media-design.md`, plano `.../plans/2026-10-20-relatorio-resultados-social-media.md`, ledger `.superpowers/sdd/2026-10-20-relatorio-resultados-social-media/progress.md` (rulings R1..R19), documento de operação `docs/operacao/relatorio-social-media.md` (ordem de implantação, mutações, limitações). 32 arquivos de teste/890 testes verdes; `typecheck:tests` tem 1 erro da main (`crm-follow-up-hook.int.spec.ts`/`estaPausado`); falha pré-existente `clientes-escrita-financeiro-comercial`.

**O que é:** rota da equipe `/relatorios/[clienteSlug]/social` (gate = item de matriz `/admin/relatorios-clientes` + carteira; `?mes=` mês fechado ou `?p=30d`); link do cliente `/resultados/[token]` (token por cliente, sem expirar, revogável, campo com `access.read:false`, não copia ao duplicar); rota de miniatura `/api/resultados/miniatura/[id]` (sessão ou `?t=`); capas de posts/reels copiadas para `media` (R2) no sync; `scripts/recuperar-miniaturas.mts` recupera capas antigas (dry-run sem rede); análise por IA (`claude-sonnet-5-5`, tool forçada, verificador de números por código, aprovação amarrada ao texto, 5 gerações/dia por cliente+mês, só mês fechado); quem gera link e aprova: admin, supervisor, administrativo e social da carteira (R12/R13).

**DEPLOY (só com autorização do Renan):** aplicar ANTES do código, nesta ordem, os 3 DDLs aditivos `scripts/ddl/2026-10-20-publicacoes-thumb-midia.sql`, `...clientes-resultados-social-token.sql`, `...relatorios-social.sql` (com `SET lock_timeout='5s'`, fora de 04:00 a 06:30 BRT) e rodar os `verificar-*.mts` (só leem o banco de teste: usar os SELECT do documento em produção). Código antes do DDL derruba o admin (42703).

**BLOQUEIO DE ENVIO A CLIENTE (C1, não bloqueia deploy):** o sync diário (07:00 UTC) grava métricas de fluxo com data=hoje UTC: `total_value` ~7h de dia parcial; `reach`/`follower_count` deslocados 1 dia (Pacífico); nunca regravado; o relatório conta como dia coberto. Confirmar com UMA leitura Graph autorizada (procedimento no doc, seção 7b) e corrigir buscando D-1 e D-2 + refazer ago-out. Pendente a autorização do Renan.

**Achados fora do escopo:** duplicar cliente copia briefingToken/briefingSocialToken/crmWebhookToken (o do CRM aceita leads em nome do original) — chip de tarefa criado; excluir cliente com posts do Instagram falhava na main (publicacoes_sociais.cliente_id NOT NULL + SET NULL), corrigido no hook desta branch. R0 pendente: 10 clientes da carteira sem ID do Instagram + 15 com ID quebrado (lista em `.superpowers/relatorio-social-ids-instagram.md`, fora do git); Renan deixou para a equipe preencher depois.

Relacionado: [[project_relatorio_perfil_ig]], [[project_agente_social_media]], [[project_programador_posts]], [[reference_payload_select_enum_ddl]].
