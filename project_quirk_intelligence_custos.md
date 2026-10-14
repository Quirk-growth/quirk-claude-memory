---
name: project_quirk_intelligence_custos
description: "Rastreio de uso/custo da IA do Quirk Intelligence (no ar 13/10/2026, só admin lê) + plano de economia de tokens com alavancas, números medidos e o que falta decidir"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-14T00:51:39.948Z
---

**No ar (commit dd90b89f, 13/10/2026):** coleção `quirk-intelligence-usos` (1 linha por colaborador/dia-Brasília/tipo `conversa`|`resumo-memoria`, contador atômico via SQL cru, custo US$ congelado na gravação). Código em `src/lib/quirkIntelligence/uso.ts`; orquestrador itera o Tool Runner (`consumirRunner`). Acesso direto: `/admin/collections/quirk-intelligence-usos` (sem item de menu — o menu é da matriz de permissões).

**Regra do Renan: só `admin` (Dono/Quirk) lê. Ninguém escreve pela API.** `supervisor`/`administrativo` ficam de fora (interpretei "administrador e acima" como o papel admin). Se ele quiser incluir o administrativo: trocar `isAdmin` na coleção + ajustar `quirkIntelligenceUsoAcesso.int.spec.ts`. Se algum dia o bot ganhar tool "quanto gastou?", tem que ser admin-only também.

**Preço Sonnet 5 (conferido na página oficial 13/10):** US$ 2 entrada / 10 saída / 0,20 leitura de cache / 2,50 escrita cache 5min por MTok; thinking já está em `output_tokens`. Tabela em `PRECOS_USD_POR_MTOK` (teste obriga preço pra todo modelo em uso).

**Medido em produção (só leitura, 13/10):** retorno da tool `metricas_clientes` = 46.006 caracteres pra admin/supervisor (143 clientes, 66 com veiculação) e 6.043 pra gestor (16 clientes). Compactar (arredondar, sem `slug`/`serieLeads`, sem-veiculação só `{id,nome}`) corta ~71% no caso pesado. Chave Anthropic só existe no Render (não no .env local) — contador oficial de tokens indisponível daqui.

**Plano (ordem):** 1) compactar retorno da tool (free win, 1 diff); 2) ler a tela de uso em 2-3 dias e decidir `effort`/thinking (troca de qualidade → precisa de ~20 perguntas reais do Renan como mini-eval); descartados por teto pequeno: cache de prefixo (~1-2K tokens, tool result é único por pergunta), memória/resumo em Haiku, trocar modelo, batch.

**Pro teto de 100/dia (outra sessão, task_abe6e475):** existe `usoDoDia(payload, usuarioId, dia)`, mas `interacoes` só incrementa DEPOIS da resposta — mensagens concorrentes furam o teto; teto estrito exige reservar a vaga antes.

Ver também [[reference_render_criar_cron_job]] (os 3 crons do bot).
