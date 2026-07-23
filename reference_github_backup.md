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

Pendências de blindagem (Dia 0 incompleto):
- 🔴 **Mac sem backup** — Time Machine com zero destinos configurados. Falta HD externo ou Backblaze.
- Duplicação: as pastas originais (lp-iscas-quirk, lp-quirk-tech, lp-quirk-auto-ads, lp-nau-marina) ainda existem separadas das cópias dentro de quirk-lps/ — decidir fonte da verdade.
- ~15 projetos ainda sem git (avaliador-copys, curso-cpl, apresentacao-social, manual-comercial, quirk-painel-relatorios, etc.).
- Backup dos blueprints Make/n8n como JSON ainda não feito.
