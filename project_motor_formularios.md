---
name: project-motor-formularios
description: "Motor genérico de formulários (sub-projeto 1 do pedido de NPS) — NO AR, engine (29/08) + redesign Frente A (30/08)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-30T14:04:35.028Z
---

Motor genérico de criação/envio/preenchimento de formulários — sub-projeto 1
de um pedido maior do Renan sobre NPS pros clientes (o NPS em si, com regras
de classificação/tarefa automática/indicação, é sub-projeto 2, ainda não
construído; dashboard de NPS é sub-projeto 3, também pendente).

**Engine no ar desde 29/08/2026** (commit `c52cdc5`, smoke test original).
**Redesign visual + link avulso (Frente A) no ar desde 30/08/2026** (commit
`d5f8e16`, smoke test: `POST /api/disparos/avulso` sem auth → 403).
**Fix do editor + apagar formulário no ar desde 30/08/2026** (commit
`1c07a5d` — ver seção própria abaixo).

## Bug real em produção + apagar formulário (30/08/2026, commit `1c07a5d`)
Renan reportou (com screenshot) que digitar o texto de uma pergunta no
editor apagava o que tinha sido escrito e mostrava "Não foi possível
salvar. Tente de novo." — regressão do ENGINE original (29/08), não do
redesign visual do dia anterior, só ficou visível quando ele usou de
verdade a tela nova.

**Causa raiz**: `EditorFormulario.tsx` disparava um PATCH a CADA TECLA
digitada (sem debounce), reaproveitando o mesmo `salvar()` usado pelas
ações estruturais (mover/remover/adicionar). O campo `texto` da pergunta é
`required: true` no schema (`Formularios.ts`) — selecionar o texto padrão
"Nova pergunta" pra digitar por cima passa por um instante com o campo
vazio, que o servidor rejeita; o handler `!r.ok` reverte a lista inteira
com um erro no meio da digitação. A suíte de testes nunca pegou isso
porque sempre mockava o fetch como sucesso, nunca simulou o campo ficando
transitoriamente vazio contra a validação real do servidor.

**Fix**: campos de texto livre (texto da pergunta, opções de múltipla
escolha, condicional, min/máx de escala, máx de contatos) agora salvam com
debounce de 600ms — a UI atualiza na hora, o PATCH só sai depois de uma
pausa na digitação. Ações estruturais continuam salvando na hora. Um
`salvar()` imediato (mover/remover/adicionar) sempre cancela um debounce
pendente antes de rodar — evita que um timer antigo "ressuscite" dados já
superados (mesmo padrão de race já resolvido antes em `TarefasFiltros`,
achado 3 da revisão de reordenar-mover-subtarefas).

**Apagar formulário**: pedido junto pelo Renan na mesma mensagem. Botão
"Apagar" na lista e no cabeçalho do detalhe (`ExcluirFormularioButton.tsx`,
mesmo padrão de `window.confirm()` + DELETE que `ExcluirClienteButton.tsx`
já usa pra clientes). Decisão de design (perguntada e confirmada pelo
Renan): **bloquear** a exclusão se o formulário já tiver algum Disparo
vinculado (não cascatear) — `protegerExclusaoFormulario.ts`, um
`beforeDelete` na coleção, mesmo padrão de `protegerExclusaoCliente.ts`.
Formulário nunca disparado apaga normal.

**Gotcha novo**: `FormulariosView.tsx` é Server Component — passar uma
função como prop (`aoExcluir={() => ...}`) pro `ExcluirFormularioButton`
(Client Component) não é serializável através da fronteira RSC. Corrigido
trocando por uma prop booleana (`redirecionarParaListaAoExcluir`) que o
botão interpreta internamente, em vez de receber um callback.

Smoke test deste deploy foi só de liveness geral (`/admin/login` → 200,
`/api/notificacoes/painel` → 403) — sem schema novo e sem endpoint
genuinamente novo nesta mudança pra fingerprint específico (o `beforeDelete`
só é observável de fato tentando apagar um formulário de verdade).

## Redesign Frente A (30/08/2026)
Pedido do Renan: "está feio, difícil de compreender" + precisa de link pra
quem não tem área de membros + queria visão de dashboard de respostas (essa
parte virou **Frente B**, explicitamente adiada — dashboard de conteúdo de
resposta ainda não existe, só a coleção nativa `Respostas` no admin cru).

Spec: `docs/superpowers/specs/2026-08-29-motor-formularios-redesign.md`.
Plano: `docs/superpowers/plans/2026-08-29-motor-formularios-redesign.md` (9
tasks, executadas via subagent-driven-development em branch própria
`feat/motor-formularios-redesign`, 3 rodadas de revisão de branch inteira).

O que mudou:
- Visual: `EditorFormulario`/`PainelDisparos` trocaram `style={{}}` solto
  (com hex hardcoded que não batia com nenhum tema) pelas classes `hub-*`/
  `var(--pt-*)` já usadas no resto do painel — zero mudança de lógica,
  todos os `data-*` de teste preservados byte-a-byte.
- Reorganização: tela de detalhe virou 4 abas por query param (`?id=X&aba=`)
  — Perguntas (editor + mensagem de introdução), Configurações (só o toggle
  Ativo), Pré-visualizar (nova — wizard local sem rede, ver abaixo), Disparos
  (bloco "Links avulsos" + o fluxo de disparo que já existia). O antigo
  `MetadadosFormulario.tsx` (título+mensagemIntro+ativo num bloco só) foi
  aposentado — os 3 campos viraram 3 componentes pequenos em 3 lugares.
- **Pré-visualizar**: desviei da spec original (que pedia um modo dual
  dentro do próprio `FormWizard`) — extraí o renderizador de pergunta-por-
  tipo pra um componente `PerguntaCampo` reaproveitado por um componente
  novo e separado (`FormWizardPreview`, sem rede, condicional reagindo ao
  vivo). Decisão de arquitetura consciente (evitar bifurcar a máquina de
  estado do wizard real), registrada e aceita nas revisões.
- **Link avulso**: mecanismo "balde" — um `Disparo` especial por formulário
  (`rotulo` reservado `__links_avulsos__`, `alvo: sem_cliente`), criado sob
  demanda, que acumula uma `Resposta` por clique em vez de criar um disparo
  novo a cada vez. Zero mudança de schema. `src/lib/formularios/linkAvulso.ts`.

## Achados sérios da revisão de branch (3 rodadas, todos corrigidos antes do merge)
- **Balde vazando pra lista de Disparos** (rodada 1): a query da aba
  Disparos não excluía o rótulo reservado — o supervisor via
  `__links_avulsos__` cru na lista, com um botão "Encerrar" funcional que
  matava TODOS os links avulsos (passados e futuros) enquanto a UI
  continuava mostrando "Copiado!" pra links mortos. Corrigido filtrando o
  rótulo na query + reabrindo o balde automaticamente se reaproveitado num
  estado não-`aberto`.
- **`LinksAvulsos` escondia links já gerados** (rodada 1): carregava a lista
  só no primeiro "Ver links", e gerar um link novo ANTES de expandir marcava
  "já carreguei" sem nunca ter buscado — links pré-existentes ficavam
  invisíveis até reload de página inteira. Corrigido: carrega no mount.
- **Cores hex hardcoded ilegíveis no tema claro** (rodada 1, plan-mandated —
  contradição do meu próprio plano com sua própria regra de "nunca hex
  hardcoded"): `PerguntaCampo` foi extraído do wizard público (fundo escuro)
  mas passou a rodar também dentro de `.hub-cartao` (fundo branco no tema
  claro) — bordas `rgba(255,255,255,0.15)` sobre branco = invisíveis.
  Trocado por `var(--pt-borda)`/`var(--pt-destaque)`/`var(--pt-destaque-suave)`.
- **`typecheck:tests` regrediu** (rodada 1, plan-mandated): 6 erros novos em
  3 arquivos de teste do próprio plano (`vi.fn()` sem tipar parâmetros
  quebra `.mock.calls[n][1]`) — mesma classe de bug já documentada abaixo
  em "Achados sérios" do engine original. Corrigido tipando os mocks.
- **O próprio fix da rodada 1 quebrou 2 testes existentes** (rodada 2): o
  fetch-no-mount do `LinksAvulsos` deslocou `fetchMock.mock.calls[0]` em 2
  testes de `painelDisparos.spec.tsx` que indexavam por posição — e o
  relatório do implementador declarou "verde" rodando só 6 arquivos
  escolhidos a dedo, sem o único consumidor do componente alterado. Achado
  pelo revisor rodando a suíte de verdade, não confiando no relatório.
  Lição reforçada: **rodar a suíte do componente CONSUMIDOR, não só a do
  componente alterado**, antes de declarar "sem regressão".
- Todos os 3 achados de bug real (balde vazando, links escondidos, hex
  hardcoded) foram reproduzidos por execução direta pelo revisor (não só
  leitura de código) antes de qualquer fix ser aceito.

## Deploy do redesign (30/08/2026)
Sem DDL (zero mudança de schema). Merge local fast-forward (branch criada
de cima do commit do engine original, sem divergência de `origin/main`
durante o desenvolvimento). Push direto — sem conflito, ao contrário do
deploy do engine original.

**Gotcha novo neste deploy**: o worktree "principal" deste repo
(`/Users/renanreal/area-membros-quirk`, dono do `.git` compartilhado) estava
em uso por OUTRA sessão paralela (branch diferente, arquivos não
commitados) no momento do merge — não dava pra fazer `git checkout main`
ali sem atrapalhar. Resolvido atualizando a ref de `main` local direto
(`git branch -f main <tip-da-feature>`, sem checkout em nenhum working
tree) já que era um fast-forward puro. Ver [[reference_git_repo_compartilhado_sessoes]].

Suíte completa rodada momentos antes do push: >1000 arquivos passaram, os
16 arquivos de teste do motor de formulários (int+unit) 100% verdes, só
caiu no fim por queda de conexão do Neon (`57P01`, infra, não código) num
teste não-relacionado — mesmo padrão já documentado.

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

## Segunda rodada de bugs reais + polish (30/08/2026, commit `f4af4cd`)
Renan testou de verdade depois do deploy anterior e achou 2 problemas novos:

- **"+ Adicionar opção" dava o mesmo erro de salvar** — mesma causa raiz do
  bug de digitação (texto vazio bate em `required: true` no servidor), só
  que num call site diferente: `adicionarOpcao()` criava a opção nova como
  `{ valor: '' }` e salvava IMEDIATO (fora do debounce, porque é uma ação
  estrutural tipo mover/remover). Corrigido dando um valor padrão não-vazio
  ("Nova opção"), mesmo padrão que `novaPergunta()` já usava pro texto.
  Teste de integração novo prova que o servidor rejeita `opcoes[].valor`
  vazio de verdade, não só o mock — mesma disciplina do fix anterior.
- **Campo de resposta e botões do wizard "feios"** — `PerguntaCampo.tsx`
  (compartilhado entre o wizard público E a aba Pré-visualizar do admin) e
  os botões Voltar/Próxima/Concluir do `FormWizard.tsx` NUNCA entraram no
  redesign de ontem (que só cobriu telas administrativas) — eram inputs e
  botões completamente crus, sem borda nem preenchimento. Redesenhados com
  os mesmos tokens `--pt-*` já usados no admin — que existem tanto em
  `custom.scss` (admin) quanto em `globals-quirk.css` (formulário público),
  então funciona nos dois lados sem precisar de classes `hub-*` (que só
  existem no CSS do admin).

Deploy confirmado (`/admin/login` 200, `/api/notificacoes/painel` 403) —
smoke test só de liveness geral, sem endpoint novo pra fingerprint
específico neste deploy.

## Frente B (dashboard de respostas) — brainstorming iniciado (30/08/2026)
Renan pediu explicitamente: na lista de formulários, um botão levando pra
(1) lista de respostas, (2) resposta individual, (3) respostas agrupadas
por pergunta pra análise. Decisões já confirmadas com ele: 5ª aba
"Respostas" no formulário (não página separada); lista só mostra respostas
CONCLUÍDAS; escala mostra média + distribuição por nota; toggle "Por
resposta / Por pergunta" dentro da aba (não as duas juntas na mesma tela).
Design completo apresentado, aguardando aprovação do Renan pra escrever a
spec formal — ver [[reference_git_repo_compartilhado_sessoes]] se outra
sessão mexer em `FormulariosView.tsx`/`Respostas.ts` em paralelo.
