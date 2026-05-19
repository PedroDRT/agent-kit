---
type: feature
module: beneficiario
layer: feature
related:
  - fluxo-busca-prestador
  - portal-self-service-beneficiario
  - criacao-atendimento
  - login-multi-portal
---

# Cadastro de Beneficiário

> Permite registrar, editar e gerenciar beneficiários vinculados a um plano de um cliente (seguradora).

## Descrição

Permite registrar, editar e gerenciar beneficiários vinculados a um plano de um cliente (seguradora). Um beneficiário é o titular do serviço de assistência — a pessoa ou veículo que será atendido em uma solicitação. O cadastro pode incluir veículos, pets e dados de contato.

---

## Entradas

### Dados Pessoais

| Campo | Tipo | Descrição |
|---|---|---|
| `nome` | string | Nome completo |
| `cpf_cnpj` | string | CPF ou CNPJ |
| `nome_social` | string | Nome social (opcional) |
| `email` | string | E-mail de contato |
| `telefones` | array | Lista de telefones |
| `data_nascimento` | date | Data de nascimento |
| `id_plano` | integer | Plano ao qual está vinculado |

### Veículos (beneficiarios_veiculos)

| Campo | Tipo | Descrição |
|---|---|---|
| `placa` | string | Placa do veículo |
| `marca` | string | Marca |
| `modelo` | string | Modelo |
| `cor` | string | Cor |
| `ano_fabricacao` | integer | Ano de fabricação |
| `ano_modelo` | integer | Ano do modelo |
| `chassi` | string | Número do chassi |

### Pets (beneficiarios_pets)

| Campo | Tipo | Descrição |
|---|---|---|
| `nome` | string | Nome do pet |
| `especie` | string | Espécie (cão, gato, etc.) |
| `raca` | string | Raça |
| `peso` | decimal | Peso em kg |
| `porte` | string | Pequeno, médio, grande |
| `sexo` | string | Macho ou Fêmea |

### Upload em Lote

| Campo | Descrição |
|---|---|
| Arquivo CSV/Excel | Template padrão para importação em massa |

## Saídas

- Registro criado/atualizado na tabela `clientes_beneficiarios`
- Veículos e pets vinculados em suas respectivas tabelas
- Beneficiário disponível para seleção em novos atendimentos
- Arquivos associados armazenados no AWS S3

---

## Regras de Negócio

- Um beneficiário é sempre vinculado a um `id_plano` de um cliente (`id_veniti`)
- CPF/CNPJ é identificador único por tenant — não pode ser duplicado no mesmo plano
- A importação em lote via upload usa `html/assistencia/beneficiarios/__acoes_upload_beneficiarios.php` (54KB de lógica)
- Beneficiários podem ser localizados via busca integrada (`BuscadorClienteBeneficiario`) que consulta bases externas (MaxPar, CDF, Tato, TempoAssist, etc.)
- Um beneficiário pode ter múltiplos veículos e múltiplos pets
- A operação de edição requer permissão específica no perfil do usuário interno
- Beneficiários com atendimentos em aberto não podem ser excluídos
- Dados de beneficiários podem ser pré-preenchidos via integrações externas (SGA, CDF)

## Casos de Borda

- Upload em lote com CPFs inválidos ou duplicados no arquivo
- Beneficiário com mesmo CPF em dois planos diferentes do mesmo cliente
- Edição de beneficiário durante atendimento ativo
- Veículo com placa já vinculada a outro beneficiário no mesmo tenant
- Pet com porte incompatível com o peso informado
- Beneficiário não encontrado nas bases externas de busca

---

## Dependências

- **Portais**: `html/assistencia/beneficiarios/` (gestão interna), `html/cliente/beneficiarios/` (visão cliente)
- **Busca externa**: `src/Models/BuscadorClienteBeneficiario.php` (MaxPar, CDF, Tato, TempoAssist)
- **Upload em lote**: `html/assistencia/beneficiarios/__acoes_upload_beneficiarios.php`
- **Storage**: AWS S3 (`src/Models/S3.php`) para arquivos vinculados
- **Banco**: `clientes_beneficiarios`, `clientes_beneficiarios_veiculos`, `clientes_beneficiarios_pets`
- **Integrações**: SGA (`src/Integracoes/SGA/`), CDF billing

## Flows Relacionados

- [[fluxo-busca-prestador]]

## Features Relacionadas

- [[portal-self-service-beneficiario]]
- [[criacao-atendimento]]
- [[login-multi-portal]]
