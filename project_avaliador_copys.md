---
name: project_avaliador_copys
description: Projeto de agente avaliador de copys da Quirk; análise inicial do corpus de copys imobiliárias no Drive e DNA estrutural extraído
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c440f6-f4b9-429b-8c1a-cb5ce0bf0e47
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
