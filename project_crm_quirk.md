---
name: project_crm_quirk
description: CRM multi-tenant dentro da área de membros (Bolten+Imobilead); v1 no ar ago/2026; hierarquia dono/gerente/vendedor; pendências UAZAPI
metadata: 
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-02T13:55:47.572Z
---

CRM próprio da Quirk dentro da [[project_area_membros_quirk]], pros CLIENTES usarem (multi-tenant). Une Bolten (contato WhatsApp vira lead, kanban editável) + Imobilead (filas por campanha, roleta de vendedores). Spec `docs/superpowers/specs/2026-08-02-crm-quirk-design.md`; 3 planos em docs/superpowers/plans/2026-08-02-crm-*.md. **v1 deployada em 02/ago/2026** (21 commits, 85a1c33..82a32bf) via subagent-driven development (16 tasks, review por task + review final).

**Arquitetura v1 (fase A — sem chat):** 4 coleções `crm-leads/vendedores/filas/conexoes` + `clientes.crmAtivo/crmEtapas/crmWebhookToken` + `users.papelCliente` (dono/gerente/vendedor — campo travado isAdminField, convites via /convidar-time). UAZAPI segura as conexões WhatsApp (QR); nosso lado só recebe webhooks — NADA de mensagens armazenadas. Fases futuras: B = chat leitura sob demanda da UAZAPI; C = chat completo (serviço separado).

**Regras-chave:** dedupe por (cliente, telefone normalizado DDI55) — UNIQUE no banco; roleta round-robin com ponteiro persistido na fila (fila vazia → round-robin geral entre ativos); vendedor só vê/edita os leads dele (bypass null corrigido); webhook genérico `/api/crm-leads/entrada/<token>` (Meta/Google/TikTok via Make/n8n); webhook UAZAPI `/api/crm/whatsapp?token=`; página `/crm` (dark) + aba CRM no hub (tema escopado `.hub-crm-tema`); ativação = switch no Cadastro (supervisor+) que gera etapas padrão + token + fila Geral.

**Pendências pro rollout:** (1) Render: `UAZAPI_BASE_URL` + `UAZAPI_ADMIN_TOKEN` (sem eles só o Conectar falha); (2) **validar payload real do webhook UAZAPI** — capturar amostra da instância do Renan e ajustar `mapearEventoUazapi` (src/lib/crm/uazapi.ts, único ponto de ajuste; endpoints /instance/* também a confirmar); (3) ativar primeiro na conta de teste renandmreal; ao criar vendedores, MARCAR na fila Geral. Follow-ups aceitos: CSV prévia+colunas etapa/vendedor; filtros campanha/origem na lista; testes int dos webhooks; UI pra gestor trocar papelCliente (campo agora é admin-only).
