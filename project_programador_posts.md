---
name: project_programador_posts
description: "Programador de posts multi-plataforma (Frente 1 do plano de Social Media) — task=post na lista Social Media, publica via Meta; NO AR 18/ago, faltam 2 passos externos pra funcionar de verdade"
metadata: 
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-19T02:24:16.866Z
---

Frente 1 do plano de 4 frentes de Social Media (ver [[reference_papel_social_media]]): agendar e publicar posts (Instagram Feed/Story + Facebook Feed, V1 só imagem) direto da [[project_tarefas]] — a task da lista "Social Media" **é** o post. Spec `docs/superpowers/specs/2026-08-16-programador-posts-social-media-design.md`, plano `docs/superpowers/plans/2026-08-16-programador-posts-social-media.md` (12 tasks), execução via subagent-driven-development.

**NO AR (18/ago, commit `eaf6c5a`)**, deploy confirmado saudável (34/34 health checks). Mas a publicação de verdade **não funciona ainda** — faltam 2 passos de fora do código (ver Pendências abaixo).

## Arquitetura

- `listas_tarefas.permite_publicacao` (boolean) é a ÚNICA fonte de verdade pra "esta lista publica posts" — nunca casar por nome. Hoje só a lista Social Media (id 6) tem.
- `tarefas.agendamento` (jsonb): `{midiaId, legenda, plataformas[], dataHora, status, publicadoEm, resultados[]}`. Helpers puros em `src/lib/tarefas/agendamento.ts` (`lerAgendamento`, `normalizarAgendamento`, `agendamentoEstaDue`, `chaveDiaLocal`, `ROTULO_PLATAFORMA`).
- Publicar: ação `publicarAgora` (UI) ou cron `POST /api/cron/publicar-posts` (a cada 5min, `PUBLICAR_POSTS_SECRET`) → `src/lib/publicacao/publicarTarefa.ts` (orquestração) → `executor.ts` (itera plataformas, injeção de `Publicador`) → `meta.ts` (Graph API v21, token de sistema `META_ACCESS_TOKEN`, mesmo padrão do CRM).
- UI: seção "Publicação" no `TarefaModal`, modo "Calendário" (grade mensal) no `TarefasBoard`, atalho de menu "Social Media" → `/admin/social-media` (resolve a lista por `permitePublicacao`, redireciona pro Calendário).

## Achados críticos da revisão final de branch (corrigidos antes do deploy, commit `ff6ba13`)

Duas revisões passaram (task-a-task) sem pegar; só a revisão de branch INTEIRA achou:

1. **`media.url` do Payload é relativa e protegida (`mediaRead` nega leitura anônima)** — o Meta busca a imagem sem credencial, então NENHUMA publicação funcionaria, e o erro seria indistinguível do "aprovação Meta pendente" (ficaria escondido). Mesmo bug já resolvido uma vez, por outro consumidor, em `src/lib/onboarding/executar.ts:resolverLogoBase64` (lá lendo bytes direto porque não precisava de URL). Aqui a correção é `src/lib/publicacao/urlPublicaMidia.ts` — URL assinada temporária do R2 (`@aws-sdk/s3-request-presigner`, 10min), sem tornar o bucket público nem enfraquecer `mediaRead`.
2. **Idempotência tinha corrida real**: ler-status-depois-escrever permitia 2 chamadas concorrentes (cron + "Publicar agora" no mesmo instante) publicarem 2x — reproduzido com `Promise.all`, teste novo prova que não acontece mais. Fix: `UPDATE tarefas SET agendamento=jsonb_set(...,'publicando') WHERE agendamento->>'status' IN ('programado','erro')` via `pool.query` raw (mesmo padrão de `Users.ts` pras preferências) — condicionado ao status ATUAL na linha, não ao que foi lido.

**LIÇÃO pra próximas frentes (2/3/4)**: sempre rodar uma revisão de BRANCH INTEIRA no fim de um plano multi-task, mesmo com revisão por task limpa em todas — bug de integração cross-cutting (aqui: a URL nunca é testada fim-a-fim porque todo teste injeta publicador fake) só aparece olhando o todo.

## Pendências (fora do código, ação do Renan)

1. **Aprovação Meta** dos escopos `pages_manage_posts`/`instagram_basic`/`instagram_content_publish` no app "BM - Clientes Quirk" — sem isso todo publish falha com erro de permissão (esperado, documentado). Única pendência restante da Frente 1.

**Infra de publicação 100% pronta (18/ago):** `PUBLICAR_POSTS_SECRET` configurado no serviço web (deploy `fb58cf7` live) + **Render Cron Job `publicar-posts`** criado (`crn-da2h737qj5pc73fqkicg`, mesmo padrão do `sync-diario-painel`: Node, branch main, build `echo sem-build`, `*/5 * * * *`, env `APP_URL`+`PUBLICAR_POSTS_SECRET`). Disparo manual de teste ("Trigger Run") rodou e terminou "successfully". Assim que o Meta aprovar os escopos, publicação começa a funcionar sozinha sem nenhuma ação extra.

## Minors abertos (não bloqueiam, ficam pra depois)

Tasks 10/11 (UI) sem teste `.spec.tsx`; thumbnail do criativo não aparece no card do Calendário (spec pedia, ficou pra depois — trivial agora que a URL é pública); tela "publicado com links" no modal fica em branco logo após "Publicar agora" (key remount + a task já virou concluída, listagem padrão esconde); DDL de `permite_publicacao` não checa linhas afetadas (mitigado nesta rodada — rodei com `RETURNING` e conferi 1 linha).
