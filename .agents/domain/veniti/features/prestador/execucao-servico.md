# Execução de Serviço pelo Prestador

## Description

Representa o conjunto de ações realizadas pelo prestador de serviço após aceitar um acionamento. Cobre todo o ciclo de execução em campo: deslocamento à origem, execução do serviço, deslocamento ao destino e finalização. O prestador interage exclusivamente pelo portal Prestador (`/prestador/`).

## Inputs

| Ação | Campo/Trigger | Descrição |
|---|---|---|
| Aceitar acionamento | `id_acionamento` | Prestador confirma aceite |
| Marcar chegada na origem | `situacao = ORIGEM` | Confirmação de chegada ao local do beneficiário |
| Marcar chegada no destino | `situacao = DESTINO` | Confirmação de chegada ao destino final |
| Finalizar atendimento | `situacao = FINALIZADO` | Conclusão do serviço |
| Cancelar acionamento | `motivo_cancelamento` | Cancela o acionamento aceito |
| Informar KM percorrido | `km_total` | Ajuste de distância real (se divergir do estimado) |

## Outputs

- Timestamps registrados por transição: `TAceite`, `TOrigem`, `TDestino`, `TFinalizado`
- `dispatch_vehicle_license_plate` e `dispatch_technician_name` registrados no aceite
- GPS coletado continuamente via `ObserverLogGeolocalization`
- Notificação enviada ao cliente quando prestador chega na origem e finaliza
- Cálculo de crédito disparado ao `FINALIZADO` (`CalcularValorCredito`)
- Histórico disponível em `/prestador/servicos/` e `/prestador/acompanhamento/`

## Status Flow (Perspectiva do Prestador)

```
[Notificação recebida]
      ↓
  AGUARDANDO ──→ Prestador Aceita ──→ ACEITO
                                        ↓
                                   Desloca à origem
                                        ↓
                                     ORIGEM  (chegou ao local do beneficiário)
                                        ↓
                                   Executa serviço
                                        ↓
                                    DESTINO  (chegou ao destino final)
                                        ↓
                                   FINALIZADO
```

**Saídas alternativas:**
- `AGUARDANDO` → `RECUSADO` (prestador recusa a solicitação)
- `AGUARDANDO` → `NAO ATENDEU` (timeout sem resposta)
- `ACEITO` → `CANCELADO` (cancelamento pela operação ou prestador)
- `AGUARDANDO` → `TEMPO EXPIROU` (cronjob expira o acionamento)

## UI Status Mapping (portal `/prestador/acompanhamento/`)

| Status | Ícone | Visual |
|---|---|---|
| ABERTO | spinner animado | badge-light |
| ACEITO | check-circle | badge-light |
| ORIGEM | arrow-up | badge-warning |
| DESTINO | arrow-down | badge-success |
| FINALIZADO | check-circle | badge-success |
| EM ESPERA | hand-paper | badge-warning |
| VALIDAÇÃO | lock | badge-warning |
| NEGATIVA / ATRASADO | minus-circle | badge-danger |
| CANCELADO | times | badge-danger |
| EM PROGRESSO | spinner animado | badge-warning |
| SOCIAL CALL | headset | badge-warning |
| REEMBOLSO | money-bill-transfer | badge-warning |

## Business Rules

- O prestador só pode avançar os status na sequência definida (não pode pular de `ACEITO` para `FINALIZADO`)
- Em broadcast: primeiro prestador a aceitar obtém o serviço; demais recebem `ACEITO POR OUTRO PRESTADOR`
- O cancelamento pelo prestador após aceite deve ter `motivo_cancelamento` preenchido
- `km_total` pode ser ajustado pelo prestador até a finalização — impacta o cálculo de crédito
- `dispatch_vehicle_license_plate` e `dispatch_technician_name` são obrigatórios no aceite
- Atendimentos em `EM ESPERA` não podem ser avançados sem liberação pela operação
- O prestador acessa o painel de acompanhamento em `/prestador/acompanhamento/` com filtros por data, protocolo, tipo de serviço e status
- O painel de serviços históricos fica em `/prestador/servicos/` (somente leitura)
- O acionamento em `FINALIZADO` alimenta o módulo de `Faturamento` do prestador — o serviço passa a ser faturável

## Edge Cases

- Prestador marca `ORIGEM` antes de chegar ao local real (fraude de execução)
- GPS não disponível no dispositivo do prestador (rastreamento falho)
- Prestador tenta finalizar atendimento sem marcar `ORIGEM` e `DESTINO`
- Serviço concluído fisicamente mas não finalizado no sistema (atendimento "fantasma")
- `km_total` informado muito diferente do `km_acionamento` calculado (possível fraude)
- Prestador finaliza atendimento para beneficiário errado
- Dois técnicos do mesmo prestador marcando progresso simultaneamente no mesmo acionamento
- Atendimento finalizado no sistema mas serviço não concluído (reclamação posterior)

## QA Notes

- **Risco crítico:** Não há evidência de validação de geolocalização real vs. status reportado — prestador pode marcar `ORIGEM` sem estar no local
- **Risco:** `km_total` ajustável pelo prestador até finalização — potencial para abuso sem limite de variação configurado
- **Validation gap:** Não está claro se há validação de sequência de status no backend (apenas no frontend via `changeSteps()`)
- **Comportamento unclear:** O que acontece com o acionamento se o app do prestador fecha durante a execução?
- **Edge case:** Cancelamento após `DESTINO` (prestador já chegou ao destino mas não finalizou) — gera custo ao cliente?
- **Risco de consistência:** Timestamps (`TOrigem`, `TDestino`) são registrados no momento da ação — e se há atraso de rede?

## Dependencies

- **Portal**: `html/prestador/acompanhamento/` (execução em tempo real), `html/prestador/servicos/` (histórico)
- **Observers**: `src/Observers/ObserverLogGeolocalization.php`
- **Use Case**: `src/UseCases/CalcularValorCredito.php` (acionado no FINALIZADO)
- **Real-time**: `src/Helpers/WebSocket.php` (Pusher — atualizações em tempo real)
- **Cronjob**: `html/__cronjob/auto_expires_dispatch.php`
- **Modelo**: `src/Models/Acionamento.php`

## Related Flows

- [[execucao-atendimento]]
- [[fluxo-despacho-prestador]]
- [[fluxo-atendimento-completo]]

## Related Features

- [[ciclo-vida-acionamento]]
- [[dispatch-automatico]]
- [[calculo-credito]]
- [[agrupamento-lote]]
