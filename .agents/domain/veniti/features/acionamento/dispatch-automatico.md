---
type: feature
module: acionamento
layer: feature
related:
  - fluxo-despacho-prestador
  - ciclo-vida-acionamento
  - gestao-prestador
---

# Dispatch Automático

> Sistema que seleciona e aciona prestadores de serviço automaticamente, sem intervenção humana, usando algoritmos de busca por proximidade, disponibilidade e cobertura.

## Descrição

Sistema que seleciona e aciona prestadores de serviço automaticamente, sem intervenção humana. Executa algoritmos de busca e seleção baseados em proximidade, disponibilidade e cobertura do serviço. Utilizado tanto para despacho inicial quanto para follow-up de acionamentos não respondidos.

---

## Entradas

| Parâmetro | Descrição |
|---|---|
| `id_atendimento` | Atendimento a ser despachado |
| `tipo_servico` | Tipo de serviço requerido |
| `Olat`, `Olng` | Coordenadas de origem do atendimento |
| `raio_busca` | Raio de busca de prestadores (km) |
| `metodo_acionamento` | Bouncing ou Broadcast |

## Saídas

- Acionamento(s) criado(s) automaticamente com `acionamento_automatico = true`
- Notificação enviada ao(s) prestador(es) elegível(is)
- Fila de despacho atualizada
- Log de cada tentativa registrado

---

## Regras de Negócio

- Somente atendimentos com `bloquear_acionamento = false` são elegíveis
- A busca de prestadores usa `BuscadorPrestadores` que filtra por:
  - Tipo de serviço (prestador deve oferecer o serviço solicitado)
  - Disponibilidade (prestador `online`)
  - Distância da origem (raio configurável)
  - Base ativa com viatura disponível
- O cronjob `add_queue_dispatch.php` insere atendimentos elegíveis na fila
- O cronjob `auto_dispatch.php` processa a fila e efetua os acionamentos
- O cronjob `auto_assignment_follow_up.php` reatribui acionamentos não respondidos
- `auto_expires_dispatch.php` expira acionamentos cujo timeout foi atingido
- Prestadores com avaliação NPS ruim podem ser despriorizados
- Se nenhum prestador for encontrado, o atendimento permanece em `ABERTO` para acionamento manual

## Métodos de Despacho

### Bouncing (Sequencial)
- Seleciona o melhor prestador disponível
- Aguarda resposta por tempo configurável
- Se não aceitar → tenta o próximo prestador
- Repete até aceite ou esgotamento de opções

### Broadcast (Simultâneo)
- Envia notificação para múltiplos prestadores ao mesmo tempo
- Primeiro que aceitar fica com o serviço
- Demais recebem status `ACEITO POR OUTRO PRESTADOR`
- Reduz tempo de espera em regiões com múltiplos prestadores

---

## Casos de Borda

- Região sem prestadores cadastrados para o tipo de serviço
- Todos os prestadores encontrados estão offline
- Prestador aceita automaticamente mas está fora da área de cobertura real
- Fila de despacho processada em ordem incorreta (prioridade por data de criação)
- Broadcast com mais prestadores do que o limite configurado
- Cronjob falha no meio do processo (acionamento criado sem notificação enviada)

## Dependências

- **Modelos**: `src/Models/MetodosAcionamentos.php` (Bouncing, Broadcast), `src/Models/BuscadorPrestadores.php`
- **Cronjobs**: `html/__cronjob/auto_dispatch.php`, `auto_assignment.php`, `auto_assignment_follow_up.php`, `auto_expires_dispatch.php`, `add_queue_dispatch.php`
- **Geolocalização**: `src/Helpers/GeoGoogle.php`, `GeoOSRM.php` (cálculo de distâncias)
- **Cronjob auxiliar**: `atualizar_distancias.php` (atualiza distâncias dos prestadores disponíveis)

## Flows Relacionados

- [[fluxo-despacho-prestador]]

## Features Relacionadas

- [[ciclo-vida-acionamento]]
- [[gestao-prestador]]
