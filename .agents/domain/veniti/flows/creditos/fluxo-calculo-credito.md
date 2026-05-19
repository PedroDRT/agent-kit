# Fluxo de Cálculo de Crédito

## Description

Descreve o processo automático de cálculo e registro do valor de crédito gerado ao cliente após a finalização de um atendimento. Este fluxo é disparado pelo sistema e não requer interação humana direta.

## Steps

### 1. Trigger de Finalização
- **Evento**: Acionamento muda para status `FINALIZADO`
- **Ator**: Sistema (automático) — disparado pela mudança de status do acionamento
- **Context**: Atendimento vinculado é identificado via `id_atendimento` no acionamento

### 2. Coleta de Dados do Acionamento
- **Sistema**: `CalcularValorCredito.php` carrega dados do acionamento:
  - `km_total`, `km_acionamento`, `km_total_original`
  - `tariffs` / `tarifas`
  - `excessPlanFees`
  - `tipo_acionamento`
  - `beneficiario_pagara_excedente`

### 3. Resolução do Método de Cálculo
- **Sistema**: `MetodoCalculoResolver.php` consulta configuração do plano
- **Input**: `id_plano` + `tipo_servico`
- **Output**: Estratégia selecionada (`ValorAtendimento`, `ValorTabela` ou `ValorManual`)
- **Fallback**: Se não configurado, usa método padrão do sistema

### 4. Execução do Cálculo
#### Método ValorAtendimento
- Busca valor fixo configurado para o tipo de serviço no plano
- Aplica multiplicadores (se houver)
- Resultado: valor único independente de km

#### Método ValorTabela
- Utiliza `km_total` como parâmetro de entrada
- Lookup na tabela de tarifas por faixa de distância
- Calcula excedente se km ultrapassa limite do plano
- Soma `excessPlanFees` se aplicável

#### Método ValorManual
- Usa valor definido diretamente no acionamento
- Sem cálculo adicional

### 5. Cálculo de Excedente (Se aplicável)
- **Condição**: `km_total > limite_km_plano`
- **Cálculo**: `(km_total - limite_km_plano) × tarifa_excedente`
- **Responsabilidade**: Se `beneficiario_pagara_excedente = true`, valor cobrado do beneficiário; caso contrário, do cliente

### 6. Registro do Crédito
- **Sistema**: Cria lançamento em `clientes_contratos_creditos`
- **Dados registrados**: valor, id_atendimento, id_acionamento, método usado, data, id_veniti
- **Sinal**: Positivo = crédito (cliente deve pagar); Negativo = débito (Veniti deve ao cliente)

### 7. Atualização de Métricas
- **Sistema**: `CalcularMetricas.php` atualiza totalizadores do período
- **Disponível**: Dashboards de faturamento nos portais Assistência e Cliente

## Entry Points

- Acionamento finalizado (automático ou manual pelo prestador/operador)

## Exit Points

- **Sucesso**: Crédito registrado, métricas atualizadas
- **Erro de configuração**: Plano sem método de cálculo → log de erro, crédito pendente para revisão manual
- **Dados inválidos**: `km_total` inválido → valor zerado registrado, alerta gerado

## Variations

### Socorre A&E
- `CalcularValorCreditoSocorreAe.php` executa lógica de cálculo específica para essa marca
- Regras de negócio diferenciadas para o contrato Socorre A&E

### Cancelamento com Reembolso ao Cliente
- Valor negativo registrado para atendimentos cancelados após aceite do prestador
- Cobre custo de deslocamento do prestador já pago

## Related Features

- [[calculo-credito]]
- [[ciclo-vida-acionamento]]
- [[gestao-status-atendimento]]
