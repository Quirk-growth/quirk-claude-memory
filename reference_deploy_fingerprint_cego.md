---
name: reference_deploy_fingerprint_cego
description: "GOTCHA: o fingerprint do JS de /admin/login pra confirmar deploy é CEGO a mudanças de CSS ou de rotas não-compartilhadas — não confirmar/negar deploy por ele nesses casos"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-14T19:59:37.692Z
---

Truque que uso pra saber quando um deploy do Render assumiu em prod (área de membros): capturo o md5 dos `static/chunks/*.js` da tela `/admin/login` antes do push e faço polling até mudar (`fp_pre.txt` + loop no scratchpad). Funciona bem pra mudanças em componentes COMPARTILHADOS (lista, chips, hub) que entram nos chunks do login.

**GOTCHA (13/ago):** o método é **cego** quando a mudança NÃO toca os assets da tela de login — ex.: só CSS (`custom.scss`), ou só uma rota específica não carregada no login (ex.: `AgendaTimeView`). O Next faz **code-split de CSS e JS por rota**, então o deploy pode assumir 100% ok e o fingerprint do login **não muda**. Fiquei 20min achando que "não subiu" quando na verdade tinha subido (a régua da agenda era CSS + route da agenda). Também tentei: grep de classe no CSS do login (inválido, CSS é route-split, a agenda carrega no route dela), e buildId do Next (não exposto no HTML do admin aqui).

**Regra:** pra mudança de CSS-only ou de rota não-compartilhada, o fingerprint do login NÃO confirma nem nega o deploy. Nesses casos: (a) confiar em `git ls-remote` (commit é o tip do main) + `tsc` limpo + site 200 estável = deploy deve ter ido; (b) confirmação de verdade = o Renan dá refresh forte na tela que mudou, ou eu abro o **painel do Render no Chrome logado dele** (claude-in-chrome). NUNCA declarar "no ar" baseado só no fingerprint quando ele é cego. Relacionado: [[feedback_acompanhar_deploy_ate_verde]], [[reference_agenda_time_grid]].

**FINGERPRINT POSITIVO quando o deploy ADICIONA uma ROTA nova (14/ago, funcionou lindo):** quando a mudança inclui um endpoint/rota que ANTES não existia, dá pra confirmar o deploy sem depender do md5 do login — a rota nova flipa de **404 (não existe) → status real** quando o código novo assume. Ex.: no link direto de briefing, `POST /api/briefing/<token-fake-24-chars>` dava **404 antes** do deploy e **401 depois** (rota existe, token inválido). Monitor em background: loop `curl` na rota nova + `/api/health`, espera 2 leituras seguidas do status novo = LIVE. Isso é POSITIVO (prova que o código novo roda), diferente do fingerprint do login que é cego. Sempre que o deploy criar rota nova, use ela como sonda. Página nova (GET) é fingerprint pior: `/rota-nova/<invalido>` pode dar 404 (notFound) IGUAL a "rota não existe" — nesse caso o smoke test serve só pra garantir que NÃO é 500 (page compila).

**ARMADILHA do fingerprint em endpoint custom de COLEÇÃO Payload (14/ago, quase declarei live falso):** um custom endpoint novo tipo `/api/clientes/crm-uso` NÃO serve como sonda por **GET** — qualquer `GET /api/clientes/<qualquer-coisa>` (inclusive path inexistente) cai no REST da coleção como `findByID('<segmento>')` e devolve **403** ("sem permissão") SEMPRE, mesmo com a rota custom ausente. Confirmei calibrando: `GET /api/clientes/zzznaoexiste` = 403 IGUAL ao meu endpoint. O 403 apareceu 20s após o push (build nem tinha acabado) = falso positivo. **Sonda confiável = POST no path custom:** `POST /api/clientes/crm-limite` devolve **`404 {"message":"Route not found"}`** enquanto a rota não está registrada e vira **400/403** (a validação/gate do meu handler) quando o deploy assume. Regra: pra endpoint custom de coleção, fingerprinte por **POST** (404 Route not found → status do handler), nunca por GET; e sempre calibre com um path vizinho inexistente antes de confiar no código.
