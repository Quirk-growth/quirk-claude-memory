---
name: reference_ytdlp_4k
description: Receita que funciona pra baixar YouTube em 4K/alta resolução com yt-dlp no Mac do Renan (contorna SABR + JS challenge)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 54ef7949-9baf-4569-88bc-8d9a59c12fca
  modified: 2026-07-28T22:16:57.977Z
---

Baixar YouTube em 4K no Mac do Renan (jul/2026). O bloqueio novo do YouTube tem DUAS camadas e a segunda é a que quase todo mundo esquece:

1. **SABR streaming** — sem cookies, dá erro "The page needs to be reloaded"; formatos DASH somem. Resolver com `--cookies-from-browser chrome`.
2. **JS challenge (a chave do 4K)** — a partir de ~2026 o YouTube exige um runtime JavaScript pra liberar os formatos DASH acima de 1080p. Sem ele, só aparece HLS/H.264 até 1080p (parece que o vídeo "não tem 4K", mas TEM). O Mac tem **node** (`/usr/local/bin/node`, v24). Passar `--js-runtimes node` faz aparecer 1440p/2160p em VP9 (313) e AV1 (401).

**Ferramenta:** o yt-dlp do pip (out/2025) é velho demais e falha no JS challenge. Usar o **binário nightly** (jul/2026):
`~/Downloads/... ` — baixei em `scratchpad/yt-dlp_bin` de `github.com/yt-dlp/yt-dlp-nightly-builds/releases/.../yt-dlp_macos`. ffmpeg já está em `~/.local/bin/ffmpeg`.

**Comando que funcionou** (4K VP9 + Opus → MKV):
```
./yt-dlp_bin --js-runtimes node --cookies-from-browser chrome \
  -f "313+251/bestvideo[height=2160][vcodec^=vp9]+bestaudio" \
  --merge-output-format mkv -o "$HOME/Downloads/%(title)s [4K].%(ext)s" URL
```
Genérico: trocar seletor por `bestvideo[height<=2160]+bestaudio`. Pra MP4 usar AV1 (401) mas o áudio só vem em Opus (mp4+opus é irregular; MKV é mais seguro).

Detalhe: o binário nightly NÃO carrega o plugin bgutil-pot (ignora `--plugin-dirs`), mas com `--js-runtimes node` ele gera o PO token nativo (yt_dlp_ejs) e nem precisa do plugin.
