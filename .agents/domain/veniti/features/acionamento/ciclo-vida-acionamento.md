# Ciclo de Vida do Acionamento

## Description

Um acionamento (dispatch) representa a atribuição de um atendimento a um prestador de serviço específico. É a ponte operacional entre a demanda do beneficiário e a execução do serviço em campo. Um atendimento pode ter múltiplos acionamentos ao longo do seu ciclo (recusas, cancelamentos, novas tentativas).

## Inputs

| Campo | Tipo | Descrição |
|---|---|---|
| `id_atendimento` | integer | Atendimento a ser despachado |
| `id_prestador` | integer | Prestador selecionado |
| `id_prestadores_viatura` | integer | Veículo do prestador |
| `id_prestadores_bases` | integer | Base de origem do prestador |
| `tipo_acionamento` | string | Tipo (WEB, automático, etc.) |
| `km_acionamento` | decimal | Distância da base ao local |
| `km_total` | decimal | Distância total estimada do serviço |
| `data_agendamento` | datetime | Para acionamentos agendados |
| `tariffs` / `tarifas` | array | Tarifas aplicáveis |
| `acionamento_tempo_chegada` | integer | Tempo estimado de chegada (min) |
| `acionamento_automatico` | boolean | Se foi criado pelo sistema automático |

## Outputs

- Registro criado na tabela `atendimentos_dispatch`
- Notificação push/SMS enviada ao prestador
- Status do atendimento atualizado conforme o subtipo — ver [[gestao-status-atendimento]]
- Cálculo de crédito/débito gerado ao finalizar (`src/UseCases/CalcularValorCredito.php`)
- `dispatch_public_id` gerado para rastreamento externo
- Informações de veículo e técnico retornadas ao portal do cliente

## Status States

> **Nota:** Estes são os status do **acionamento** (`atendimentos_dispatch`). O status do **atendimento** (`atendimentos`) segue fluxo paralelo e distinto — ver [[gestao-status-atendimento]].

```
[criado] → AGUARDANDO → ACEITO → ORIGEM → DESTINO → FINALIZADO
                      ↘ RECUSADO → [novo acionamento criado]
                      ↘ NAO ATENDEU
                      ↘ TEMPO EXPIROU → [expirado por cronjob]
                      ↘ NAO DISPONIVEL
           CANCELADO (qualquer estado)
           ACEITO POR OUTRO PRESTADOR
```

| Status | Significado |
|---|---|
| `AGUARDANDO` | Notificação enviada, aguardando resposta do prestador |
| `ACEITO` | Prestador confirmou o serviço |
| `ORIGEM` | Prestador chegou ao endereço de origem |
| `DESTINO` | Prestador chegou ao destino final |
| `FINALIZADO` | Serviço concluído e registrado |
| `RECUSADO` | Prestador recusou (novo acionamento pode ser tentado) |
| `NAO ATENDEU` | Prestador não respondeu à notificação |
| `TEMPO EXPIROU` | Timeout de aceitação atingido |
| `NAO DISPONIVEL` | Prestador indisponível no momento |
| `CANCELADO` | Acionamento cancelado pela operação |
| `ACEITO POR OUTRO PRESTADOR` | Em modo broadcast, outro prestador aceitou primeiro |

## Business Rules

- Somente atendimentos com status `ABERTO` e `bloquear_acionamento = false` podem ser acionados
- O campo `beneficiario_pagara_excedente` define se o excedente de km é cobrado ao beneficiário
- Timestamps de cada transição são registrados: `TAceite`, `TOrigem`, `TDestino`, `TFinalizado`, `TRecusa`
- Cronjob `auto_expires_dispatch.php` expira automaticamente acionamentos sem resposta no tempo limite
- Dois métodos de despacho automático existem: **Bouncing** (sequencial) e **Broadcast** (simultâneo para múltiplos prestadores)
- O campo `excessPlanFees` armazena taxas de excedente do plano para cálculo posterior
- Após `FINALIZADO`, o sistema dispara `CalcularValorCredito` para gerar o crédito/débito do cliente
- `ObserverLogGeolocalization` registra a posição GPS do prestador em intervalos durante o serviço
- O `dispatch_vehicle_license_plate` e `dispatch_technician_name` são capturados no momento do aceite

## Edge Cases

- Prestador aceita mas não se move para `ORIGEM` por tempo prolongado
- Múltiplos acionamentos simultâneos para o mesmo atendimento (broadcast)
- Prestador sem viatura ativa disponível no momento do acionamento
- Acionamento automático falha por falta de prestadores elegíveis na região
- Distância total calculada diverge da percorrida (km_total vs km_total_original)
- Acionamento gerado para atendimento já cancelado (race condition)
- Beneficiário pagará excedente mas não autoriza previamente

## Dependencies

- **Use Cases**: `src/UseCases/CalcularValorCredito.php`
- **Modelos**: `src/Models/Acionamento.php`, `src/Models/MetodosAcionamentos.php`
- **Service Assignments**: `src/Models/ServiceAssignments/` (Dispatch, Followup)
- **Observers**: `src/Observers/ObserverLogGeolocalization.php`
- **Cronjobs**: `html/__cronjob/auto_dispatch.php`, `auto_expires_dispatch.php`, `auto_assignment.php`
- **Geolocalização**: `src/Helpers/GeoGoogle.php`, `GeoHere.php`, `GeoTomTom.php`, `GeoOSRM.php`
- **Banco**: tabelas `atendimentos_dispatch`, `prestadores`, `prestadores_viaturas`, `prestadores_bases`

## Related Flows

- [[fluxo-despacho-prestador]]
- [[fluxo-atendimento-completo]]

## Related Features

- [[dispatch-automatico]]
- [[gestao-status-atendimento]]
- [[calculo-credito]]
- [[gestao-prestador]]
