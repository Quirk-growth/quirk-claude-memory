---
name: project_tarefas
description: Módulo de Tarefas do time interno (Portal da Equipe) na área de membros; gerenciador estilo ClickUp; Parte 1 + v2 (lista/filtros/etiquetas/editor de status) no ar ago/2026; Parte 2 (menções+inbox) pendente
metadata: 
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-06T20:37:26.273Z
---

Gerenciador de tarefas do **time interno da Quirk** dentro da [[project_area_membros_quirk]] (cockpit/Portal da Equipe — nunca aparece pro cliente `member`). Inspirado na lista "Tarefas Tráfego" do ClickUp deles (list 901702083148, space Gestão de Projetos), mas com **cliente estruturado** em vez de `[CLIENTE]` no título. Spec/plano `docs/superpowers/specs|plans/2026-08-06-tarefas-parte1-*`.

**Parte 1 (NO AR, 06/ago, branch feat/tarefas → main edea30b):** feito via subagent-driven (8 tasks TDD + review por task + review final MERGE).
- **Collections:** `listas-tarefas` (status configuráveis POR LISTA, estilo ClickUp: `{slug,nome,cor,ordem,tipo:aberto|andamento|concluido}`, `minRows:1`) + `tarefas` (titulo/descricao/lista/status/responsaveis[users hasMany]/autor/prazo/prioridade[maxima|media|normal]/cliente[opcional]/concluidaEm). Status é validado contra a lista no hook `beforeChange` (`normalizarStatusTarefa`); `concluidaEm` derivado do `tipo=concluido`; `autor` sempre o criador. Acesso = `ehEquipe` em tudo (helper novo `isEquipe`); delete tbm = equipe.
- **Endpoint** `POST /api/tarefas/acao` (criar/editar/mover/excluir), gate `ehEquipe` 403.
- **Camada** `src/lib/tarefas/{tipos,listas,status,consulta}.ts`: `LISTAS_SEED`, helpers puros, `montarWhereTarefas`+`listarTarefas` (busca por título em memória; TarefaCard).
- **Telas:** view admin `TarefasView` no menu Tráfego+Comercial (`/admin/tarefas`, lista agrupada por status + modal criar/editar, componente client `TarefasLista` reutilizável) + **aba "Tarefas" no Hub** (`ABAS_HUB` += 'tarefas', `HubAbaTarefas`, escopo por cliente via `listarTarefas({clienteId})`, modo `clienteFixo`).
- **Seeds:** listas **Tráfego** (A fazer·Demanda recebida·Em andamento·Concluído) e **Criativos** (Demanda recebida·Em produção·Revisão de copy·Copy revisada·Criativo finalizado·Criativo revisado·Criativo aprovado·Concluído). Script `src/scripts/seedListasTarefas.ts` (idempotente por nome, exige `CONFIRMAR_SEED=1`, rodar `NODE_ENV=production`).
- **DDL prod aplicado à mão** (extraído do banco de TESTE, endpoint direto sem -pooler): tabelas `listas_tarefas`, `listas_tarefas_statuses` (array), `tarefas`, `tarefas_rels` (responsaveis hasMany) + enums `enum_listas_tarefas_statuses_tipo`/`enum_tarefas_prioridade` + 6 FKs. Seed rodado (Tráfego id 1 / Criativos id 2).

**Decisões:** status POR LISTA (não enum global); sem recorrência no v1; view "lista" (não kanban por arrasto); cliente opcional; time inteiro vê todas as tarefas (sem escopo de carteira no v1). Minors deferidos: 500-vs-400 em enum inválido no endpoint, sem confirmação de exclusão na UI, teste HTTP 403 do endpoint.

**Reforma de UI (06/ago, branch feat/tarefas-ui → main aa79998, NO AR):** trocou o `TarefasLista` cru por **`TarefasBoard`** (client) — card-padrão branco/azul com **toggle Kanban↔Lista** (padrão de todas as tarefas), avatares dos responsáveis (`AvatarPessoa` + `fotosDosUsuarios`), chip do cliente no card, drag→mover status. **`TarefaModal`** (detalhe editável + **comentários** estilo ClickUp + **cliente digitável** `CampoClienteBusca`). **Comentários:** campo `tarefas.anotacoes` (jsonb, DDL manual em prod) + ação `anotar` no `/acao` (autor = req.user carimbado no servidor, append cap ~50); `TarefaCard.anotacoes`. `OpcoesTarefas` mora em `TarefaModal.tsx`. Feito via subagent-driven (5 tasks + review final MERGE). Fixes: cliente stale no autocomplete, drag cross-lista bloqueado no Hub (valida slug contra a lista da tarefa). Página CLARA (≠ comercial escuro).

**v2 (06/ago, branch feat/tarefas-v2 → main 9821de3, NO AR):** subagent-driven, 10 tasks + review final opus (READY TO MERGE). (A) **Lista em linhas** compactas (grid 6 colunas: Tarefa·Empresa·Responsáveis·Criado por·Criada·Vencimento; atrasada em vermelho + ícone; divisórias 0.5px); (B) card Kanban com contorno azul claro `#9dc3f7`; (C) aba Tarefas removida do Comercial (só Tráfego); (D) **barra de filtros** `TarefasFiltros` (URL params `de/ate/cliente/resp/autor/etq/prio/eu`; Modo EU = `meuId` do `req.user` server-side; etiqueta filtrada em memória, dropdown calculado ANTES do filtro); (E) campo **`etiquetas`** jsonb + ação `etiquetas` no `/acao` (trim/dedupe/cap 15) + chips; (F) `CampoResponsavelBusca` no modal (autocomplete hasMany); (G) **EditorStatus** self-service (endpoint `/api/listas-tarefas/gerir`, supervisor+, slug estável no rename, delete bloqueado se houver tarefa no status, ≥1 status) — botão "Gerenciar status" só no menu (nunca no Hub/união). Datas `de/ate` validadas por regex antes do where (URL adulterada dava 500). DDL prod: só `ALTER TABLE tarefas ADD COLUMN etiquetas jsonb` (não mexe em tabelas compartilhadas).

**Minors deferidos (v2):** `concluidaEm` não re-deriva quando supervisor muda o `tipo` de um status em uso; teste int de etiquetas não exercita o dedupe via endpoint; input color exige hex 6 dígitos.

**PENDENTE:** verificação user-facing (login de equipe do Renan em /admin/tarefas + aba do Hub). **Parte 2 = MENÇÕES (@) nos comentários (Comercial/Anotações/Rotina) + CAIXA DE ENTRADA por pessoa + sino/contador** — próximo módulo, pluga em cima desta base (spec/plano próprios).
