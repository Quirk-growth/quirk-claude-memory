---
name: project_remediacao_auditoria_membros
description: Plano de remediação da auditoria da área de membros (set/2026) — 37 de 41 itens no ar em 9 lotes; o que ficou pendente e por quê
metadata: 
  node_type: memory
  type: project
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-09-21T11:05:04.506Z
---

Auditoria da área de membros em **18/09/2026** (nota geral 5,8/10: velocidade 5,0 · banco/espaço 6,0 · segurança 6,0 · código/testes 6,8 · operação/custo 5,2) virou plano de 41 itens em 9 lotes. Execução concluída em **21/09/2026**: **37 itens no ar**, todos verificados em produção depois do deploy.

Plano: `docs/superpowers/plans/2026-09-18-plano-remediacao-auditoria.md`. Diário com SHAs e lições por lote: `.superpowers/sdd/progress.md` (gitignored — some com `git clean`).

## O que ficou pendente, e de quem é

**Do Renan (painéis que Claude não acessa)** — roteiro pronto em `docs/operacao/`:
- 7 perguntas no fim de `restauracao-do-banco.md` (retenção do Neon, plano, backup fora do Neon, quem acessa, versionamento R2, uso × limite, alerta de cota);
- ligar o gate de deploy (`gate-de-deploy.md`) — **só depois** dos 6 testes de integração ficarem verdes, senão trava o time;
- decidir sobre 5 arquivos de mídia incertos (~14 MB), listados em `.superpowers/sdd/rem-7-midia-orfa.md`;
- minutos de SLA por fila (destrava o item 6.3) e monitor externo de disponibilidade.

**Adiado por decisão dele:** retenção de criativos/métricas (item 7.1). Motivo: a proposta de "detalhe só nos últimos 90 dias" conflita com os filtros `365d`/`6m`/`12m`/`max` que a interface já oferece. Espera a cota real do Neon para ser decidida.

**Não é código:** "leads sem dono" — 4 clientes têm 100% dos leads sem vendedor porque o CRM deles não tem vendedor configurado.

## Ganhos que dá para medir

- Funil comercial: **−14,9% de HTML e −34% no tempo até a tela pronta** — veio de parar de carregar leads de etapa de saída, que nunca viravam card (500 de 1.295). Os outros três itens de velocidade não moveram esse número.
- Painel de relatórios: 13,3 s **sempre** → 0,82 s a partir do 2º acesso.
- `criativos`: 119 MB → 110 MB, e as atualizações voltaram a poder ser HOT.
- Suíte: 2.811 → **3.028** testes unitários.

## Como esse trabalho foi feito (o que funcionou)

Ciclo por lote: briefing escrito com a evidência medida → agente implementa → revisão adversarial → correção → verificação em **worktree limpa e destacada** → DDL manual em produção quando havia schema → push → verificação comportamental em produção.

**Nenhum lote passou de primeira.** Três foram rejeitados e corrigidos. Medir em produção derrubou afirmações da auditoria repetidas vezes — ver [[feedback_medir_antes_de_corrigir]].

Relacionado: [[project_area_membros_quirk]], [[reference_render_cloudflare_ip]], [[gotcha_afterlogin_transacao_pendura]].
