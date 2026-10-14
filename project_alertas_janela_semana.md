---
name: project_alertas_janela_semana
description: "Alertas da home e do sino só cobram a ÚLTIMA SEMANA (7 dias) — regra única em lib/alertas/janela.ts; backlog antigo some do alerta mas segue no módulo"
metadata:
  node_type: memory
  type: project
---

Pedido do Renan (14/out/2026): "o painel inicial de cada função, os alertas devem se referir a task recentes daquela semana, e não mostrar task ou notificações abertas a meses." Antes, a home dele mostrava "463 parados no funil (há 270 dias)", "2 contratos sem assinatura — agora" (contrato sem `criadoEm` valia "agora") e "22 pré-fechamentos há 189 dias".

**Regra única:** `src/lib/alertas/janela.ts` — `DIAS_JANELA_ALERTAS = 7`, `dentroDaJanelaDeAlertas(evento, agora)` (dias INTEIROS por piso: 7d23h59 conta, 8d+1min não, futuro não) e `inicioDaJanelaDeAlertas(agora)` = `agora − (7+1) dias` (o corte do banco TEM que ser 7+1 por causa do piso; cortar em 7 encolhe a janela pra 6 dias). Usada por home, crons e sino — nunca duplicar a conta.

**Onde vale:** home (`lib/home/acaoTarefas|acaoComercial|acaoContratual.ts`; `acaoClientes` já era 7d), crons `lib/tarefas/avisoVencidas.ts` e `lib/crm/avisoRetornosPendentes.ts` (piso no `where`), sino `lib/notificacoes/painel.ts` → `consultarSino` (notificações, contador de não lidas e "atribuídas a mim" = só ABERTAS criadas na semana).

**Efeitos pra lembrar ao monitorar:**
- contrato: janela 30→7 dias; sem `criadoEm` NÃO conta mais;
- "parado" no funil = parado há entre 5 e 12 dias (cruzou o limite dentro da semana); pré-fechamento sem `etapaDesde` fora;
- `notificacoesGeradas: 0` nos crons deixou de ser, sozinho, sinal de cron morto (ver docs/operacao/jobs-agendados.md);
- nada é apagado: notificação antiga fica no banco e no perfil da pessoa; tarefa/lead/contrato continuam nas telas do módulo.

**Não testado (limite conhecido):** o pré-filtro de banco do `acaoComercial` (`or` de etapaDesde/slaVenceEm/dataAgendamento) existe pra não estourar `limit: 500`; a função pura filtra de qualquer jeito, então apagar um piso de data no banco não derruba teste (só a fronteira exata do corte derruba). Truncamento por limite exigiria 500+ linhas de fixture.

**How to apply:** qualquer alerta novo (home, sino, cron) deve importar de `lib/alertas/janela.ts` em vez de inventar janela. Pra mudar "uma semana" pra outro valor, só `DIAS_JANELA_ALERTAS` — mas conferir `DIAS_LEAD_PARADO + DIAS_JANELA_ALERTAS` no `acaoComercial`.
