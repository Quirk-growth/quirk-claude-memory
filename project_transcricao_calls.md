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

**No ar em 16/10/2026:** Lotes 1, 2 e 3. Falta o **Lote 4** (tela de detalhe,
renomear locutor, busca/filtros, descarte do áudio em 30 dias) e a **Fase 2**
(item de ação do resumo vira tarefa com um clique — por isso o resumo é
estruturado em json, pra não reprocessar nada depois).

**Decisões que não são óbvias no código:**
- O resumo sai ESTRUTURADO (pontos/decisões/itensDeAcao/riscos), não em prosa —
  é o que a Fase 2 consome.
- `speaker_labels` SÓ no canal da aba: o microfone é uma pessoa só, já
  identificada pelo login.
- Basta UM canal pronto pra call ficar `pronta`. Numa call pelo app do Zoom o
  canal da aba sai mudo; falhar tudo ali apagaria a única voz que existe.
- Numa call `pronta`, o campo `erro` carrega um AVISO, não uma falha.
- Corte de silêncio ficou de fora (exigiria ffmpeg, que não tem no Render).
  Custo medido ~R$420/mês no volume previsto. Medir o real antes de otimizar.

Ver [[reference_render_criar_cron_job]], [[reference_payload_select_enum_ddl]],
[[feedback_medir_antes_de_corrigir]].
