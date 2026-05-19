# Integração — Endpoint de Consulta de Reembolsos

## Resumo

Endpoint REST para que **sistemas externos** consultem os dados de reembolsos cadastrados no Veniti. Retorna informações completas do reembolso, incluindo dados do beneficiário, atendimento, valores financeiros, perguntas respondidas, arquivos, tags e status.

---

## Endpoint

```
GET /api/reembolsos
```

---

## Parâmetros de Consulta (Query Params)

| Parâmetro | Descrição |
|---|---|
| `protocolo` | Filtra pelo protocolo do atendimento vinculado |
| `numero_reembolso` | Filtra pelo número do reembolso |
| `beneficiario` | Filtra pelo nome do beneficiário |
| `cliente` | Filtra pelo cliente associado ao atendimento |
| `data_inicio` | Filtra pela data inicial de abertura da solicitação |
| `data_fim` | Filtra pela data final de abertura da solicitação |
| `status` | Filtra pelo status padrão do reembolso |
| `status_adicional` | Filtra pelo status adicional (módulo status personalizados) |

---

## Estrutura de Retorno

| Campo | Descrição |
|---|---|
| `associado` | Beneficiário vinculado ao reembolso |
| `protocolo` | Protocolo do atendimento relacionado |
| `chassi` | Chassi do veículo |
| `cliente` | Cliente associado ao atendimento |
| `colaborador_responsavel` | Usuário que abriu o reembolso |
| `arquivos` | Lista de arquivos anexados (com data e hora de envio) |
| `data_recebimento_demanda` | Data de abertura da solicitação |
| `numero_reembolso` | Número identificador do reembolso |
| `placa` | Placa do veículo |
| `placa_mercosul` | Placa no padrão Mercosul (quando aplicável) |
| `solicitante` | Nome do solicitante do reembolso |
| `tipo_servico` | Tipo de serviço relacionado ao atendimento |
| `categoria` | Categoria do atendimento ou serviço |
| `valor_aprovado` | Valor aprovado para reembolso |
| `valor_solicitado` | Valor inicialmente solicitado |
| `possui_rastreador` | Indica se o veículo possui rastreador |
| `tags` | Lista de tags vinculadas ao reembolso |
| `valor_excedente` | Valor excedente calculado na aprovação |
| `perguntas` | Lista de perguntas respondidas no fluxo |
| `motivo_reembolso` | Motivo selecionado no cadastro |
| `reincidente` | Se o beneficiário já teve outro reembolso aberto |
| `status_atual` | Status padrão atual do reembolso |
| `status_adicional` | Status personalizado configurado pela assistência |
| `observacoes` | Campo de observações do reembolso |

---

## Exemplo de Retorno JSON

```json
{
  "numero_reembolso": "123",
  "protocolo": "SAT000001/1",
  "cliente": "SAT SEGUROS",
  "beneficiario": {
    "nome": "Carlos Eduardo Santos",
    "placa": "ABC1323",
    "placa_mercosul": "ABC1D23",
    "chassi": "9BWZZZ377VT004251",
    "possui_rastreador": "SIM"
  },
  "solicitante": "Carlos Eduardo Santos",
  "colaborador_responsavel": "João Almeida",
  "tipo_servico": "REBOQUE LEVE",
  "categoria": "Pane Mecânica",
  "motivo_reembolso": "Pagamento direto ao prestador",
  "data_recebimento_demanda": "2026-03-10T14:35:21",
  "valores": {
    "valor_solicitado": 350.00,
    "valor_aprovado": 300.00,
    "valor_excedente": 50.00
  },
  "status": {
    "status_atual": "Aguardando documentação",
    "status_adicional": "Em análise financeira"
  },
  "reincidente": "sim",
  "tags": [
    { "nome": "Prioridade Alta" },
    { "nome": "Cliente VIP" }
  ],
  "arquivos": [
    {
      "nome": "nota_fiscal.pdf",
      "url": "https://api.veniti.com/arquivos/nota_fiscal.pdf",
      "data_envio": "2026-03-10T14:40:02"
    }
  ],
  "perguntas": [
    {
      "pergunta": "O pagamento foi realizado pelo associado?",
      "resposta": "Sim"
    },
    {
      "pergunta": "Possui comprovante de pagamento?",
      "resposta": "Sim"
    }
  ],
  "observacoes": "Reembolso autorizado conforme análise da assistência."
}
```

---

## Dependências

- [[solicitacao-reembolso]]
- [[perguntas-reembolso]]
- [[tags-reembolso]]
- [[status-personalizados-reembolso]]
- [[processamento-pagamento-reembolso]]
