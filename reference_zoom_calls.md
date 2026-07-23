---
name: reference_zoom_calls
description: Como acessar e varrer as gravações/transcrições de Zoom do Renan via conector MCP — mecânica, pegadinhas e mapa de cobertura dos últimos 6 meses.
type: reference
originSessionId: 798c9556-748c-4496-87e4-6ab001869e18
---
Conector **Zoom MCP** ativo (prefixo `mcp__276933cf-b8c0-4fb9-b3dd-87b81e9b5803__`). Confirmado funcionando jun/2026.

**Mecânica:**
- `recordings_list` (from/to em `yyyy-mm-dd`, **máx 1 mês por chamada**, page_size até 300) lista gravações com `uuid`, `topic`, `start_time`, `duration` e arquivos (TRANSCRIPT VTT, SUMMARY, etc).
- `get_recording_resource` lê conteúdo: `meetingId` = uuid **DUPLO-ENCODADO** (`/`→`%252F`, `+`→`%252B`, `=`→`%253D`). `types`: `transcript`, `summary`, `nextStep`, `playUrl`.
- PEGADINHA: `types:"summary"` traz um array gigante `smart_pic_url`/`thumbnailUrls` que estoura contexto — ignorar, ler só `overall_summary` e `items[].summary`.

**Pegadinhas de descoberta:**
- A **busca semântica** (`search_meetings` com `q`) volta VAZIA — os tópicos das calls são todos genéricos ("Sala Pessoal do 'Renan Real'"), então o índice não acha por tema. Para varrer, é mês a mês via `recordings_list` + triagem pelos summaries.
- Fuso: America/Sao_Paulo.

**Mapa de cobertura dez/2025–jun/2026 (varredura jun/2026):** dez/25 = 0 gravações; jan/26 = 1 (Assessoria Comercial); fev/26 = 0; mar/26 = 2 (cultura/protagonismo + 4 etapas da venda); abr/26 = 1 ouro (Fórmula Redução do CAC); mai/26 = várias (Oferta de Alto Impacto, SPIN Selling c/ Chaiene, Linha Editorial que Vende); jun/26 = onboarding cliente. As mentorias de metodologia (marketing + comercial) são a fonte mais rica de IP da Quirk.
