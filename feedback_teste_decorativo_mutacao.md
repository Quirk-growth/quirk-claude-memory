---
name: feedback_teste_decorativo_mutacao
description: Teste com o nome certo e o caminho errado é o defeito mais comum aqui — só aparece por mutação; sete casos numa sessão só
metadata:
  type: feedback
---

O defeito de teste mais frequente neste projeto não é teste faltando: é **teste com o nome certo que não protege o caminho certo**. Em uma única sessão (22-23/09/2026) apareceram **sete**, todos achados do mesmo jeito — quebrando o código de propósito e vendo se alguém reclamava. Nenhum apareceria numa leitura de código.

**A regra:** antes de dizer que algo está coberto, **reintroduza a falha que o teste alega detectar e confirme que ele FALHA**. Se continuar verde, o teste é decoração. Isso vale inclusive — e principalmente — para teste escrito para fechar um achado de teste decorativo: dois dos sete foram exatamente isso.

**Os sete mecanismos reais, porque eles se repetem:**

1. O teste inspeciona **o lugar errado** — olhava `logger.error`, mas o dado sensível ia na mensagem passada a outra função, que estava mockada.
2. O nome promete **múltiplas ocorrências**, o cenário monta **uma** — o caminho usado só consegue produzir uma; a flag `g` da regex ficava destravada.
3. O contrato **não tinha teste nenhum** — removi a chamada inteira e 11 testes seguiram verdes.
4. O teste prova a **ordem das instruções**, não a proteção — "falha no envio não impede o card de mover" passava sem o `.catch()`, porque a gravação já acontecia antes.
5. **Eu mesmo escrevi um decorativo tentando consertar o 4** — escutar `unhandledRejection` não funciona sob Vitest, que intercepta o tratamento de promises. Passava com e sem a proteção.
6. A **outra trava bloqueia primeiro** — teste de eco com a chave desligada: a criação do lead já era barrada pela chave, por motivo alheio ao eco. Passava mesmo removendo o tratamento de eco inteiro.
7. A asserção é **inalcançável por construção** — o mapeador só reconhece um tipo de evento, então o corpo do teste caía em "não reconhecido" antes de chegar ao código sob prova.

**Quando a mutação não é possível** (Vitest interceptando promises, comportamento externo): varredura do código-fonte é feia mas honesta, e já é padrão da casa (`envDocumentadas.spec.ts`, o anti-esquecimento do `avisarFalha`). Melhor uma trava explícita e sem charme do que um teste elegante que nunca falha.

Relacionado: [[feedback_medir_antes_de_corrigir]], [[reference_suite_testes_area_membros]].
