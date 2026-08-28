---
name: project_auto_ads_migracao_codigo
description: "Proposta (em STAND-BY) de migrar o cérebro de decisão do Auto Ads do n8n pra um serviço em código; lembrar o Renan no futuro"
metadata:
  node_type: memory
  type: project
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-28T12:03:26.869Z
---

**STATUS: STAND-BY (24/ago/2026).** O Renan pediu pra parar e **lembrá-lo futuramente** — resurjo quando a gente esbarrar de novo em fragilidade/escala do [[project_quirk_auto_ads]].

**Contexto:** depois de uma sessão achando bugs de silêncio no motor n8n (CONFIRMAR caindo na rota de gestão, áudio/localização mudos, metade dos verbos de gestão — reativar/encerrar/alterar geo — sem executar, webhook sem auth), propus mover a **lógica de decisão** (classify, roteamento, máquina de estado, chamadas Meta) pra um **serviço Node/TS** — mesmo stack da [[project_area_membros_quirk]]. n8n vira encanamento (webhook, envio, crons). Ganho principal: **testabilidade** (o tipo `Promise<Reply[]>` + try/catch central torna dead-end silencioso impossível). Esboço/proposta em artifact: https://claude.ai/code/artifact/f891b909-79bf-4a42-9ebf-912613842838

**Minha recomendação (registrada):** migrar incremental (não big-bang) — porque o Auto Ads é produto pago e escalando, e a fragilidade compõe com cada cliente. Caminho de menor risco: (1) fechar segurança do webhook no n8n, (2) Fase 0 = esqueleto + 1 intent (SALDO) ponta a ponta, (3) decidir continuar. Esforço: ~3–4 semanas em tempo parcial pra migração completa.

**⚠️ Buracos de segurança do n8n mapeados (fechar independente da migração):** (1) **webhook `/webhook/quirk-auto-ads` SEM autenticação** — dá pra injetar mensagem falsa de cliente (validar assinatura `X-Hub-Signature-256` do WhatsApp); (2) `meta_access_token` em **texto puro** em `auto_ads.config`; (3) SQL por interpolação de string em vários nós (risco de injection; parametrizar). O #1 é o mais crítico.
