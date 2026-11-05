---
name: project-lp-a2-usucapiao
description: "LP de usucapião para o cliente A2 Arquitetura — converte por clique no WhatsApp, sem formulário; branch no GitHub, ainda não publicada"
metadata:
  node_type: memory
  type: project
  originSessionId: 82379061-5612-4773-b98e-38e0debf380e
  modified: 2026-11-05T11:19:22.686Z
---

LP de cliente para a **A2 Arquitetura e Construção**, vendendo Assistência Técnica
para Usucapião Judicial e Extrajudicial (ticket médio R$ 7.500). Criada em 05/11/2026.

Mora em `~/quirk-lps/a2-usucapiao/`, branch **`feat/lp-a2-usucapiao`** já no GitHub
(`Quirk-growth/LPs-quirk`). **Ainda não publicada em lugar nenhum.**

**Converte por clique direto no WhatsApp — sem formulário, sem webhook, sem backend.**
Decisão do Renan: a A2 sempre atendeu pelo botão do WhatsApp e esse procedimento fica.
Uma primeira versão tinha quiz de 5 etapas + webhook; foi removida (está no histórico
do git). Um workflow n8n chegou a ser criado e foi deletado sem nunca rodar.

A triagem que sobrou: **cada botão abre o WhatsApp com mensagem diferente** conforme a
seção (advogado / orçamento / risco / padrão), para a A2 separar o tipo de lead antes
de responder. Ver [[gotcha-wa-me-nao-dispara-pixel]] — o formato do link é crítico.

Eixo da copy: **risco de processo**, não entregável. Os 7 concorrentes de SP vendem
"planta e memorial" como item avulso; competir nesse plano perde por preço.

Plano de medição combinado: clique no WhatsApp é a conversão no navegador, e depois o
dado volta pelo **CRM via API de Conversões**, casando por telefone com hash. Os dois
são etapas diferentes do funil e **não** devem ser deduplicados.

**Pendências com a A2** (não inventar, tudo não confirmado): número de WhatsApp
(página usa 5511970382322), autoria de 4 dos 6 depoimentos, tempo de mercado
(apresentação de 2021 diz "18 anos"), e CAU/CREA — que não aparece em nenhuma página
do site deles.

Detalhes e auditoria do GTM do cliente em `a2-usucapiao/README.md` e `a2-usucapiao/GTM.md`.
