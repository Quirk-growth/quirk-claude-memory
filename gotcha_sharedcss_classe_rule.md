---
name: gotcha_sharedcss_classe_rule
description: "Nos decks HTML→PNG da Quirk (shared.css), a classe .rule é RESERVADA (é o risquinho azul de destaque). Reusá-la em outro contexto quebra o layout silenciosamente só no render headless."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 798c9556-748c-4496-87e4-6ab001869e18
  modified: 2026-09-16T18:41:03.145Z
---

No pipeline de slides HTML da Quirk (`shared.css` + Chrome headless `--screenshot`), o `shared.css` já define **`.rule`** como o risquinho de destaque: `width:96px;height:6px;background:var(--blue)`. E também `.slide`, `.brand`, `.cresce-tag`, `.title`, `.accent`, `.lbl`, `.center`, `.soft`, `.mut`.

**Gotcha:** se você criar uma classe `.rule` (ou reusar qualquer uma acima) no `<style>` do slide sem sobrescrever `width`/`height`, as duas regras MERGE — seu elemento herda `width:96px;height:6px` do shared.css → texto quebra 1 palavra/linha e itens absolutos colapsam. Custou ~5 renders no Programa de Parceiros (set/2026).

**Pegadinha de diagnóstico:** o preview do Claude (Browser pane) abre arquivo fora do projeto como **data: URL** e NÃO carrega o `shared.css` relativo → ali renderiza CERTO, escondendo o bug. Só o headless (file://, que carrega o shared.css) mostra quebrado. Não confie no preview pane pra validar; valide sempre pelo PNG do headless.

**Regra:** prefixe classes locais do slide (ex: `.ritem`, `.pcard`) pra nunca colidir com o `shared.css`. Se precisar reusar um nome, sobrescreva explicitamente `width`/`height`.
