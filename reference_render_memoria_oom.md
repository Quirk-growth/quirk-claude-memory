---
name: reference_render_memoria_oom
description: Área de membros no Render — instância Standard 2GB (era 512MB e dava OOM/502); start precisa de --max-old-space-size; acompanhar deploy até verde
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-11T00:54:44.724Z
---

O serviço `area-membros-quirk` no Render (srv-d9a34jucjfls73969rp0, blueprint `render.yaml`) rodava em **Starter 512 MB** e cresceu até OOMar: o Node capa o heap em ~256 MB num container de 512 MB e o processo morre com `FATAL ERROR: Reached heap limit — JavaScript heap out of memory` → **502 em tudo** (não é bug de código; build e boot passam local). Aconteceu no deploy da Fase 2a do relatório social (10/ago/2026): o footprint extra empurrou por cima do teto. Resolvido: **upgrade pra Standard (2 GB, 1 CPU, $25/mês)** + fix no `package.json`: o `start` NÃO tinha `--max-old-space-size` (só `--no-deprecation`), agora `NODE_OPTIONS="--no-deprecation --max-old-space-size=1536"`; e `render.yaml` `plan: standard` (senão o blueprint reverte pra 512 MB). O `build` já usava `--max-old-space-size=8000`.

Regras daqui pra frente: (1) o app é faminto de memória — não voltar pra 512 MB; (2) diagnóstico de 502 vive no **Deploy/Application logs** do Render (OOM aparece lá, não no build); rollback = `git push --no-verify --force-with-lease origin <commit-bom>:main` (o pre-push hook dá falso-positivo); (3) SEMPRE **acompanhar o deploy de prod até o health check verde** antes de declarar pronto ([[feedback_acompanhar_deploy_ate_verde]]). Relacionado: [[reference_neon_bancos]], [[project_area_membros_quirk]].
