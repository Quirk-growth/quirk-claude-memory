---
name: project-painel-relatorios
description: Painel de relatórios de tráfego próprio da Quirk pra substituir o Reportei; multi-cliente dentro da área de membros
metadata: 
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
---

Projeto iniciado 23/jul/2026 (importante e estratégico): **painel de relatórios de tráfego próprio** pra **substituir o Reportei** (gasto alto; ~79 clientes no plano). Vira ativo da agência aproveitando o acesso de API que a Quirk já tem.

**Decisões travadas no brainstorming:**
- Fontes v1: **Meta Ads + Google Ads** (TikTok no backlog). Meta já puxa via app na BM central; Google via **MCC** centralizada.
- Multi-tenant: gestor vê todos os ~79; cliente vê só o dele. **Construído DENTRO da área de membros** ([[project-area-membros-quirk]]) — Next.js + Payload + Postgres, reaproveitando usuários/auth/design. Público de tráfego ≈ mesmos usuários da área de membros.
- Arquitetura: **sync diário (Vercel Cron, em código) → Postgres → painel**. Nada de API ao vivo.
- Granularidade: métrica por **campanha/anúncio por dia** (habilita atribuição por objetivo + principais criativos + qualquer período).
- **Regra crítica de atribuição:** custo por cadastro e custo por conversa NUNCA misturam objetivos de campanha; conversa de rebote conta como lead mas não no custo por conversa.
- Funil comercial: **só dados, SEM análise** (decisão 23/jul — Quirk é multi-nicho, benchmark de imobiliária não vale pra clínica/arquitetura). Removidos índice de qualidade, Índice de Eficiência Comercial, "ponto de aprimoramento" e projeção de VGV; o motor `engine.js` NÃO é usado. Etapas genéricas fixas: Leads(auto) → Atendimentos → Agendamentos (era "Visitas") → Vendas + ticket, com % sobre leads. Dados digitados pelo gestor no admin Payload; análise/leitura vai só no campo de comentário à mão. Ingestão da planilha de KPIs fica pro backlog.
- Duas telas validadas em mockup: relatório do cliente (rolagem única) + cockpit do gestor (portfólio com período até "Máximo", filtro de veiculação, totais da agência).

**Local:** spec + mockups em `/Users/renanreal/quirk-painel-relatorios/` (git). **Código em `/Users/renanreal/area-membros-quirk/`** (é a área de membros — Payload 3.86 + Next 16 + Postgres). Planos em `docs/superpowers/plans/`.

**Progresso (23/jul):**
- **Plano 1 (fundação dados + sync): CONCLUÍDO e MERGED na main local.** 6 coleções (clientes, contas-ads, metricas-diarias/criativos/sociais/resultado), isolamento multi-tenant (`isGestorOuDonoCliente`), lógica pura em `src/lib/relatorios/` (objetivo, atribuição c/ regra do rebote, agregação), adapters Meta/Google em `src/lib/sync/`, upsert idempotente, `runSync` (isolamento por conta + param opcional `clienteIds`), rota cron `api/cron/sync` (x-sync-secret).
- **Plano 2 (relatório do cliente): CONCLUÍDO e MERGED na main local.** Página `src/app/(frontend)/(app)/relatorios/page.tsx` + `RelatorioCompleto` reutilizável + componentes em `src/components/relatorios/` + camada de consulta. Funil SÓ DADOS.
- **Plano 3 (cockpit do gestor + acesso por carteira): CONCLUÍDO e MERGED na main local.** Acesso em 3 níveis (Dono/admin vê tudo, Gestor vê só `clientes.gestores` atribuídos, Cliente vê o próprio); `clientesVisiveis`/`isGestorOuDonoCliente` async fail-closed; cockpit `src/app/(frontend)/(app)/portfolio/page.tsx` escopado ao papel; rota guardada `relatorios/[clienteSlug]`. Papéis Users: admin|gestor|member.
- Suíte 83 testes verde. Executado via subagent-driven-development (ledger em `.superpowers/sdd/`).
- **Colisão de sessão (23/jul):** outra sessão commitou `docs/crm/` na branch do Plano 3; isolei numa branch limpa `feat/painel-relatorios-plano3-focus` (mergeada). Branch antiga `feat/painel-relatorios-plano3` ainda tem os commits de CRM (não é nossa).
- **Plano 4 (export PDF com logo): CONCLUÍDO e MERGED na main local.** Botão Exportar PDF (`BotaoExportarPdf`, window.print) + `CabecalhoPdf` (logo `/logo-quirk.png`) + print CSS (`@media print` + `print-color-adjust:exact`, `.no-print`/`.only-print`) em `globals-quirk.css`; embutido no `RelatorioCompleto` (vale cliente + gestor).

**Os 4 planos de construção estão CONCLUÍDOS e MERGED na main local (não pushed).** Falta só operacional/produção:
- **Push pro GitHub** (main ahead ~42 commits; dispara deploy — requer OK do Renan).
- ~~Bloqueio de build~~ **RESOLVIDO (23/jul, merge 042db8b):** `next build` passa. Fix: `Clientes.ts` read tipado como `Where`; `tsconfig` exclui `tests/`; `vitest.config.mts` ganhou alias `@/`→`src` explícito (o exclude quebrava a resolução de paths do vitest). Suíte 83 verde + build provado.
- **Env vars que FALTAM pro sync real (não estão no .env):** `META_ACCESS_TOKEN`, `GOOGLE_ADS_ACCESS_TOKEN`, `GOOGLE_ADS_DEVELOPER_TOKEN`, `GOOGLE_ADS_MCC_ID`, `SYNC_SECRET` (segredo do cron — defina um aleatório). Já configuradas: APP_URL, DATABASE_URI, PAYLOAD_SECRET, MAIL_*, R2_* (media).
- Validar adapters Meta/Google contra API real + tokens (META_ACCESS_TOKEN, GOOGLE_ADS_*); completar criativos[]/sociais[].
- Verificação visual autenticada (login real) do relatório + do PDF (print preview) — não automatizável.
- Deploy da área de membros.
- Minors acumulados p/ limpeza (ver ledgers .superpowers/sdd/progress-plano*.md): ex. `.q-card` inerte no print CSS, back-link do gestor imprime, idsDaCarteira duplica query.
Backlog Plano 5+: TikTok, ingestão planilha KPIs, envio automático por e-mail (+ PDF server-side), alertas, ordenação/busca no cockpit.

**Pendências:**
- **NÃO pushed pro GitHub** (main ahead ~26 commits; push provavelmente dispara deploy — requer OK do Renan).
- **Bloqueio de build pré-existente:** `main` tem ~14 erros tsc em arquivos de TESTE que travam `next build` (tsconfig inclui `**/*.ts`); limpar antes do deploy.
- Validar adapters Meta/Google contra API real + tokens (META_ACCESS_TOKEN, GOOGLE_ADS_*); completar criativos[]/sociais[] nos adapters; decidir GET p/ Vercel Cron.
- Deploy da área de membros (já era pendente).

**Próximo:** Plano 3 (cockpit do gestor §9.2 + PDF com logo + acesso do gestor a qualquer cliente).
