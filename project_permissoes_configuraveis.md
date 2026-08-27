---
name: project-permissoes-configuraveis
description: "Feature \"Permissões configuráveis\" (matriz papel×item + exceções por pessoa) no ar em produção desde 27/ago/2026 — o que existe, onde mexer, e as pendências conhecidas"
metadata: 
  node_type: memory
  type: project
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-27T05:27:42.650Z
---

Visibilidade de tela/menu do admin da área de membros Quirk deixou de ser
hardcoded por papel e virou configurável: matriz papel×item editável em
`/admin/permissoes` (só admin/supervisor/administrativo), com exceções
pontuais por pessoa na ficha de cada um. No ar em produção desde 27/ago/2026
(commit `451cc56`, `main`).

**Peças centrais**: `src/lib/relatorios/permissoes.ts` (`podeVerItem`/
`podeVerItens`/`resolverPermissao`, resolução exceção→padrão→nega, 1 busca em
lote ao global `permissoes-padrao`, não N); `src/lib/menuAdmin.ts`
(`TODAS_AS_SECOES` é a lista canônica de itens; `ITENS_FORA_DA_MATRIZ` e
`ITENS_COLECAO_NATIVA` são as duas categorias de exceção — ver abaixo);
`src/lib/relatorios/permissoesSeed.ts` (`montarRegrasIniciais`, cópia
congelada do comportamento pré-feature, usada pro seed inicial E pelo teste
de invariante de cobertura — **NÃO apagar este arquivo**, vira o "padrão de
fábrica" versionado).

**Duas categorias de item fora da matriz** (`ITENS_FORA_DA_MATRIZ`, Set em
`menuAdmin.ts`): itens cujo acesso é fixo por código/regra de negócio mais
funda, force-added em `secoesDoAdmin` (não passam pela matriz nem aparecem
editáveis nas telas): `/admin/permissoes` (evita autolockout), `/admin/
trilha-time` (`ehLiderancaTrilha`, regra "quem vê quem" da trilha, não é
toggle de papel), `/propostas` e `/analises` (`podeVerPropostas`/
`podeVerAnalises`, ferramentas fora do namespace `/admin/*`, gate próprio na
página). Diferente disso, `ITENS_COLECAO_NATIVA` (7 hrefs de coleção nativa
do Payload — Aulas, Módulos, Vertentes, etc.) SÃO editáveis na matriz, mas só
controlam o link do MENU — o `access.read` de cada coleção continua sendo a
fonte real do acesso; `MatrizPermissoes.tsx` mostra um aviso (tooltip) nesses
itens especificamente.

**Padrão obrigatório pra QUALQUER item novo fora do namespace `/admin/*`**
(ferramenta com gate próprio de página, tipo propostas/análises): extrair um
helper puro `podeVer<Coisa>(role)` num `src/lib/<coisa>/acesso.ts`, usar essa
MESMA função em: a página, a rota de API, o `access` da coleção (se houver),
E o force-add em `menuAdmin.ts` + entrada em `ITENS_FORA_DA_MATRIZ`. Nunca
reimplementar o `role === 'x' || role === 'y'` em cada lugar — foi a causa
de um Important real numa revisão (4 cópias divergentes em potencial).

**Bugs de segurança reais corrigidos durante a implementação** (não
pré-existentes, introduzidos e pegos na própria revisão adversarial):
endpoint `/permissoes-extras` (grava exceção por pessoa) reintroduzia o bug
"payload.update desloga" (ver [[reference_payload_update_sessions]]) — só
foi exercitado de verdade quando a UI da ficha da pessoa passou a chamá-lo;
corrigido gravando via `pool.query` na coluna, igual `/preferencias`. Mesmo
endpoint não checava `rowCount`, respondendo 200 mesmo sem gravar nada pra
`userId` inexistente.

**Pendências conhecidas, de baixa prioridade** (não bloqueiam nada):
- Papel `social` tem link morto de "Meu perfil" no rodapé (`PAPEIS_COM_AGENDA`
  não inclui `social`) — bug PRÉ-EXISTENTE a esta feature (16/ago), achado de
  passagem na revisão final. Spawnado como task separada (`task_e55effad`),
  ainda não resolvido no momento em que esta memória foi escrita.
- Mensagens de erro fixas tipo "Acesso restrito (admin/supervisor)" em
  algumas views (`AgendaTimeView`, `HistoricoSatisfacaoView`, `BiFinanceiroView`)
  ficaram desatualizadas/imprecisas agora que a decisão real é configurável
  pela matriz — cosmético, nota de polish futuro.

**Merges de features paralelas absorvidos no deploy**: esta feature levou 24
commits próprios + 2 merges reais de `origin/main` durante o processo (outras
sessões: "guia comercial" com `podeVerGuia`, "propostas comerciais", "Agente
de Análise de Presença") — todos os 3 itens de menu novos das outras
features (`/propostas`, `/analises`, e a lógica de `podeVerGuia` em "Meu
treinamento") foram integrados à arquitetura desta feature (force-add +
`ITENS_FORA_DA_MATRIZ`) durante a resolução de conflito, não pelas sessões
originais (que nunca viram este código). Ver [[feedback_git_repo_compartilhado_sessoes]]
pro detalhe de como esses conflitos foram resolvidos.
