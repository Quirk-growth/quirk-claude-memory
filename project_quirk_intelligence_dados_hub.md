---
name: project-quirk-intelligence-dados-hub
description: "Bot do Telegram agora lê Handoff CS, Gestão de Rotina e relatório semanal do Hub (3 ferramentas só-leitura); commit e98f9074 NO AR em 14/10 (deploy atrasado por incidente do Render); regras de acesso e o que ficou pendente"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-14T20:47:34.234Z
---

**Entregue (14/10/2026, commit `e98f9074` na main, branch `feat/qi-dados-hub`):** ferramentas `handoff_cs`, `gestao_rotina` e `relatorio_semanal` no bot (`src/lib/quirkIntelligence/dadosHub.ts`, registro no `orquestrador.ts`). Cada uma tem modo cliente (`clienteId`) e modo carteira (sem id; `handoff_cs` aceita `apenasEmRisco`, `gestao_rotina` aceita `atencaoMinima`/`revisaoPendente`; relatório é sempre de UM cliente e devolve `mensagemPronta`, o texto de sexta). O cálculo do relatório saiu do Hub para `lib/relatorios/dadosRelatorioSemanal.ts` (fonte única com o bot; refactor do `HubAbaRotina`/`HubRelatorioSemanal` equivalente em 5.000 casos).

**Regras de acesso (decisão minha, avisar o Renan se ele quiser afrouxar):** papel = aba do Hub (`abasDoPapel`) E leitura da coleção. Handoff: equipe + social. Rotina e relatório semanal: SÓ a equipe (admin, supervisor, administrativo, gestor) — `comercial` fica de fora mesmo o menu do Hub mostrando a aba Rotina a ele (a coleção `gestao-rotina` nega leitura ao comercial e tem comentário interno do supervisor). Cliente fora da carteira = inexistente (mesma resposta); modo carteira exclui status `churn`.

**Dados reais medidos na produção (14/10):** 578 registros de rotina, só 7 handoffs em 127 clientes vivos (adoção do handoff é baixa: o bot vai achar pouco). Textos: rotina até 1.013 caracteres, handoff até 712. `semana`/`data_referencia` gravadas em 00:00 UTC (o `slice(0,10)` é o dia digitado).

**Pendências:**
- **Deploy: NO AR.** Push 17:36 BRT; o Render teve incidente ("Builds are not starting in all regions", 19:32–21:43 UTC) e o build só rodou ~18:43; "Deploy succeeded" para e98f907 (1h13m, quase tudo fila). Site estável (login 200, rota protegida 403). Não há rota nova pra smoke test: a prova é o primeiro uso real no Telegram.
- **Primeiro uso real:** pedir ao Gerson "como está o handoff da Primare?" / "quais dos meus clientes estão em risco?".
- **Bug do Hub (task separada, chip `task_d8b9fcf1`, o Renan pediu para NÃO mexer):** `HubAbaRotina` calcula a janela de 7 dias com `new Date()` em UTC; das 21h às 23h59 BRT inclui o dia corrente. O bot usa `hojeEmBrasilia()` e diverge nesse intervalo.
- **Teste instável que já existia na main:** `quirkIntelligenceSaldoContas.int` ("média EXATAMENTE 7 dias") falha com `expected 3 to be 0` quando roda junto com `quirkIntelligenceAlertaSupervisor.int` (conta contas de todo o banco compartilhado); reproduzido em `origin/main` puro. Falhas também reproduzidas em `origin/main`: tarefas-recorrencia, clientes-escrita-financeiro-comercial, mensagens-onboarding-destinatarios, tarefa-templates-aplicar, converter-cria-tarefa. A suíte completa não foi rodada de ponta a ponta (714 arquivos, ~1h30); rodei todas as unitárias (430 arquivos, verdes) e as 26 de integração das áreas tocadas.
- **Próximo pedido do Renan:** espelhar no Telegram dele o que os gestores perguntam ao bot (ver [[project-quirk-intelligence-auditoria-conversas]]).

Ver também [[project-quirk-intelligence-tarefas-escopo]] e [[project-quirk-intelligence-custos]].
