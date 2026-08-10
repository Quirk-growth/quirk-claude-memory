---
name: project_relatorio_perfil_ig
description: "Relatório de crescimento de perfil do Instagram na área de membros — Fase 1 (métricas de perfil) + Fase 2a (bloco de conteúdo posts/reels/stories), no ar; pendências do lado Meta"
metadata: 
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-10T21:01:05.477Z
---

Relatório de social media da Quirk (substitui o que se perdeu do Reportei), dentro da área de membros ([[project_painel_relatorios]]). Reusa o pipeline social (coleção `metricas-sociais`, sync `instagram.ts` com token System User em `META_ACCESS_TOKEN`, componentes do relatório de mídia — sem estilo novo, restrição do Renan).

- **Fase 1 (no ar):** métricas de crescimento de perfil (seguidores, alcance, visualizações, visitas, contas engajadas, interações, cliques, demografia idade/gênero/cidade). Config no hub aba Relatório: toggle "mostrar perfil do Instagram" + quais contas entram.
- **Fase 2a — bloco de Conteúdo (deploy ago/2026):** coleção nova `publicacoes-sociais` (só o sync escreve, leitura isolada por cliente); sync busca posts/reels/stories com insights (`data[0].values[0].value`, SEM metric_type) e faz upsert por `[cliente, mediaId]` no cron diário; `topPublicacoesDoCliente` traz o top-8 por alcance da janela; UI é um bloco inline no `SecaoCrescimentoPerfil` (miniatura + selo tipo + legenda + alcance/interações/visualizações + link) — o `ListaRanking` real não tinha miniatura. DDL de prod pegou a lição do enum ([[reference_payload_select_enum_ddl]]).

**Pendências do Renan (lado Meta, o bloco fica ZERADO até isso):** (1) ligar "mostrar perfil do Instagram" nos clientes de social; (2) reconciliar ~21 `igUserId` quebrados (perfis fora da BM que o token enxerga) — de-para já levantado. Stories não têm retroativo (só acumulam do deploy pra frente); posts/reels o cron popula até o teto de 50 mídias.

**Fase 2b (futuro, spec própria):** 4 métricas de perfil — comparação com período anterior, alcance seguidores×não-seguidores, taxa de engajamento, melhores horários (`online_followers`).
