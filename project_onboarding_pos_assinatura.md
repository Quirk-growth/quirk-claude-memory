---
name: project_onboarding_pos_assinatura
description: "Onboarding pós-assinatura na área de membros: contrato assinado → grupo WhatsApp + pasta Drive, com squad selecionável e mensagens editáveis"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-05T15:00:58.388Z
---

Automação **pós-assinatura** trazida do Make (cenário 3570588) pra dentro da [[project_area_membros_quirk]] (ago/2026). Gatilho: **webhook do Autentique** — quando o contrato vira `assinado`, dispara `executarOnboarding(payload, clienteId)` (não-bloqueante) que, idempotente:
- **Grupo WhatsApp** (instância "Comercial Quirk" da UAZAPI — env `UAZAPI_COMERCIAL_TOKEN`, base `UAZAPI_BASE_URL`): cria "{1º nome do cliente} | Quirk Growth", adiciona squad + cliente, promove squad a admin, foto (logo), descrição "Link Google Drive: {pasta}", boas-vindas no grupo, DM do convite pro cliente. Lib `src/lib/onboarding/whatsappGrupo.ts`.
- **Pasta Drive** (mesmo OAuth dos contratos [[reference_contratos_autentique]], env `GOOGLE_CLIENTES_FOLDER_ID`=`108-zggH-ZSz9-phrcoO0hPBgCAylh4jB` "Clientes Quirk"): pasta "{Razão Social}" → subpastas CRM/Material/Copys/Anúncios → copia só "Acessos" (`1KAgLzzu…`) → compartilha com o email do gestor. Lib `src/lib/onboarding/drive.ts`. Rastreio no cliente (`onboardingStatus`, `grupoWhatsappJid`, `pastaDriveId`…) + botão "Reprocessar" no hub.

**Squad:** bloco na aba Cadastro do cliente (`clientes.squad` hasMany→users), pré-marca o **squad padrão** (`users.squadPadrao`: Renan/Chaiene/Anna) + os gestores do cliente; desmarcável. Cada membro precisa de `users.telefone`.

**Mensagens editáveis:** global `mensagens-onboarding` (aba em /admin/contratos) com nomeGrupo/mensagemGrupo/mensagemClienteDM + **logoGrupo** (upload, foto do grupo; sobe em Configurações). Placeholders `{{primeiroNome}}`, `{{razaoSocial}}`, `{{linkPasta}}`, `{{linkGrupo}}`.

**Gotchas:** (1) grupo SÓ via UAZAPI — a WhatsApp Cloud API oficial não cria grupos. (2) `normalizarTelefone` prefixa **55** em número BR de 10-11 dígitos (os cadastrados sem DDI funcionam sozinhos). (3) foto do grupo é base64 PNG (`src/lib/onboarding/logoQuirk.ts` = fallback "Q"; override pelo upload do global). (4) **DDL manual + snake_case do Payload:** o campo `mensagemClienteDM` virou coluna `mensagem_cliente_d_m` (cada MAIÚSCULA vira separador → `DM`=`d_m`), mas o DDL manual criou `mensagem_cliente_dm` → o SELECT do global quebrava (500) e o admin mostrava **"Nada encontrado"**. Lição: em coluna nova por DDL manual, checar a snake_case do Payload letra a letra (caps consecutivas = `_x_y`), ou extrair do banco de teste com push ON. Erro real fica nos logs do Render (o cliente só vê "Something went wrong"/"Nada encontrado"). Spec/plano em docs/superpowers/. Envs `UAZAPI_COMERCIAL_TOKEN` + `GOOGLE_CLIENTES_FOLDER_ID` no Render.

**3 BUGS MUDOS corrigidos em 11/ago (commits `cd10e38`, `ca1bd7b`, `4ca5f66`) — achados na Plannus (cliente 119: contrato assinado, grupo não abriu).** Os três quebravam TODO cliente novo e os três tinham TESTE VERDE cobrindo o comportamento errado:
1. **Formato da UAZAPI mudou** — devolve os campos NO TOPO, o código lia `data.group.JID`. O grupo ERA criado no WhatsApp e `criarGrupo` lançava "sem JID": a execução parava sem foto/descrição/admins/mensagem e sem gravar o JID → a próxima tentativa criaria grupo DUPLICADO. Hoje aceita os dois formatos. Diagnóstico: `POST /group/info` e olhar se a resposta tem `data` ou não.
2. **A logo configurada NUNCA era usada** — `resolverLogoBase64` fazia `fetch` na `url` do Payload, que é RELATIVA (`/api/media/file/...`, o fetch do Node recusa) e, mesmo absoluta, a rota exige sessão (403 anônimo, pelo escopo de leitura de Media). Caía no catch em silêncio e usava o ícone genérico. Agora lê do **R2 direto** (S3 client + `filename`) e LOGA quando cai no fallback.
3. **A DM de convite não saía com UMA ausência** — a condição era `participantes < convidados`, mas o número da PRÓPRIA instância conta como participante e empatava a conta. Só disparava com 2+ faltando. Como o WhatsApp barra ser adicionado por quem não é contato, **o convite é o caminho NORMAL** — era o que menos funcionava. Hoje `infoGrupo` devolve os `numeros` e a decisão é a PRESENÇA do telefone do cliente.
+ o passo do Drive agora espelha o link em `clientes.pastaArquivos` (aba "Links e assets" do hub), sem sobrescrever link posto à mão.

⚠️ **Recuperação de grupo já criado sem JID gravado:** pegar o JID em `GET /group/list`, gravar em `clientes.grupo_whatsapp_jid`, e rodar o onboarding de novo — ele pula a criação e completa o resto. A mensagem de boas-vindas NÃO reenvia nesse reprocesso (só sai quando o grupo nasce na mesma execução); mandar à mão se precisar.
⚠️ **Antes de afirmar que uma mensagem do global está vazia, LEMBRAR da coluna `mensagem_cliente_d_m`** (gotcha 4 acima). Em 11/ago consultei `mensagem_cliente_dm`, veio vazio e eu concluí que o texto não existia — estava configurado o tempo todo, e o diagnóstico falso quase escondeu o bug 3.
