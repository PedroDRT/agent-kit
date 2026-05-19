---
type: feature
module: fechamento
layer: feature
related:
  - fechamento-financeiro
  - agrupamento-lote
  - calculo-credito
---

# Serviços de Prestadores Parceiros em Fechamento por Clientes

> Permite configurar a exibição e contabilização de atendimentos realizados por prestadores parceiros no fechamento por cliente.

## Descrição

Permite configurar, no contrato do cliente e por tipo de cobrança, a exibição e contabilização de atendimentos realizados por prestadores parceiros no fechamento por cliente. A configuração é feita via dois checkboxes com dependência hierárquica: "Exibir atendimentos de prestadores parceiros" e "Contabilizar valor de atendimento" (visível apenas quando o primeiro está habilitado). Inclui também nova coluna "Parceiro" (SIM/NÃO) nos datatables dos tipos de cobrança "Atendimento mais" e "Tabela de valor".

---

## Entradas

### Novas Configurações (UI)

**Localização:** Contrato do cliente > tipo de cobrança elegível ("Atendimento mais R$/%" ou "Tabela de valores")

#### Checkbox 1: "Exibir atendimentos de prestadores parceiros"
- **Tooltip:** "Ao habilitar esta configuração, o sistema passará a exibir no fechamento os atendimentos realizados por prestadores parceiros. Esses atendimentos serão apresentados com valor zerado, não impactando os cálculos financeiros."
- **Comportamento quando habilitado:**
  - Atendimentos de prestadores parceiros passam a ser exibidos no fechamento
  - São apresentados com valor zerado
  - Não impactam o valor total do lote

#### Checkbox 2: "Contabilizar valor de atendimento"
- **Visibilidade:** Só é exibido quando o checkbox 1 está habilitado
- **Tooltip:** "Ao habilitar esta configuração, os atendimentos de prestadores parceiros terão seus valores calculados e considerados no fechamento, impactando o valor total."
- **Comportamento quando habilitado:**
  - Atendimentos de parceiros deixam de ter valor zerado
  - Passam a ser contabilizados normalmente
  - Seguem integralmente as regras do tipo de cobrança configurado

#### Nova Coluna "Parceiro" nos Datatables
- Presente nos datatables de "Atendimento mais" e "Tabela de valor" (Fechamento > Por Clientes > Detalhes)
- Valores: SIM / NÃO
- Oculta por padrão, habilitável via botão "Altere as colunas"
- Quando habilitada, incluída nas exportações (XLS, CSV, PDF)
- NÃO deve aparecer em outros tipos de cobrança

## Saídas

- Configuração persistida por tipo de cobrança no contrato do cliente
- Fechamento exibe ou oculta atendimentos de parceiros conforme configuração
- Coluna "Parceiro" disponível nos datatables elegíveis
- Exportações (XLS, CSV, PDF) incluem coluna "Parceiro" quando habilitada

---

## Regras de Negócio

### Regras Gerais (Matriz de Configuração)

| Exibir | Contabilizar | Comportamento |
|---|---|---|
| ❌ | — (oculto) | Comportamento atual mantido — atendimentos de parceiros não aparecem |
| ✅ | ❌ | Atendimentos aparecem com valor zerado, sem impacto no total |
| ✅ | ✅ | Atendimentos aparecem com valor calculado, somados ao total, regras do tipo de cobrança aplicadas |

- Tipos de cobrança elegíveis: apenas "ATENDIMENTOS MAIS" e "TABELA DE VALOR"
- Ao desmarcar "Exibir atendimentos de prestadores parceiros", o checkbox "Contabilizar valor de atendimento" é automaticamente desmarcado e ocultado
- A relação de parceria é por cliente, não é uma flag global do prestador (verificar em Negócio > Clientes > aba "Parceiros")
- Cada tipo de cobrança tem configuração independente; podem coexistir no mesmo fechamento
- Formatos de exportação XLS, CSV e PDF devem incluir a coluna "Parceiro" quando habilitada
- Configuração afeta apenas fechamentos futuros (não há retroatividade)

## Critérios de Aceitação

1. Dado que o usuário acessa o contrato do cliente, quando visualizar os tipos de cobrança configuráveis, então o sistema deve apresentar as novas opções de configuração para prestadores parceiros
2. Dado que o tipo de cobrança seja um dos elegíveis ("Atendimento mais R$/%" ou "Tabela de valores"), quando a tela de configuração for exibida, então deve existir o checkbox "Exibir atendimentos de prestadores parceiros"
3. Dado que o checkbox "Exibir atendimentos de prestadores parceiros" esteja desabilitado, quando o fechamento for processado, então atendimentos de prestadores parceiros não devem ser exibidos
4. Dado que o checkbox "Exibir atendimentos de prestadores parceiros" esteja habilitado, quando o fechamento for processado, então os atendimentos de prestadores parceiros devem ser exibidos com valor zerado
5. Dado que o checkbox "Exibir atendimentos de prestadores parceiros" esteja habilitado, quando a tela for renderizada, então o checkbox "Contabilizar valor de atendimento" deve ser exibido
6. Dado que o checkbox "Contabilizar valor de atendimento" esteja desabilitado, quando os atendimentos de parceiros forem exibidos, então seus valores devem permanecer zerados e não impactar o total
7. Dado que o checkbox "Contabilizar valor de atendimento" esteja habilitado, quando o fechamento for processado, então os atendimentos de prestadores parceiros devem ser calculados e somados ao total do lote
8. Dado que os atendimentos de parceiros estejam sendo contabilizados, quando o sistema aplicar as regras de cálculo, então deve utilizar as mesmas regras do tipo de cobrança aplicado
9. Dado qualquer configuração não habilitada, quando o fechamento for processado, então o comportamento atual não deve ser impactado

## Comportamentos Confirmados

| Comportamento | Confirmado | Caso de teste |
|---|---|---|
| Checkbox 2 aparece apenas quando Checkbox 1 está habilitado | ✅ Sim | CT001.5 |
| Desmarcar Checkbox 1 → Checkbox 2 desmarcado e ocultado automaticamente | ✅ Sim | CT001.6 |
| Exibir OFF → parceiros não aparecem no fechamento | ✅ Sim | CT002.1 |
| Exibir ON + Contabilizar OFF → parceiros com valor R$ 0,00 (total inalterado) | ✅ Sim | CT002.2, CT002.4 |
| Exibir ON + Contabilizar ON → parceiros com valor calculado, somado ao total | ✅ Sim | CT002.3 |
| Regras de cálculo do tipo de cobrança aplicadas igualmente a parceiros | ✅ Sim | CT002.5 |
| Coluna "Parceiro" oculta por padrão em ambos os datatables elegíveis | ✅ Sim | CT003.1, CT003.4 |
| Coluna "Parceiro" habilitável via "Altere as colunas" | ✅ Sim | CT003.2, CT003.5 |
| Coluna "Parceiro" exibe SIM/NÃO corretamente por atendimento | ✅ Sim | CT003.3 |
| Coluna "Parceiro" ausente em tipos não elegíveis | ✅ Sim | CT003.6 |
| Exportação XLS/CSV/PDF inclui coluna "Parceiro" quando habilitada | ✅ Sim | CT004.1–CT004.3, CT004.5 |
| Exportação sem coluna "Parceiro" quando oculta (padrão) | ✅ Sim | CT004.4 |
| Toast "Valor cadastrado com sucesso" ao salvar configuração | ✅ Sim | CT001.8 |
| Configuração persiste após reload da página | ✅ Sim | CT001.8 |
| Cada tipo de cobrança com configuração independente | ✅ Sim | CT001.9 |
| Sem configuração habilitada → comportamento atual inalterado | ✅ Sim | CT002.6 |
| Tipos elegíveis: apenas "ATENDIMENTOS MAIS" e "TABELA DE VALOR" | ✅ Sim | CT001.1–CT001.3 |

---

## Escopo de Teste

- Configuração dos checkboxes no contrato do cliente (Negócio > Clientes > selecionar cliente > contrato > tipo de cobrança)
- Dependência hierárquica entre os dois checkboxes (Exibir → Contabilizar)
- Comportamento do fechamento por cliente (Fechamento > Por Clientes > Detalhes) nas 3 combinações:
  - Não exibir (comportamento atual mantido)
  - Exibir com valor zerado
  - Exibir e contabilizar normalmente
- Nova coluna "Parceiro" (SIM/NÃO) nos datatables de "Atendimento mais" e "Tabela de valor"
  - Coluna oculta por padrão, habilitável via botão "Altere as colunas"
  - Exibida para todos os atendimentos (parceiros e não parceiros)
- Exportação do fechamento (XLS, CSV, PDF) incluindo coluna "Parceiro" quando habilitada
- Verificar que a coluna "Parceiro" NÃO aparece em outros tipos de cobrança
- Verificar que tipos de cobrança não elegíveis não exibem os checkboxes de configuração
- Retrocompatibilidade: configuração desabilitada = comportamento atual inalterado

## Fora do Escopo

- Fechamento com prestador (este é fechamento por **cliente**)
- Negociação de tarifas (fluxo bilateral Assistência ↔ Prestador)
- Cálculo de crédito em si (apenas a aplicação das regras existentes aos atendimentos de parceiros)
- Cadastro/gestão de prestadores parceiros
- Fechamentos já processados (a configuração afeta apenas fechamentos futuros)

---

## Notas de QA

- **Retrocompatibilidade confirmada:** Nenhuma configuração habilitada = comportamento anterior inalterado (CT002.6).
- **Parceria por cliente:** A relação de parceria é configurada em Negócio > Clientes > aba "Parceiros" — não é flag global do prestador.
- **Independência por tipo de cobrança confirmada:** Dois tipos elegíveis no mesmo contrato podem ter configurações diferentes (CT001.9).
- **Coluna "Parceiro" exibe ambos (SIM e NÃO):** Quando habilitada, aparece para todos os atendimentos — não apenas os de parceiros.

## Dependências

- Prestadores parceiros configurados para o cliente de teste (Negócio > Clientes > aba "Parceiros")
- Contrato do cliente com tipos de cobrança "Atendimento mais R$/%" e/ou "Tabela de valores" configurados
- Atendimentos vinculados a prestadores parceiros disponíveis no ambiente de testes

## Features Relacionadas

- [[fechamento-financeiro]]
- [[agrupamento-lote]]
- [[calculo-credito]]
