---
name: gotcha_tokens_css_admin_vs_frontend
description: "Na área de membros existem DUAS famílias de token CSS e elas não se cruzam: --pt-* só no /admin, --accent/--ink/--soft só no frontend. Usar a errada renderiza sem cor, sem erro nenhum."
metadata:
  node_type: memory
  type: reference
---

Descoberto em **22/10/2026**, na revisão do ranking de performance ([[project_conversoes_meta_cliente]]).

**Duas famílias, dois layouts, zero sobreposição:**

| família | definida em | carregada por |
|---|---|---|
| `--pt-destaque`, `--pt-texto`, `--pt-texto-fraco`, `--pt-borda-suave`, `--pt-card`, `--pt-fundo` | `src/app/(payload)/custom.scss` | layout do **/admin** |
| `--accent`, `--ink`, `--soft`, `--muted`, `--card-1`, `--border-soft` | `src/app/(frontend)/globals-quirk.css` | layout do **frontend** |

O CSS do próprio Payload não define nenhum dos dois conjuntos.

**O modo de falha é mudo.** Um componente do `/admin` que usa `var(--accent)`
renderiza **sem cor nenhuma** — `getComputedStyle(...).backgroundColor` volta
string vazia. Nenhum erro, nenhum aviso, nada no console. Aconteceu com a barra
do funil do ranking: a peça central da tela era espaço em branco.

**Por que conferir "o token existe" não basta:** ele existe — no arquivo
errado. A pergunta certa é *qual layout carrega o CSS onde ele está definido*.

**E o teste que deveria pegar passou.** Ele afirmava que o estilo
"contém `var(--`" e "não contém hex" — valida a FORMA da string, não a cor
resolvida. Passa para qualquer variável, inclusive uma que não existe ali.
O guarda que funciona é **lista branca** dos tokens daquele layout:

```ts
const TOKENS_DO_ADMIN = ['--pt-destaque', '--pt-texto', '--pt-texto-fraco', '--pt-borda-suave', '--pt-card']
const usados = container.innerHTML.match(/--[a-z0-9-]+/gi) ?? []
for (const t of usados) expect(TOKENS_DO_ADMIN).toContain(t)
```

Caso de [[feedback_teste_decorativo_mutacao]]: nome certo, caminho errado.

**Como verificar de qual família a tela é:** veja qual layout a envolve.
`src/components/admin/**` e as abas do Hub → `--pt-*`. `src/app/(frontend)/**`
→ a outra. Em dúvida, `grep -o "\-\-pt-[a-z-]*" src/app/\(payload\)/custom.scss | sort -u`.
