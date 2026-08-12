---
name: reference_deploy_fingerprint_cego
description: "GOTCHA: o fingerprint do JS de /admin/login pra confirmar deploy é CEGO a mudanças de CSS ou de rotas não-compartilhadas — não confirmar/negar deploy por ele nesses casos"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-12T21:02:01.892Z
---

Truque que uso pra saber quando um deploy do Render assumiu em prod (área de membros): capturo o md5 dos `static/chunks/*.js` da tela `/admin/login` antes do push e faço polling até mudar (`fp_pre.txt` + loop no scratchpad). Funciona bem pra mudanças em componentes COMPARTILHADOS (lista, chips, hub) que entram nos chunks do login.

**GOTCHA (13/ago):** o método é **cego** quando a mudança NÃO toca os assets da tela de login — ex.: só CSS (`custom.scss`), ou só uma rota específica não carregada no login (ex.: `AgendaTimeView`). O Next faz **code-split de CSS e JS por rota**, então o deploy pode assumir 100% ok e o fingerprint do login **não muda**. Fiquei 20min achando que "não subiu" quando na verdade tinha subido (a régua da agenda era CSS + route da agenda). Também tentei: grep de classe no CSS do login (inválido, CSS é route-split, a agenda carrega no route dela), e buildId do Next (não exposto no HTML do admin aqui).

**Regra:** pra mudança de CSS-only ou de rota não-compartilhada, o fingerprint do login NÃO confirma nem nega o deploy. Nesses casos: (a) confiar em `git ls-remote` (commit é o tip do main) + `tsc` limpo + site 200 estável = deploy deve ter ido; (b) confirmação de verdade = o Renan dá refresh forte na tela que mudou, ou eu abro o **painel do Render no Chrome logado dele** (claude-in-chrome). NUNCA declarar "no ar" baseado só no fingerprint quando ele é cego. Relacionado: [[feedback_acompanhar_deploy_ate_verde]], [[reference_agenda_time_grid]].
