---
name: reference_agenda_time_grid
description: "Agenda na área de membros — grid de horário do time em colunas, vínculo só pelo perfil, orientação de link PÚBLICO iCal; arquivos e gotchas"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-12T20:26:59.699Z
---

Feature de agenda da área de membros, refeita ago/2026 (commit `b7a5559`, sem DDL).

**Vínculo:** cada pessoa vincula a PRÓPRIA agenda por um link **iCal PÚBLICO** (não o secreto — o secreto foi a orientação errada que não trazia os eventos reais). Passo certo no Google: Configurações da agenda → Acesso e permissões → "Disponibilizar publicamente" → Integrar agenda → copiar "Endereço público no formato iCal". Dica: criar uma agenda separada só de trabalho e publicar só ela. O controle vive SÓ no **perfil** (`PessoaView`, card "Agenda", guardado por `ehMeuPerfil` + `PAPEIS_COM_AGENDA`) — reusa `ConectarAgendaForm` (ações conectar/trocar/desconectar/sincronizar via `/api/agenda/minha`). A rota antiga `/admin/minha-agenda` (`MinhaAgendaView`) agora só **redireciona** pro próprio perfil; o item do menu (rodapé Configurações, `menuAdmin.ts` ITENS_CONFIGURACOES) virou **"Meu perfil"** (`ti-user`).

**Agenda do time** (`AgendaTimeView`, supervisor+): deixou de ser lista em linha e virou **grid de horário em colunas** — `AgendaTimeGrid.tsx` (client). Coluna por pessoa, eventos posicionados por início→fim; filtro/seleção de pessoas (chips) + rolagem horizontal, **horários livres em comum** (verde), **linha do "agora"** (vermelha, só se é hoje), **clique no evento → detalhes** (overlay). Quem não vinculou fica listado embaixo.

**Onde fica a lógica:** `src/lib/agenda/grid.ts` (PURO, testado em `tests/unit/agendaGrid.spec.ts` 8/8): `unirIntervalos`, `livresComuns(ocupados, janela, minimoMin=30)` = complemento da união dos ocupados, `janelaDoGrid` (padrão 8h–18h, estica pra caber eventos). Trabalha em MINUTOS do dia. Conversão Date→minutos SP em `src/lib/agenda/dia.ts` `minutosNoDia(d, dia)` (usa `diaLocal`/`horaLocal`; gruda em 0/1440 fora do dia). `EventoDoDia` ganhou `fim`. O server (`AgendaTimeView`) serializa cada evento como `{titulo, local, horario, diaTodo, inicioMin, fimMin}` (nada de Date cruzando pro client).

**Gotchas:** (1) não dá pra validar o grid no browser daqui (admin exige login) — validação foi tsc + unit + esboço aprovado; conferir posicionamento/altura/faixa de horas em prod. (2) eventos sobrepostos da MESMA pessoa hoje se sobrepõem visualmente (sem algoritmo de lanes) — melhoria futura. (3) CSS `.agenda-grid*` em `custom.scss`, cores por pessoa via paleta translúcida (funciona nos 2 temas). Relacionado: a agenda do Renan que "não aparecia" também tinha o bug do Make [[reference_agendamento_reuniao_make]] (conexão Google na conta errada) — são coisas diferentes.
