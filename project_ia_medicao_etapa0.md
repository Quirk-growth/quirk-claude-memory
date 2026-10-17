---
name: ia-medicao-etapa0
description: "Medição do uso de IA do Hub (tabela ia_chamadas + relatório), Etapa 0 PRONTA em branch local spec/ia-medicao, não enviada; deploy em um passo só, DDL antes; decisões e armadilhas"
metadata:
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-10-17T13:34:52.735Z
---

**Pedido do Renan (16/10/2026):** reestruturar o sistema inteiro para gastar menos tokens, em etapas e sem derrubar nada. Etapa 0 = MEDIR sem mudar comportamento. Etapas seguintes (não iniciadas): 1 porta única de IA + tetos/kill switches/cache, 2 leitura sem modelo (dados em seções, busca antes de ler, resumos guardados), 3 otimizações finas.

**Estado (17/10/2026):** worktree `~/area-membros-quirk-ia-medicao`, branch `spec/ia-medicao` (HEAD 0f67ef64, 36 commits sobre a origin/main 38141bc2, mescla limpo, NÃO enviada, upstream removido de propósito). Spec `docs/superpowers/specs/2026-10-16-ia-medicao-etapa0-design.md`, plano `docs/superpowers/plans/2026-10-16-ia-medicao-etapa0.md` (roteiro de implantação na Task 11). Suíte completa: 7457 ok; 16 falhas em 12 arquivos, 6 já falham na origin/main limpa e 6 passam isolados (flake de carga/banco de teste compartilhado). Nenhuma regressão da medição.

**O que é:** tabela nova `ia_chamadas` (só metadados, custo congelado, `created_at` = instante), coleção Payload `ia-chamadas` (só admin lê, menu "Chamadas de IA"), módulo `src/lib/ia/` (precos, custo, `registrarChamada` que NUNCA lança e espera no máximo 2,5 s, `IA_MEDICAO=off` desliga a gravação), relatório `scripts/ia-relatorio.mts` (só leitura, zero token) e verificador de produção `scripts/ddl/verificar-ia-chamadas.mts` (só leitura). 6 pontos ligados: `analises.gerar`, `crm.agente`, `crm.followup`, `linha_editorial.gerar`, `propostas.gerar`, `transcricoes.resumir`. O Quirk Intelligence já tinha medição própria (`quirk-intelligence-usos`).

**Deploy (decisão: UM deploy da branch toda, não um por ponto):** (1) aplicar `scripts/ddl/2026-10-16-ia-chamadas.sql` em produção ANTES (skill payload-migracao-prod; código antes do DDL derruba o admin com 42703), (2) rodar o verificador contra produção, (3) deploy com `IA_MEDICAO=off`, (4) smoke de um fluxo real por ponto, (5) ligar a gravação, (6) conferir pela seção por função do relatório, (7) janela de conferência com o Console da Anthropic começa no DIA SEGUINTE (a chave do Hub é exclusiva no Console). Sempre conferir se a origin/main andou e mesclar de novo antes do push.

**Decisões que valem lembrar:** ref_id de propostas fica nulo (a rota só cria o doc depois da geração); linha editorial usa união por REFERÊNCIA (ref_tipo `linha-editorial-geracoes` + id do job), `resumo-call` do QI é fonte antiga de `transcricoes.resumir` com regra por dia, `transcricao` (áudio) vai para "Fora de LLM"; a 10ª chamada paga (esteira de copy do QI, flag `QI_COPY`) fica FORA do relatório: conferir com `QI_COPY` desligado ou somar os pedidos; a tela nova "Uso da IA" (`/admin/uso-ia`) do Renan também conta `resumo-call`, não somar com o relatório sem tratar (Etapa 1).

**Achados fora de escopo (viraram a branch `fix/followup-envio-duplo-e-vazio`, worktree `~/area-membros-quirk-followup-fix`, 4 commits, não enviada):** follow-up podia enviar a mesma etapa 2x ou adiantada (reivindicação não exigia lead vencido; corrigido) e repetia chamada paga a cada ciclo com resposta vazia (recuo de 30 min, encerra após 2 h do horário da etapa; decisão do Renan pendente: encerrar a sequência inteira ou só pular a etapa) e o cliente do modelo agora tem timeout 60 s e 1 retentativa. Conflito textual simples com spec/ia-medicao em followUp.ts: o bloco de medição fica ANTES do tratamento novo da resposta vazia.

**Armadilhas deste trabalho:** banco de TESTE é compartilhado entre worktrees e o globalSetup trunca: nunca rodar duas suítes ao mesmo tempo (`pgrep -fl vitest`); o push do schema de uma worktree derruba tabelas da outra (ia_chamadas some; reaplicar o DDL só no banco de teste); fixtures com ids coincidentes (cliente=lead=1) escondem troca de colunas, usar helper de sequência; node_modules por symlink de um checkout com o MESMO package.json/lock.

Relacionado: [[project_quirk_intelligence_custos]], [[project_agente_social_media]], [[reference_neon_bancos]], [[gotcha_push_payload_trava_testes_int]].
