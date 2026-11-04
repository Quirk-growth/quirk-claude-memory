---
name: gotcha-lead-ads-chave-do-rotulo
description: No Lead Ads a CHAVE do campo é derivada do rótulo que o gestor digitou; lista fixa de chaves descarta lead em silêncio
metadata:
  node_type: memory
  type: project
  originSessionId: 01cb3caa-2faf-418f-851d-4b7662439309
  modified: 2026-11-04T22:11:43.567Z
---

Na Meta, a chave de cada campo do formulário é derivada do **rótulo digitado pelo gestor**, não é um nome canônico. "Número do WhatsApp" chega como `número_do_whatsapp`. Lista fixa de chaves é, portanto, aposta no idioma.

Incidente 04/11/2026: 35 leads de 3 clientes (Altto Design id 12, Aline Cangiani id 153, Mirae Imóveis id 146) descartados, o mais antigo com 29 dias. O telefone caía em `respostas` como pergunta de qualificação, o lead chegava sem telefone, e `criarOuAtualizarLead` devolvia `invalido` — registrado com `logger.info`, idêntico a sucesso. Corrigido em `62f38e28`: mapeamento em duas passadas em `src/lib/crm/meta.ts` (chaves canônicas primeiro, depois padrão `whats|phone|fone|celular` sem acento, exigindo que o VALOR normalize como telefone), e descarte agora sai como `warn` listando as chaves recebidas.

**Como diagnosticar de novo (o caminho que funcionou):**
1. `clientes.meta_lead_forms` tem os formulários conectados? (json, não entra no `where` do Payload)
2. `GET /{pagina}/subscribed_apps` com o token DA PÁGINA — `leadgen` inscrito?
3. `GET /{pagina}/leadgen_forms?fields=id,name,status,leads_count` — tem lead do lado do Meta?
4. `GET /{leadgen_id}?fields=field_data` e **olhar as chaves cruas**. Foi o passo que resolveu; eu primeiro li a chave errada (`phone_number`) e conclui que o lead não tinha telefone.
5. Cruzar por telefone contra `crm_leads` do cliente — parte dos leads pode já estar lá com `origem='whatsapp'`, sem as respostas e sem atribuição de campanha (a pessoa preencheu e depois mandou mensagem).

**Não é a fila.** Fila sem vendedor vinculado cai de volta em todos os ativos do cliente (`fila.vendedores.length > 0 ? fila.vendedores : [...ativos]`), e sem fila vai pra roleta geral. A fila nunca descarta lead.

**Dois buracos que seguem abertos** (Renan decidiu em 04/11 não recuperar os 35 e pediu só garantia daqui pra frente):
- A seleção de formulários no Hub é MANUAL e feita uma vez. Formulário novo de campanha nova não entra sozinho, e os leads caem no chão com "form não vinculado a nenhum cliente" no log. A Altto tem 5 formulários na página e 2 conectados.
- Ninguém compara os dois lados. Proposta feita e **não aprovada ainda**: reconciliação diária (leads no Meta × cards no CRM, por formulário) + alerta de formulário ativo com lead que ninguém conectou, no padrão de [[project_conversoes_meta_cliente]].

Criar lead **não** arma cadência de follow-up (`dadosArmarFollowUp` só é chamado pela ação `iniciar`, clique humano). Mas **manda e-mail**: `notificarLeadNovo`, travado só por `origem !== 'csv'` — então backfill em massa precisa de uma trava explícita. Ver [[reference_suite_testes_area_membros]].
