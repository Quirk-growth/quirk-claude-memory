---
name: feedback_medir_antes_de_corrigir
description: "Medir em produção antes de corrigir — na remediação da auditoria, a medição derrubou achado atrás de achado, inclusive meus"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-09-21T11:05:20.027Z
---

Durante a remediação da auditoria da área de membros (set/2026), **quase toda afirmação conferida contra a produção mudou de tamanho — ou caiu**. O padrão se repetiu vezes demais para ser coincidência.

**Por quê:** leitura de código produz hipótese, não fato. Relatório de auditoria (inclusive escrito por mim) herda essa incerteza e a apresenta com a confiança de um número.

**Como aplicar:** antes de corrigir um achado, medir o que ele afirma. Antes de apagar qualquer coisa, medir o que se perde. E quando o agente contraria a instrução com medição, a medição ganha — aconteceu três vezes, e nas três eu estava errado.

Casos reais, todos de 18–21/09/2026:

| Afirmado | Medido |
|---|---|
| 834 tarefas vencidas | 46 |
| Vazamento cross-tenant de fotos | só time da Quirk tem foto |
| Validade de 30 dias "não quebra nada" | 11 de 19 links leriam "expirado" no 1º acesso |
| 389 leads parados sem dono | 430, mas 4 clientes com **100%** — é CRM sem vendedor configurado, não lead parado |
| 152 leads sem telefone | 11 |
| "Índice com 3 usos em 18 dias é inútil" | dropá-lo deixaria a tela 600× mais lenta (0,18 ms → 110 ms) |
| 33 arquivos de mídia órfãos | 8 estavam **em uso** — id dentro de `jsonb` sem FK; apagar destruiria Cartões CNPJ de clientes |
| Duplicata de telefone não existe (**eu disse isso**) | existe, e diverge no sentinela de inválido |
| `await` no hook de login "não é fuga de deadlock" (**eu mandei escrever**) | pendura o login; o agente mediu e recusou |
| `system_identifier` como prova de identidade (**sugestão minha**) | branch do Neon herda do pai — recusaria a restauração legítima |

**Duas armadilhas de método que apareceram:**

- Contador do Postgres (`idx_scan`) cobre só desde o último restart — 18 dias ali. Não prova nada sobre rotina mensal ou anual.
- Rodar a suíte inteira e comparar com um subconjunto **não é comparação válida**: 2 testes "quebrados" por um lote passavam isolados, era contaminação de ordem no banco de teste compartilhado.

Relacionado: [[project_remediacao_auditoria_membros]], [[feedback_acompanhar_deploy_ate_verde]].
