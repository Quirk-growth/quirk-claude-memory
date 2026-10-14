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

**Corte 1 NO AR (e9012481, deploy live 14/10):** `compactarMetricasClientes` (`src/lib/quirkIntelligence/compactarMetricas.ts`) — medido de novo em produção: 45.951 → 18.953 caracteres (−59%; 143 clientes, 66 com veiculação). Tira `slug` (6.156 chars) e `serieLeads` (6.194), arredonda, sem-veiculação vira só id/nome/status/gestores (status importa: standby/onboarding/churn não têm veiculação e nenhuma outra tool devolve status; só id+nome daria −65% mas perde isso). Mantém `{linhas,totais}` e os nomes de campo (testes int dependem de `linhas[].clienteId`). **Ainda sem baseline em tokens reais: a tabela `quirk_intelligence_usos` estava VAZIA em 14/10** (ninguém mandou mensagem ao bot) — pra medir o efeito, mandar "métricas de todos os clientes" no Telegram e ler a tela de uso.

**Plano (ordem):** ~~1) compactar retorno da tool~~ (feito); 2) ler a tela de uso em 2-3 dias e decidir `effort`/thinking (troca de qualidade → precisa de ~20 perguntas reais do Renan como mini-eval); descartados por teto pequeno: cache de prefixo (~1-2K tokens, tool result é único por pergunta), memória/resumo em Haiku, trocar modelo, batch.

**Alertas proativos — 1ª execução real conferida (14/10, logs do Render):** `quirk-intelligence-alertas-custo` (07:00 BRT, 17 s) → `{"ok":true,"picosDetectados":3,"alertasEnviados":2}`; `quirk-intelligence-alerta-saldo` (07:00 BRT, 1m28s) → `{"ok":true,"contasDetectadas":9,"alertasEnviados":8}`. `alertasEnviados` conta MENSAGENS (alerta × gestor vinculado), não alertas — não dá pra ler "9→8" como "1 falhou". **Lacuna de adoção:** o alerta só vai pra quem está em `cliente.gestores` E com Telegram `vinculado`, sem plano B, e cliente sem ninguém vinculado some em silêncio (não aparece em nenhum contador). Em 14/10 só Gerson (14 clientes), Phillipe (13) e Anna (9) estavam vinculados; Beatriz (16), Carlos (12), Debora Waller (12), Gustavo (12) — as maiores carteiras de gestor — nunca recebem. `gestores` do cliente inclui supervisores. **Plano B NO AR (commit 92cc6b42, 14/10, decisão do Renan: "o supervisor pode receber quando o gestor não estiver vinculado"):** `lib/quirkIntelligence/destinatariosAlerta.ts` (`enviarAlertas`) — gestor vinculado recebe individual como sempre; sem gestor alcançável (ninguém vinculado OU envio falhou) os supervisores ATIVOS com Telegram vinculado recebem UM resumo por supervisor (com o nome do gestor de cada cliente); sem supervisor consultado quando todo alerta achou gestor; admin NÃO entra. Não existe hierarquia gestor→supervisor, então o resumo vai pra TODOS os supervisores vinculados (hoje Phillipe e Anna). Resultado dos crons ganhou `alertasViaSupervisor`, `resumosAoSupervisor`, `semDestinatario` (+ `logger.warn` com ids dos clientes). Continua valendo: pedir a Beatriz, Carlos, Debora e Gustavo que vinculem o Telegram (tela do perfil). Render avisou manutenção em 14/10 22:00 BRT (15/10 01:00 UTC).

**Pro teto de 100/dia (outra sessão, task_abe6e475):** existe `usoDoDia(payload, usuarioId, dia)`, mas `interacoes` só incrementa DEPOIS da resposta — mensagens concorrentes furam o teto; teto estrito exige reservar a vaga antes.

Ver também [[reference_render_criar_cron_job]] (os 3 crons do bot).
