---
name: reference_render_criar_cron_job
description: "Como criar Cron Job novo no Render pra área de membros — formulário vem como Docker, precisa trocar pra Node + `echo sem-build`; SYNC_SECRET é colado pelo Renan; jobs do Quirk Intelligence criados em 13/10/2026"
metadata:
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-13T20:34:44.533Z
---

Crons do Quirk Intelligence criados no Render em 13/10/2026 (todos Node, Virginia, 0.5 CPU/512 MB, build `echo sem-build`, env `PORT`+`APP_URL`+`SYNC_SECRET`):
- `quirk-intelligence-limpeza-memoria` (`crn-db78h5ks728c73fhpevg`) — `0 8 * * *`
- `quirk-intelligence-alertas-custo` (`crn-db78r7bbc2fs73e8hoj0`) — `0 10 * * *`
- `quirk-intelligence-alerta-saldo` (`crn-db7964favr4c73fcjh90`) — `0 10 * * *`

Transcrição de calls (16/10/2026), mesmo molde:
- `transcricoes-sessoes-orfas` (`crn-db93s4jbc2fs73conbkg`) — `0 9 * * *`, fecha
  gravação presa em "gravando" há mais de 6h. Criado com o Claude preenchendo
  tudo e o Renan colando o `SYNC_SECRET`; validado por **Trigger Run manual**
  (verde em 18,9s). O `-fsS` do curl é o que torna esse teste conclusivo:
  segredo errado viraria 401 e a execução ficaria VERMELHA, não verde silencioso.

**Gotcha do formulário (`/cron/new`):** o Render autodetecta **Docker** (o repo tem Dockerfile) e preenche build `npm install; npm run build`. Os crons existentes são **Node** com build `echo sem-build` — trocar Language e Build Command, senão builda a app inteira à toa. Não há Environment Group, então cada cron leva as próprias variáveis (ver [[gotcha_render_cron_sem_app_url]]).

**SYNC_SECRET:** é segredo — o Claude não digita nem lê. Preenche tudo o resto, deixa o `value` vazio e o Renan cola (copiar do env do `sync-diario-painel`). Nunca clicar em "Generate" (cria valor aleatório diferente do que o app espera).

**Validar sem disparar alerta:** Web Shell do próprio cron (`/cron/<id>/shell`, leva ~15s pra conectar) e chamar a rota inofensiva de limpeza (`.../quirk-intelligence-limpeza-memoria`) com `$APP_URL`/`$SYNC_SECRET` — prova que as variáveis daquele cron estão certas sem mandar Telegram. As rotas `alertas` e `alerta-saldo` mandam mensagem de verdade a cada chamada.
