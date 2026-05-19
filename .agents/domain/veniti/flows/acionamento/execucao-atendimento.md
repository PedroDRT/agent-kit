---
type: flow
module: acionamento
layer: flow
related:
  - execucao-servico
  - ciclo-vida-acionamento
  - dispatch-automatico
  - gestao-status-atendimento
  - calculo-credito
  - fluxo-despacho-prestador
  - atendimento-lifecycle
  - faturamento-lote
---

# Execução do Atendimento (Perspectiva do Prestador)

> Descreve o fluxo de execução de um serviço de assistência do ponto de vista do prestador — desde o recebimento da notificação até a finalização em campo.

## Descrição

Descreve o fluxo de execução de um serviço de assistência do ponto de vista do prestador — desde o recebimento da notificação até a finalização em campo. Este fluxo ocorre integralmente no portal Prestador (`/prestador/`) e é o responsável pela progressão de status que alimenta os módulos financeiros posteriores.

---

## Fluxo

### 1. Recebimento da Notificação
- **Ator:** Prestador (sistema)
- **Canal:** SMS e/ou push notification
- **Conteúdo:** Tipo de serviço, endereço de origem, distância estimada, tempo estimado
- **Sistema:** Acionamento em status `AGUARDANDO`
- **Timer:** Contagem regressiva iniciada para aceite

### 2. Visualização no Painel
- **Ator:** Prestador
- **Portal:** `/prestador/acompanhamento/`
- **Informações visíveis:**
  - Protocolo do atendimento
  - Tipo de serviço
  - Nome do beneficiário
  - Endereço de origem (com mapa)
  - Veículo (placa, marca, modelo, cor)
  - Distância estimada e tempo de chegada
- **Ação disponível:** Botão "Aceitar" ou "Recusar"

### 3A. Aceite do Serviço
- **Ator:** Prestador
- **Ação:** Clica em "Aceitar" no painel
- **Dados registrados obrigatoriamente:**
  - `dispatch_vehicle_license_plate` — placa do veículo usado
  - `dispatch_technician_name` — nome do técnico responsável
- **Sistema:** Acionamento → `ACEITO`; `TAceite` timestamp gravado
- **Atendimento:** Status avança para `EMBUSCA`
- **Notificação:** Cliente (seguradora) e portal Assistência notificados com dados do prestador

### 3B. Recusa do Serviço
- **Ator:** Prestador
- **Ação:** Clica em "Recusar"
- **Sistema:** Acionamento → `RECUSADO`; `TRecusa` timestamp gravado
- **Consequência:** Sistema busca próximo prestador elegível (Bouncing) ou outro aceita (Broadcast)

### 3C. Timeout — Sem Resposta
- **Ator:** Cronjob `auto_expires_dispatch.php`
- **Trigger:** Tempo máximo de resposta atingido sem ação do prestador
- **Sistema:** Acionamento → `TEMPO EXPIROU`
- **Status prestador:** Registrado como `NAO ATENDEU` para fins de métricas

### 4. Deslocamento à Origem
- **Ator:** Prestador em campo
- **GPS:** Rastreamento coletado continuamente via `ObserverLogGeolocalization`
- **Visibilidade:** Operação acompanha posição em tempo real no mapa (`/assistencia/operacao/`)
- **ETA:** Tempo estimado de chegada exibido ao portal do cliente

### 5. Confirmação de Chegada na Origem
- **Ator:** Prestador
- **Ação:** Marca chegada no portal/app
- **Sistema:** Acionamento → `ORIGEM`; `TOrigem` timestamp gravado
- **Notificação:** Beneficiário e cliente notificados que o prestador chegou

### 6. Execução do Serviço
- **Ator:** Prestador em campo
- **Atividades:** Reboque, troca de pneu, recarga de bateria, assistência mecânica, etc.
- **GPS:** Monitoramento continua durante execução
- **Notas:** Prestador pode registrar observações/ocorrências via portal

### 7. Deslocamento ao Destino (Se aplicável)
- **Ator:** Prestador transportando veículo/beneficiário
- **GPS:** Monitoramento de rota completa registrada

### 8. Confirmação de Chegada no Destino
- **Ator:** Prestador
- **Ação:** Confirma chegada ao endereço de destino
- **Sistema:** Acionamento → `DESTINO`; `TDestino` timestamp gravado

### 9. Finalização do Serviço
- **Ator:** Prestador
- **Ação:** Confirma conclusão do serviço
- **Validação no frontend:** `changeSteps()` verifica se `ORIGEM` e `DESTINO` foram marcados antes de liberar "Finalizar"
- **Sistema:** Acionamento → `FINALIZADO`; `TFinalizado` timestamp gravado
- **Atendimento:** Status → `FINALIZADO` (ou `RECUPERADO`/`NAO RECUPERADO` conforme resultado)
- **Trigger:** `CalcularValorCredito` executado automaticamente
- **NPS:** Pesquisa de satisfação disparada ao beneficiário (status `NPS_PENDENTE` no atendimento)

### 10. Serviço Disponível para Faturamento
- **Sistema:** Acionamento `FINALIZADO` entra no pool de itens faturáveis
- **Visível em:** `/prestador/servicos/` (histórico somente leitura)
- **Próximo passo:** Inclusão em Lote → [[faturamento-lote]]

---

## Pontos de Entrada

- Notificação SMS/push recebida pelo prestador
- Acesso manual ao painel `/prestador/acompanhamento/` com filtro de data/protocolo

## Pontos de Saída

| Saída | Status Final | Consequência |
|---|---|---|
| Serviço concluído | `FINALIZADO` | Crédito calculado, entra no pool de faturamento |
| Prestador recusou | `RECUSADO` | Novo prestador acionado |
| Sem resposta | `TEMPO EXPIROU` / `NAO ATENDEU` | Novo prestador acionado |
| Cancelamento operação | `CANCELADO` | Serviço encerrado sem faturamento |

## Variações

### Atendimento Extended Status (além de FINALIZADO)

| Status | Gatilho | Significado |
|---|---|---|
| `EM ESPERA` | Operação | Pausa forçada — aguardando autorização para continuar |
| `VALIDAÇÃO` | Sistema/Operação | Atendimento em análise interna antes de finalizar |
| `NEGATIVA` | Operação | Cobertura negada durante execução |
| `EM PROGRESSO` | Sistema | Serviço em andamento (estado intermediário) |
| `SOCIAL CALL` | Operação | Contato de acompanhamento com beneficiário |
| `REEMBOLSO` | Sistema | Atendimento redirecionado para fluxo de reembolso |
| `NPS_PENDENTE` | Sistema | Aguardando resposta da pesquisa de satisfação |
| `CONTATO_BENEFICIARIO` | Operação | Tentativa de contato com beneficiário |

---

## Notas de QA

- **Risco crítico:** Sequência de status validada apenas no frontend (`changeSteps()` em JS) — o backend aceita transições inválidas diretamente via API?
- **Risco:** GPS pode falhar silenciosamente (dispositivo sem sinal) sem alertar a operação
- **Risco:** `TFinalizado` registrado sem verificar se `TOrigem` e `TDestino` existem no backend — inconsistência de dados possível
- **Validation gap:** `km_total` ajustável pelo prestador até o momento da finalização — há limite máximo de variação em relação ao estimado?
- **Comportamento unclear:** O status `EM ESPERA` bloqueia o prestador de avançar — mas o que notifica o prestador quando a espera é liberada?
- **Risco de fraude:** Prestador pode marcar `ORIGEM` sem estar no local — comparar lat/lng do prestador com o endereço de origem é uma verificação necessária
- **Edge case:** Prestador finaliza atendimento correto mas com veículo errado (`dispatch_vehicle_license_plate` diferente do veículo que executou)
- **Risco de consistência:** `NPS_PENDENTE` — o que acontece se o beneficiário nunca responde? O status do atendimento fica preso?

## Features Relacionadas

- [[execucao-servico]]
- [[ciclo-vida-acionamento]]
- [[dispatch-automatico]]
- [[gestao-status-atendimento]]
- [[calculo-credito]]
