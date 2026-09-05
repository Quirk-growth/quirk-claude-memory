---
name: reference_migracao_historico_clickup_crm
description: "Importação do histórico completo do ClickUp (3.384 cards) para o CRM do cliente 104 — CONCLUÍDA 04/09/2026; como rodar de novo e o que verificar"
metadata:
  node_type: memory
  type: project
---

Migração do **histórico** de leads da lista "prospectos" (ClickUp `900902474139`, space Pipeline Quirk) para o CRM interno do cliente 104 ("Quirk — Comercial"). É outra coisa da [[project_migracao_leads_clickup_crm]], que ligou os funis do Make ao CRM daqui pra frente — esta trouxe o passado.

**Concluída em 04/09/2026.** 3.384 cards → 1.190+1.837 criados, ~250 completados, **0 não entraram**. CRM do 104 foi de 130 pra 3.177 leads e de 80 pra 9.026 anotações. Invariantes conferidos no fim: 0 carimbos duplicados, 0 telefones duplicados, 0 SLA armado, 0 leads sem `ultimaInteracao`, e os 41 leads vivos todos dentro da janela de 500 do quadro.

**Why:** o SDR trabalhava sem memória — abria um lead no CRM sem saber que o time já tinha falado com aquela pessoa meses antes.

**How to apply:** o script é `scripts/importar-leads-clickup.mts` (repo `area-membros-quirk-adm`, branch `feat/onboarding-administrativo`), lógica testável em `src/lib/crm/importClickUp/`. Rodar com `PAYLOAD_MIGRATING=true npx tsx scripts/importar-leads-clickup.mts [--seco] [--limite N]`. É **retomável** por `crm_leads.clickup_task_id` — pode matar e relançar à vontade, ele pula o que já entrou antes de qualquer chamada cara ao ClickUp.

- **`CLICKUP_TOKEN` no `.env`.** O ClickUp NÃO deixa copiar um token existente: o botão "Copiar" fica travado e só libera depois de "Regenerar" (que mata o token anterior) + reconfirmação de identidade pelo Google. As conexões do Make e do n8n são **OAuth**, não usam esse token — regenerar não as quebra (conferido via `connections_list`, `accountType: oauth`).
- **Escrita silenciosa:** Local API com `overrideAccess`, nunca `/api/crm-leads/entrada/<token>` — aquele caminho armaria SLA, roleta, e-mail e primeira mensagem de WhatsApp pra 3 mil contatos antigos.
- **Regras de negócio que valem lembrar:** `campanha = "Leads ClickUp"` fixo nos criados (e campanha vazia de lead pré-existente fica vazia — carimbar corromperia atribuição do BI); leads do Bruno Daspett (192 deles) foram pro Rodrigo Ikeda com nota de sistema; 152 entraram com placeholder `SEM-TELEFONE-<id>` + etiqueta `sem-telefone`; 37 ficaram sem dono porque o card não tinha responsável nenhum.

**Efeito colateral esperado, não é bug:** os leads entram com a data real de criação, então a home do comercial passa a contar milhares de "parados no funil" (`DIAS_LEAD_PARADO = 5`). Se incomodar, excluir os importados (`clickup_task_id is not null`) dessa conta.
