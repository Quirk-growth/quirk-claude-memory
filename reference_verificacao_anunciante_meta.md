---
name: reference_verificacao_anunciante_meta
description: "Erro Meta subcode 3858634 (verificação de anunciante) que impede clientes de subir campanha no Auto Ads"
metadata:
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
  modified: 2026-08-06T18:58:41.839Z
---

No [[project_quirk_auto_ads]], clientes podem não conseguir subir campanha com o erro Meta **subcode 3858634** (`compliance_section`, "O anunciante está ausente / Forneça um anunciante verificado para veicular nas localizações selecionadas"). Falha no `meta_d2_adset` (criação do conjunto).

**Causa raiz (05/ago/2026, caso Chrystian):** NÃO é o nosso targeting nem a campanha. Testado com `validate_only` na conta dele: **com E sem** os campos DSA (`dsa_beneficiary`/`dsa_payor`) dá o mesmo 3858634 → é exigência **da CONTA**: o negócio dono precisa completar a **verificação de anunciante** (identidade) no Meta. A conta de teste **Ignite funciona** porque roda sob um BM verificado da Quirk (`BM - Contas de Anúncio Quirk 1`); a conta do Chrystian (`act_2254881678623214`) roda sob o negócio próprio dele ("Depaulacorretor"), **não verificado**. `verification_status` da API vem `None` nas duas — não é esse campo; é a verificação de anunciante/compliance, sem campo limpo na API.

**Fix de mensagem (commit e427703):** `check_meta_results` detecta 3858634 → `classe='verificacao'` + motivo claro; `build_agente_body` força mensagem determinística (proíbe "falha técnica", proíbe citar região/Perdizes, proíbe pedir reenvio/SUBIR DENOVO — só a verificação da conta resolve). O `rollback` (também no check_meta_results) apaga a campanha órfã da tentativa.

**Ação REAL (fora do sistema):** o dono da conta precisa concluir a verificação no **Gerenciador de Negócios → Central de Segurança → verificação de identidade do anunciante**. A Quirk pode fazer se tiver acesso admin ao BM do cliente. **Follow-up sugerido:** precheck no ONBOARDING (rodar um `validate_only` de adset) pra avisar o cliente que precisa verificar ANTES de montar campanha, evitando frustração.
