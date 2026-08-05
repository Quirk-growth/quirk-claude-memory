---
name: project_onboarding_pos_assinatura
description: "Onboarding pós-assinatura na área de membros: contrato assinado → grupo WhatsApp + pasta Drive, com squad selecionável e mensagens editáveis"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-05T14:00:29.616Z
---

Automação **pós-assinatura** trazida do Make (cenário 3570588) pra dentro da [[project_area_membros_quirk]] (ago/2026). Gatilho: **webhook do Autentique** — quando o contrato vira `assinado`, dispara `executarOnboarding(payload, clienteId)` (não-bloqueante) que, idempotente:
- **Grupo WhatsApp** (instância "Comercial Quirk" da UAZAPI — env `UAZAPI_COMERCIAL_TOKEN`, base `UAZAPI_BASE_URL`): cria "{1º nome do cliente} | Quirk Growth", adiciona squad + cliente, promove squad a admin, foto (logo), descrição "Link Google Drive: {pasta}", boas-vindas no grupo, DM do convite pro cliente. Lib `src/lib/onboarding/whatsappGrupo.ts`.
- **Pasta Drive** (mesmo OAuth dos contratos [[reference_contratos_autentique]], env `GOOGLE_CLIENTES_FOLDER_ID`=`108-zggH-ZSz9-phrcoO0hPBgCAylh4jB` "Clientes Quirk"): pasta "{Razão Social}" → subpastas CRM/Material/Copys/Anúncios → copia só "Acessos" (`1KAgLzzu…`) → compartilha com o email do gestor. Lib `src/lib/onboarding/drive.ts`. Rastreio no cliente (`onboardingStatus`, `grupoWhatsappJid`, `pastaDriveId`…) + botão "Reprocessar" no hub.

**Squad:** bloco na aba Cadastro do cliente (`clientes.squad` hasMany→users), pré-marca o **squad padrão** (`users.squadPadrao`: Renan/Chaiene/Anna) + os gestores do cliente; desmarcável. Cada membro precisa de `users.telefone`.

**Mensagens editáveis:** global `mensagens-onboarding` (aba em /admin/contratos) com nomeGrupo/mensagemGrupo/mensagemClienteDM + **logoGrupo** (upload, foto do grupo; sobe em Configurações). Placeholders `{{primeiroNome}}`, `{{razaoSocial}}`, `{{linkPasta}}`, `{{linkGrupo}}`.

**Gotchas:** (1) grupo SÓ via UAZAPI — a WhatsApp Cloud API oficial não cria grupos. (2) `normalizarTelefone` prefixa **55** em número BR de 10-11 dígitos (os cadastrados sem DDI funcionam sozinhos). (3) foto do grupo é base64 PNG (`src/lib/onboarding/logoQuirk.ts` = fallback "Q"; override pelo upload do global). Spec/plano em docs/superpowers/. Envs `UAZAPI_COMERCIAL_TOKEN` + `GOOGLE_CLIENTES_FOLDER_ID` no Render.
