---
name: reference_papel_social_media
description: "Papel 'social' (Social Media) na área de membros — carteira própria socialMedias, Hub restrito, isolamento; como ativar + a lição de vazamento cross-tenant"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 74e3c39b-64af-48ce-9da1-ebcfb16c3a2b
  modified: 2026-08-12T01:44:47.353Z
---

Papel novo `role: 'social'` (Social Media) na área de membros, no ar ago/2026 (commit 30384eb). Entra no /admin como time, mas com visão RESTRITA à **carteira própria** — campo `clientes.socialMedias` (relationship hasMany→users, espelha `gestores`, `filterOptions` role=social). **Ativar:** criar user role=Social Media + preencher "Social medias responsáveis" nos clientes da carteira dele.

O que vê: menu **Início** (Visão geral + Mensagens) + **Social** (Clientes + Relatórios dos clientes); Hub só dos clientes da carteira, 6 abas (resumo/relatorio/conta/briefing/acesso/links — vincula o igUserId na Conta, Acesso é read-only pro social); portfólio + relatório escopados; Mensagens = a MESMA do time. NÃO vê: Financeiro, CRM, Funil, Cadastro, Rotina, Tarefas.

**Identificação na Lista de clientes + Hub (ago/2026, commit 9dd3bad):** cada cliente mostra os responsáveis como **chips foto+primeiro nome** — gestor azul (`--pt-destaque-suave`/`--pt-destaque`), social rosa (`--pt-rosa-fundo`/`--pt-rosa-texto`, já nos 2 temas). Componente reusável `ChipPessoa`/`ChipsResponsaveis` (`src/components/comum/ChipPessoa.tsx`). Clicar abre o MESMO popover das menções (`PerfilPopover` → Ver perfil + Chat) — mas o `PopoverPessoaProvider` NÃO é global (montado por feature: CRM, Anotações, Comentários, e agora a tabela da lista + ilha do cabeçalho do hub). Pro papel social os chips são estáticos (`podeAbrir=ehEquipe`, e `/api/pessoas/perfil` é equipe-only). Filtros da lista: gestor (azul) + social (rosa, com "Sem social"). `dadosDaListaHub`/`carregarClienteDoHub` resolvem nome+foto em lote (`resolverPessoasChip`, overrideAccess escopado). `PessoaView` (`/admin/pessoa`) ganhou seção **Clientes** (carteira: gestor OU social) + rótulo "Social Media". Tipos `LinhaLista`/`ClienteDoHub` mantêm `gestor`/`gestorNome` (contrato dos testes) e ganharam `gestores`/`socialMedias: PessoaChip[]`.

**Design de isolamento (chave):** o social NÃO foi adicionado a `ehEquipe`/`PAPEIS_EQUIPE`/`isGestorOuDonoCliente` — assim fica **fail-closed** nas coleções sensíveis (Vendas/financeiro/CRM); lê só pelas páginas server-side (overrideAccess) escopadas por `clientesVisiveis` (que ganhou o ramo `social` → `idsDaCarteiraSocial`). `podeGerenciarCliente` idem. Chat usa `podeUsarChat` (=ehEquipe||social) + `usuariosParaChat`, SEPARADOS das consts de coleção. Entrada no /admin via `podeEntrarNoAdmin`. Enum `role` = pgEnum ([[reference_payload_select_enum_ddl]]): precisou `ALTER TYPE ADD VALUE 'social'` em prod; `socialMedias` reusa `clientes_rels` (sem coluna nova).

**LIÇÃO (apareceu 3× no review):** liberar uma tela/aba pra um papel restrito novo pode **vazar dado cross-tenant** por listas internas que assumem 'todos' pra não-gestor — pegou em: gate da home (PAPEIS_COM_HOME), aba Acesso (escopoRestrito → lista de logins de OUTROS clientes), e o link Clientes. Ao dar acesso a uma tela pra papel de carteira, SEMPRE checar as listas/contadores DENTRO dela e escopar por `clientesVisiveis`. Relacionado: [[feedback_isolamento_dados_clientes]]. Futuro: aba Social/Calendário editorial (reservado, spec própria).
