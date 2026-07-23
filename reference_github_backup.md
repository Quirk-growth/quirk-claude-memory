---
name: reference_github_backup
description: "Estrutura de backup no GitHub (org Quirk-growth) — quais repos guardam o quê, auth SSH, e o que ainda falta blindar"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ecaae04f-fdc3-46e0-b5d4-982b4223e31a
---

Backup de código da Quirk vive na org **Quirk-growth** no GitHub (repos privados), acessado via **chave SSH ed25519** (`~/.ssh/id_ed25519`, sem passphrase) — autentica como "Quirk-growth". Identidade git: Renan Real / contato@quirkgrowth.com.br.

Repositórios (jul/2026):
- **quirk-auto-ads** ← pasta local `quirk_auto_ads/` (automação Make+n8n). Ver [[project_quirk_auto_ads]].
- **tdtc** ← `Desktop/TDTC/` (branches main, aba2, feat/conserto-instrumento). Ver [[project_tdtc]].
- **Calculadora-vgv-quirk** ← `lp-vgv-quirk/` (separado por ter motor de cálculo). Ver [[project_lp_vgv]].
- **LPs-quirk** ← monorepo `quirk-lps/` com subpastas: iscas, tech, auto-ads, nau-marina. LPs menores entram aqui como nova subpasta.
- **area-membros-quirk** ← já existia. Ver [[project_area_membros_quirk]].

- **quirk-claude-memory** ← backup destas próprias fichas de memória (`~/.claude/projects/-Users-renanreal/memory/`). **Auto-push configurado**: hook `Stop` em `~/.claude/settings.json` commita+empurra a memória pro GitHub ao fim de cada sessão (async). Numa máquina nova, clonar esse repo pra pasta de memória local restaura a continuidade.

Regra: código → GitHub; assets pesados/docs → Google Drive; NUNCA versionar `.env`/tokens/node_modules (já no .gitignore).

## Modelo de continuidade (o que dá pra "continuar de onde paramos")
Continuidade NÃO vem das conversas (transcripts ficam locais em ~/.claude/projects/*.jsonl, ~324M, NÃO sobem pra nuvem e NÃO seguem pra outra máquina — e não precisam). Vem de 3 camadas portáteis: (1) código → GitHub org Quirk-growth; (2) servidores/fluxos → contas na nuvem (Make/n8n/Meta), só logar; (3) memória → repo quirk-claude-memory. O que sincroniza é a MEMÓRIA CURADA (estas fichas), não o chat cru.

## Setup numa máquina nova (Mac ou Windows) — restaurar continuidade
Passos únicos, uma vez por máquina:
1. Instalar Claude Code + `git clone` dos repos da org Quirk-growth.
2. Clonar `quirk-claude-memory` PRA DENTRO da pasta de memória local daquela máquina (o caminho muda por SO/projeto — no Windows é diferente; conferir o path real antes).
3. Recriar o hook `Stop` de auto-push no `settings.json` DAQUELA máquina (o hook é config local, NÃO viaja junto). Sem isso, a memória das conversas naquela máquina NÃO sobe.

## Sincronização entre máquinas (CUIDADO)
O auto-push é por-máquina (config local). Numa máquina nova só sobe se o hook for recriado lá. Risco de conflito git se editar memória em 2 máquinas sem dar `git pull` antes — para uso solo, disciplina "pull no início / push no fim" resolve. Quando ele realmente configurar a 2ª máquina, considerar adicionar hook SessionStart que faz `git pull --rebase` na memória.

Pendências de blindagem (Dia 0 incompleto):
- 🔴 **Mac sem backup** — Time Machine com zero destinos configurados. Falta HD externo ou Backblaze.
- Duplicação: as pastas originais (lp-iscas-quirk, lp-quirk-tech, lp-quirk-auto-ads, lp-nau-marina) ainda existem separadas das cópias dentro de quirk-lps/ — decidir fonte da verdade.
- ~15 projetos ainda sem git (avaliador-copys, curso-cpl, apresentacao-social, manual-comercial, quirk-painel-relatorios, etc.).
- Backup dos blueprints Make/n8n como JSON ainda não feito.
