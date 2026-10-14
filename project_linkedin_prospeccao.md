---
name: project-linkedin-prospeccao
description: Estudo (13/10/2026) pra Quirk criar automação de prospecção no LinkedIn estilo Content Soluções; recomendação Unipile dentro do CRM
metadata:
  type: project
---
Renan quer replicar o modelo da Content Soluções (Netwall; chat.contentsolucoes.com.br): convida/segue ~30/dia no LinkedIn, msg automática no aceite, resposta = levantada de mão → time agenda call.
Relatório em ~/reports/Automação de prospecção LinkedIn.md (notas em ~/research_notes/Automação de prospecção LinkedIn/).
Recomendação: build dentro do [[project-crm-quirk]] sobre Unipile (€49/mês até 10 contas, LinkedIn+WhatsApp), piloto antes com trial HeyReach. 30/dia passa do teto (~100/semana); vender 15-20/dia. Funil esperado 1-2 reuniões/conta/mês. Viola ToS 8.2 — risco de ban é do dono do perfil, precisa cláusula no contrato.

DECISÃO 13/10: robô 100% próprio (Playwright, Chrome real) no PC Windows do Renan, SÓ o perfil do Renan, cérebro no CRM (cliente 104), busca do LinkedIn como origem, msg1+2 follow-ups, mão levantada = lead + WhatsApp (time assume pelo WhatsApp). Spec em worktree ~/area-membros-quirk-robo-linkedin branch feat/robo-linkedin, docs/superpowers/specs/2026-10-13-robo-linkedin-design.md + planos A (cérebro, 10 tasks) e B (executor, 9 tasks) em docs/superpowers/plans/2026-10-13-robo-linkedin-{A-cerebro,B-executor}.md (commit 5e844854, não pushado). Sales Navigator fora do MVP. Executor vai em repo novo robo-linkedin-quirk. CRM exige lead sem telefone (índice único cliente+telefone).

STATUS 14/10: Plano A (cérebro) IMPLEMENTADO na branch feat/robo-linkedin (worktree ~/area-membros-quirk-robo-linkedin, ~20 commits até afbc9fde, NÃO mergeado/pushado/deployado). DDL em scripts/ddl/2026-10-13-robo-linkedin.sql (provado só em schema descartável do banco de teste). R1-R3 corrigidos. NO AR 14/10 (main 0fa84222): DDL aplicada em prod e conferida, LINKEDIN_ROBO_TOKEN criado no Render via Generate (valor só no Render — copiar de lá pro config.json do executor), cron linkedin-verificar criado. Robô nasce pausado + ensaio. git push e DDL de prod são bloqueados pelo classifier até o Renan dizer explicitamente 'execute'. Ledger: .superpowers/sdd/2026-10-13-robo-linkedin-A-cerebro/progress.md. Plano B (executor) não iniciado. Tela: menu Comercial → Robô LinkedIn (só admin) — só aparece após deploy.

**Why:** novo produto de outbound pra Quirk.
**How to apply:** status volátil — perguntar ao Renan se o piloto começou antes de assumir.
