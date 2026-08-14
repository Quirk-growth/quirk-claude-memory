---
name: project-lp-quinta-arvoredo
description: "LP do Residencial Quinta do Arvoredo (Zanardi, Itapeva/MG) — copy pronta espelhando estrutura da LP Vitória Régia; lançamento 01/09/2026, sem preço público ainda"
metadata: 
  node_type: memory
  type: project
  originSessionId: 69b205c4-a049-4a8f-9f4d-4104babe65f5
  modified: 2026-08-14T22:01:44.067Z
---

LP de lançamento do **Residencial Quinta do Arvoredo** (Zanardi Empreendimentos + gestora Neximob): loteamento fechado em Itapeva/MG (Sul de Minas, região Extrema/Monte Verde), **221 lotes a partir de 600 m²**, portaria 24h, 12 itens de lazer (beach tênis, gourmet, pomar, horta, galinheiro, capela…), poço+reservatório próprios. Distâncias-âncora: 10 min centro de Itapeva, 35 min Extrema, 50 min Monte Verde, **85 min de SP**. Acesso: Rod. Tancredo Neves, 123 / Estrada Municipal da Capitinga, Bairro Pedrosos.

- Copy completa entregue em `/Users/renanreal/zanardi-quinta-arvoredo/copy-lp-quinta-do-arvoredo.md` (14/08/2026), estrutura dobra a dobra espelhada da LP **Residencial Vitória Régia** (zanardiempreedimentoslp.com.br/residencial-vitoria-regia); a home do mesmo domínio é a LP Rancho Terra Nova.
- **Sem preço/condições no material** — lançamento é 01/09/2026 (meeting de corretores c/ Edgar Ueda); 3 placeholders `[DEFINIR]` no doc esperam a tabela.
- Material-fonte: pasta Drive `1Qw4OfXj-RuE96dnRXRH5WvcioOr-r-F3` (dona: pscuderi@scuderi.com.br). Book 67MB não baixável via MCP (limite 10MB); LP antiga e anúncio são PDFs sem camada de texto (CDR) — extrair renderizando via PyMuPDF (pip, sem brew nesse Mac).
- GOTCHA das LPs Zanardi no ar: conteúdo mora num iframe "bundled page" (HTML estático em /wp-content/uploads/...) — WebFetch/get_page_text da página-mãe voltam vazios; navegar direto pra URL do iframe e esperar o unpack.
- Site oficial quintadoarvoredo.com.br ainda fora do ar (cert de parking) em 14/08/2026.
