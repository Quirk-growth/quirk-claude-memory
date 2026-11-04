---
name: reference_r2_credenciais
description: Chave do Cloudflare R2 recusada (401) desde 24/10/2026 derrubou fotos, uploads e transcrições; como diagnosticar, trocar e o que já avisa sozinho
metadata:
  type: reference
---

O bucket R2 `quirk-membros` guarda TODA a mídia (coleção `media` via s3Storage: fotos de perfil, banner, aulas, anexos/áudios das Conversas do CRM, posts, onboarding, miniaturas do relatório social, prints do LinkedIn) E o áudio das calls (`transcricoes/<id>/<canal>/`). Mesma chave pra tudo: `R2_ACCESS_KEY_ID`/`R2_SECRET_ACCESS_KEY` no Render (serviço area-membros-quirk).

**Incidente (descoberto 04/11/2026):** a chave foi recusada (HTTP 401 "Unauthorized") ≈ 24/10 05h. Sintomas: 15 calls seguidas "falhou/Unauthorized" (26/10→03/11, áudio provavelmente nunca subiu), fotos de perfil sumiram no Hub, `media` sem nenhuma linha nova desde 24/10 04:56. Causa provável: token expirado/revogado (só o painel da Cloudflare diz; o Chrome do Renan NÃO estava logado lá). Correção é DELE (criar token "Leitura e escrita de objetos" no bucket, TTL "Para sempre", trocar só as 2 variáveis no Render sem apagar as outras; criar/colar chave é proibido pra mim). Passo a passo em `docs/operacao/r2-credenciais.md`.

**Como verificar (estado é volátil, não confiar nesta nota):** `npx tsx scripts/r2-saude.mts` (grava+relê objeto de teste e lê uma foto de perfil; precisa do `.env` local com a chave nova), depois `npx tsx scripts/transcricoes-requeue.mts` (simulação; `--aplicar` recoloca na fila só as calls com áudio no R2).

**O que já avisa/protege (82c4ad9e, 04/11):** abrir gravação testa o R2 e barra com 503 + e-mail ao admin (rotina `transcricoes/armazenamento`); cron `transcricoes-processar` deixa a call em `processando` e dá 503 se a chave for recusada; cron de sessões órfãs não descarta mais por erro de listagem. Detecção de credencial: `ehErroDeCredencialR2` (`src/lib/transcricoes/saudeArmazenamento.ts`).

**Lição:** o relatório de uso mostrava só o custo; o primeiro sinal real foi o gestor reclamar. Quando "tudo que sobe arquivo" para de uma vez, olhar a chave do R2 primeiro (rodar a consulta de `media` por dia mostra o dia exato da quebra).

**Desfecho (04/11/2026):** Renan trocou a chave no Render às ~10:30; `/api/health/armazenamento` passou a dar `{"storage":"up"}` (rota nova, b94921c0, pública, só leitura, 200/503 — apontar UptimeRobot nela). As 15 calls (#31–#45) foram recolocadas em `processando` e TODAS voltaram `falhou` com "Nenhum áudio foi guardado": o R2 listou sem erro e não achou nenhum pedaço, então o áudio nunca subiu (chave morta na gravação) — PERDIDAS, sem recuperação. Falta confirmar: fotos de perfil voltaram no Hub (ele confere a olho) e a escrita no R2 (1º upload/sincronia da madrugada ou 1ª gravação de call).
