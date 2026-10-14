---
name: project_quirk_intelligence_tarefas_escopo
description: "Quirk Intelligence (bot Telegram) — tarefas só do colaborador (responsável OU carteira), minhas/prazo resolvidos no servidor, resposta vazia explicada; incidente do Gerson 13/10, NO AR 14/10"
metadata:
  node_type: memory
  type: project
---

**Incidente (13/10, gestor Gerson id 526):** "quais tarefas estão em atraso" e "avalie apenas as tarefas em que estou atribuído" → `Não consegui gerar uma resposta.`; "quais tarefas tenho pra fazer hoje" → 25 tarefas de Zanardi/Clarivie/Gustavo Matiro (clientes que NÃO eram dele). Achado lendo a memória do bot (`quirk_intelligence_memorias` + `_mensagens`, por usuário) — é o jeito de ver o que o bot respondeu a alguém.

**Causas medidas em produção (leitura):** `ferramentaListarTarefas` só validava carteira quando vinha `clienteId`; sem ele `listarTarefas` filtrava só pela visibilidade de LISTA por papel → pro Gerson 206 tarefas abertas do Hub, 182 que não eram dele, ~237 mil caracteres de JSON por pergunta. O modelo não conhece o próprio id e não tinha filtro de data. O "sem resposta" era `finalMessage` sem bloco de texto (fallback silencioso, sem log, gravado na memória); causa exata no passado NÃO provável (nada era logado) — candidatos: `max_tokens` 1024 (o `thinking` adaptativo conta dentro dele) ou `max_iterations` 8.

**NO AR (commit 3056b65e, deploy live 14/10 11:20):**
- `FiltrosTarefas.escopo` {usuarioId, clienteIds} em `lib/tarefas/consulta.ts` → where `(responsaveis = ele) OU (cliente in carteira)`; carteira vazia vira só o ramo do responsável (nunca `in []`). Só a IA preenche; telas humanas inalteradas. `role`/`escopo` vêm SEMPRE do colaborador (zod também descarta campo forjado — medido). admin/supervisor/administrativo = `todos` (sem recorte); gestor/comercial/social = carteira própria.
- `listar_tarefas` ganhou `minhas:true` (servidor resolve o id) e `prazo` (atrasadas | hoje | ate_hoje | amanha | proximos_7_dias, calendário de Brasília; `prazoTarefas.ts`). `ate_hoje` = atrasadas + hoje ("o que tenho pra fazer hoje"). Resultado compacto `{total, mostrando, tarefas[≤50]}` (`compactarTarefas.ts`): Gerson 236 mil → ~4,5 mil caracteres.
- `buscar_tarefa` também abre tarefa em que ele é responsável. O inverso não vale: tarefa sem cliente e de outra pessoa não aparece na lista mas abre por id (decisão antiga; ids só saem da lista).
- Resposta vazia (`respostaFinal.ts`): motivo identificado, `payload.logger.warn` com stop_reason/blocos/usuário (sem texto de conversa), mensagem específica, e NÃO grava na memória. `tool_use` pendente vence um "Vou verificar…" escrito antes (reproduzido com o SDK real); `max_tokens` com texto parcial vale como resposta mas loga.
- `MAX_TOKENS_SAIDA` 1024 → **4096** (contraria a decisão de 12/10 do Renan de 1024; justificativa: o raciocínio come o teto; brevidade segue no prompt "Seja BREVE"; teto por rodada US$0,010→0,041 só se usado). Se ele quiser voltar, o risco é o "sem resposta" voltar.
- `enviarMensagemTelegram` divide em partes ≤4096 caracteres (`dividirMensagem.ts`) — o Telegram recusa mais que isso (HTTP 400) e o bot não entregava nada; se uma parte falha não manda as seguintes.

**Memória do Gerson LIMPA (14/10 12:13, com OK do Renan):** só ele tinha pergunta de tarefa/fallback (Renan, Anna, Phillipe estavam limpos). Zerei as 14 `mensagens` brutas da memória id 3 (usuário 526), mantendo `resumo` (310 chars, sobre Primare) e `atualizadoEm`; script com guarda (só rodava se `atualizadoEm` ainda fosse o inspecionado) e backup local no scratchpad.

**Pendente / não verificado:** (1) [resolvido acima] (2) Nunca vi o Sonnet de fato escolher `minhas=true`/`prazo` com o prompt novo — só com mensagem real no Telegram (chave Anthropic só no Render). (3) `totalPodeSerMaior` é calculado antes do filtro `busca` em memória.

Ver [[project_quirk_intelligence_custos]], [[feedback_isolamento_dados_clientes]], [[feedback_teste_decorativo_mutacao]].
