---
name: reference-neon-bancos
description: "Bancos Neon da área de membros/painel Quirk — produção vs teste separados, e a lição da cota compartilhada"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-07-24T22:46:46.275Z
---

Postgres da [[project-area-membros-quirk]] / [[project-painel-relatorios]] é **Neon**, org "Quirk Growth" (plano **Launch/pago** desde 24/jul/2026).

**Dois projetos Neon SEPARADOS (importante):**
- **Produção:** projeto "Area de Membros - Quirk", host `ep-patient-hill-acxg3aix.sa-east-1` (São Paulo), banco `neondb`. É o `DATABASE_URI` no `.env`.
- **Testes:** projeto "area-membros-testes", host `ep-rough-thunder-acyzl9k2-pooler.sa-east-1` (São Paulo), banco `neondb`. É o `DATABASE_URI_TEST`. Criado 24/jul.

**LIÇÃO (incidente 24/jul):** antes, o banco de teste (`neondb_test`) ficava no MESMO projeto Neon da produção. A **cota de transferência do Neon é POR PROJETO** — então rodar a suíte (dezenas de `npx vitest run`, cada um puxa schema + ~142 testes de integração pela rede) consumiu a cota compartilhada e **derrubou a produção** (todas as rotas que tocam o banco → 500 "exceeded the data transfer quota"; só `/login`, que é client component, respondia). Resolvido: Renan subiu pro plano Launch (volta na hora) + separamos o teste num projeto Neon próprio. `vitest.setup.ts` já recusa rodar se `DATABASE_URI_TEST === DATABASE_URI`.

**Regra prática:** teste nunca deve compartilhar projeto Neon com produção. Ideal futuro: Postgres local (Renan recusou instalar Postgres.app em jul/26). Sem Postgres/Docker/brew na máquina do Renan.

**Pendência de higiene:** a senha do projeto de teste (`npg_...`) passou pelo chat — rotacionar no Neon (Reset password do projeto area-membros-testes). Não urgente (banco descartável, sem dado real).

**Regra de credencial mantida:** eu NÃO escrevo strings de conexão com senha / tokens no `.env` — o Renan cola. (String de teste tem senha embutida = credencial.)
