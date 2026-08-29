---
name: reference_make_ignore_engole_falha
description: Make — tratador de erro "Ignore" mata o resto da rota e marca SUCESSO; foi o que sumiu com e-mail/WhatsApp de leads quando o ActiveCampaign caiu por falta de pagamento (402)
metadata:
  type: reference
---

⚠️ **No Make, o tratador de erro `Ignore` NÃO é "ignora e continua" — ele DESCARTA o pacote e ABANDONA o resto da rota**, e ainda marca a execução como **SUCESSO** (bolinha verde). Um módulo intermediário quebrado derruba silenciosamente tudo que vem depois dele.

**Caso real (29/ago/2026, cenário 4080950 "Leads Quirk [Comercial Time]"):** o ActiveCampaign começou a responder `[402] The request could not be processed due to account payment issues` (conta bloqueada por pagamento). O módulo tinha `onerror: Ignore` → o fluxo morria no 3º de 9 módulos. Resultado: o lead entrava no CRM (módulo 2) mas **NÃO** recebia e-mail interno, e-mail pro lead, linha na planilha, tarefa no ClickUp **nem a mensagem automática de WhatsApp** — e nada aparecia como erro. O Renan só percebeu porque não chegou e-mail.

**COMO DIAGNOSTICAR ISSO RÁPIDO — a assinatura é a CONTAGEM DE OPERAÇÕES.** `executions_list` mostra `operations` por execução: o fluxo saudável fazia 9, o quebrado fazia 3, todos com `status: 1` (sucesso). Queda súbita e estável no número de operações = rota abandonada no meio. Cruzar o horário das execuções truncadas com `created_at` dos leads em `crm_leads` identifica exatamente quais leads foram afetados.
- ⚠️ O MCP do Make (`executions_get-detail`) devolve **só `{"status":"SUCCESS"}`**, inútil pra achar o módulo. A mensagem de erro real só sai no **painel web**: `us1.make.com/385200/scenarios/<id>/logs/<execId>` → clicar no módulo com ⚠️ → bloco "Handled error".
- ⚠️ O canvas do editor do Make **não responde a automação de browser** (right-click, tecla Delete, hover não abrem menu). Editar blueprint = `scenarios_update` pela API (substitui o blueprint INTEIRO — buscar com `scenarios_get`, editar o JSON, mandar completo). O upload de arquivo pro "Import blueprint" foi barrado pelo classificador de permissão.
- ⚠️ Se sobrar aba do editor aberta com estado pendente, ela pede "Leave site?" e **não dá pra fechar por automação** — e se alguém clicar Save nela, DESFAZ a alteração feita pela API.

**Antes de remover um módulo, procurar referências à saída dele** (`{{20.` etc.) no blueprint — no caso, só o próprio par do ActiveCampaign usava `{{20.id}}`, então dava pra remover os dois juntos.

**Estado em 29/ago/2026:** ActiveCampaign removido do 4080950 (fluxo agora: webhook → CRM → e-mail interno → e-mail lead → planilha → ClickUp → WhatsApp). **Outros 12 cenários ativos ainda tinham módulo do ActiveCampaign** (Calculadora VGV 4779375, Isca Digital 4755230, Kit Pago 4802778, Clinics 4727812, Google Ads 4493717, NOVA LP 4805666, Form Nativo 4497128, HT insta 4493740, RMKT Form 4497144, VSL RMKT 4536766, TikTok 4597833, Leads Meta 3376524) — todos com a mesma bomba armada. Renan decidiu não usar mais ActiveCampaign. Relacionado: [[reference_agendamento_reuniao_make.md]] (mesmo padrão de "Ignore esconde"), [[project_crm_quirk]].
