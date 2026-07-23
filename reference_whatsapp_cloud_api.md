---
name: reference_whatsapp_cloud_api
description: "Migração Auto Ads uazapi→WhatsApp Cloud API oficial; IDs, credenciais, arquitetura e gotchas do n8n"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
---

Em 2026-07-02 o [[project_quirk_auto_ads]] migrou de uazapi (não-oficial) pra **WhatsApp Cloud API oficial**. Motivo: número foi registrado no oficial (sai da uazapi — um número só vive numa plataforma). **VALIDADO EM PRODUÇÃO** no mesmo dia: conversa real de 2 turnos pelo WhatsApp (Renan como cliente em_onboarding) — IA respondeu e mensagens foram entregues com wamid.

**IDs**
- Phone Number ID: `1320571937797802` (número +55 11 95213-6200 "Quirk growth", status CONNECTED/CLOUD_API)
- WABA ID: `2088588845203405`
- App: "Quirk Auto Ads" (`3005061513218441`) — business verificado
- Graph API: `https://graph.facebook.com/v25.0`

**Credenciais / segredos** (fora do git)
- Token permanente (System User "Quirkautoads", `expires_at:0`): `~/.config/n8n-quirk/wa_cloud_token.txt`
- Phone id / WABA id: `~/.config/n8n-quirk/wa_phone_number_id.txt` / `wa_waba_id.txt`
- Credencial n8n "Quirk WhatsApp Cloud" (Header Auth `Authorization: Bearer`): id `0tSLCLWrTWl9xSyO`
- Verify token do webhook: `quirk_wa_28efc561d81a16b3`

**Arquitetura** (scripts `scripts/e_01`..`e_04`)
- Workflow "WhatsApp Cloud Inbound" (`7vhoapaFk2zY8ptL`, path `whatsapp-cloud`): recebe webhook Meta → `route` (parseia GET verify + POST mensagem) → repassa pro webhook do principal no formato `{message:{type,text,id,from,sender_pn}, chat:{phone}}`.
- Principal "Quirk Auto Ads" (`fBUin1UPt5xJEp6g`, path `quirk-auto-ads`): entrada intocada; 12 envios de texto + download de mídia trocados pro Graph API.
- Gateway (`2ZnZqb4wFous4uEs`): welcome WhatsApp proativo DESLIGADO (exigiria template); email + status mantidos. Cliente fala primeiro → IA responde na janela de 24h.

**Gotchas do n8n (custaram tempo)**
- Webhook com `multipleMethods:true` tem **2 saídas** (GET=índice 0, POST=índice 1). PRECISA ligar as DUAS no `route`, senão POST (mensagem) não roteia.
- Update via API pública **não re-registra webhook**: fazer deactivate+activate depois de alterar.
- API pública rejeita `settings` extras (callerPolicy/binaryMode/availableInMCP): filtrar pro subconjunto permitido.
- Nós de IA enviam `JSON.stringify($json)` inteiro → campos auxiliares (`_msg_atual_user`) vazavam e a Anthropic dava 400 "Extra inputs are not permitted". Fix: filtrar chaves top-level que começam com `_`.

**Upload de mídia pro Meta (criativos) — gotcha crítico**
- O **code node do n8n NÃO faz multipart/form-data binário** (`formData` e Buffer cru dão HTTP 400; `require('form-data')` é bloqueado; `helpers.request` quebra). Só o **HTTP Request node NATIVO** (contentType `multipart-form-data` + parameterType `formBinaryData` + `inputDataFieldName`) envia arquivo binário.
- Padrão que funciona (validado): code baixa bytes com Bearer + `prepareBinaryData` → nó nativo faz o multipart. Fluxo do criativo: `meta_d3_prep` (code) → `meta_d3_upload` (nativo) → `meta_d3_creative` (code: poll vídeo + thumbnail obrigatória + adcreative). Vídeo CTWA exige `image_url` (thumbnail de `/{video_id}/thumbnails`).
- URL da Cloud API (lookaside) exige Bearer — `file_url` do `/advideos` NÃO funciona (Meta não autentica). Por isso baixa+sobe bytes.
- Adset CTWA (`destination_type:WHATSAPP`) exige a **Página com WhatsApp Business conectado**, senão erro 2446886.
- ⚠️ **Lição de operação**: NÃO rodar uploads pesados (vários MB × N) numa execução só de code node — derruba o n8n por OOM. Testar upload sempre leve (1 arquivo).

**Pendências pro go-live** (ver checklist na conversa): configurar callback URL no app Meta + assinar campo `messages`; garantir app em **Live mode** (senão só envia pra números de teste); teste real; aposentar uazapi. Persistência de mídia recebida (URLs da Cloud API são temporárias) é refino futuro.

**GOTCHA CRÍTICO — 9º dígito BR (debugado jul/2026):** o WhatsApp/Meta entrega `wa_id` de números BR ora COM ora SEM o 9 (ex: chega `554198443588`, cadastro Asaas guarda `5541998443588`). Isso quebrou em 2 camadas no workflow principal `fBUin1UPt5xJEp6g`: (1) `select_cliente` fazia match exato → cliente pago virava "desconhecido" e o fluxo morria calado no `respond_immediate`; (2) depois de achar, ~15 nós (conversas/campanhas/estado) usam `telefone_normalizado` e a FK `conversas/campanhas → clientes.telefone` violava quando a forma não batia com a PK. **Solução (commits 95e945c + 5ee5310):** o nó `normalize_phone` canonicaliza `telefone_normalizado` SEMPRE pra forma COM o 9 (regra `55+DDD+[9]+8`), e mantém `telefone_variantes` (com/sem 9) que o `select_cliente` casa via `WHERE telefone IN (...)`. `clientes` guarda com o 9 (default do Asaas), então canonicalizar-pra-com-9 alinha tudo sem migrar dado. Também: `select_cliente.alwaysOutputData=true` (commit e311ba2) pra número desconhecido cair em not_found → `send_not_found` em vez de morrer em silêncio. Helper `scripts/_dump_prompt_vivo.py` extrai o prompt vivo (embutido no nó `build_agente_body`, campo jsCode→estavelBlock; o `prompts/agente_principal.md` está DESATUALIZADO — fonte da verdade é o nó).

**Onboarding auto-detecção (jul/2026, commits h_01..h_04):** o cliente NÃO precisa mais digitar o ID da Conta de Anúncios. `revisao_meta` detecta conta+página **por nome** entre os ativos SEM DONO (não presentes em `clientes.ad_account_id/page_id`) compartilhados na BM Quirk (`1612905538806887`) — match por `nome_cliente` + o que o cliente cita. Fluxo: cliente diz "terminei" → `<REVISAO_REQUEST/>` → detecta → se 1 conta+1 página → `precisa_confirmar` → grava candidatos em `estado_json.candidatos` + `etapa=aguardando_confirmacao` → manda "confirma? SIM". Cliente responde afirmativo → `if_aguardando` (no em_onboarding) roteia direto pra revisão → `revisao_meta` bloco CONFIRMA-ATIVA (checa afirmativo pela msg ATUAL via `normalize_phone.mensagem_texto`) → valida acesso + atribui System User (`122093025345347834`) → ativa. 0/ambíguo → pergunta (nunca adivinha). 3 falhas → `estado_json.travado_onboarding=true` + alerta no WhatsApp do Renan (`5511980838409`, hardcoded no nó `send_alerta_humano`). Nós novos: load_assigned_assets, load_estado_revisao, load_estado_onb, if_aguardando, if_precisa_confirmar, persist_candidatos, send_confirma_msg, incr_falhas, if_escape, mark_travado, send_alerta_humano, send_time_cliente. Estado em `auto_ads.conversas.estado_json`. **Validado:** confirma→ativa end-to-end (replay real); detecção isolada; contador SQL. NÃO testado end-to-end o caminho detect→precisa_confirmar com ativo real sem-dono (todos atribuídos) — validado por partes (wiring+SQL+lógica isolada).
