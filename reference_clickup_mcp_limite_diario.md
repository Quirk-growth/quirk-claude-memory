---
name: clickup-mcp-limite-diario
description: O conector MCP do ClickUp tem teto de 1.000 chamadas/dia por workspace e get_task/filter_tasks dão timeout às vezes; orçar chamadas antes de delegar extração em massa.
metadata:
  node_type: memory
  type: reference
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-10-15T02:35:17.491Z
---

- **Limite:** "Daily MCP limit reached (1,000/1,000 calls used)" (`RATE_LIMIT_EXCEEDED`), libera cerca de 18h depois. Estourou em 14/10/2026 depois de ~5 agentes em paralelo + tentativas do dia anterior (cada peça = 1 chamada `get_task`).
- **Orçamento:** extrair texto de uma peça custa 1 chamada. ~1.000 chamadas ≈ ~700 peças no dia, divididas entre TODOS os agentes. Calcular antes de lançar vários agentes; deixar margem.
- **O que funciona:** `clickup_search` (com `location.subcategories=[idLista]`) acha tarefas e IDs; `get_task` com `include:["subtasks"]` lista as peças de uma mãe; `get_task` com `include:["description","checklists"]` traz o texto. Documentos (`get_document_pages`) funcionam bem.
- **O que NÃO é confiável:** `clickup_filter_tasks` deu timeout sistemático; a lista de subtarefas mostra `description:""` mesmo quando a peça tem texto (só a leitura individual vale); `due_date` está desatualizada em muitas peças (usar o prefixo DD.MM do título).
- **Não usar o navegador** para ler o ClickUp: a tela carrega por partes, `get_page_text` devolve quase nada e a árvore da página tem 44 mil caracteres de menu.
- **Subagentes:** um agente delegou a sub-agentes por conta própria (custo extra): dizer "não crie sub-agentes". Subagentes não têm chave de API do ClickUp nem devem usar uma.
- Relacionado: [[project_agente_social_media_base_conhecimento]]
