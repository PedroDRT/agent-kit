# Fluxo de Busca de Prestador

## Description

Permite ao beneficiário localizar prestadores de serviço disponíveis em sua região através do portal self-service. Utiliza geolocalização do dispositivo ou endereço informado para exibir prestadores no mapa e listar suas informações de contato e cobertura.

## Steps

### 1. Acesso ao Portal do Beneficiário
- **Ator**: Beneficiário
- **Portal**: `/beneficiario/a/` via link único recebido por SMS/e-mail
- **Token**: Link contém token que identifica o beneficiário e o contexto do serviço

### 2. Exibição do Mapa
- **Sistema**: Carrega `map_beneficiario.php` com Leaflet.js
- **Base cartográfica**: OpenStreetMap ou Google Maps (configurável por tenant)
- **Localização inicial**: Geolocalização do dispositivo (browser API) ou endereço informado

### 3. Busca de Prestadores Próximos
- **Sistema**: Chama `search_provider.php` com coordenadas do beneficiário
- **Backend**: `BuscadorPrestadores` filtra por:
  - Tipo de serviço (inferido do contexto do atendimento ou selecionado)
  - Raio de distância configurável
  - Prestadores ativos e online
- **Resultado**: Lista de prestadores com nome, distância, telefone e serviços

### 4. Visualização no Mapa
- **Sistema**: Pins plotados no mapa para cada prestador encontrado
- **Interação**: Clique no pin exibe detalhes (nome, telefone, serviços oferecidos)
- **Atualização**: Mapa recarrega se beneficiário mover a visualização

### 5. Contato com Prestador (Opcional)
- **Ator**: Beneficiário
- **Ação**: Clica no número de telefone exibido → inicia chamada direta (mobile)
- **Sistema**: Não registra o contato automaticamente — registro manual pela operação se necessário

### 6. Retorno ao Fluxo Principal
- **Ator**: Beneficiário
- **Ação**: Retorna ao portal para acompanhar atendimento ou completar reembolso
- **Sistema**: Histórico de navegação preservado pelo token de sessão

## Entry Points

- Link único recebido por SMS/e-mail: `https://{tenant}/beneficiario/a/?token=...`
- A partir do portal de acompanhamento do atendimento

## Exit Points

- **Prestador encontrado e contatado**: Beneficiário segue para atendimento autônomo → possível reembolso posterior
- **Nenhum prestador encontrado**: Sistema exibe mensagem e sugere contato com a central
- **Retorno ao portal**: Beneficiário volta ao fluxo de acompanhamento ou reembolso

## Variations

### Busca por Endereço Manual
- Beneficiário digita o endereço quando geolocalização do dispositivo não está disponível
- Sistema geocodifica o endereço via Google Maps API

### Filtro por Tipo de Serviço
- Beneficiário seleciona o tipo de assistência necessária
- Lista filtrada por serviços compatíveis

### Modo Sem Internet (Degradado)
- Mapa não carrega
- Sistema exibe número de telefone da central como fallback

## Related Features

- [[portal-self-service-beneficiario]]
- [[cadastro-beneficiario]]
- [[dispatch-automatico]]
