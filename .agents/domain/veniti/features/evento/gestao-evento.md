---
type: feature
module: evento
layer: feature
related:
  - atendimento-lifecycle
  - fluxo-atendimento-completo
  - criacao-atendimento
  - vinculo-plano-beneficiario
  - ciclo-vida-acionamento
---

# Gestão de Evento

> O Evento é a entidade raiz do ciclo operacional do Veniti, representando o incidente ou ocorrência reportada pelo beneficiário.

## Descrição

O **Evento** é a entidade raiz do ciclo operacional do Veniti. Representa o incidente ou ocorrência reportada pelo beneficiário (ex: colisão, pane, roubo). É o container pai que agrupa um ou mais Atendimentos. Todo serviço de assistência começa com a abertura de um Evento.

**Hierarquia:** `Evento → Atendimento(s) → Acionamento(s)`

---

## Entradas

| Campo | Tipo | Descrição |
|---|---|---|
| `id_beneficiario` | integer | Beneficiário que reportou o incidente |
| `id_tipo_evento` | integer | Tipo do evento (colisão, furto, pane, etc.) |
| `id_plano` | integer | Plano do beneficiário |
| `id_veiculo` | integer | Veículo envolvido (se aplicável) |
| `id_pet` | integer | Pet envolvido (se aplicável) |
| `solicitante` | string | Nome de quem está solicitando (pode diferir do titular) |
| `solicitante_telefones` | string | Telefone(s) de contato |
| `solicitante_email` | string | E-mail de contato |
| `obs` | text | Observações iniciais do incidente |
| `data_agendamento` | datetime | Para eventos agendados (não imediatos) |

## Saídas

- Registro criado na tabela `eventos`
- `protocolo` gerado automaticamente no formato `YYYYMM` + 6 dígitos aleatórios
- Status inicial: `ABERTO`
- Evento visível no painel de operação: `/assistencia/operacao/eventos.php`
- Pode ter zero ou mais `Atendimentos` vinculados
- Evento `AttendanceCreated` propagado via Observer (`SplSubject`)

---

## Regras de Negócio

- Um Evento representa **o incidente** — pode exigir múltiplos tipos de serviço (ex: reboque + assistência médica no mesmo evento)
- Cada serviço distinto é modelado como um **Atendimento** separado dentro do mesmo Evento
- O `protocolo` do Evento é o número de referência apresentado ao beneficiário
- Um Evento pode existir **sem atendimentos** (caso de registro de incidente sem execução de serviço)
- O painel de operação permite filtrar "Eventos sem atendimentos" — indicando eventos não atendidos
- O `id_tipo_evento` determina a natureza do incidente (configurado em `/assistencia/tipos_eventos/`)
- O Evento é vinculado ao `id_veniti` (tenant) — multi-tenant por design
- O Observer pattern (`SplSubject`) é implementado — mudanças no Evento notificam listeners registrados
- O método `generate_protocol()` em `src/Models/Eventos.php` garante unicidade do protocolo

## Estados

| Status | Descrição |
|---|---|
| `ABERTO` | Evento ativo — atendimentos podem ser criados |
| `FINALIZADO` | Todos os atendimentos concluídos |
| `CANCELADO` | Evento cancelado sem execução |

## Casos de Borda

- Evento criado sem beneficiário identificado (beneficiário não localizado nas bases)
- Evento duplicado para o mesmo beneficiário no mesmo momento (double-click ou retry)
- Evento do tipo que não possui prestadores cadastrados para atendimento
- Evento agendado com data no passado
- Evento com múltiplos atendimentos onde um é finalizado e outro é cancelado — qual o status final do Evento?
- Tipo de evento não mapeado para nenhum tipo de serviço disponível no plano

---

## Notas de QA

- **Risco:** O painel lista "Eventos sem atendimentos" — possível gap de atendimento não monitorado ativamente (SLA breach)
- **Risco:** Protocolo é gerado com componente aleatório — verificar se há garantia de unicidade em concorrência alta
- **Comportamento unclear:** Quando o Evento fecha automaticamente vs. requer fechamento manual
- **Validation gap:** Não está claro se o sistema valida cobertura do plano antes de criar o Evento ou somente ao criar o Atendimento
- **Edge case crítico:** Evento com `data_agendamento` → cronjob processa corretamente se o sistema estiver indisponível no horário?

## Dependências

- **Modelo**: `src/Models/Eventos.php` (SplSubject, generate_protocol)
- **Portal**: `html/assistencia/operacao/eventos.php`
- **Configuração**: `html/assistencia/tipos_eventos/` (tipos de evento)
- **Banco**: tabelas `eventos`, `tipos_eventos`
- **Observer**: `src/Domain/Attendance/Events/AttendanceCreated`

## Flows Relacionados

- [[atendimento-lifecycle]]
- [[fluxo-atendimento-completo]]

## Features Relacionadas

- [[criacao-atendimento]]
- [[vinculo-plano-beneficiario]]
- [[ciclo-vida-acionamento]]
