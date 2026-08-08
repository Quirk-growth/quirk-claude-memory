---
name: reference-uazapi-qr-expira
description: "GOTCHA UAZAPI — QR code de conexão do WhatsApp expira e é renovado do lado da UAZAPI; UI precisa reler a cada poll, não cachear a imagem do /conectar"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 635d4787-0e22-45b2-b202-ef558aebae16
  modified: 2026-08-08T22:28:00.522Z
---

O QR code de conectar o WhatsApp (`/instance/connect` da UAZAPI, usado no CRM em `src/collections/CrmConexoes.ts`) **expira e é renovado periodicamente do lado da UAZAPI** enquanto a instância está em `connecting`. `statusInstancia()` (`src/lib/crm/uazapi.ts`) devolve o `qrcode` atualizado nesse estado.

**Sintoma se a UI não reler:** exibe o QR uma única vez (da resposta do `/conectar`) e nunca atualiza a imagem. Se o usuário demora mais que a janela de validade pra escanear, o app do WhatsApp recusa com uma mensagem genérica — "não é possível conectar no momento, verifique se está tudo dentro da normalidade" — que NÃO vem do nosso código (não existe esse texto no repo), é o próprio WhatsApp rejeitando um QR morto.

**Fix (commit `ce6e4f4`, 2026-08-08):** `statusInstancia()` passou a devolver `qrcode` também; `situacaoFrescaDaConexao()` e o endpoint `/situacao` repassam; o polling de [[project_area_membros_quirk]] em `ConexaoWhatsapp.tsx` (`useConexaoWhatsapp`) agora troca a imagem do QR a cada tick de 6s em vez de deixá-la congelada.

**Como aplicar:** qualquer fluxo novo que exiba QR de conexão (WhatsApp ou outro provider com padrão parecido) deve reler o código periodicamente enquanto não conectado — nunca assumir que a primeira resposta continua válida até o usuário agir.
