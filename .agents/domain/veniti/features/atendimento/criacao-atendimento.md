---
type: feature
module: atendimento
layer: feature
related:
  - fluxo-atendimento-completo
  - fluxo-despacho-prestador
  - gestao-status-atendimento
  - ciclo-vida-acionamento
  - calculo-credito
---

# Criação de Atendimento

> Feature responsável por registrar uma nova solicitação de assistência (atendimento).

## Descrição

Feature responsável por registrar uma nova solicitação de assistência (atendimento). Representa a entrada principal do ciclo de vida de um serviço no sistema Veniti. Um atendimento pode ser criado manualmente pela operação ou de forma automatizada via extensão externa.

---

## Entradas

| Campo | Tipo | Obrigatoriedade |
|---|---|---|
| `id_plano` | integer | Obrigatório |
| `tipo_servico` | string | Obrigatório |
| `beneficiario_nome` | string | Obrigatório |
| `beneficiario_cpf_cnpj` | string | Obrigatório |
| `beneficiario_nome_social` | string | Opcional |
| `solicitante` | string | Obrigatório |
| `solicitante_telefones` | string | Obrigatório |
| `solicitante_email` | string | Opcional |
| `veiculo_placa` | string | Condicional (serviços veiculares) |
| `veiculo_marca` | string | Condicional |
| `veiculo_modelo` | string | Condicional |
| `veiculo_cor` | string | Condicional |
| `veiculo_ano_fabricacao` | integer | Condicional |
| `Ologradouro`, `Ocidade`, `Obairro`, `Ocep` | string | Obrigatório (endereço origem) |
| `Olat`, `Olng` | decimal | Obrigatório (geolocalização origem) |
| `Dlogradouro`, `Dcidade`, `Dbairro`, `Dcep` | string | Opcional (endereço destino) |
| `Dlat`, `Dlng` | decimal | Opcional (geolocalização destino) |
| `obs` | text | Opcional |
| `data_agendamento` | datetime | Condicional (atendimento agendado) |
| `numero_bo` | string | Opcional (boletim de ocorrência) |
| **Dados de pet** | | |
| `pet_nome`, `pet_raca`, `pet_especie` | string | Condicional (serviços pet) |
| `pet_peso`, `pet_porte`, `pet_sexo` | string/decimal | Condicional |

## Saídas

- Registro de atendimento criado na tabela `atendimentos`
- `id` e `protocolo` gerados automaticamente
- Status inicial: `ABERTO`
- Evento `AttendanceCreated` disparado (vincula tags automaticamente via `LinkIdentifierTagsToAttendanceListener`)
- Atendimento disponível no painel de operação dos portais Assistência e Cliente

---

## Regras de Negócio

- Todo atendimento é vinculado a um `id_veniti` (empresa/cliente tenant)
- O `protocolo` é único e gerado pelo sistema — não pode ser definido manualmente
- `tipo_servico` deve existir na tabela de serviços cadastrados para o plano
- Se `bloquear_acionamento = true`, o atendimento não pode ser despachado para prestadores
- Atendimentos agendados requerem `data_agendamento` e `data_agendamento_inicio`
- Serviços com animais de estimação exigem todos os campos `pet_*` preenchidos
- O campo `reembolso` define se o atendimento foi solucionado via reembolso
- Um atendimento pode ser criado pelo portal Assistência (interno) ou pela extensão externa (`html/extensao/create_attendance.php`)
- O campo `id_veniti` é sempre inferido da sessão autenticada, nunca da entrada do usuário

## Casos de Borda

- Atendimento criado com endereço de origem inválido (geolocalização ausente ou fora do Brasil)
- Plano vencido ou beneficiário sem cobertura ativa
- Tipo de serviço não coberto pelo plano do beneficiário
- Atendimento duplicado para o mesmo beneficiário no mesmo período
- Criação via extensão com token de API inválido ou expirado
- Pet registrado com peso incompatível com porte declarado

---

## Dependências

- **APIs/Serviços externos**: Google Maps / HERE Maps / TomTom (validação e geocodificação de endereços)
- **Banco de dados**: tabelas `atendimentos`, `clientes_planos`, `clientes_beneficiarios`, `tipos_servicos`
- **Integrações**: `html/extensao/create_attendance.php`, `src/Integracoes/AlteracoesAtendimento/`
- **Módulos internos**: Autenticação de sessão, `src/Domain/Attendance/Events/AttendanceCreated`

## Flows Relacionados

- [[fluxo-atendimento-completo]]
- [[fluxo-despacho-prestador]]

## Features Relacionadas

- [[gestao-status-atendimento]]
- [[ciclo-vida-acionamento]]
- [[calculo-credito]]
