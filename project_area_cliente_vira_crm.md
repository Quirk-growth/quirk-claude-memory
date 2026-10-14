---
name: area-cliente-vira-crm
description: Área do cliente virou CRM (Mentorias aposentadas em 13/10/2026) e passou pro domínio crm.quirkgrowth.com.br via APP_URL_CLIENTE; Quirk Hub (/admin) e APIs ficam em membros.
metadata:
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-10-14T02:07:46.012Z
---

**Estado (13-14/10/2026, NO AR):** a área de membros do CLIENTE é um CRM. A home `/` só roteia (`lib/destinoInicial.ts`): member com CRM → `/crm`; sem CRM → `/relatorios` (ou `/conta` sem empresa); não-member → `/admin`. Menu sem "Mentorias"; CRM é o 1º item. A escolha de tela inicial foi removida de vez. Commits: `820c5343`, `0fd7dda2` (corte), `228f0f91` (prepara domínio).

**Domínio:** `crm.quirkgrowth.com.br` = área do cliente; `membros.quirkgrowth.com.br` = Quirk Hub (/admin), APIs, webhooks, formulários públicos por token. Mesmo serviço Render (`srv-d9a34jucjfls73969rp0`), dois hosts. O interruptor é a env `APP_URL_CLIENTE` (no Render, ligada). `APP_URL` continua = membros. Apagar `APP_URL_CLIENTE` desfaz tudo.

**Why:** Renan quer a área do cliente "toda voltada ao CRM", mais profissional e apresentável; o Hub (CRM interno) não muda de endereço, então nenhum webhook/integração precisou migrar.

**How to apply / armadilhas:**
- Redirects por host (`lib/redirectsDominio.ts` → `next.config.ts`) são calculados no BUILD: mexer em `APP_URL_CLIENTE` no Render dispara deploy (~4-5 min) e é isso que aplica.
- Link de e-mail segue o papel: member → `crm.`, equipe → `membros.` (`lib/urlCliente.ts`). Webhook, conexão e link de entrada de lead ficam em `APP_URL` (membros).
- Cookie de sessão é host-only: cliente loga uma vez no `crm.`; equipe que cair no `crm.` vai pro `membros.../admin` e loga de novo.
- DNS (Cloudflare, zona `quirkgrowth.com.br`): `crm` e `membros` são CNAME → `area-membros-quirk.onrender.com`, **Somente DNS** (sem proxy). O `crm` ANTES era CNAME → `whitelabel.bolten.io` (proxied, sem uso, Renan autorizou trocar); para voltar atrás, restaurar esse valor.
- Render: plano com limite de **2 domínios personalizados** (já usados: membros + crm). Um terceiro exige upgrade.
- Render avisou manutenção de infra em 14/10 22h (Brasília).
- Depois de trocar o DNS, o Render mostrou "Certificate Error" até clicar no ⟳ (refresh) do domínio; aí virou "Certificate Issued".
- Campo `clientes.telaInicialPadrao` ficou no schema (oculto) de propósito: tirar faria o push do Drizzle pedir confirmação interativa ([[gotcha_drizzle_push_interativo_trava]]) e não há DDL em prod.
- Páginas/coleções de aula seguem no código/banco; `/vertente` e `/aula` redirecionam pra `/` (next.config). `/assistir/[token]` NÃO redireciona (entrega pública da aula avulsa paga, `/api/aula/liberar` ainda emite link). "Materiais e Mídias" (pasta do Drive) virou item do menu, sem vendedor.
- 15 pessoas assistiram aula nos 90 dias antes do corte e perderam o acesso (decisão do Renan, sem aviso).

**Pendente:** (1) conferir com conta `renandmreal` login em `crm.` + convite de senha de cliente de teste com link `crm.`; (2) "Corte 2": deixar o CRM profissional/apresentável (não começou; precisa olhar as telas); (3) limpeza definitiva do código/coleções de mentoria (CardVertente, BannerHomeView, lib/progresso, cacheConteudo…) só se o Renan decidir; (4) Omnichannel: caixa única IG+WhatsApp+e-mail, abordagem B recomendada (tabela `crm-threads`), Renan disse que vai pensar.

Relacionado: [[project_crm_quirk]], [[reference_render_cloudflare_ip]].
