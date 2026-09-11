---
name: project_avaliador_copys
description: Projeto de agente avaliador de copys da Quirk; análise inicial do corpus de copys imobiliárias no Drive e DNA estrutural extraído
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c440f6-f4b9-429b-8c1a-cb5ce0bf0e47
  modified: 2026-09-11T13:28:21.615Z
---

Renan está construindo um **agente avaliador de copys da Quirk Growth** (jun/2026). Primeira etapa pedida: avaliar os docs de copy no Drive e mapear pontos de similaridade entre as copys imobiliárias. Vai passar mais infos (critérios, formato de nota, exemplos bom vs ruim) depois.

**Corpus no Drive:** centenas de docs "Copy *". Dois tipos distintos:
- **Tipo 1 — copy de produto imobiliário** (anúncio de imóvel/lançamento pro cliente da imobiliária). É o foco do avaliador. Estrutura `H1/H2 + tópicos + valor + CTA`.
- **Tipo 2 — copy institucional Quirk** (@renanreal/@quirkgrowth, atrai incorporadora; cases, autoridade). Lógica diferente. *Pendente: confirmar se o avaliador cobre os dois.*

**Doc-chave com a metodologia escrita:** "Infos Pubs imóveis Quirk" (fileId 1CO6oNu5wio3zjy1ysUUjE1JfB7W8-a2GkOysv9QDth8) — tem ordem de prioridade do gancho, empilhamento de oferta, briefing, públicos Meta, "80% da copy está no gancho". Também relevante: "04_copies_prontas.md", "[QUIRK][PB Boutique] Copy imóveis".

**DNA extraído (17 copys de clientes/autores diferentes):**
1. Esqueleto universal GANCHO → BODY → CTA.
2. Sempre 2-6 variações da mesma oferta, cada uma com ângulo diferente (teste A/B embutido).
3. Gancho carrega tipo+local+valor; 80% do peso. Ordem: tipo→valor→localização→quartos→banheiros→metragem.
4. Body: dorms/suítes → ambientes → vagas → metragem (faixa) → lazer listado → localização-âncora (shopping/praia/vias).
5. CTA único, baixa fricção, do repertório: "Cadastre-se", "Agende sua visita", "Fale com um especialista", "Me chama". Nunca "compre".
6. ZERO emojis (~100%), 2ª pessoa, tom limpo.
7. "Empilhamento de oferta" (escassez + bônus + dado específico).

**3 eixos que reescrevem a copy (estrutura fica intacta):** Público (morador↔investidor, a variável-mãe) · Padrão (alto padrão↔econômico/MCMV) · Objetivo (venda↔captação). Relacionado: [[project_quirk_auto_creative]] (skill quirk-auto-creative gera copy tipo 1, ZERO CTA visual) e [[project_quirk_auto_ads]].

**REFINAMENTO (jun/2026) — padrão canônico confirmado pelo Renan:**
- O prompt-avaliador foi entregue e testado; Renan achou genérico e deu o contexto definitivo abaixo.
- **2 modelos oficiais:** Modelo 1 ANÚNCIO (Gancho→Body→CTA) e Modelo 2 BANNER (Headline 1 + Headline 2 + 3-4 tópicos). A reescrita do avaliador entrega os DOIS.
- **Gancho é o ponto nº1 (peso 35%):** tem que BATER as características de MAIOR DESTAQUE do produto logo no início, direto e conciso — liderar pelo campo "Destaque do produto" + tipo+local+valor. Gancho morno/genérico = erro nº1. Ordem: tipo→valor→localização→quartos→banheiros→metragem.
- **Body é DIRETO e factual, NÃO floreado.** Quirk NÃO busca encantamento. "Encantamento" (emocional/aspiracional floreado, storytelling longo, adjetivação excessiva) é exceção a QUESTIONAR; demais é ruim pro topo de funil. O avaliador sinaliza, explica a ressalva e entrega versão direta como padrão.
- Existe prompt GERADOR de copy num GPT da Quirk (Renan colou) — é a fonte do padrão; o formulário de briefing oficial (Cidade, Bairro, Proximidades, Valor, Ficha técnica, Quartos, Banheiros, Metragem, Vaga, Lazer, Renda, Lançamento, Target, Rentabilidade, Destaque do produto) está embutido no avaliador.
- Arquivos: /Users/renanreal/avaliador-copys-quirk/ → `prompt-avaliador-copys.md` (mestre) e `gpt-avaliador-copys.md` (Custom GPT, Instructions ~5.9k chars, cabe no limite 8k). Próximo passo combinado: rodar uma copy real pra calibrar.

**GERADOR de copy — evolução (11/set/2026):** além do avaliador, Renan mantém o GPT GERADOR de copy. Nova frente: a partir do briefing, gerar **elementos criativos de reforço ao gancho** (objeto/ação/prova no 1º segundo do vídeo que dá destaque e retenção) — ex. real: água de coco=João Pessoa, print de diária Airbnb, trena que fecha, bloco de concreto (trocadilho c/ "Bloco Construções"), chave "só existe em 2029". Precisam ser MUITO criativos, "que ninguém no mercado faz". Prompt gerador v2 em /Users/renanreal/gerador-copys-quirk/`prompt-gerador-copys.md` (~5.8k chars): motor de 8 lentes de criatividade + padrão de produção de vídeo (gancho 1 take câmera em movimento; body 3-4 takes; CTA 1 take plano médio parado; nunca 2 planos iguais seguidos) + 3-5 variações por briefing, cada uma com ângulo e elemento criativo distintos. REGRA: emoji NUNCA dentro da copy; só como ícone-rótulo do elemento criativo (direção interna pro time). Relacionado: [[project_quirk_auto_creative]].

**MESCLAGEM gerador+avaliador (11/set/2026) — gerador v3:** Renan pediu pra fundir os 2 prompts pra saída mais assertiva. Solução: gerador AUTOCORRETIVO — escreve, se autoavalia internamente contra a rubrica Quirk (gancho 35% etc.), reescreve o que ficar <8,5 e só entrega copy que passaria no Avaliador. Decisões dele: (1) CTA ampliado pra incluir os falados de vídeo ("Clique no botão e fale com a nossa equipe", "Me chama") além do repertório estático do avaliador — porque o exemplo real do Nakhon usava CTA de vídeo fora do repertório; (2) SELO DE QUALIDADE compacto no fim (nota do gancho por variação, ≥8,5), autoavaliação interna não é mostrada. Arquivo prompt-gerador-copys.md atualizado pra v3 (~7,2k chars). Alternativa registrada p/ futuro: pipeline 2 etapas (gerador→avaliador→volta) via n8n quando quiser relatório de nota registrado/escala.
