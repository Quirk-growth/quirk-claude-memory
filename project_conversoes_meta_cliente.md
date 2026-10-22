---
name: project_conversoes_meta_cliente
description: "Mandar sinais de progresso do CRM pro pixel DO CLIENTE + ranking de performance por campanha no Hub. Lotes 1 e 5 no ar (out/2026); 2, 3 e 4 pendentes."
metadata:
  node_type: memory
  type: project
---

Pedido do Renan em **20/10/2026**. Dois lados do mesmo dado, no
[[project_crm_quirk]] / [[project_area_membros_quirk]]:

- **Enxergar** — ranking de campanha e conjunto no Hub, cruzando o funil do CRM
  com o custo do sync ([[project_painel_relatorios]]). Do lado da Quirk só.
- **Automatizar** — eventos de progresso pro conjunto de dados **do cliente**,
  pra Meta otimizar por reunião e venda em vez de por lead.

Spec: `docs/superpowers/specs/2026-10-20-conversoes-meta-cliente-design.md`.

## Estado (22/10/2026)

| lote | o quê | estado |
|---|---|---|
| 1 | Atribuição por id no lead | **NO AR 21/10** (`f54ab1be`), DDL antes |
| 5 | Ranking no Hub, aba `Performance` | **NO AR 22/10** (`64cb7f69`), sem DDL |
| 2+3 | Envio por cliente + tela no Hub (aba CRM) | **NO AR 22/10** (`df096408`), DDL antes |
| 4 | Vigilância: alerta + tela global | **NO AR 22/10** (`769788c0`), DDL + dado de permissão antes |

**PROJETO COMPLETO.** Nenhum cliente ligado ainda — ligar é decisão do Renan,
cliente a cliente, pelo Hub → aba CRM → Conversões pro Meta.

## Decisões do Renan

- **Escada de eventos** por volume: `Lead` (em atendimento) → `Schedule`
  (agendado) → `Purchase` (venda). Cada degrau liga separado; começa pelo de
  cima porque a Meta precisa de ~50 eventos/semana.
- **Cliente NUNCA vê o ranking.** Fica no Hub, e o papel `social` também não vê.
- Níveis de decisão: **campanha e conjunto**. Anúncio só no card do lead.
- Vigilância: alerta de silêncio **e** lista global de todos os clientes.

## Fatos medidos (re-medir antes de confiar)

- **Elegibilidade da escada passa**: atendimento 29,8%, agendamento 5,4% dos
  leads de Lead Ads — a Meta exige entre 1% e 40%, e 100% dos atendimentos
  cabem na janela de 28 dias. Armadilha: se o atendimento passar de 40% o
  degrau deixa de ser elegível — melhorar desqualifica.
- **Valor da venda está vazio**: 2 de 24 cards ganhos têm valor. Por isso a v1
  manda `Purchase` SEM valor, contra o pedido original. Reversível.
- **Pixels**: de 109 contas Meta ativas, 50 já expõem pixel pro nosso token,
  55 não têm pixel nenhum, 4 sem permissão. **Quase todos são da BM DO
  CLIENTE**, compartilhados — não nossos.
- **Escrita**: `POST /{pixel}/events` devolveu 200 + `events_received: 1` num
  pixel de cliente (VLL Construtora, 21/10). MAS o evento **não apareceu** no
  Gerenciador dele. Renan decidiu seguir considerando que funciona. Por isso o
  botão "Enviar evento de teste" por cliente é **requisito** do Lote 2, não
  enfeite — cada cliente confirma o próprio acesso ao ligar.
- Não existe sandbox: `test_event_code` inventado é processado como produção.

## Armadilhas do desenho

- **Atribuição é BLOCO**, não campo a campo: ou tudo do primeiro toque, ou
  nada. Preencher buraco monta linha Frankenstein (anúncio A + nome do B).
- **Período conta pela data do EVENTO**, não da criação — então
  `atendimentos > leads` é NORMAL e nada pode assumir monotonicidade.
- **Custo desconhecido ≠ custo zero**: campanha sem verba no período com
  "R$ 0 por agendamento" subia pro topo do ranking como a mais barata.
- **Nome da campanha vem do SYNC**, nunca do lead: o lead grava na entrada e
  não acompanha rename (medido: 354 leads num id apontando pra outra campanha).
- **O sync fica 1-2 dias atrás do CRM** — o período efetivo para no último dia
  sincronizado, senão o custo por agendamento sai mais barato do que é.
- Tokens CSS: ver [[gotcha_tokens_css_admin_vs_frontend]].

## Onde mexer

- **Ligar um cliente:** Hub → aba CRM → Conversões pro Meta. Escolhe o
  conjunto da lista (só os das contas DAQUELE cliente), marca os degraus,
  salva. O botão "Enviar evento de teste" confirma o acesso na hora — use
  sempre, porque a escrita nunca foi provada de ponta a ponta.
- **Ver todos:** `/admin/conversoes-meta` (Tráfego). Três estados: parado,
  enviando, desligado.
- **Alerta:** vai no cron diário do QI, por Telegram, roteado pro gestor da
  carteira. Não usa `ALERTA_ROTINAS_EMAIL` de propósito — ela pode estar vazia.

## Armadilhas que já custaram caro (todas corrigidas, todas com teste)

- **Falso positivo mata o alarme.** A vigilância media silêncio em 3 dias e
  cards numa janela fixa de 7: os mesmos cards que já geraram envios aceitos
  reapareciam como "nada saiu". Medido: 16,6% dos cliente-dias, 66,2% nos que
  só ligam agendamento. O corte tem que ser o ÚLTIMO ENVIO ACEITO, e só nos
  degraus LIGADOS.
- **Recusa não é envio.** Contar recusa como envio fazia cliente 100% quebrado
  exibir "Enviando".
- **Destino sem degrau** é o estado natural de quem acabou de configurar — não
  é "parou".
- **Venda não é vigiada**: o lead não carimba data de ganho.
- **O teste do gatilho era decorativo** — `payload.update` não passa pelo
  endpoint `/acao`, onde o gatilho mora. Use `chamarEndpoint`.

## Pendências

- **Rollup anúncio → campanha pro CTWA**: `criativos.campanhaId` existe pra
  isso e não foi usado. Hoje inócuo (1 lead de WhatsApp com id de anúncio),
  mas quando a captura do CTWA render, todos caem em "sem campanha".
- **Backfill de `criativos.campanhaId`**: ~172 mil linhas com nulo e o cron só
  re-sincroniza 3 dias. É `runSync` com `intervaloBackfill(hoje, 90)` — decisão
  do Renan, não rodado.
- **Nunca abri a tela num navegador.** O achado crítico dos tokens veio de
  análise, não de olhar. Vale conferir Hub → Performance com olho.
- `referral.source_type` não é checado: CTWA de post orgânico entra como
  anúncio inexistente.
