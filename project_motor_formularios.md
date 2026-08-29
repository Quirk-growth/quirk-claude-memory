---
name: project-motor-formularios
description: "Motor genérico de formulários (sub-projeto 1 do pedido de NPS) — NO AR 29/08, 12 tasks, coleções Formularios/Disparos/Respostas"
metadata: 
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-29T17:15:20.653Z
---

Motor genérico de criação/envio/preenchimento de formulários — sub-projeto 1
de um pedido maior do Renan sobre NPS pros clientes (o NPS em si, com regras
de classificação/tarefa automática/indicação, é sub-projeto 2, ainda não
construído; dashboard de NPS é sub-projeto 3, também pendente).

**No ar em produção desde 29/08/2026** (commit `c52cdc5`, deploy confirmado
por smoke test: `GET /api/formularios` → 403 sem auth, `/formularios/<token
inválido>` → 404, `/admin/formularios` → 200).

## Arquitetura
- 3 coleções Payload: `Formularios` (definição + perguntas — 5 tipos:
  texto_curto/texto_longo/escala/multipla_escolha/bloco_contatos, com
  condicional "só mostra se..."), `Disparos` (rodada de envio: alvo =
  todos_clientes_ativos/clientes_especificos/sem_cliente, prazo opcional),
  `Respostas` (uma por destinatário, token público, `cliente` opcional pra
  suportar prospect/time interno).
- Preenchimento: link público sem login (`/formularios/[token]`) OU área de
  membros logada (`/formularios/responder/[respostaId]` — segmento `responder/`
  é OBRIGATÓRIO, sem ele colide com a rota pública e o Next se recusa a
  buildar "Ambiguous app routes").
- `estadoParaWizard`/`salvarEtapa` (`src/lib/formularios/estadoResposta.ts`)
  são o único motor de submissão, reaproveitado literalmente pelas 2 rotas.
- Editor admin em `/admin/formularios` (gate fixo por `ehSupervisorOuAcima`,
  fora da matriz de Permissões Configuráveis — decisão do Renan, v1).
- Painel de disparos: criar/encerrar, prazo, seletor de clientes específicos,
  "Ver respostas" com link público copiável por resposta.

Ver `docs/superpowers/specs/2026-08-28-motor-formularios-design.md` e
`docs/superpowers/plans/2026-08-28-motor-formularios.md` (12 tasks — as
últimas 4 foram adicionadas ao vivo depois da revisão final de branch achar
gaps reais entre a spec aprovada e o plano original).

## Achados sérios ao longo da implementação (SDD, 12 tasks + 2 revisões de branch inteira)
- Task 11 (`perguntaId` editável) foi um caso real de whack-a-mole: 4
  rodadas de fix no MESMO arquivo (`EditorFormulario.tsx`) até achar a
  causa raiz — rastrear o painel "Detalhes" expandido por VALOR editável ou
  por ÍNDICE são as duas formas erradas óbvias (cada uma resolve um bug e
  introduz outro equivalente). A solução certa foi uma chave interna
  estável (`_chave`, gerada uma vez, nunca enviada ao servidor, sobrevive a
  edição de campo E a reordenação porque o padrão de spread já usado no
  arquivo preserva objetos não tocados). O bug mais sério dessa task:
  `perguntaId` duplicado rejeitado pela UI ainda vazava pro banco assim que
  o operador tocasse em QUALQUER outro campo (validação só rodava no
  handler específico do campo de ID, não em `salvar()`) — fechado com
  validação centralizada em `salvar()` (bloqueia até TUDO ficar válido) +
  `validate` server-side na própria collection (defesa em profundidade).
- Revisão final de branch (opus) achou 1 Critical de build real: rota
  pública e rota logada colidiam na forma de URL — reproduzido com build
  isolado, não só teoria.
- `npm run typecheck:tests` (script separado de `tsc --noEmit`, que exclui
  `tests/`) nunca rodou em nenhuma das 12 tasks nem nas 2 revisões de
  branch — só descoberto no checklist de deploy. Achou ~36 erros reais
  (fetchMock sem params tipados quebrando `.mock.calls[n][1]`, `clientes`
  criado sem `status` required, `opcoes` com campo `rotulo` que não existe
  no schema). Lição: **sempre rodar os dois scripts de typecheck**, não só
  `tsc --noEmit` — ver skill `deploy-quirk`.

## Deploy
DDL manual aplicada em prod ANTES do push (skill `payload-migracao-prod`) —
10 tabelas + coluna de lock em `payload_locked_documents_rels` pras 3
coleções, script em `scripts/ddl/2026-08-29-motor-formularios.sql`,
extraída por introspecção do banco de teste (não escrita de cabeça),
confirmada por introspecção em produção depois de aplicar. Merge real com
`origin/main` antes do push (56 commits de outras sessões desde que a
feature começou) — 5 conflitos, incluindo aposentar meu `CampoClienteFiltro.tsx`
porque outra sessão paralela generalizou o mesmo padrão num `SeletorBusca`
reutilizável (com um fix de race condition de staleness que a minha versão
nem sabia que tinha) — ver [[reference_git_repo_compartilhado_sessoes]].

## Pendente (fora do escopo deste sub-projeto)
- Sub-projeto 2: regras de negócio do NPS (classificação por nota,
  criação automática de tarefa detrator/neutro/promotor, fluxo de
  indicação → lead comercial, ocultar pergunta de serviço não contratado).
- Sub-projeto 3: dashboard de NPS.
- Testes manuais em browser (wizard público + fluxo logado) — não
  executados nesta sessão (local dev = produção, ver
  [[reference_modeloscontrato_corpo_orfa]] — verificação real só via HTTPS
  de produção).
