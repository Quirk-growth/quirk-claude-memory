---
name: gotcha_ia_crm_tres_chaves
description: "IA do CRM tem TRÊS chaves independentes (iaHabilitada, iaModoAusente, followUpAutomaticoHabilitado) + iaAtiva por lead; 'IA desativada' só é real com iaHabilitada=false. Incidente 14/09/2026"
metadata:
  node_type: memory
  type: reference
---

A IA do CRM ([[project_crm_quirk]]) NÃO tem um interruptor só. Incidente de 14/09/2026 no funil comercial (cliente 104): o time jurava que a IA estava desligada e ela seguiu escrevendo pra prospect — 15 mensagens em 7 dias.

**As chaves e o que cada uma de fato controla:**
- `clientes.iaHabilitada` — botão "Ativar/Desativar Agente de IA". É a chave GERAL. Desde o fix `ab95d0b`, desligada = zero resposta E zero follow-up.
- `clientes.iaModoAusente` — checkbox "Modo ausente". Liga/desliga `iaAtiva` em MASSA nos leads sem atendente.
- `clientes.followUpAutomaticoHabilitado` — reengajamento escrito pelo Claude.
- `crm-leads.iaAtiva` — por lead. A trava de resposta (`deveResponder` em `agenteIa.ts`) exige `iaHabilitada && lead.iaAtiva && sem atendente`.

**Why:** três defeitos somados deixavam mensagem sair. (1) Desligar o modo ausente só mudava a flag do cliente e deixava `iaAtiva: true` nos leads que o ligar tinha tocado — havia até um teste travando isso de propósito, sem justificativa. (2) O follow-up ignorava `iaHabilitada`. (3) O botão vermelho "Desativar Agente de IA" mostrava a AÇÃO e era lido como ESTADO — com a IA ligada, parecia desligada.

**How to apply:** quando alguém disser "a IA está desligada mas mandou mensagem", NÃO confie no relato nem no botão: leia `ia_habilitada`, `ia_modo_ausente`, `follow_up_automatico_habilitado` do cliente e conte leads com `ia_ativa=true`. Contenção rápida e reversível: salvar os ids e `update crm_leads set ia_ativa=false` no cliente. Autoria das mensagens fica em `crm_mensagens.autor_tipo` ('ia' | 'humano' | 'lead') — filtrar por ele mostra o que a IA realmente escreveu.
