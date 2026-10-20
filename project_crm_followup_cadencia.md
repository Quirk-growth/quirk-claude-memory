---
name: project_crm_followup_cadencia
description: "Follow-up do CRM (área de membros) só existe como cadência em DIAS, uma mensagem por fase, sem padrão nenhum — e a flag followUpAutomaticoHabilitado é zumbi"
metadata:
  node_type: memory
  type: project
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-10-19T18:33:21.560Z
---

Follow-up do [[project_crm_quirk]] depois da limpeza de **19/10/2026** (commit `0aa3e7c5`, no ar; a remoção do legado veio antes, em `70483ca4`).

**Só existe UM follow-up: a cadência personalizada por cliente.** Descontinuados pelo Renan — o legado de 3 mensagens de IA em 1h/6h/23h ("experiência ruim com o lead") e o "Reengajamento de leads parados" (nunca disparou em produção).

**Modelo (`src/lib/crm/followUpCadencia.ts`):** cada FASE é UMA mensagem (não existe mais `quantidade`), com `intervaloDias` **inteiro** de 1 a 365 — fração é recusada de propósito, porque 0,5 dia seria a cadência de horas voltando pela porta dos fundos (risco de bloqueio do número). Limites por cliente: `{ maxMensagens, intervaloMinimoDias }`, teto global 30 mensagens / 1 dia. O PLANO gravado no lead (`crm_leads.follow_up_plano`) segue em HORAS (`offsetHoras` absoluto) — é snapshot: mudar a cadência não altera sequência em andamento.

**Ciclo de vida (20/10/2026, commit `d2baecc6`): conversa viva PAUSA.** O
gatilho de `crm-mensagens` trata os dois lados — ENTRADA do lead e SAÍDA nossa
(humano ou IA reativa) — e PAUSA em vez de cancelar. Antes a entrada do lead
**não fazia nada** (ele respondia e seguia recebendo "por que você sumiu?"; só
parava se nós mandássemos algo, e a IA ligada escondia o defeito respondendo por
nós), e a saída APAGAVA etapa/plano, tornando "retomar" impossível. Pausa NÃO
precisa de campo novo: cancelar e concluir apagam o plano, então "inativo COM
plano guardado" é pausa — é o que `estaPausado` lê. Entrada nunca ARMA (só a 1ª
saída nossa arma).

**`/follow-up` (CrmMensagens) tem 3 ações**, e ausência de ação = `pausar` (o
que o botão da tela manda): `pausar`, `retomar` (re-ANCORA o prazo —
`dadosRetomarFollowUp`: os offsets são absolutos desde o início, então retomar
sem re-ancorar faria tudo sair de uma vez ou morrer no teto de atraso) e
`iniciar` (arma do zero com a cadência atual do cliente, pra lead que nunca
entrou ou já concluiu). Decisão do Renan: **retomada só na mão**, nunca
automática por tempo quieto — coerente com ele ter matado o reengajamento.
Em Conversas há três estados: 🔄 ativa, ⏸ pausada (com "Retomar") e sem cadência
(botão "Iniciar follow-up" dentro da conversa).

**Regra de ouro: não existe padrão pra cair em cima.** Cliente sem cadência salva não arma; lead sem snapshot válido tem a sequência ENCERRADA no envio. Qualquer `?? PLANO_LEGADO` que aparecer é o bug voltando.

**`followUpAutomaticoHabilitado` é flag ZUMBI** — continua no banco (não foi dropada) e **nada** deve lê-la. Três defeitos vivos saíram exatamente disso: ligar o follow-up exigia ela (115 clientes ativos não conseguiam ligar), a trava de ENVIO consultava ela (armava e nunca enviava), e o envio caía no plano legado sem snapshot. Quem manda hoje é `followUpPersonalizadoAtivo(cliente)` + `iaHabilitada`.

**Desligar `followUpPersonalizadoHabilitado` PARA as sequências em andamento** (precedente do incidente 104: interruptor tem que desligar). **Canal oficial (`cloud_api_hibrido`) não tem follow-up automático** — fora da janela de 24h a Meta recusa texto livre.

**Aviso de cadência pesada** (`avaliarRiscoCadencia` + `AVISO_META_BLOQUEIO`): amarelo em 2 dias seguidos / 3 na 1ª semana / 6 no total; vermelho em 3 / 4 / 10. É aviso, não bloqueio. O texto não inventa número da Meta: a conta cai por **reclamação** (bloqueio/denúncia derrubam a qualidade do número), não por volume.

**Pra saber o estado real** (volátil — medir, não supor): `SELECT nome, follow_up_fases, follow_up_personalizado_habilitado FROM clientes WHERE follow_up_fases IS NOT NULL OR follow_up_personalizado_habilitado` e `SELECT count(*) FROM crm_leads WHERE follow_up_ativo`. Em 19/10/2026 era: nenhum cliente com cadência salva, nenhum lead armado, só riccelly com a flag ligada. Os campos são `json` — trocar o formato das fases NÃO pede DDL.

Pendência conhecida: `HubFollowUpAtivarButton` + endpoint `/follow-up-ativar` ficaram sem uso (escreviam a flag zumbi); a tela do comercial interno perdeu o interruptor mas o endpoint segue de pé.
