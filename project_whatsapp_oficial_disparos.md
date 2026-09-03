---
name: project-whatsapp-oficial-disparos
description: "WhatsApp Oficial — Disparos e Templates (Fase 2): feature nova no ar (main a156071+582ecde, 02/set/2026) — templates via Graph API, disparo em massa com filtro de audiência, rastreio de entrega, tela na aba Comercial"
metadata:
  node_type: memory
  type: project
  originSessionId: 7b5edbe3-a2f1-4c16-bf89-5ff55a540ed7
  modified: 2026-09-03T22:07:50.005Z
---

Fase 2 do WhatsApp oficial (Cloud API) na [[project_area_membros_quirk]], construída em cima da Fase 1 (conexão real via Embedded Signup, já em produção desde 19/ago). Spec: `docs/superpowers/specs/2026-09-02-whatsapp-oficial-disparos-templates-design.md` (só existe na branch antiga `feat/whatsapp-cloud-api-conexao`, não foi cherry-picked pra `main` — pendência de arquivo, não de código). Plano: `docs/superpowers/plans/2026-09-02-whatsapp-oficial-disparos-templates.md`.

**O que entrega**: tela "WhatsApp Oficial" dentro da aba Comercial (item fixo de menu, gate por papel admin/administrativo), com 4 abas — Conexão (reaproveita o botão já existente), Conversas (leads do canal oficial, rastreado por campo próprio), Templates (CRUD ao vivo contra a Graph API do Meta, sem persistência local), Disparos (filtro de audiência por etapa/campanha/dias-sem-interação + envio em massa de template, processado em background por um cron do Render com claim atômico via SQL cru).

**Executado via `subagent-driven-development`**: 13 tasks do plano original + revisão final do branch inteiro (que achou 3 Critical + vários Important) + 3 rodadas de correção pós-revisão-final antes de mergear. Ver [[reference_payload_update_where_nao_atomico]] pro gotcha técnico mais importante descoberto (claim de fila não é atômico via `payload.update({where})`).

**3 Critical achados só na revisão do branch inteiro (não pegos por revisão task-a-task)**:
1. Deadlock de bootstrap — o botão de conectar só existia dentro da tela que exigia conexão já existir pra abrir. Corrigido: aba Conexão (e o item de menu) abre só por papel; as outras 3 abas ficam escondidas até conectar.
2. `mapearStatusCloudApi` procurava `field==='statuses'`, mas o Meta manda status de entrega sob `field==='messages'` — rastreio de entrega era código morto, mascarado por 20 testes com fixture fabricada errada.
3. Aba Conversas filtrava `vendedor:{exists:false}` achando que identificava "canal oficial" — mas a roleta atribui vendedor a TODO lead novo (medido: 0/122 leads reais sem vendedor). Corrigido com campo próprio `crm-leads.canalOficial`, setado só pelo webhook Cloud API.

**3 DDLs de produção aplicadas nesta sessão** (endpoint direto, script Node descartável, confirmado por introspecção antes/depois): `crm-disparos`/`crm-disparos-itens` (coleções novas + bloco `payload_locked_documents_rels`), `crm_leads.canal_oficial` (coluna aditiva numa tabela já com 473 linhas reais, sem rewrite), enum `enviando` em `enum_crm_disparos_itens_status`.

**Merge pra `main`**: feito via worktree temporário detached em `origin/main` (não no worktree raiz compartilhado, que estava ocupado por outra sessão) — ver [[feedback_git_repo_compartilhado_sessoes]]. Commits finais: `a156071` (merge) + `582ecde` (fix de typecheck:tests pós-merge). Smoke test: `POST /api/crm/disparos-processar` sem secret → 401 (rota só existe por causa desta feature).

**Pendências conhecidas, documentadas como follow-up (não bloqueiam a feature em produção)**:
- Render Cron Job pro endpoint `/api/crm/disparos-processar` ainda **não foi criado no painel** — o processamento em background fica inerte até alguém configurar (doc: `docs/operacao/render-cron-disparos.md`).
- Filtro de "fila" no Disparos: backend já suporta (`audienciaDisparo.ts`), UI não expõe (não existe endpoint de listar filas pra popular um dropdown) — gap do próprio plano, não do código.
- `/crm-disparos/historico` sem teste dedicado (revisor mediu 6 cenários manualmente, sem bug).
- Regex de detecção de variável de template (`{{n}}`) tolerava espaço só no front — fix server-side (`CrmDisparos.ts` `templateTemVariaveis`) ficou pra um chip de `spawn_task` separado, aplicado pelo usuário depois do merge (fora desta sessão).
- Verificação visual ao vivo (login real na tela) nunca aconteceu — toda validação foi estática/teste de integração mockado, por falta de credenciais na sessão.
