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

EXECUTOR (Plano B) PRONTO PRO ENSAIO 15/10: repo local ~/robo-linkedin-quirk (sem remoto ainda; commit 1ecd60a, 239 testes), login do LinkedIn salvo em ~/robo-linkedin-quirk/perfil-chrome. Ledger: area-membros-quirk-robo-linkedin/.superpowers/sdd/2026-10-13-robo-linkedin-B-executor/progress.md. I4 (desde=msg1Em??convidadoEm??aceitoEm) NO AR 15/10 (main 14011896); ensaio real; seletores sintetizados (Pendente, modal Enviar sem nota, bolha, retirar) verificar ao vivo; criar repo GitHub privado. Messaging do LinkedIn só expõe URN (/in/ACoA…) → resolvedor no executor com cache.

ENSAIO REAL rodando desde 15/10 no Mac (npm start + caffeinate): 1º dia achou 2 bugs só visíveis ao vivo — top card real é DIV [componentkey$=Topcard] com nome em h2 (8d010a6); tsx injeta __name em funções passadas ao page.evaluate → usar string via funcaoDaPagina (e550f36). Após fixes: convidar ensaio ok (ação 10). 4 prospects ficaram em revisão (nada enviado). Ainda não visto ao vivo: modal Enviar sem nota, Pendente, bolha de mensagem, retirar.

STATUS 17/10: painel novo NO AR (main aeaef0fa, DDL 2026-10-15 aplicada): status com cor, computador conectado (trava 1 por vez, cap 240s), comandos desligar/reiniciar remotos. Executor 0.2.2 (dc0a1d8) rodando no Mac com caffeinate; Renan JÁ SAIU DO ENSAIO 16/10 (inicioOperacao 16/10) — 1º convite real seg 19/10 9h. Fixes ao vivo: nome com parênteses, menu Mais, bolha compose da msg1, conversa só nome idêntico. Pacote PC atualizado em ~/Desktop/Robo-LinkedIn-Quirk(.zip). 5 prospects em revisão (nada enviado). Teste de integração do cérebro: banco de teste compartilhado vive clobbered por outros worktrees → usar banco isolado temporário no host de teste.

1º DIA REAL 19/10: 5 convites enviados, 1 aceite (msg1 sai 20/10 9h), 1 botao_indisponivel; manhã pausou por 'conversa aberta que não fecho' (causa desconhecida → v0.2.1 loga diagnóstico mascarado + Escape/feed seguro). 8 prospects em revisão (nada enviado).

GOTCHA 21/10: overlay de mensagens do LinkedIn vive em shadow root (#interop-outlet) — page.evaluate/document.querySelector NÃO enxerga, só locators; bolha que fica aberta após enviar msg1 travava os convites seguintes (fix dc0a1d8: fecha logo após enviar, fechar só no cabeçalho).

MIGROU PRO PC 21/10: executor 0.2.2 rodando no PC 'DESKTOP-2P9JE7K' (Windows); Mac DESLIGADO (não religar no Mac — trava 1 computador). Após '2 - Entrar no LinkedIn.bat' a tarefa não iniciou sozinha; 'Ligar o programa.bat' resolveu — conferir se a tarefa AtLogOn liga após reiniciar o PC. Atualizar o PC = novo pacote ZIP (sem GitHub ainda).

**Why:** novo produto de outbound pra Quirk.
**How to apply:** status volátil — perguntar ao Renan se o piloto começou antes de assumir.
