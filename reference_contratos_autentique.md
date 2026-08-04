---
name: reference_contratos_autentique
description: "Contratos na área de membros: geração via Google Docs (OAuth) + assinatura Autentique; envs, pasta Drive, placeholders"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-04T21:29:51.382Z
---

Gestão de contratos via **Autentique** na área de membros ([[project_area_membros_quirk]]), escolhida a **Opção C** (completa: sync + webhook + painel), jul/2026.

**API Autentique:** GraphQL em `https://api.autentique.com.br/v2/graphql`, header `Authorization: Bearer <token>`, rate limit 60/min. Query `document(id){ name created_at files{original signed} signatures{ name email signed{created_at} rejected{created_at} viewed{created_at} } }`; `documents(limit,page){total data{...}}`. Webhook: POST com `event.type` + `event.data.object`, validado por HMAC-SHA256 no header `X-Autentique-Signature` (secret = do endpoint). A conta tem ~386 documentos; os nomes trazem o nome do cliente (dá pra casar por busca).

**Token:** `AUTENTIQUE_TOKEN` — Renan cola no Render (Environment). O token foi compartilhado no chat uma vez → **recomendei rotacionar** e usar um novo só no Render. Local (.env gitignored) só pra validação.

**Feito — Opção C completa:** (1/2, 7d621d9) cliente GraphQL `src/lib/autentique/api.ts` (buscarDocumento/listarDocumentos + `statusDe`); coleção `contratos` com endpoints `/vincular` `/buscar` `/atualizar` `/desvincular` (gate ehSupervisorOuAcima); seção "Contratos (Autentique)" na aba Cadastro. DDL `scripts/ddl/2026-07-30-contratos.sql` (com lock) aplicado. (3/4, 204060d) `syncContratos` em lote plugado no cron diário; webhook `src/app/api/autentique/webhook/route.ts` valida HMAC-SHA256. (5, ea61cb8) painel `/admin/contratos` (ListaContratosView, item no menu Administrativo).

**Env que Renan precisa pôr no Render:** `AUTENTIQUE_TOKEN` (API) e `AUTENTIQUE_WEBHOOK_SECRET` (validação do webhook) + configurar a callback URL `https://membros.quirkgrowth.com.br/api/autentique/webhook` no painel Autentique. Sem esses, vincular/sync/webhook ficam inertes (erro amigável), o resto funciona.

**Geração de contratos (Google Docs, ago/2026):** trouxemos pra dentro da plataforma o fluxo que era do Make — 3 etapas **Gerar → Revisar → Enviar** no hub do cliente (gate `ehGestaoFinanceira` = admin/administrativo; vendedor tbm alcança). Motor `src/lib/contrato/googledocs.ts`: duplica um **Google Doc modelo** → arquiva a cópia na pasta de contratos → substitui placeholders (`docs.batchUpdate replaceAllText`) → exporta PDF (`drive.files.export`) → manda pro Autentique (`criarDocumento` multipart + `assinarDocumento` = Quirk auto-assina). Coleção `modelos-contrato` agora tem campo **`googleDocId`** (URL/ID do Doc), não mais `corpo` Lexical — aba "Modelos de contrato" em `/admin/contratos`. Modelo id 1 → template `1QtyrZDtorwxxY1eMNevnYMgXs4et3QdU2vqy82-AbKM` ("Make - Contrato Quirk").

**Auth = OAuth, NÃO service account.** Uma SA **não tem cota de Drive própria** → não consegue criar/copiar arquivos em My Drive (erro 403 `storageQuotaExceeded`); só lê/exporta. Então usamos OAuth (igual o Make): OAuth client "Contratos - Area de Membros" (tipo Desktop) no projeto GCP **quirk-painel-ads**, client_id `102402168523-s0ass8oak86lqpb1ldgh7d27fc6rv8kq...`; refresh token da conta **renan.reeal@gmail.com** (Gmail pessoal, dona do Drive de contratos e a mesma que o Make usa — NÃO é contato@quirkgrowth). Tela de consentimento External + Em produção → refresh token não expira. Envs no Render: `GOOGLE_OAUTH_CLIENT_ID` / `GOOGLE_OAUTH_CLIENT_SECRET` / `GOOGLE_OAUTH_REFRESH_TOKEN` (+ opcional `GOOGLE_CONTRATOS_FOLDER_ID`). Pra regerar o refresh token: fluxo loopback local (scopes drive+documents, `access_type=offline&prompt=consent`).

**Pasta de destino:** "* Contratos Clientes *" = `1pWqnKYEsPe28ysaCaOFlDJleDserACoC` (Drive da renan.reeal@gmail.com). Cópia arquivada nomeada "Contrato Quirk Growth | {razão social}". Docs nativos do Google não contam cota; o PDF exportado não é salvo no Drive (só vai pro Autentique). Prévia (`/previa-contrato`) é efêmera: gera e **apaga** a cópia. Envio (`/gerar-contrato`) arquiva a cópia + idempotência (409 se já há contrato pendente/parcial/assinado).

**Placeholders do template (padrão Make, `{{...}}`):** NOME DA EMPRESA DO CLIENTE, CNPJ DO CLIENTE, ENDEREÇO COMPLETO DO CLIENTE, NÚMERO DA CASA, CEP, CIDADE, ESTADO, E-MAIL DO CLIENTE, E-MAIL FINANCEIRO DO CLIENTE, PLATAFORMAS, VALOR MENSAL, DIA DO PAGAMENTO, Vigência, DATA EX. Gotchas em `montarPlaceholders` (variaveis.ts): `{{VALOR MENSAL}}` entra **cru em milhar** ("1.500") porque o template já escreve "R$ …,00"; `{{Vigência}}` vira "12 meses"; e-mail financeiro cai pro e-mail do cliente se vazio.

**Puppeteer removido:** a 1ª tentativa gerava o PDF com Chromium/Puppeteer, mas **não roda no plano free do Render** (512MB, libs nativas faltando) — daí o pivô pro Google Docs. `pdf.ts` + dep `puppeteer` + `PUPPETEER_CACHE_DIR` foram removidos.
