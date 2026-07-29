---
name: reference_agendamento_reuniao_make
description: "Bug do cenário [AHL] Agendamento de reunião no Make — conexão Google na conta errada causa 404 e reuniões não entram na agenda"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 15ff4359-90d3-4459-acda-749f36f31504
  modified: 2026-07-29T15:20:46.480Z
---

Cenário Make **3704159 "[AHL] Agendamento de reunião"** (team 385200): webhook ClickUp → getATask → Router por responsável (Renan `55187723`, Yuri `89283218`, Bruno `290621161`; 4ª rota é placeholder "COLOCAR O ID"). Cria evento no Google Calendar e grava o link do Zoom no custom field "Meeting" (`534a9d61-...`) do card em prospectos (lista 900902474139).

**Bug (jul/2026, investigado a fundo):** reuniões da rota do Renan pararam de entrar na agenda `contato@quirkgrowth.com.br` desde ~15/mai/2026. Causa REAL provada com teste ao vivo + RPC `listCalendars`: a conexão Make **"Quirk Growth" (id 3645550)** está autenticada na **conta ERRADA = renan.reeal@gmail.com** (a pessoal/dona do Make), NÃO na contato@. A RPC listCalendars da conexão retorna só: Kommo-renanreeal, FESF, Feriados, e `renan.reeal@gmail.com (Primary)` — a agenda contato@ nem aparece. A metadata da conexão MOSTRA "contato@quirkgrowth.com.br" mas está mentindo/desatualizada. Por isso `calendar: contato@quirkgrowth.com.br` dá **[404] Not Found**, mas `calendar: primary` cria (na agenda pessoal do Renan). Outros fluxos (Drive/Sheets) "funcionam" porque a conta pessoal tem acesso compartilhado aos arquivos — mas Calendar é por conta. Reautorizar cai de novo em renan.reeal@gmail pq o navegador está logado nela (reconnect silencioso).

Descartado como causa: conteúdo do card (Edgard funcionou e Thiago falhou com cards idênticos), roteamento, datas, e-mail markdown, edição do cenário (intocado desde 10/abr). O tratador `builtin:Ignore` no módulo de calendar ESCONDE o 404 (execução marca SUCCESS) — por isso passou 2,5 meses sem ninguém notar. Renan optou por manter o Ignore.

**RESOLVIDO (29/jul/2026):** Renan criou conexão nova **"Calls Contato@quirk" (id 4832988)** logada de verdade na contato@ (listCalendars confirma que ela enxerga `contato@quirkgrowth.com.br`), e apontou o módulo 3 (rota EMAIL RENAN) pra ela. Testado ponta a ponta: evento criado na agenda contato@ sem 404. Módulo 3 agora = conn 4832988, calendar contato@quirkgrowth.com.br (Yuri=4000944, Bruno=4735479 intactos).

**Pendências/riscos:** (1) a conexão antiga **"Quirk Growth" (3645550)** continua LOGADA na renan.reeal@gmail apesar do label dizer contato@ — ela ainda é usada por outros cenários ([AHL] Contrato Parte 1 = 3328391, LP Zanardi Rancho = 4822743); esses rodam como conta pessoal do Renan. (2) Backlog de reuniões perdidas 15/mai→29/jul (cards em prospectos com data marcada e campo "Meeting" vazio) não entrou na agenda — recriar manualmente se necessário. Método de teste da conexão: RPC `rpc_execute` google-calendar@5 `listCalendars` com `data:{__IMTCONN__: <id>}` retorna as agendas inline (revela a conta real).

Ver [[reference_quirk_docs]]. Evento perdido do Matheus (31/jul 11:30) foi recriado manualmente via MCP em 29/jul.
