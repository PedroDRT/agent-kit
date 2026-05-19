# Vínculo de Plano ao Beneficiário

## Description

Processo de associação de um beneficiário (pessoa física ou jurídica) a um plano contratado por um cliente (seguradora). O vínculo determina quais serviços o beneficiário pode solicitar, os limites de cobertura, e como os créditos serão calculados. Inclui também o vínculo de veículos e pets ao plano.

**Hierarquia de vínculo:**
```
Cliente (Seguradora)
  └── Plano (ex: "Plano Ouro Reboque + Assistência")
        └── Beneficiário (titular da apólice)
              ├── Veículos vinculados ao plano
              └── Pets vinculados ao plano
```

## Inputs

### Vínculo Beneficiário ↔ Plano

| Campo | Tipo | Descrição |
|---|---|---|
| `id_beneficiario` | integer | Beneficiário a ser vinculado |
| `id_plano` | integer | Plano ao qual será vinculado |
| `data_vigencia_inicio` | date | Início da cobertura |
| `data_vigencia_fim` | date | Fim da cobertura (opcional — planos abertos) |
| `numero_apolice` | string | Número da apólice do beneficiário |
| `id_contrato` | integer | Contrato do cliente com a seguradora |

### Vínculo Veículo ↔ Plano (`beneficiarios_veiculos_planos`)

| Campo | Tipo | Descrição |
|---|---|---|
| `id_beneficiario_veiculo` | integer | Veículo do beneficiário |
| `id_plano` | integer | Plano |
| `data_vigencia_inicio` | date | Início cobertura do veículo |
| `data_vigencia_fim` | date | Fim cobertura |

### Vínculo Pet ↔ Plano (`beneficiarios_pets_planos`)

| Campo | Tipo | Descrição |
|---|---|---|
| `id_beneficiario_pet` | integer | Pet do beneficiário |
| `id_plano` | integer | Plano |
| `data_vigencia_inicio` | date | Início cobertura |
| `data_vigencia_fim` | date | Fim cobertura |

## Outputs

- Registro em `clientes_beneficiarios` com `id_plano` ativo
- Registros em `beneficiarios_veiculos_planos` e/ou `beneficiarios_pets_planos`
- Beneficiário disponível para criação de Eventos com o plano vinculado
- Créditos associados ao plano calculados conforme configuração (`per_tipo_veiculo`, `por_categorizacao_veiculo`)

## Business Rules

- Um beneficiário pode estar vinculado a **múltiplos planos** de **múltiplos clientes** (multi-tenant)
- A cobertura é validada pela vigência do plano no momento da criação do Evento/Atendimento
- O plano define os **tipos de serviço cobertos** — atendimentos fora do escopo são negados
- A precificação varia conforme configuração do plano: `per_tipo_veiculo` (por tipo de veículo) ou `por_categorizacao_veiculo` (por categoria)
- Beneficiários importados em lote (`__acoes_upload_beneficiarios.php`) são vinculados automaticamente ao plano da importação
- O vínculo pode ser desativado sem excluir o beneficiário (suspensão de cobertura)
- Planos têm limitações por tipo de evento (`modal-register-limitation-event-type.php`) — ex: máximo de 2 reboques por ano
- Dados do vínculo são cruzados com o cálculo de crédito em `src/UseCases/CalcularValorCredito.php`

## Edge Cases

- Vínculo criado com data de início no passado (cobertura retroativa)
- Beneficiário solicita serviço com plano expirado (`data_vigencia_fim` no passado)
- Limite de uso do plano atingido (ex: 2 reboques/ano esgotados)
- Veículo vinculado a plano diferente do beneficiário titular
- Beneficiário com dois planos ativos simultaneamente do mesmo cliente
- Importação em lote sobrepõe vínculos existentes (atualiza ou duplica?)
- Pet de espécie não coberta pelo plano vinculado

## QA Notes

- **Risco crítico:** Validação de cobertura no momento do Evento — o sistema bloqueia criação ou apenas registra para análise posterior?
- **Risco:** Limite de usos do plano (ex: 2 reboques/ano) — quando e onde é verificado? No Evento, no Atendimento, ou no Acionamento?
- **Validation gap:** Não há evidência de validação de CPF/CNPJ do beneficiário nas bases externas antes do vínculo
- **Comportamento unclear:** Se um veículo é vinculado a dois planos diferentes, qual plano é usado na criação do Atendimento?
- **Edge case:** Importação em lote com arquivo contendo beneficiários sem CPF/CNPJ válido — o sistema rejeita toda a linha ou ignora o campo?

## Dependencies

- **Portais**: `html/assistencia/beneficiarios/` (cadastro e vínculo), `html/assistencia/planos/`
- **Upload lote**: `html/assistencia/beneficiarios/__acoes_upload_beneficiarios.php`
- **Busca externa**: `src/Models/BuscadorClienteBeneficiario.php`
- **Banco**: `clientes_beneficiarios`, `beneficiarios_veiculos_planos`, `beneficiarios_pets_planos`, `clientes_planos`
- **Crédito**: `src/UseCases/CalcularValorCredito.php`

## Related Flows

- [[atendimento-lifecycle]]

## Related Features

- [[cadastro-beneficiario]]
- [[gestao-evento]]
- [[calculo-credito]]
