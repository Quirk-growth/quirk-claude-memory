---
name: project-quirk-intelligence-auditoria-conversas
description: "Pedido do Renan (14/10/2026) de ver o que cada gestor pergunta/solicita ao bot Quirk Intelligence; hoje NÃO existe log de conversa (só memória rolante escondida e uso numérico); fila: espelho no Telegram (opção 1 escolhida), depois dos dados do Hub"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-14T20:02:40.456Z
---

O Renan (admin) precisa **acesso às conversas de todos os gestores com o bot do Telegram** para entender o que estão perguntando e solicitando (pedido de 14/10/2026, dito depois de aprovar o desenho "dados do Hub no bot").

**Estado hoje (verificado no código em 14/10):**
- `quirk-intelligence-memorias`: 1 linha por colaborador, só as últimas 14 mensagens em texto + resumo automático das antigas; `admin.hidden` e `read: () => false` — nem o admin lê pelo produto; expira em 90 dias sem uso. O que passa de 14 mensagens vira resumo (perde o texto original).
- `quirk-intelligence-usos`: só números (interações, tokens, custo) por pessoa/dia; só admin lê (`podeVerUsoIA`).
- Chamadas de ferramenta (qual, com que argumentos) NÃO são registradas em lugar nenhum. O webhook ignora tudo que não é texto (foto/arquivo), então "subir" = mensagens de texto; com o criar-tarefa, as propostas ficam em `quirk-intelligence-acoes-pendentes` (já desenhado).

**Decisão do Renan (14/10, depois de ver as 3 opções):** ele quer SÓ o espelho no Telegram dele (a cada pergunta de um gestor, o bot manda uma cópia com a resposta ao Telegram do admin, que já está vinculado). Registro de conversas e tela ao vivo no Hub NÃO foram pedidos por ora. Nada implementado ainda: falta apresentar o desenho curto (ponto de entrada em `processarAtualizacaoTelegram`, depois de responder; chave `QI_ESPELHO=off`; destinatários = admins vinculados, exceto quem falou; texto puro cortado por caractere; falha do espelho nunca atrapalha a resposta do gestor; aviso de transparência no `/start`) e ter o OK dele. Só começar depois que o deploy dos dados do Hub estiver no ar. O criar-tarefa (plano em `docs/superpowers/plans/2026-10-14-quirk-intelligence-criar-tarefa.md`, branch `feat/qi-criar-tarefa`, só docs) fica depois.

**Pontos de desenho a decidir quando começar (brainstorming):** coleção nova de log (pergunta, resposta, nomes das ferramentas + argumentos, erro/resposta vazia; NÃO guardar o resultado das ferramentas, que carrega dado de cliente) com DDL em produção antes do push; só admin lê (mesma regra de `podeVerUsoIA`); retenção definida; tela admin por gestor/dia com busca; e avisar os gestores no `/start` que as conversas ficam registradas e visíveis à diretoria (transparência/LGPD).

**Why:** o Renan quer entender a demanda real dos gestores pelo bot (e supervisionar o uso), e hoje isso é impossível.
**How to apply:** ao começar, conferir o código antes (`QuirkIntelligenceMemorias.ts`, `memoria.ts`, `uso.ts`, `webhook.ts`) — o estado acima pode ter mudado. Ver [[project-quirk-intelligence-custos]] e [[project-quirk-intelligence-tarefas-escopo]].
