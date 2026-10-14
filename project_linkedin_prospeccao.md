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

**Why:** novo produto de outbound pra Quirk.
**How to apply:** status volátil — perguntar ao Renan se o piloto começou antes de assumir.
