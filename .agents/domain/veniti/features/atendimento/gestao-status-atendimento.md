---
type: feature
module: atendimento
layer: feature
related:
  - fluxo-atendimento-completo
  - criacao-atendimento
  - ciclo-vida-acionamento
  - solicitacao-reembolso
---

# Gestão de Status do Atendimento

> Controla o ciclo de vida de um atendimento através de transições de status.

## Descrição

Controla o ciclo de vida de um atendimento através de transições de status. O status reflete a situação operacional atual do serviço e determina quais ações estão disponíveis para cada perfil de usuário (operação interna, cliente, prestador).

---

## Entradas

| Campo | Descrição |
|---|---|
| `id_atendimento` | Identificador do atendimento |
| `status` | Novo status a ser aplicado |
| `obs` | Observação/justificativa da mudança (obrigatória em alguns estados) |
| `numero_bo` | Boletim de ocorrência (obrigatório para NEGADO em alguns casos) |

## Saídas

- Status atualizado na tabela `atendimentos`
- Registro no `log` de auditoria com usuário e timestamp
- Notificação para o cliente via `NotificacaoCliente` (dependendo do status)
- Notificação interna para CCO via `NotificacaoCCO` (estados críticos)
- Evento enviado ao TimescaleDB para histórico temporal
- Atualização refletida em tempo real nos dashboards via WebSocket (Pusher)

---

## Regras de Negócio

- Somente usuários com permissão de operação podem alterar o status
- Transições de status geram entrada obrigatória no log de auditoria
- O status `NEGADO` normalmente exige justificativa via campo `obs`
- Um atendimento `FINALIZADO` não pode voltar para estados anteriores
- O status `EMBUSCA` pode ser ativado quando o acionamento é aceito pelo prestador — **comportamento não confirmado**; verificar se ocorre automaticamente ou requer ação manual, e se se aplica a todos os tipos de atendimento ou apenas ao tipo Assistência
- O campo `refund_status` controla a situação do reembolso vinculado, independente do status do atendimento
- Atendimentos com `bloquear_acionamento = true` ficam em `ABERTO` indefinidamente até liberação manual
- Observadores (`ObserverChangeService`) são disparados em toda mudança de status para notificações externas

## Estados

O fluxo de status varia conforme o subtipo do atendimento. Os fluxos abaixo descrevem o comportamento observado na UI para atendimentos do tipo **Assistência**:

**Assistência — Veicular:**
```
ABERTO → AGUARDANDO ACEITE → ACEITO → ORIGEM → DESTINO → FINALIZADO
       ↘ CANCELADO
       ↘ NEGADO
```

**Assistência — Beneficiário / Pet:**
```
ABERTO → AGUARDANDO ACEITE → ACEITO → ORIGEM → FINALIZADO
       ↘ CANCELADO
       ↘ NEGADO
```

> ⚠️ **Ambiguidade:** O status `EMBUSCA` pode pertencer ao fluxo de atendimentos do tipo **Roubo/Furto** — não confirmado. Os status `RECUPERADO`, `NAO RECUPERADO` e `VALIDACAO` listados abaixo são referenciados no código mas seu fluxo por tipo de atendimento não está mapeado.

| Status | Contexto Provável | Significado Operacional |
|---|---|---|
| `ABERTO` | Todos os tipos | Atendimento registrado, aguardando acionamento |
| `AGUARDANDO ACEITE` | Assistência (veicular / benef. / pet) | Acionamento criado, prestador notificado |
| `ACEITO` | Assistência (veicular / benef. / pet) | Prestador confirmou o serviço |
| `ORIGEM` | Assistência (veicular / benef. / pet) | Prestador chegou ao endereço de origem |
| `DESTINO` | Assistência (veicular) | Prestador chegou ao destino final |
| `FINALIZADO` | Todos os tipos | Serviço concluído |
| `CANCELADO` | Todos os tipos | Atendimento cancelado |
| `NEGADO` | Todos os tipos | Solicitação negada (sem cobertura, fraude, etc.) |
| `EMBUSCA` | Roubo/Furto (não confirmado) | Possivelmente: equipe em busca do veículo |
| `RECUPERADO` | Roubo/Furto (não confirmado) | Veículo/beneficiário recuperado |
| `NAO RECUPERADO` | Roubo/Furto (não confirmado) | Encerrado sem recuperação |
| `VALIDACAO` | Não confirmado | Atendimento em análise interna |

## Casos de Borda

- Tentar finalizar um atendimento sem acionamento vinculado
- Cancelamento após o prestador já ter chegado ao local (`ORIGEM`)
- Atendimento marcado como `RECUPERADO` sem veículo localizado
- Mudança de status por dois operadores simultaneamente (race condition)
- Atendimento em `VALIDACAO` sem resolução por tempo prolongado

---

## Dependências

- **Modelos**: `src/Models/Atendimento.php`, `src/Observers/ObserverChangeService.php`
- **Notificações**: `src/Models/NotificacaoCliente.php`, `src/Models/NotificacaoCCO.php`
- **Eventos**: `src/Models/ConnectionTimescaleDB.php` (registro temporal)
- **Real-time**: `src/Helpers/WebSocket.php` (Pusher)
- **Integrações**: `src/Integracoes/AlteracoesAtendimento/` (notifica sistemas externos como Agiliza, MaxPar)

## Flows Relacionados

- [[fluxo-atendimento-completo]]

## Features Relacionadas

- [[criacao-atendimento]]
- [[ciclo-vida-acionamento]]
- [[solicitacao-reembolso]]
