---
name: gotcha_render_cron_sem_app_url
description: "Cron Job do Render falha em silêncio quando falta APP_URL nas variáveis DELE (cada cron tem o próprio conjunto); a lista de serviços mostra status de BUILD, não de execução"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-09-18T21:15:49.566Z
---

**Incidente real (18/09/2026):** o aviso de tarefa vencida (`Area-Membros-Notifications`, `crn-d9sgikijobas73fa6n90`) não gerava notificação desde 11/08 — 5 semanas. O agendamento existia, rodava todo dia às 07:00 UTC e **falhava todo dia**: `curl: (3) URL using bad/illegal format or missing URL`, porque o comando é `curl -fsS -X POST "$APP_URL/api/cron/tarefas-vencidas" -H "x-sync-secret: $SYNC_SECRET"` e **`APP_URL` não existia nas variáveis daquele cron** (só `PORT` e `SYNC_SECRET`). No Render, **cada Cron Job tem o próprio conjunto de variáveis** — não herda nada do serviço web. Conserto: adicionar `APP_URL=https://membros.quirkgrowth.com.br` no próprio cron. Verificado: `HTTP 200 {"ok":true,"tarefasVencidas":46,"notificacoesGeradas":56}` (56 avisos, 12 pessoas).

**Duas armadilhas de diagnóstico que custaram tempo:**
1. **A lista de serviços do dashboard mostra o status do BUILD, não da execução.** O cron aparecia como "Successful build" — e todas as 30 execuções estavam com ✗. O sinal certo está na página do cron: a frase **"No successful runs yet"** e os ícones vermelhos na lista de Runs. Compare entre crons: os que funcionam não têm essa frase.
2. **O número da auditoria estava errado.** O relatório dizia "834 tarefas vencidas"; medindo com o mesmo filtro do código (`concluida_em is null AND prazo < hoje 00:00 UTC`) são **46**, das quais 28 têm responsável. Sempre reproduzir o filtro do código antes de citar número.

**How to apply — diagnosticar cron do Render sem expor segredo:** abrir `/cron/<id>/settings` e ler o campo `command` (o valor usa `$VARIAVEL`, não o segredo); conferir os NOMES em `/cron/<id>/env`; e, pra testar de verdade, usar o **Web Shell do próprio cron** (`/cron/<id>/shell`) com `curl -sS -o /tmp/r.json -w "HTTP %{http_code}\n" -X POST "$APP_URL/<rota>" -H "x-sync-secret: $SYNC_SECRET"; cat /tmp/r.json` — digita-se só o NOME da variável, e a saída traz status + corpo. O botão "Trigger Run" não registrou execução nas tentativas (a página não atualizou); o Shell resolveu.

Os outros 4 crons (`sync-diario-painel`, `crm-follow-up`, `publicar-posts`, `limpeza-midia-posts`) foram conferidos no mesmo dia e têm execuções bem-sucedidas. Nenhum cron está no `render.yaml` — existem só no painel, ver [[project_area_membros_quirk]] e o plano de remediação em `docs/superpowers/plans/2026-09-18-plano-remediacao-auditoria.md`.
