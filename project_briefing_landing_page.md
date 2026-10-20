---
name: project-briefing-landing-page
description: 3º briefing do Hub (aba Briefing e Recursos): Landing Page, 24 perguntas em 8 seções extraídas de 10 copys de LP; NO AR 20/10/2026 (26265e32, DDL em prod antes); molde do Social (token 30 dias, rate limit, isolamento)
metadata:
  node_type: memory
  type: project
---

**Pedido do Renan (20/10/2026):** briefing de Landing Page para os clientes, na aba "Briefing e Recursos" do Hub. Base: revisão de 10 copys de LP do Drive (Luham, VLL, Guaíra, Árvore do Sorriso, Ufos, estética Bruna, Holly Coffee, Promotoboy, Importadores de Elite, Full Expresso; pastas `12vzfQJ1MkpgZnUAGJixy19iAS4nHTOIs` e `197HVrJ1WbjDQ5LLlK2QZhs_hKsE2YY_r`). Padrão das LPs: dobras fixas (hero → apresentação → para quem é/dores → serviços → prova social → como funciona → autoridade humana → CTA → FAQ → rodapé); o gargalo são os materiais (as copys têm "[espaço para depoimentos/foto/vídeo]").

**No ar (26265e32):** 24 perguntas obrigatórias em 8 seções (negócio e objetivo 5, oferta 5, público e dores 4, posicionamento 2, prova e autoridade 3, FAQ 1, tom e restrições 2, materiais 2), cortadas de 42 (limites: objetivo+ação fundidos, logo+pasta fundidas, dados do rodapé vêm do cadastro, campos do formulário a Quirk define). Modelo em `src/lib/briefing/modelosLanding.ts`; campos `briefingLanding`/`briefingLandingToken`/`briefingLandingTokenEm` em `clientes` (DDL `scripts/ddl/2026-10-20-briefing-landing-clientes.sql`, aplicado em prod ANTES e conferido contra o banco de teste); Hub: `HubBriefingLandingForm`; cliente: `/briefing-landing/[token]` + menu; rascunho parcial (só conta como preenchido com as 24, decidido no servidor); rate limit/validade iguais ao Social; vendedor do cliente NÃO lê o campo pela API.

**Decisões aceitas / pendências:** reenvio sobrescreve (duas abas/pessoas ao mesmo tempo podem apagar respostas — aceito na fase 1; se virar problema, merge por pergunta + versão); fase 2 = módulos por nicho (imobiliário/lançamento: nome, tipologias, valor, previsão de obra; saúde/estética; varejo; infoproduto). Minors do revisor: pergunta 4 "Outro" sem campo para descrever; token sem data nunca expira (campo novo, só via PATCH manual); vendedor ainda lê `briefing`/`briefingSocial`/`briefingSocialToken` pela API crua (gap antigo, follow-up).

Relacionados: [[project-area-membros-quirk]], [[feedback-isolamento-dados-clientes]], [[feedback-push-squash-base-velha]] (rebasear + `diff --stat` antes do push — seguido neste deploy).
