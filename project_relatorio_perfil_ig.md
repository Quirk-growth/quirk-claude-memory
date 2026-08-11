---
name: project_relatorio_perfil_ig
description: "Relatório de crescimento de perfil do Instagram na área de membros — Fase 1 (métricas de perfil) + Fase 2a (bloco de conteúdo posts/reels/stories), no ar; pendências do lado Meta"
metadata: 
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-11T12:22:21.094Z
---

Relatório de social media da Quirk (substitui o que se perdeu do Reportei), dentro da área de membros ([[project_painel_relatorios]]). Reusa o pipeline social (coleção `metricas-sociais`, sync `instagram.ts` com token System User em `META_ACCESS_TOKEN`, componentes do relatório de mídia — sem estilo novo, restrição do Renan).

- **Fase 1 (no ar):** métricas de crescimento de perfil (seguidores, alcance, visualizações, visitas, contas engajadas, interações, cliques, demografia idade/gênero/cidade). Config no hub aba Relatório: toggle "mostrar perfil do Instagram" + quais contas entram.
- **Fase 2a — bloco de Conteúdo (deploy ago/2026):** coleção nova `publicacoes-sociais` (só o sync escreve, leitura isolada por cliente); sync busca posts/reels/stories com insights (`data[0].values[0].value`, SEM metric_type) e faz upsert por `[cliente, mediaId]` no cron diário; `topPublicacoesDoCliente` traz o top-8 por alcance da janela; UI é um bloco inline no `SecaoCrescimentoPerfil` (miniatura + selo tipo + legenda + alcance/interações/visualizações + link) — o `ListaRanking` real não tinha miniatura. DDL de prod pegou a lição do enum ([[reference_payload_select_enum_ddl]]).

O deploy da Fase 2a (10/ago) **derrubou a prod com OOM/502** — não era bug de código; a instância Render de 512 MB não aguentou o footprint extra. Rollback + upgrade pra Standard 2 GB resolveu e a Fase 2a subiu de pé (commit 4e9b142). Lição em [[reference_render_memoria_oom]].

**Pendências do Renan (lado Meta, o bloco fica ZERADO até isso):** (1) ligar "mostrar perfil do Instagram" nos clientes de social; (2) reconciliar ~21 `igUserId` quebrados (perfis fora da BM que o token enxerga) — de-para já levantado. Stories não têm retroativo (só acumulam do deploy pra frente); posts/reels o cron popula até o teto de 50 mídias.

**Fase 2b (no ar, ago/2026, commit cfcf5b8):** 3 métricas na seção de perfil — comparação com período anterior (setas de delta nos KPIs; N/A até o histórico acumular), taxa de engajamento (Interações÷Alcance) e alcance seguidores×não-seguidores (barra de proporção, `reach&breakdown=follow_type&metric_type=total_value`, +2 colunas `alcance_seguidores`/`alcance_nao_seguidores`). **Melhores horários (`online_followers`) CORTADO** pelo Renan. Deploy nos 2 GB sem OOM. GOTCHA que o review final pegou: `pct()` (Intl percent) já multiplica por 100 — não pré-multiplicar taxa/deltas (são fração); e o token do azul da marca é `--accent` (`#1d80ff`), NÃO `--primary` (inexistente). Fora da 2b: heatmap dia×hora, follows_and_unfollows.
