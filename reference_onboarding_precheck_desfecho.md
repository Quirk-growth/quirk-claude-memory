---
name: reference_onboarding_precheck_desfecho
description: "Auto Ads: precheck de prontidão no onboarding (trava até verde) + desfecho determinístico da criação (fim do 'validando e subindo' falso)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-27T20:48:24.515Z
---

Dois ajustes de robustez no [[project_quirk_auto_ads]] (workflow `fBUin1UPt5xJEp6g`), deploy 24/ago/2026, norte do Renan: "tudo precisa ser avisado e revisado pra ferramenta ser o mais autônoma possível". Specs/plano em `quirk_auto_ads/docs/`.

**#2 — Desfecho determinístico da criação (`scripts/e_18`):** bug real (Aquilino): a criação falhou no `meta_d2_adset` (página sem WhatsApp), o `check_meta_results` diagnosticou certo (`ok:false, classe:'dado', motivo:...`), MAS o `switch_resposta_meta` roteia `ok=false` pro `build_agente_body → agente_principal` (IA), que respondeu **"Validando e subindo, te aviso quando estiver no ar"** — mensagem falsa, motivo engolido. Causa: `build_agente_body` só tinha bloco determinístico p/ `classe==='verificacao'`. Fix: bloco novo p/ `check_meta_results.ok===false && classe!=='verificacao'` que injeta o `motivo` como diagnóstico de ação + PROÍBE "subiu/subindo/te aviso". Sucesso já era determinístico (`build_resposta_ativa`). Verificado na Ignite com falha controlada (page quebrada → diagnóstico certo, sem "te aviso").

**#1 — Precheck de prontidão no onboarding (`scripts/e_19`, decisão "trava até verde"):** na confirmação do onboarding, entre `if_revisao_ok`(true) e `update_cliente_ativo`, roda `load_token_precheck → precheck_prontidao → switch_prontidao → [verde: update_cliente_ativo | pendência: record_prontidao → send_checklist_prontidao]`. Checklist: conta ativa · saldo>0 · fanpage · WhatsApp-na-página · verificação. Verde → ativa; pendência → NÃO ativa (mantém `em_onboarding`), grava coluna `prontidao` jsonb e manda checklist ao cliente (quem resolve o quê). **Sem alerta ativo à Quirk** (decisão Renan: "só registra + cliente vê"). Colunas novas: `auto_ads.clientes.prontidao` e `.aquecimento` (jsonb). Fail-open só se faltar token.

**Truque validado:** UM `validate_only` de adset CTWA (`OUTCOME_LEADS` + `optimization_goal:CONVERSATIONS` + `destination_type:WHATSAPP` + `promoted_object.page_id`) pega de uma vez **WhatsApp-na-página (subcode 2446886)** E **verificação de anunciante (3858634)**. Ver [[reference_verificacao_anunciante_meta]]. Saldo via `funding_source_details.display_string` (ver [[reference_saldo_whatsapp]]).

**⚠️ GOTCHA n8n (custou um bug crítico quase-em-prod):** nó **Postgres executeQuery SUBSTITUI o item** de saída (não passa o input adiante). Se você põe um `load_token_*` (Postgres) ANTES de um code node, o `$input.first().json` do code vira `{o_resultado_do_postgres}`, perdendo os campos do item original. Um code node depois de Postgres deve ler os campos originais por `$('no_de_origem')`, não por `$input`. (Aqui: `precheck_prontidao` lê `inp` de `$('if_revisao_ok')`, não de `$input` — senão gravaria `ad_account_id=undefined` no cadastro na ativação.)

Ordem executada: #2 → #1 → (retomar aquecimento, plano `2026-08-24-aquecimento-conta-plan.md` Tasks 3–7). Ver [[reference_onboarding_confirmacao_flow]], [[reference_saldo_whatsapp]] e [[reference_aquecimento_conta]].

**⚠️ Review "TUDO responde, nada trava" (24/ago, `e_24`+`e_25`):** o Renan exige que NENHUMA ação termine sem resposta. Dois achados/fixes:
- **CONFIRMAR caindo na rota de gestão (`e_24`):** `switch_a_ou_b` roteava por `estado.gestao != null` — mas gestão ATIVA chega lá por `process_gestao_step` (que pula `classify_intent`), enquanto criação chega por `classify_intent` CONFIRMAR. Resíduo de `estado.gestao` (ex: STATUS anterior) fazia o CONFIRMAR de criação ir pro `execute_gestao_action` (sem ação válida) e **morrer calado**. Fix: nó `decide_a_ou_b` (try/catch) — se `classify_intent.intent` é CONFIRMAR/SUBIR_DENOVO → rota CRIACAO sempre; senão respeita `estado.gestao`. `switch_a_ou_b` roteia por `$json.rota`.
- **Switches sem fallback = silêncio (`e_25`):** auditoria de grafo (todo ramo alcança um nó de envio?) achou 3 switches sem `fallbackOutput`: `switch_type` (áudio/figurinha/localização → mudo!), `execute_gestao_action`, `switch_acao_gestao`. Cada um ganhou `fallbackOutput:'extra'` → nó de mensagem. Método de auditoria: nós terminais que não são envio + ramos de switch/if vazios ou que não alcançam `/messages`. `normalize_phone` retorna `[]` só no dedup de wamid (intencional). `if_cadastrado` é órfão inalcançável.
