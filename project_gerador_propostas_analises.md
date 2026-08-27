---
name: gerador-de-propostas-agente-de-an-lise-rea-de-membros
description: "Duas ferramentas comerciais na área de membros — divisão de frentes entre sessões, decisões travadas e estado das branches"
metadata: 
  node_type: memory
  type: project
  originSessionId: abd490d3-d546-47fd-9269-c2122b669bb6
  modified: 2026-08-27T14:29:15.410Z
---

Ferramentas do comercial na área de membros (ago/2026), nascidas da skill [[proposta-quirk]] (~/.claude/skills/proposta-quirk/ — processo, regras de copy do Renan, 3 exemplos reais, spec do agente de análise em references/agente-analise-presenca.md).

**Decisões do Renan (25/08):** acesso APENAS comercial + admin (fail-closed) · vendedor digita valores livremente · PDF sai direto sem aprovação · Caminho B escolhido (ferramenta no portal via API do Claude; ninguém do time precisa de assinatura Claude).

**Divisão de frentes entre sessões paralelas (coordenada por mensagem):**
- **renanreal-79**: Gerador de Propostas inteiro — branch `feat/gerador-propostas` no checkout principal (Propostas.ts, src/lib/propostas/*, /api/propostas, página (app)/propostas). ELA MERGEIA PRIMEIRO.
- **Esta sessão (renanreal-35)**: Agente de Análise de Presença — branch `feat/analise-presenca` no worktree `/Users/renanreal/area-membros-quirk-analise`. Commitada, SEM push (aguarda DDL manual + merge da 79 pra rebasear e reutilizar o renderizarPdf compartilhado dela).
- **renanreal-40**: materiais estáticos (proposta-incorporadora, manual-comercial, guia-perfil) — dona do proposta.html da incorporadora daqui pra frente.

**O Agente de Análise entrega:** avaliação sem preços de 3 pilares (Site auto-fetch · Instagram e Anúncios colados pelo vendedor na v1) com nota 0-10 → PDF identidade Quirk + e-mail personalizado (≤120 palavras) pronto pra copiar. Limite 10 análises/dia por usuário. Exporta `gerarObservacoes({site,instagramDados})` pro campo observacoes do gerador.

**Gotchas técnicos descobertos:** args do @sparticuz/chromium travam launch do Chrome desktop no Mac (teste local = CHROME_EXECUTABLE_PATH + args mínimos) · puppeteer-core 25.9 sem networkidle0 no setContent (usar 'load' + document.fonts.ready) · worktree novo não tem .env (fica no checkout principal) · ANTHROPIC_API_KEY só existe no Render, sem teste de API local.

**DEPLOY TRAVADO (27/08 ~10h BRT):** pushes depois da feature de permissões (451cc56, comprovadamente no ar via 403 em /api/permissoes/salvar-padrao) NÃO estão chegando em produção: 1h40 após o push de 7023f5d (views do portal) o marcador público (/propostas deve 307→/admin/propostas; segue 307→/login) não virou, nem com redeploy por commit vazio (cd77ac8). Build de produção local do mesmo commit passa VERDE. Site estável no build antigo. Diagnóstico precisa do painel do Render (Events/Deploys) — pedido ao Renan. Commits represados: 97847c0 (fotos), 88e42b7+623d8c3 (guia), fec081f (prints evidência), ec580f1 (prompt janelas abertas), 7023f5d (views portal).

**v2 (27/08, commit `fec081f` na main):** prints de evidência reais no PDF — site, perfil público do Instagram (campo novo `instagramUrl`, DDL `instagram_url` aplicada em teste+prod ANTES do push) e Biblioteca de Anúncios por frase exata (`search_type=keyword_exact_phrase`; unordered traz lixo). Captura via `comBrowser` novo exportado do renderizarPdf (mesma fila). Claude recebe placeholders `__LOGO__`/`__PRINT_*__`; substituição no pós-processamento (base64 nunca passa pelo modelo). Logo = ícone oficial 128px base64 (SVG recriado distorcia). Instagram deslogado: derrubar `div[role=dialog]` (modal de cadastro) e detectar "não está disponível". GOTCHA runner de DDL: lê `DATABASE_URI` (não DDL_URI) e a falha por env faltando imprime só "Node.js v24..." — conferir SEMPRE por introspecção. Deploy sem rota nova não tem marcador público (fingerprint cego): prova final é gerar uma análise no ar.

**NO AR (26/08):** Análise de Presença deployado — main em `55fe6b9`, smoke test verde (POST /api/analises/gerar = 403; página /analises redireciona pro login). DDL de prod aplicada antes do push. Renderizador DELEGA pro renderizarPdf compartilhado da 79 (fila única de Chromium). Página fica no grupo `(equipe)` — o grupo `(app)` tem gate da área do cliente que barra o papel comercial (fix espelhado do 0d2d4e7 do gerador). Chamada ao Claude em **streaming** (stream()+finalMessage()) — SDK recusa create() não-streaming com max_tokens alto (fix espelhado do f5ed9e4). Runner de DDL: scripts/aplicar-ddl.mts (bloqueia -pooler). Falta validar geração real com usuário logado (ANTHROPIC_API_KEY só no Render).

**GOTCHA novo do banco de teste:** virou terra disputada — sessões paralelas rodando suíte/push com código da main DROPAM tabelas que não estão no schema delas (minha analises sumiu 2x). Validação atômica = aplicar DDL + introspectar NA MESMA CONEXÃO. E o drizzle push interativo trava em prompt "create or rename" por causa da tabela órfã permissoes_padrao no teste (não existe em nenhuma collection do código).

**Pré-existente na main (não meu, verificado no checkout principal em 0d2d4e7):** typecheck:tests com 98 erros (tipos gerados dessincronizados por outra sessão) e 4 testes vermelhos (tarefa-templates-aplicar, tarefas-recorrencia, agenda-conectar-form, menuAdmin "Pessoas…role" — esse último era da feature de permissões, que na época ainda não tinha dado push).

**ATUALIZAÇÃO (27/08): a feature de permissões (`-crm`) deu push (`451cc56`) e absorveu tanto o Gerador de Propostas quanto o Agente de Análise no processo.** `menuAdmin.ts` virou ponto de conflito real entre as duas frentes — `secoesDoAdmin`/`itensDoColaborador` da feature de permissões (assíncronas, matriz configurável) trocaram de lugar com a versão síncrona que Propostas/Análises tinham estendido (`podeVerGuia`). Quem resolveu o merge (a sessão -crm) teve que: extrair `podeVerPropostas`/`podeVerAnalises` como helpers puros (pra não duplicar `role==='comercial'||'admin'` em 4 lugares cada), e adicionar `/propostas`/`/analises` como force-add em `secoesDoAdmin` (fora da matriz de permissões — `ITENS_FORA_DA_MATRIZ`), já que essas 2 ferramentas moram fora do namespace `/admin/*` e têm gate próprio na página. Ver [[project_permissoes_configuraveis]] pro detalhe da arquitetura resultante — QUALQUER ferramenta nova fora de `/admin/*` com gate de página própria deve seguir esse mesmo padrão (helper puro + force-add), não reimplementar a checagem de papel em cada lugar.
