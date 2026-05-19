# Fluxo de Reembolso Completo

## Description

Descreve o ciclo completo de um reembolso — desde a criação pela operação, passando pela interação do beneficiário via portal self-service, até o pagamento final. Envolve o portal Assistência (gestão interna) e o portal público do Beneficiário.

## Módulos de Configuração (Pré-requisitos)

| Módulo | Caminho | Descrição |
|---|---|---|
| Perguntas de reembolso | Configurações → Financeiro → Perguntas de reembolso | Configura perguntas exibidas no fluxo |
| Status personalizados | Configurações → Financeiro → Status reembolso | Configura status complementares |

## Steps

### 1. Decisão de Reembolso
- **Ator**: Operador (portal Assistência)
- **Contexto**: Prestador não disponível, beneficiário custeou serviço próprio, ou cobertura é por reembolso
- **Ação**: Acessa o atendimento em `/assistencia/operacao/` → Editar → Ação → Reembolso
- **Modal de cadastro exibe:**
  - Motivo do Reembolso (obrigatório)
  - Nome do Solicitante / Telefone do Solicitante (auto-preenchidos do atendimento)
  - Limite reembolso (R$) (opcional)
  - Valor solicitado (R$)
  - Observação (opcional)
  - Seção PERGUNTAS (quando existem perguntas configuradas que correspondem ao contexto do atendimento)
- **Tags herdadas:** tags com Tipo de Evento = 'TODOS' vinculadas ao atendimento são pré-selecionadas automaticamente no campo de tags
- **Sistema**: Registro criado em `reembolsos` com status `NOTA_PENDENTE`

### 2. Envio do Link ao Beneficiário
- **Sistema**: Gera short URL única vinculada ao reembolso e ao beneficiário
- **Canal**: SMS e/ou e-mail enviado ao beneficiário com o link de acesso
- **Link**: `https://{tenant}/reembolso?token=...` (URL curta com token embutido)

### 3. Acesso ao Portal pelo Beneficiário
- **Ator**: Beneficiário
- **Portal**: `/reembolso/` (portal público self-service)
- **View**: `introduction.php` → `default_information.php`
- **Informações visíveis**: protocolo, motivo, valor solicitado, status atual
- **Feature**: [[portal-self-service-beneficiario]]

### 4. Envio de Documentação pelo Beneficiário
- **Ator**: Beneficiário
- **Ação**: Preenche dados financeiros (PIX, TED, DOC) e faz upload de documentos (múltiplos arquivos simultâneos permitidos)
- **Sistema**: Arquivo armazenado no AWS S3; `id_arquivo_nota` atualizado no reembolso
- **Status**: Reembolso → `NOTA_RECEBIDA`
- **Notificação**: Operação notificada internamente
- **Arquivos enviados via link** recebem badge "BENEFICIÁRIO" na aba Arquivos do reembolso

### 5. Análise pela Operação
- **Ator**: Operador (portal Assistência)
- **Portal**: `/assistencia/reembolso/editar.php`
- **Ação**: Visualiza nota fiscal (link S3 assinado, 10 min de validade), confere dados bancários
- **Decisão**: Aprovar, Agendar ou Recusar

### 6A. Aprovação
- **Ator**: Operador
- **Modal de aprovação exibe:** Valor Solicitado, Valor Excedente (calculado automaticamente se houver limite de reembolso), Valor Aprovado, Diferença
- **Cálculo:** `Valor excedente = Valor solicitado − Limite de reembolso` (preenchido automaticamente; editável)
- **Ação**: Define `valor_final` (pode ser igual ou inferior ao solicitado)
- **Sistema**: Status → `APROVADO`; lançamento criado em `contas_pagar`
- **Notificação**: Beneficiário notificado que o reembolso foi aprovado
- **View no portal**: `refund_approved.php`

### 6B. Agendamento
- **Ator**: Operador
- **Ação**: Define `data_agendamento` para pagamento futuro
- **Sistema**: Status → `AGENDADO`
- **Notificação**: Beneficiário notificado com data prevista
- **View no portal**: `refund_scheduled.php`

### 6C. Recusa
- **Ator**: Operador
- **Ação**: Preenche justificativa da recusa
- **Sistema**: Status → `RECUSADO`; ocorrência gerada em `ocorrencias`
- **Notificação**: Beneficiário notificado com motivo da recusa
- **View no portal**: `refund_rejected.php`

### 7. Geração do Pagamento
- **Ator**: Sistema financeiro / Operação
- **Ação**: Pagamento gerado no sistema bancário
- **Sistema**: Status → `PAGAMENTO_GERADO`; `numero_documento` preenchido

### 8. Confirmação do Pagamento
- **Sistema**: Confirmação de débito bancário recebida
- **Status**: Reembolso → `PAGO`
- **View no portal**: `refund_sent.php`
- **Encerramento**: Ciclo do reembolso concluído

## Telas do Reembolso (Abas)

| Aba | Descrição |
|---|---|
| Visualizar | Dados gerais do reembolso |
| Editar | Card "INFORMAÇÕES DO REEMBOLSO" com campos editáveis (tags, status adicional, motivo, valores, etc.) |
| Perguntas | Perguntas configuradas para o contexto; respostas editáveis |
| Arquivos | Lista de arquivos (arquivos do beneficiário identificados com badge "BENEFICIÁRIO") |
| Ocorrências | Histórico de ocorrências manuais e automáticas |

## Entry Points

- Portal Assistência: `/assistencia/operacao/` → criação manual pela operação
- Portal Assistência: `/assistencia/reembolso/` → gestão de reembolsos existentes

## Exit Points

- **Sucesso**: Status `PAGO` — pagamento confirmado
- **Recusa**: Status `RECUSADO` — ciclo encerrado sem pagamento
- **Link expirado**: Beneficiário não acessa o link no prazo → operação deve reenviar

## Variations

### Reembolso sem Upload de Nota
- Em casos onde a documentação é dispensada pelo contrato
- Operação aprova diretamente sem exigir envio pelo beneficiário

### Pagamento para Terceiro
- `pagamento_para` ≠ `BENEFICIARIO`
- Dados financeiros preenchidos para terceiro autorizado

### Reembolso com Aprovação Parcial
- `valor_final` < `valor` solicitado
- Beneficiário notificado do valor aprovado diferente do solicitado

### Reembolso com Limite e Valor Excedente
- Operador informa `limite_reembolso` no cadastro
- No modal de aprovação, o sistema calcula e exibe automaticamente o `valor_excedente`

## Related Features

- [[solicitacao-reembolso]]
- [[processamento-pagamento-reembolso]]
- [[perguntas-reembolso]]
- [[tags-reembolso]]
- [[ocorrencias-reembolso]]
- [[badge-arquivos-reembolso]]
- [[status-personalizados-reembolso]]
- [[link-reembolso]]
- [[portal-self-service-beneficiario]]
- [[criacao-atendimento]]
- [[api-reembolsos]]
