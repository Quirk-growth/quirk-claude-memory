---
name: reference-dev-local-cookie-seguro
description: "GOTCHA — npm run dev local (área de membros) não guarda sessão de login: cookie Secure exige HTTPS, localhost é HTTP puro"
metadata:
  node_type: memory
  type: reference
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-14T12:51:04.566Z
---

Login em `http://localhost:3000/admin/login` parece funcionar (redireciona pro `/admin`, mostra o shell com avatar), mas `GET /api/users/me` volta `{"user":null}` — nenhuma sessão real fica ativa. Toda view/endpoint com gate de papel (`PODE.includes(user.role)`) mostra "Acesso restrito", mesmo pra conta admin.

**Causa**: o cookie de sessão do Payload é configurado como `Secure`, que só é aceito pelo navegador em HTTPS. `npm run dev` local roda em HTTP puro (`localhost:3000`), então o cookie nunca é salvo de fato — mesmo que o POST de login em si tenha ido com sucesso.

Confirmado batendo em `/api/users/me` (retorna `user:null`) e reproduzindo em MAIS de uma tela (inclusive `comercial-mensagens`, feature antiga já em produção — não é bug de código, é o ambiente). A mesma tela funciona normal em `https://membros.quirkgrowth.com.br` (produção real, HTTPS).

**Consequência prática**: verificação visual/browser de mudanças de UI **não é viável rodando `npm run dev` local** neste projeto, mesmo que o `DATABASE_URI` local já aponte pra produção (ver [[reference_modeloscontrato_corpo_orfa]]). Views/endpoints autenticados sempre vão parecer quebrados localmente por esse motivo, não por bug real.

**Caminho alternativo pra verificação visual**: fazer o deploy primeiro e verificar direto em produção (HTTPS de verdade, cookie funciona), em vez de tentar validar no `localhost` antes de subir.
