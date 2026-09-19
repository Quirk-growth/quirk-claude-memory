---
name: gotcha-hidratacao-fuso-admin
description: "React #418 na área de membros: Render roda em UTC e o navegador em BRT, então componente client que formata data sem fixar timeZone quebra a hidratação — helpers em formato.ts"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5467ce4b-95f4-4600-948e-6e15b8f0a0c1
  modified: 2026-09-19T15:15:50.060Z
---

**Render roda em UTC; o navegador do time roda em `America/Sao_Paulo`.** Qualquer componente **client** da [[project_area_membros_quirk]] que formatava data com `toLocaleDateString`/`toLocaleTimeString`/`toLocaleString` **sem `timeZone`** lia o fuso de QUEM renderiza. O mesmo timestamp entre 00:00 e 03:00 UTC — a janela das **21h à meia-noite** daqui — virava um dia no HTML do servidor e outro no primeiro render do cliente. O React compara `textContent` na hidratação, não bate, e **regenera a árvore inteira**: é o `Minified React error #418` do console de produção. Fora da virada, a hora ainda saía 3h errada dos dois lados.

**Corrigido em 19/09/2026** na branch `fix/hidratacao-fuso-admin` (commit `41b15555`, 26 arquivos): `src/lib/relatorios/formato.ts` ganhou `FUSO = 'America/Sao_Paulo'` e os helpers `data`/`dataCurta`/`dataAnoCurto`/`dataPorExtenso`/`hora`/`dataHora`/`dataHoraAnoCurto`/`dataCurtaHora`/`dataHoraCompleta`/`mesAno` + `diaISO`/`mesISO`/`diasEntre`, ao lado dos de moeda que já moravam lá. **Regra: componente client NUNCA formata data na mão — importa de `formato.ts`.**

**O mesmo erro tem um segundo sabor, sem hidratação:** `new Date().toISOString().slice(0,10)` pra "hoje" (default de `<input type=date>`, comparação de prazo) é o **dia UTC** — a partir das 21h daqui já é amanhã. Usar `diaISO()`. Foi o que fazia tarefa do dia nascer "atrasada" no servidor e em dia no navegador, e a grade do Calendário pôr o post na célula errada (`chaveDiaLocal`).

**Moeda NÃO tem esse problema** — `Intl.NumberFormat` não depende de fuso. O `norm()` de `formato.ts` (troca NBSP por espaço) é cosmético, não é correção de hidratação.

**Como reproduzir de verdade (não dá pra ver em dev local):** localmente servidor e navegador estão no MESMO fuso, então o bug some. Precisa subir o build de produção com `TZ=UTC` (`TZ=UTC npx next start`) e abrir no navegador em BRT. `tests/unit/formatoDataFuso.spec.tsx` faz isso offline: `renderToString` com `process.env.TZ='UTC'` + `hydrateRoot` com `TZ='America/Sao_Paulo'`, capturando `onRecoverableError`. Trocar `process.env.TZ` em runtime FUNCIONA no Node (avisa o V8 na hora).

⚠️ **`#418` em produção é SÓ mismatch de texto/elemento.** Conferido no `react-dom` de produção: a única checagem é `instance.textContent === "" + type`. **Atributo divergente não gera #418** — o aviso "some attributes of the server rendered HTML didn't match" existe apenas no bundle de DEV. Por isso o script anti-flash de tema do `NavQuirk`, que escreve `data-tema`/`data-theme` no `<html>` antes da hidratação, suja o console em dev mas **não é** o #418 de produção (segue em aberto, é ruído de dev).

⚠️ **`/admin/bi-financeiro` não tinha o bug** — é 100% server component. Quem vê #418 ali provavelmente carregou outra tela antes (o console guarda entre navegações do SPA). Telas realmente afetadas: Análises, Tarefas/Calendário, CRM (board e conversas), Hub do cliente, Propostas, Chat.
