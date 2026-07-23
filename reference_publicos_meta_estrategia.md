---
name: reference_publicos_meta_estrategia
description: "Estratégia de públicos do Quirk Auto Ads — escada de qualificação mensurável + secundários imobiliários, com IDs reais da Meta"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 29ede2ee-f613-4b1b-b415-bf274185df44
---

Estratégia de públicos da Quirk (validada com Renan em 2026-07-03) usada no extrator do [[project_quirk_auto_ads]]. Princípio: **base em qualificação MENSURÁVEL pela Meta** (comportamentos), empilhada (cada "E" = flexible_spec separado = estreita), com interesses imobiliários/luxo só como **secundário**, e **iPhone (`user_os:["iOS"]`)** por cima. Pub 0 = aberto (só geo+idade); quanto mais alto o nível, mais estreito/premium.

**Comportamentos mensuráveis reais (a Meta depreciou quase tudo — estes existem):**
- Viajantes frequentes `6002714895372`
- Viajantes internacionais frequentes `6022788483583`
- Valor intermediário e alto BR `6110813675983`
- Valor alto BR `6046096201583`
- Compradores envolvidos (engaged shoppers) `6071631541183`
- Proprietários de pequenas empresas `6002714898572`
- Busca de comportamento: `GET /search?type=adTargetingCategory&class=behaviors` NÃO aceita `q` (só navega a lista toda ~285; filtrar client-side). Interesses: `type=adinterest&q=...` aceita busca.

**Interesses imobiliários secundários (validados):** Investimento imobiliário `6003446239080`, Propriedade de imóveis `6003693537583`, Desenvolvimento imobiliário `6003332796032`, Real Estate `6002979192120`, Condomínio fechado `6003077334693`, Condomínio `6003435139283`, Apto cobertura `6003055396585`, Casa de campo `6003195200868`, portais (OLX Brasil `6002965402168`, zap imóveis `6014552641654`, imovelweb `6018886000820`, Portais de imóveis `6788101567252`), Arquitetura e Construção `6011884210354`, Hipoteca `6003288338951`, Bens de luxo `6007828099136`. Corretores: Corretor `6003569393903`, Corretagem residencial `6778210171187`.
- **Descartados:** marcas de construtora (Tenda/Tecnisa/Even — regionais, não universais); interesses amplos demais (Apartamento 243M, Imóveis 424M, Hipotecas 144M, Aluguel 106M).

**A escada (12 públicos no extrator, TABELA DE PÚBLICOS):** Pub 0 aberto · Pub 1 viajante · Pub 1.1 viajante+portais · Pub 1.2 viajante+interesse imob · Pub 2 viajante+comprador médio/alto · Pub 3 (viajante/internacional)+comprador ALTO · Pub 4 +alto padrão(cobertura/RE/luxo) · Invest · Veraneio · Primeira Moradia · Profissões · Corretores. Regra de escolha: objetivo+faixa_valor+trilho → Pub.

**Motor de piso de público (nó `audience_floor`, entre meta_d1 e meta_d2):** antes de subir o conjunto, estima o tamanho via `GET /act_{id}/delivery_estimate?optimization_goal=REACH&targeting_spec=...` (retorna `estimate_mau_lower_bound`). Se < **50k**, afrouxa sozinho (decisão interna, cliente não vê) na ordem: camadas extras → iOS → base viajante (última), re-estimando a cada passo, até ≥50k ou abrir de vez (região pequena aceita como está). `meta_d2_adset` usa `$('audience_floor').targeting_meta`. Motivo: público pequeno = leilão apertado = CPM alto.

Arquivos de origem em `/Users/renanreal/quirk_auto_ads/`: PUBLICOS_*.md, interests_ids.json, resolve_interests.py.
