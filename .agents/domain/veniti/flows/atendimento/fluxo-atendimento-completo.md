# Fluxo de Atendimento Completo

## Description

Descreve o ciclo de vida completo de uma solicitação de assistência — desde o registro inicial pelo operador ou sistema externo até a finalização do serviço e geração do crédito. Este é o fluxo central do sistema Veniti.

## Steps

### 1. Registro do Atendimento
- **Ator**: Operador interno (portal Assistência) ou sistema externo (extensão via API)
- **Ação**: Preenche dados do beneficiário, tipo de serviço, endereço de origem e destino
- **Sistema**: Cria registro em `atendimentos` com status `ABERTO`
- **Evento**: `AttendanceCreated` é disparado → tags identificadoras vinculadas automaticamente
- **Feature**: [[criacao-atendimento]]

### 2. Seleção de Prestador (Manual ou Automático)
- **Ator**: Operador ou cronjob de dispatch automático
- **Ação**: Seleciona prestador disponível (por distância, disponibilidade, tipo de serviço)
- **Sistema**: Cria registro em `atendimentos_dispatch` com status `AGUARDANDO`
- **Notificação**: SMS/push enviado ao prestador
- **Feature**: [[ciclo-vida-acionamento]], [[dispatch-automatico]]

### 3. Resposta do Prestador
- **Ator**: Prestador (portal Prestador)
- **Opção A — Aceite**: Status muda para `ACEITO` → Atendimento muda para `EMBUSCA`
- **Opção B — Recusa**: Status muda para `RECUSADO` → Novo acionamento é tentado (voltar ao passo 2)
- **Opção C — Timeout**: Cronjob expira o acionamento → Novo acionamento tentado

### 4. Deslocamento para a Origem
- **Ator**: Prestador em campo
- **Ação**: Confirma chegada ao endereço de origem via portal Prestador
- **Sistema**: Acionamento muda para `ORIGEM`
- **Rastreamento**: GPS registrado via `ObserverLogGeolocalization`

### 5. Execução do Serviço e Chegada ao Destino
- **Ator**: Prestador
- **Ação**: Conclui o serviço e confirma chegada ao destino
- **Sistema**: Acionamento muda para `DESTINO`

### 6. Finalização
- **Ator**: Prestador ou Operador
- **Ação**: Confirmação de conclusão do serviço
- **Sistema**: Acionamento muda para `FINALIZADO` → Atendimento muda para `FINALIZADO`/`RECUPERADO`/`NAO RECUPERADO`
- **Trigger**: `CalcularValorCredito` é disparado automaticamente
- **Feature**: [[calculo-credito]]

### 7. Geração de Crédito
- **Sistema**: Calcula valor de crédito/débito baseado no plano e método configurado
- **Resultado**: Lançamento criado em `clientes_contratos_creditos`
- **Disponível**: No extrato do cliente (portal Cliente) e na gestão interna (portal Assistência)

### 8. NPS (Opcional)
- **Sistema**: Dispara pesquisa de satisfação ao beneficiário após finalização
- **Resultado**: Pontuação NPS registrada na tabela `nps`

## Entry Points

- Portal Assistência: `/assistencia/operacao/atendimentos.php` (criação manual)
- API externa: `/extensao/create_attendance.php` (criação programática)
- Portal Cliente: `/cliente/operacao/atendimentos.php` (criação pelo cliente)

## Exit Points

- Atendimento `FINALIZADO` + crédito gerado (sucesso)
- Atendimento `CANCELADO` (cancelamento antes da finalização)
- Atendimento `NEGADO` (negação de cobertura)
- Atendimento `NAO RECUPERADO` (serviço sem sucesso)

## Variations

### Atendimento Agendado
- Na criação: `data_agendamento` preenchida
- O acionamento automático só é tentado próximo à data/hora do agendamento
- Cronjob monitora agendamentos pendentes

### Atendimento com Reembolso
- Quando não há prestador disponível ou beneficiário optou por serviço próprio
- Atendimento criado → Reembolso gerado → [[fluxo-reembolso-completo]] iniciado em paralelo

### Cancelamento
- Pode ocorrer em qualquer estado antes de `FINALIZADO`
- Se prestador já aceitou: notificação de cancelamento enviada ao prestador

## Related Features

- [[criacao-atendimento]]
- [[gestao-status-atendimento]]
- [[ciclo-vida-acionamento]]
- [[dispatch-automatico]]
- [[calculo-credito]]
