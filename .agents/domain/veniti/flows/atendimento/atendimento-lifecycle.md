---
type: flow
module: atendimento
layer: flow
related:
  - vinculo-plano-beneficiario
  - gestao-evento
  - criacao-atendimento
  - gestao-status-atendimento
  - ciclo-vida-acionamento
  - dispatch-automatico
  - execucao-servico
  - calculo-credito
  - fechamento-financeiro
  - agrupamento-lote
---

# Atendimento Lifecycle (Ciclo de Vida Completo)

> Documenta o ciclo de vida end-to-end de um atendimento de assistência no Veniti — desde o cadastro do beneficiário até o fechamento financeiro.

## Descrição

Documenta o ciclo de vida end-to-end de um atendimento de assistência no Veniti — desde o cadastro do beneficiário até o fechamento financeiro. Este é o **fluxo central e mais crítico** do sistema, envolvendo todos os módulos e todos os atores.

> Este flow implementa o ciclo descrito no contexto de negócio:
> Cadastro → Vínculo → Evento → Atendimento → Acionamento → Execução → Finalização → Fechamento → Lote → Contas a Pagar

---

## Fluxo

```
[Pré-requisitos]                     [Operação]                           [Execução]                    [Financeiro]

Cadastro Beneficiário                Criação do Evento                    Aceite Prestador
       ↓                                    ↓                                    ↓
Vínculo com Cliente                  Criação Atendimento             Deslocamento → ORIGEM
       ↓                                    ↓                                    ↓
Vínculo Plano/Veículo/Pet           Acionamento do Prestador         Execução → DESTINO
                                            ↓                                    ↓
                                     Prestador notificado             FINALIZADO
                                                                              ↓
                                                              Cálculo de Crédito
                                                                              ↓
                                                                     Fechamento
                                                                              ↓
                                                                   Agrupamento Lote
                                                                              ↓
                                                                   Contas a Pagar
```

### FASE 1 — PRÉ-REQUISITOS (Setup)

#### Passo 1: Cadastro do Beneficiário
- **Ator:** Operação interna ou importação em lote
- **Portal:** `/assistencia/beneficiarios/`
- **Ação:** Registra dados pessoais, CPF/CNPJ, contatos
- **Feature:** [[cadastro-beneficiario]]

#### Passo 2: Vínculo com Cliente
- **Ator:** Operação interna
- **Ação:** Associa o beneficiário a um cliente (seguradora) com número de apólice e contrato
- **Banco:** `clientes_beneficiarios`

#### Passo 3: Vínculo de Plano ao Beneficiário / Veículo / Pet
- **Ator:** Operação interna
- **Ação:** Vincula o plano ao beneficiário; vincula veículos e/ou pets ao plano com vigência
- **Define:** Quais serviços o beneficiário pode usar, limites, tarifas
- **Feature:** [[vinculo-plano-beneficiario]]

### FASE 2 — REGISTRO DO INCIDENTE

#### Passo 4: Criação do Evento
- **Ator:** Operador (central de atendimento) — beneficiário liga reportando incidente
- **Portal:** `/assistencia/operacao/eventos.php`
- **Ação:** Cria o Evento com tipo (colisão, pane, furto) e dados do beneficiário/veículo
- **Sistema:** `protocolo` único gerado (YYYYMM + 6 dígitos); status `ABERTO`
- **Feature:** [[gestao-evento]]

#### Passo 5: Criação do Atendimento dentro do Evento
- **Ator:** Operador
- **Portal:** `/assistencia/operacao/atendimentos.php`
- **Ação:** Cria Atendimento vinculado ao Evento com tipo de serviço (reboque, assistência mecânica, etc.)
- **Dados:** Origem, destino, veículo, observações
- **Sistema:** Status `ABERTO`; evento `AttendanceCreated` disparado → tags vinculadas
- **Feature:** [[criacao-atendimento]]
- **Nota:** Um único Evento pode gerar múltiplos Atendimentos (ex: reboque + troca de pneu)

### FASE 3 — ACIONAMENTO DO PRESTADOR

#### Passo 6: Seleção e Acionamento do Prestador
- **Ator:** Operador (manual) ou Cronjob (automático)
- **Portal:** `/assistencia/operacao/acionamento.php`
- **Ação:** Seleciona prestador elegível por tipo de serviço, distância e disponibilidade
- **Sistema:** Cria acionamento em `atendimentos_dispatch` com status `AGUARDANDO`
- **Notificação:** SMS/push enviado ao prestador
- **Features:** [[ciclo-vida-acionamento]], [[dispatch-automatico]]

#### Passo 7: Resposta do Prestador
- **Ator:** Prestador (portal `/prestador/`)
- **Opção A:** Aceita → `ACEITO`; Atendimento avança para `EMBUSCA`
- **Opção B:** Recusa → `RECUSADO`; próximo prestador é acionado (volta ao Passo 6)
- **Opção C:** Timeout → `TEMPO EXPIROU`; cronjob reabre busca
- **Feature:** [[execucao-servico]]

### FASE 4 — EXECUÇÃO EM CAMPO

#### Passo 8: Execução do Atendimento pelo Prestador
- **Ator:** Prestador em campo
- **Portal:** `/prestador/acompanhamento/`
- **Transições:**
  1. Prestador desloca à origem → confirma chegada → `ORIGEM`
  2. Executa o serviço (reboque, troca de pneu, etc.)
  3. Chega ao destino → confirma → `DESTINO`
- **Rastreamento:** GPS coletado continuamente
- **Flow:** [[execucao-atendimento]]

#### Passo 9: Finalização do Atendimento
- **Ator:** Prestador ou Operador
- **Ação:** Confirma conclusão do serviço
- **Sistema:** Acionamento → `FINALIZADO`; Atendimento → `FINALIZADO`/`RECUPERADO`/`NAO RECUPERADO`
- **Trigger automático:** `CalcularValorCredito` disparado imediatamente
- **Feature:** [[gestao-status-atendimento]], [[calculo-credito]]

### FASE 5 — FECHAMENTO FINANCEIRO

#### Passo 10: Fechamento e Negociação de Tarifas
- **Ator:** Assistência + Prestador (bilateral)
- **Portal (Assistência):** `/assistencia/faturamento/prestador/`
- **Portal (Prestador):** `/prestador/fechamento/`
- **Ação:** Assistência inicia fechamento do período; prestador revisa e pode apresentar contraproposta de valores
- **Negociação:** Status: `ABERTO → EM NEGOCIAÇÃO → CONTRAPROPOSTA → ACEITO`
- **Feature:** [[fechamento-financeiro]]

#### Passo 11: Agrupamento em Lote
- **Ator:** Assistência (operação financeira)
- **Portal:** `/assistencia/faturamento/prestador/`
- **Ação:** Acionamentos finalizados e acordados são agrupados em lote (`fechamento_prestador`)
- **Sistema:** Lote numerado com itens; status `ABERTO → APROVADO → AGENDADO`
- **Feature:** [[agrupamento-lote]]
- **Flow:** [[faturamento-lote]]

#### Passo 12: Envio para Contas a Pagar
- **Ator:** Sistema (automático ao aprovar lote)
- **Portal:** `/assistencia/contas_pagar/`
- **Ação:** Lançamento criado em `contas_pagar` com valor total do lote
- **Pagamento:** Agendado conforme forma de pagamento (PIX, TED, etc.) e data definida
- **Status final:** Lote → `REALIZADO` (pagamento confirmado)

---

## Pontos de Entrada

| Ponto de Entrada | Ator | Portal |
|---|---|---|
| Início do setup do beneficiário | Operação | `/assistencia/beneficiarios/` |
| Abertura de Evento (incidente) | Operação (ligação do beneficiário) | `/assistencia/operacao/eventos.php` |
| Criação via API externa | Sistema externo | `/extensao/create_attendance.php` |
| Acionamento manual | Operação | `/assistencia/operacao/acionamento.php` |

## Pontos de Saída

| Saída | Condição |
|---|---|
| Lote `REALIZADO` | Pagamento ao prestador confirmado — ciclo completo |
| Atendimento `CANCELADO` | Cancelamento antes da execução |
| Atendimento `NEGADO` | Cobertura negada — sem serviço |
| Reembolso `PAGO` | Alternativa ao acionamento — [[fluxo-reembolso-completo]] |
| Acionamento sem prestador | Sem elegíveis disponíveis — intervenção manual necessária |

## Variações

### Via Reembolso (sem prestador)
- No Passo 6: sem prestadores disponíveis ou beneficiário já custeou serviço
- Desvio para [[fluxo-reembolso-completo]] em paralelo ao atendimento

### Múltiplos Atendimentos por Evento
- No Passo 5: operador cria 2+ atendimentos para o mesmo evento
- Cada atendimento segue o ciclo independentemente (Passos 6-9)
- Fechamento consolida todos os acionamentos do período

### Acionamento Agendado
- No Passo 6: `data_agendamento` definida
- Acionamento não é enviado imediatamente — cronjob processa no horário correto

### Atendimento em Espera (`EMESPERA`)
- Em qualquer ponto da Fase 3-4: operação coloca atendimento em espera
- Acionamento não pode avançar até liberação manual

---

## Notas de QA

- **Risco crítico (Passo 3→4):** Validação de cobertura do plano — o sistema verifica cobertura antes de criar o Evento ou apenas ao criar o Atendimento? Gap pode causar atendimentos indevidos
- **Risco (Passo 6→7):** Race condition no broadcast — dois prestadores podem aceitar simultaneamente? Verificar atomicidade da operação de aceite
- **Risco (Passo 9→10):** `CalcularValorCredito` falha silenciosamente? Há alertas para créditos não gerados?
- **Risco (Passo 10):** Negociação bilateral sem histórico de versões — qual proposta foi aceita fica apenas no status final, sem linha do tempo
- **Risco (Passo 11→12):** Lote aprovado mas lançamento em `contas_pagar` falha — monitoramento necessário
- **Validation gap (Passo 5):** Limite de uso do plano (ex: 2 reboques/ano) — validado em qual passo exatamente?
- **Comportamento unclear:** Quando o Evento fecha automaticamente vs. manual após todos os atendimentos finalizarem

## Features Relacionadas

- [[vinculo-plano-beneficiario]]
- [[gestao-evento]]
- [[criacao-atendimento]]
- [[gestao-status-atendimento]]
- [[ciclo-vida-acionamento]]
- [[dispatch-automatico]]
- [[execucao-servico]]
- [[calculo-credito]]
- [[fechamento-financeiro]]
- [[agrupamento-lote]]
