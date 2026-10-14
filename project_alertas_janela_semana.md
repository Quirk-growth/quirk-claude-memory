---
name: project_alertas_janela_semana
description: Alertas da home e do sino só cobram a ÚLTIMA SEMANA (7 dias) — regra única em lib/alertas/janela.ts; backlog antigo some do alerta mas segue no módulo
metadata:
  node_type: memory
  type: project
  originSessionId: 00a59483-fcd7-4633-8233-fdeee4aea677
  modified: 2026-10-14T09:47:42.219Z
---

Pedido do Renan (14/out/2026): "o painel inicial de cada função, os alertas devem se referir a task recentes daquela semana, e não mostrar task ou notificações abertas a meses." Antes, a home dele mostrava "463 parados no funil (há 270 dias)", "2 contratos sem assinatura — agora" (contrato sem `criadoEm` valia "agora") e "22 pré-fechamentos há 189 dias".

**Regra única:** `src/lib/alertas/janela.ts` — `DIAS_JANELA_ALERTAS = 7`, `dentroDaJanelaDeAlertas(evento, agora)` (dias INTEIROS por piso: 7d23h59 conta, 8d+1min não, futuro não) e `inicioDaJanelaDeAlertas(agora)` = `agora − (7+1) dias` (o corte do banco TEM que ser 7+1 por causa do piso; cortar em 7 encolhe a janela pra 6 dias). Usada por home, crons e sino — nunca duplicar a conta.

**Onde vale:** home (`lib/home/acaoTarefas|acaoComercial|acaoContratual.ts`; `acaoClientes` já era 7d), crons `lib/tarefas/avisoVencidas.ts` e `lib/crm/avisoRetornosPendentes.ts` (piso no `where`), sino `lib/notificacoes/painel.ts` → `consultarSino` (notificações, contador de não lidas e "atribuídas a mim" = só ABERTAS criadas na semana).

**Efeitos pra lembrar ao monitorar:**
- contrato: janela 30→7 dias, data = `criadoEm` (Autentique) senão `createdAt` (fallback). ⚠️ O contrato ENVIADO PELO HUB (`/enviar-contrato`, Clientes.ts) nasce SEM `criadoEm` — só o `/vincular` grava — então "sem criadoEm → fora" (minha 1ª versão) teria escondido todo contrato novo do Hub; achado na revisão independente, medido em prod (14/out: 2 parciais sem criadoEm, 3 pendentes com). Sem nenhuma das duas datas → fora;
- "parado" no funil = parado há entre 5 e 12 dias (cruzou o limite dentro da semana); pré-fechamento sem `etapaDesde` fora;
- `notificacoesGeradas: 0` nos crons deixou de ser, sozinho, sinal de cron morto (ver docs/operacao/jobs-agendados.md);
- nada é apagado: notificação antiga fica no banco e no perfil da pessoa; tarefa/lead/contrato continuam nas telas do módulo.

**Texto da fila vazia:** "Tudo em dia" virou "Nada novo nesta semana" (+ contador `nada novo na semana` em `lib/home/fila.ts`) — com janela de 7 dias a fila vazia NÃO quer dizer que não há pendência antiga; o texto diz onde o resto continua (Tarefas, Funil, Contratos).

**Bug pré-existente achado junto (corrigido à parte em 14/out, branch `worktree-fix-prazo-dia-civil`, NÃO commitado/mergeado até o Renan ver o diff):** 205 de 207 tarefas abertas têm `prazo` gravado 00:00 UTC (= 21h do dia anterior em BRT), então `classificarPrazo` mostrava "atrasada" pra tarefa que vence hoje e "hoje" pra a de amanhã. Correção: em `lib/home/regras.ts` o dia do PRAZO é `toISOString().slice(0,10)` (data gravada) e o de `agora` segue `diaLocal` (São Paulo) — lados diferentes de propósito. `acaoTarefas.ts` e `janela.ts` NÃO mudaram. Mutação provou os 3 pontos (prazo, hoje, amanhã). Sobras conhecidas, não mexidas: entre 21h e 24h BRT o rótulo "há N dias" e a borda dos 7 dias (`dentroDaJanelaDeAlertas`, filtro de banco) ficam ~3h adiantados pra prazo 00:00Z, e `estaVencida`/cron `avisoVencidas` usam "hoje" em UTC (prazo de hoje vira "vencida" a partir das 21h BRT).

**Não testado (limite conhecido):** o pré-filtro de banco do `acaoComercial` (`or` de etapaDesde/slaVenceEm/dataAgendamento) existe pra não estourar `limit: 500`; a função pura filtra de qualquer jeito, então apagar um piso de data no banco não derruba teste (só a fronteira exata do corte derruba). Truncamento por limite exigiria 500+ linhas de fixture.

**How to apply:** qualquer alerta novo (home, sino, cron) deve importar de `lib/alertas/janela.ts` em vez de inventar janela. Pra mudar "uma semana" pra outro valor, só `DIAS_JANELA_ALERTAS` — mas conferir `DIAS_LEAD_PARADO + DIAS_JANELA_ALERTAS` no `acaoComercial`.
