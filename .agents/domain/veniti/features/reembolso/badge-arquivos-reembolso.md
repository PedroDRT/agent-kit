# Badge de Identificação de Arquivos do Beneficiário

## Resumo

Na aba de **Arquivos** do reembolso, todos os arquivos enviados pelo beneficiário através do **link do beneficiário** são identificados com uma badge visual, indicando a origem do arquivo.

---

## Comportamento

- A badge é exibida na listagem de arquivos do reembolso, junto ao arquivo
- Apenas arquivos enviados via **link do beneficiário** recebem a badge
- Arquivos enviados internamente pela operação **não** recebem a badge
- A badge exibe o label **"BENEFICIÁRIO"** (conforme screenshot do PDF)

---

## Localização

**Aba:** Arquivos (dentro do reembolso)

---

## Critérios de Aceitação

1. Arquivos enviados via link do beneficiário devem exibir badge de identificação
2. Arquivos enviados internamente pela operação não devem exibir badge
3. A badge deve ser visualmente identificável na listagem de arquivos

---

## Dependências

- [[link-reembolso]] — arquivos são enviados via link do beneficiário
- Aba Arquivos do reembolso
