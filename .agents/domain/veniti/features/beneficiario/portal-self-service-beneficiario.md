---
type: feature
module: beneficiario
layer: feature
related:
  - fluxo-reembolso-completo
  - fluxo-busca-prestador
  - solicitacao-reembolso
  - cadastro-beneficiario
  - login-multi-portal
---

# Portal Self-Service do Beneficiário

> Portal público acessível via link único enviado ao beneficiário, sem necessidade de conta cadastrada no sistema.

## Descrição

Portal público acessível via link único enviado ao beneficiário. Permite que o beneficiário acompanhe o status do atendimento ou reembolso, envie documentos, consulte prestadores disponíveis e interaja com o serviço sem necessidade de conta cadastrada no sistema.

---

## Entradas

| Elemento | Descrição |
|---|---|
| Link único (short URL) | Gerado pelo sistema internamente e enviado por SMS/e-mail |
| Token de acesso | Embutido no link — identifica o atendimento e o beneficiário |
| Documentos (upload) | Notas fiscais, fotos, documentos solicitados |
| Dados bancários/PIX | Para fluxo de reembolso |

## Saídas

- Visualização do status atual do atendimento/reembolso
- Upload de documentos armazenados no AWS S3
- Dados financeiros salvos no reembolso vinculado
- Confirmação visual por estado (aprovado, recusado, pago, etc.)
- Mapa com prestadores próximos (`html/beneficiario/a/search_provider.php`)

---

## Regras de Negócio

- O acesso é por link único — sem login com usuário e senha
- O link contém um token que identifica o atendimento e o beneficiário via short URL
- Links expirados (token inválido ou prazo vencido) mostram `link_expired.php`
- O beneficiário só pode editar informações enquanto o status é `NOTA_RECEBIDA`
- Uploads de documentos vão direto para AWS S3
- A busca de prestadores no mapa usa dados de geolocalização em tempo real (Leaflet.js + API)
- O portal não requer autenticação — a segurança está no unicidade e validade do token
- O estado exibido é determinado pelo `situacao` do reembolso vinculado
- Cada valor exibido (protocolo, valor, motivo) é carregado dinamicamente via `get_refund`

## Portal Views

| View | Path | Condição de exibição |
|---|---|---|
| `introduction.php` | `/reembolso/views/introduction.php` | Primeira visita ao link |
| `default_information.php` | `/reembolso/views/` | Reembolso aberto/pendente |
| `refund_details.php` | `/reembolso/views/` | Detalhes do reembolso |
| `refund_edit_info.php` | `/reembolso/views/` | Edição (status NOTA_RECEBIDA) |
| `refund_approved.php` | `/reembolso/views/` | Reembolso aprovado |
| `refund_rejected.php` | `/reembolso/views/` | Reembolso recusado |
| `refund_scheduled.php` | `/reembolso/views/` | Pagamento agendado |
| `refund_sent.php` | `/reembolso/views/` | Pagamento enviado/pago |
| `link_expired.php` | `/html/beneficiario/a/` | Link inválido ou expirado |
| `search_provider.php` | `/html/beneficiario/a/` | Busca de prestadores no mapa |
| `map_beneficiario.php` | `/html/beneficiario/a/` | Visualização de mapa |
| `aguardando_documentacao.php` | `/html/beneficiario/a/` | Aguardando docs (status específico) |
| `files_beneficiario.php` | `/html/beneficiario/a/` | Upload de arquivos |

## Casos de Borda

- Link compartilhado com terceiro (acesso indevido por conhecimento da URL)
- Beneficiário envia nota fiscal após o prazo máximo configurado
- Prestador buscado no mapa não está disponível no momento (dados desatualizados)
- Upload falha por arquivo acima do limite de tamanho
- Beneficiário acessa o link múltiplas vezes e altera dados bancários após agendamento
- Página carregada sem conexão (Progressive Web App behavior não suportado)

---

## Dependências

- **Portais**: `html/reembolso/` (fluxo de reembolso), `html/beneficiario/a/` (portal beneficiário)
- **Short URL**: sistema interno de geração de links curtos (`src/Models/ShortUrl.php`)
- **Maps**: Leaflet.js (`html/__mods/leafletjs/`), Google Maps API
- **Storage**: AWS S3 para documentos
- **Notificações**: SMS (`src/Models/SMS.php`), e-mail (PHPMailer)
- **Banco**: `reembolsos`, `arquivos`, `atendimentos`

## Flows Relacionados

- [[fluxo-reembolso-completo]]
- [[fluxo-busca-prestador]]

## Features Relacionadas

- [[solicitacao-reembolso]]
- [[cadastro-beneficiario]]
- [[login-multi-portal]]
