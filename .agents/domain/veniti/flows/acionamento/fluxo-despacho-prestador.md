---
type: flow
module: acionamento
layer: flow
related:
  - ciclo-vida-acionamento
  - dispatch-automatico
  - gestao-status-atendimento
  - calculo-credito
---

# Fluxo de Despacho ao Prestador

> Descreve o processo de seleção, notificação e acompanhamento do prestador durante a execução de um atendimento, desde a busca do prestador até a conclusão do serviço em campo.

## Descrição

Descreve o processo detalhado de seleção, notificação e acompanhamento do prestador durante a execução de um atendimento. Foca na perspectiva operacional do acionamento, desde a busca do prestador até a conclusão do serviço em campo.

---

## Fluxo

### 1. Verificação de Elegibilidade
- **Sistema**: Confirma que `bloquear_acionamento = false` no atendimento
- **Sistema**: Verifica se o atendimento está em status `ABERTO`
- **Sistema**: Confirma que não há acionamento ativo já vinculado

### 2. Busca de Prestadores Elegíveis
- **Sistema**: `BuscadorPrestadores` executa busca por:
  - Tipo de serviço compatível
  - Distância da base ao endereço de origem (raio configurável)
  - Prestadores com status `online`
  - Viaturas disponíveis na base
- **Resultado**: Lista ranqueada de prestadores candidatos

### 3. Criação do Acionamento
- **Sistema**: Cria registro em `atendimentos_dispatch`
- **Campos gerados**: `dispatch_public_id`, `tipo_acionamento`, `km_acionamento`, `tarifas`, `acionamento_tempo_chegada`
- **Status inicial**: `AGUARDANDO`

### 4. Notificação ao Prestador
- **Sistema**: SMS e/ou push notification enviados ao prestador selecionado
- **Timer**: Contagem regressiva inicia (tempo máximo para resposta)
- **Cronjob**: `auto_expires_dispatch.php` monitora o timeout

### 5A. Prestador Aceita
- **Ator**: Prestador — confirma aceite via portal (`/prestador/`)
- **Registro**: `TAceite` timestamp gravado
- **Sistema**: Acionamento → `ACEITO`, Atendimento → `EMBUSCA`
- **Dados capturados**: `dispatch_vehicle_license_plate`, `dispatch_technician_name`
- **Notificação**: Cliente notificado com dados do prestador e tempo estimado

### 5B. Prestador Recusa
- **Ator**: Prestador rejeita a solicitação
- **Registro**: `TRecusa` timestamp gravado
- **Sistema**: Acionamento → `RECUSADO`
- **Ação automática**: Próximo prestador elegível é acionado (volta ao passo 3)

### 5C. Timeout Expirado
- **Ator**: Cronjob `auto_expires_dispatch.php`
- **Sistema**: Acionamento → `TEMPO EXPIROU`
- **Ação**: Novo candidato tentado (volta ao passo 2-3)

### 6. Deslocamento e Acompanhamento
- **Ator**: Prestador em campo
- **Rastreamento**: GPS coletado periodicamente via `ObserverLogGeolocalization`
- **Visualização**: Operação acompanha em tempo real via WebSocket (Pusher) no mapa do painel

### 7. Confirmação de Chegada na Origem
- **Ator**: Prestador
- **Ação**: Marca chegada no app/portal
- **Sistema**: Acionamento → `ORIGEM`, `TOrigem` gravado

### 8. Execução e Deslocamento ao Destino
- **Ator**: Prestador executa o serviço (reboque, troca de pneu, etc.)
- **Sistema**: Rastreamento GPS continua

### 9. Chegada ao Destino
- **Ator**: Prestador
- **Ação**: Confirma chegada ao destino
- **Sistema**: Acionamento → `DESTINO`, `TDestino` gravado

### 10. Finalização do Serviço
- **Ator**: Prestador ou Operador
- **Ação**: Confirma conclusão
- **Sistema**: Acionamento → `FINALIZADO`, `TFinalizado` gravado
- **Trigger**: [[calculo-credito]] disparado automaticamente

---

## Pontos de Entrada

- Operador acessa `/assistencia/operacao/acionamento.php` e seleciona prestador manualmente
- Cronjob `auto_dispatch.php` seleciona atendimentos elegíveis da fila e aciona automaticamente
- Broadcast mode: múltiplos prestadores notificados simultaneamente

## Pontos de Saída

- **Sucesso**: Acionamento `FINALIZADO` → crédito gerado
- **Cancelamento**: Acionamento `CANCELADO` pela operação
- **Esgotamento**: Sem prestadores disponíveis → atendimento permanece em `ABERTO` para intervenção manual
- **Reembolso**: Operação decide reembolsar ao invés de acionar → [[fluxo-reembolso-completo]]

## Variações

### Broadcast (Simultâneo)
- Múltiplos prestadores notificados de uma vez
- Primeiro aceite "ganha" o serviço
- Demais acionamentos → `ACEITO POR OUTRO PRESTADOR`

### Acionamento Manual por Operador
- Operador seleciona prestador diretamente na tela de acionamento
- Ignora algoritmo de busca automática

### Reagendamento
- Acionamento agendado para data/hora futura
- `data_agendamento` e `data_agendamento_inicio` controlam a janela de execução

## Features Relacionadas

- [[ciclo-vida-acionamento]]
- [[dispatch-automatico]]
- [[gestao-status-atendimento]]
- [[calculo-credito]]
