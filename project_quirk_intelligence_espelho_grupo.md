---
name: project-quirk-intelligence-espelho-grupo
description: "Espelho das conversas do bot num grupo do Telegram com Tópicos (um tópico por colaborador): código NO AR desde 15/10/2026 (e1d60e5a, DDL antes) mas INERTE até o Renan criar o grupo e definir QI_ESPELHO_GRUPO_ID; hoje as cópias ainda caem no privado dele"
metadata:
  node_type: memory
  type: project
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-10-15T18:37:11.142Z
---

**Pedido do Renan (15/10/2026):** ver a conversa de CADA gestor separada (opção B de três: A tela no Hub, B grupo com tópicos, C responder ao gestor). Escolheu B. Grupo privado só com o **Renan e a Chaiene** (ela é `administrativo`, sem vínculo no bot); Anna Silva e Phillipe (supervisores, vinculados) e o Gerson (gestor) NÃO entram no grupo por enquanto — as conversas de Anna e Phillipe SERÃO espelhadas no grupo (têm tópico próprio), visíveis só a quem estiver no grupo.

**No ar:** commit único `e1d60e5a` na main (15/10, "Deploy live" ~15:35 BRT). DDL aplicado em produção ANTES: `espelho_grupo_id varchar` e `espelho_topico_id numeric` (nulas) em `telegram_vinculos`, conferido por introspecção só-leitura; 5 vínculos, 0 chats de grupo. Smoke: login 200, rotas protegidas 403 (6x), `/api/telegram-vinculos` logado = 200 com os 5 docs. **Falta o Renan ligar** (ver abaixo). Para o estado de hoje: `git ls-remote origin main` e a variável no Render.

**Como funciona (docs/operacao/quirk-intelligence-espelho.md, seção "Modo grupo"):** com `QI_ESPELHO_GRUPO_ID` (id do supergrupo, `-100…`), a cópia de cada colaborador NÃO-admin vai pro tópico dele ("Nome (papel)"), criado na 1ª conversa e guardado no vínculo. Qualquer falha de ENTREGA cai no DM aos admins (a cópia não se perde); falha só de GRAVAR o tópico não cai no DM (fica em memória e regrava na próxima). Prazo geral de 10 s, piso de 1,5 s para chamar o Telegram, ordem criar → enviar → gravar. Tópico apagado é recriado uma vez. Conversa de admin segue o caminho antigo (DM aos outros admins). `QI_ESPELHO=off` desliga tudo; apagar `QI_ESPELHO_GRUPO_ID` volta ao privado. O webhook agora IGNORA mensagens de chats não-privados (incluindo `/start <token>` em grupo, que antes vinculava o chat do grupo ao usuário) — só `/grupo` de admin vinculado responde.

**O que o Renan precisa fazer (nesta ordem):** (1) criar um grupo privado no Telegram, ligar **Tópicos** nas configurações ANTES de tudo (o id muda quando o grupo vira supergrupo); (2) adicionar a Chaiene; (3) "Add members" só para admins, sem link de convite, nunca público; (4) adicionar o bot como administrador com **Gerenciar tópicos**; depois no BotFather `/setjoingroups` → Disable; (5) mandar `/grupo` no grupo (como membro-admin, NÃO como admin anônimo) e copiar o id (`-100…`); (6) no Render, `QI_ESPELHO_GRUPO_ID=<id>` com **Save and deploy** (a tela Environment substitui a lista inteira — conferir `DATABASE_URI`/`PAYLOAD_SECRET` depois); (7) um colaborador NÃO-admin (Gerson/Anna/Phillipe) manda uma mensagem ao bot → conferir o tópico no grupo, a linha em `telegram_vinculos` (`espelho_topico_id` preenchido) e que os logs não têm "grupo falhou" nem "topico criado mas nao gravado".

**Limitações documentadas:** tópico vazio a mais pode aparecer se o Telegram aceitar um pedido que abortamos (apagar à mão); cópia dobrada (grupo + DM) é possível nessa mesma janela; cópia >4096 unidades pode ficar partida (parte no grupo, completa no DM); nomes de tópico não acompanham renomeação/troca de papel; fila em memória (uma instância); pior caso de latência do espelho ~26 s; `/grupo` não olha `is_forum` (grupo sem Tópicos falha no 1º envio e cai no DM). Sem registro persistente nem busca por dia (isso seria a opção A, não pedida). Opção C (responder ao gestor) não feita. Não confirmei nada contra a Bot API real: as descrições de "tópico inválido" vêm do conhecimento da API — o 1º uso real valida.

Relacionados: [[project-quirk-intelligence-auditoria-conversas]] (o espelho no privado, que continua o padrão sem a variável) e [[project-quirk-intelligence-criar-tarefa]]. Branches locais `feat/qi-criar-tarefa` e `feat/qi-espelho-topicos` ficaram redundantes (conteúdo igual ao da main).
