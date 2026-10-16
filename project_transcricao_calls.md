---
name: project-transcricao-calls
description: "Transcrição de calls no Hub (estilo Tactiq) — Lotes 1-3 no ar em 16/10/2026, falta Lote 4 (tela de detalhe) e Fase 2 (item de ação vira tarefa)"
metadata:
  node_type: memory
  type: project
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-10-16T16:52:15.488Z
---

Recurso de transcrição de calls dentro da área de membros, inspirado no Tactiq.
Worktree `~/area-membros-quirk-transcricoes`, branch `feat/transcricao-calls`
(sobe direto pra `main`). Spec e plano em `docs/superpowers/`, ledger de decisões
em `.superpowers/sdd/2026-10-15-transcricao-de-calls/progress.md` — **ler o
ledger antes de mexer**, ele tem o porquê de cada escolha.

**Como funciona:** popup `/admin/gravar-call` grava DOIS canais (microfone e aba
do navegador) em pedaços no R2. O cron `transcricoes-processar` (5 em 5 min)
junta, manda pra AssemblyAI, intercala os canais e pede o resumo estruturado à
Anthropic. Resultado em `/admin/transcricoes` e no Hub do cliente.

**No ar em 16/10/2026:** Lotes 1-3, tela de detalhe (conversa recolhida com
busca e destaque), filtros (cliente/autor/estado/período/busca), balão
flutuante de gravação (Document PiP) e parada automática por silêncio (avisa
aos 5 min, encerra aos 12, motivo escrito no campo `erro`).
Falta: **renomear locutor**, **descarte do áudio aos 30 dias** (Tarefa 4.4) e a
**Fase 2** (item de ação do resumo vira tarefa com um clique).

**Criar tarefa da seleção de texto** (pedido em 16/10) já está NO AR e vale no
sistema inteiro: mora no `NavQuirk`, abre painel rápido, reusa `acao: 'criar'`
de `Tarefas.ts`. Tela que quiser vincular cliente declara `data-cliente-id`.

**Decisões que não são óbvias no código:**
- O resumo sai ESTRUTURADO (pontos/decisões/itensDeAcao/riscos), não em prosa —
  é o que a Fase 2 consome.
- `speaker_labels` nos DOIS canais. Tentei economizar no microfone e a
  transcrição voltou como UM bloco sem horário — na call real isso ordenaria
  tudo de quem gravou antes de tudo do outro lado.
- Basta UM canal pronto pra call ficar `pronta`. Numa call pelo app do Zoom o
  canal da aba sai mudo; falhar tudo ali apagaria a única voz que existe.
- Numa call `pronta`, o campo `erro` carrega um AVISO, não uma falha.
- GOTCHA que derrubou a tela: operador de data no Payload é
  `greater_than_equal`, SEM "or". Inventei `greater_than_or_equal` e a página
  morreu com "This page couldn't load". Teste unitário não pegou porque
  afirmava a minha invenção — só `payload.find` de verdade pega.
- Corte de silêncio ficou de fora (exigiria ffmpeg, que não tem no Render).
  Custo medido ~R$420/mês no volume previsto. Medir o real antes de otimizar.

Ver [[reference_render_criar_cron_job]], [[reference_payload_select_enum_ddl]],
[[feedback_medir_antes_de_corrigir]].
