---
name: project-mapa-sistema-obsidian
description: "Mapa do sistema (área de membros) em grafo estilo Obsidian: vault em ~/vault-sistema-quirk, gerador regenerável, dicionário de nomes em português editável (nomes.json)"
metadata:
  type: project
---

Pedido do Renan (14/out/2026): usar o Obsidian pra **entender as conexões do sistema** [[project_area_membros_quirk]]; o grafo tinha que ser parecido com o do Obsidian, mas com nomes que ele entenda (os nomes do código — `logSeguro`, `CrmLeads`, `rota-api-cron-…` — não serviam).

**Onde está:** `/Users/renanreal/vault-sistema-quirk/` (fora do repo, nada commitado). `mapa-grafo.html` = página única, abre em qualquer navegador, offline, canvas com física própria (sem lib). `codigo/` + `memoria/` = notas Markdown com `[[links]]` pro Obsidian (`memoria/` é CÓPIA da memória). **Regerar:** `bash ~/vault-sistema-quirk/_gerador/gerar.sh [caminho-do-repo]` (padrão `/Users/renanreal/area-membros-quirk`, que pode estar numa branch velha — passar o worktree/main certo).

**Como funciona:** `_gerador/gerar_vault.py` lê o código por REGEX (não AST): relações entre coleções (`relationTo`), imports de `src/lib`, `collection: '<slug>'`, rotas (`route.ts`), e menções nas memórias. `_gerador/nomes.json` é o dicionário curado por mim a partir dos comentários de cabeçalho do código: cada peça → `[nome em português, área do negócio, descrição de uma frase]`, 191 entradas (57 coleções, 73 módulos, 61 rotas) + 13 áreas com cor. **O Renan edita o nomes.json e regera** — é o jeito de corrigir um nome. Peça nova do código sem entrada aparece com nome técnico e é listada em "sem nome amigável no dicionário".

**Fatos que não saem do código:**
- 9 descrições estão marcadas `true` (inferidas do nome porque o código não as explica): módulo `vgv` + rotas `asaas/normalizar-notificacoes`, `chat/ler|mensagens|resumo`, `satisfacao/fechar-mes`, `sync/backfill-conta|onboard|pendentes`. A página mostra um aviso âmbar nelas — confirmar com o Renan antes de tratar como verdade.
- Título/frase das memórias vêm do índice `MEMORY.md` (formato `- [Título](arquivo.md) — frase`); cada memória herda a ÁREA do código que cita (39 de 106 têm; o resto é "sem área").
- NÃO coberto ainda: camada de telas (`components/admin`, páginas) e crons/integrações externas (Render, Make, n8n, Asaas, Meta). Por isso módulo "sem ligação" ≠ morto (ex.: `tema`, `formatarData`, `vgv` são usados só por tela).
- O gerador e o dicionário NÃO estão no repo. Levar pra `scripts/` exige aval do Renan.

**Verificação (gotcha):** o painel de navegador do app recusa ferramentas de página em `file://` (só tira screenshot "estático") e o hook de worktree-isolado recusa Bash com variável de shell ou caminho entre aspas. O que funcionou: Chrome headless via um script `.sh` com caminhos literais (`--headless=new --dump-dom` pra rodar o JS e ler o resultado de uma cópia de teste; `--screenshot` pra ver). Navegar duas vezes pra mesma URL no painel NÃO recarrega a página (reaproveita a aba e o estado do teste anterior).

**How to apply:** pra "como as peças se conectam?", abrir o mapa; pra impacto de mudança, olhar o detalhe da coleção/módulo (quem usa / quem toca). Antes de apresentar uma descrição ao Renan como fato, checar se é uma das 9 inferidas.
