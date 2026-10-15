---
name: project-quirk-intelligence-auditoria-conversas
description: "Pedido do Renan (14/10/2026) de ver o que cada gestor pergunta/solicita ao bot Quirk Intelligence; hoje NÃO existe log de conversa (só memória rolante escondida e uso numérico); ESPELHO NO TELEGRAM NO AR em 14/10 (93020499); log persistente/tela ao vivo não pedidos"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-15T18:37:17.028Z
---

O Renan (admin) precisa **acesso às conversas de todos os gestores com o bot do Telegram** para entender o que estão perguntando e solicitando (pedido de 14/10/2026, dito depois de aprovar o desenho "dados do Hub no bot").

**Estado hoje (verificado no código em 14/10):**
- `quirk-intelligence-memorias`: 1 linha por colaborador, só as últimas 14 mensagens em texto + resumo automático das antigas; `admin.hidden` e `read: () => false` — nem o admin lê pelo produto; expira em 90 dias sem uso. O que passa de 14 mensagens vira resumo (perde o texto original).
- `quirk-intelligence-usos`: só números (interações, tokens, custo) por pessoa/dia; só admin lê (`podeVerUsoIA`).
- Chamadas de ferramenta (qual, com que argumentos) NÃO são registradas em lugar nenhum. O webhook ignora tudo que não é texto (foto/arquivo), então "subir" = mensagens de texto; com o criar-tarefa, as propostas ficam em `quirk-intelligence-acoes-pendentes` (já desenhado).

**ENTREGUE (14/10/2026, commit `93020499`, NO AR; Render live às ~19:5x BRT):** espelho no Telegram. Cada pergunta de um colaborador ao bot gera uma cópia (pergunta + resposta) pro Telegram dos admins ativos vinculados (hoje só o Renan), exceto quem está falando. Código: `src/lib/telegram/espelho.ts` (formato, chave, limite de 8 s), `webhook.ts` (cópia só DEPOIS da resposta ao colaborador), `adminsVinculados` em `destinatariosAlerta.ts`. Também espelha falha do bot (⚠️) e resposta que o Telegram recusou (⚠️ não chegou). NÃO espelha /start, chat sem vínculo nem a conversa do próprio admin. Desligar sem deploy: `QI_ESPELHO=off` no Render (também false/0/no/desligado/disabled). Doc: `docs/operacao/quirk-intelligence-espelho.md`. O `/start` agora avisa "As conversas com este bot são acompanhadas pela diretoria da Quirk." (só com o espelho ligado).
**Pendências do espelho:** (1) quem JÁ estava vinculado antes do deploy (Gerson, Anna, Phillipe) não vê o aviso do /start — o Renan precisa avisar essas pessoas; (2) teste real: um não-admin (Gerson/Anna/Phillipe) manda mensagem ao bot e o Renan recebe a cópia (a conversa do próprio Renan não é espelhada); (3) se o volume incomodar com os 8 gestores, silenciar por pessoa é a 2ª etapa (não feita).
Registro de conversas persistente e tela ao vivo no Hub: NÃO pedidos (o Renan escolheu só o espelho). **15/10:** ele pediu ver cada gestor em conversa separada e escolheu o grupo do Telegram com um tópico por colaborador — código no ar, inerte até ele definir `QI_ESPELHO_GRUPO_ID`: ver [[project-quirk-intelligence-espelho-grupo]].

**Pontos de desenho a decidir quando começar (brainstorming):** coleção nova de log (pergunta, resposta, nomes das ferramentas + argumentos, erro/resposta vazia; NÃO guardar o resultado das ferramentas, que carrega dado de cliente) com DDL em produção antes do push; só admin lê (mesma regra de `podeVerUsoIA`); retenção definida; tela admin por gestor/dia com busca; e avisar os gestores no `/start` que as conversas ficam registradas e visíveis à diretoria (transparência/LGPD).

**Why:** o Renan quer entender a demanda real dos gestores pelo bot (e supervisionar o uso), e hoje isso é impossível.
**How to apply:** ao começar, conferir o código antes (`QuirkIntelligenceMemorias.ts`, `memoria.ts`, `uso.ts`, `webhook.ts`) — o estado acima pode ter mudado. Ver [[project-quirk-intelligence-custos]] e [[project-quirk-intelligence-tarefas-escopo]].
