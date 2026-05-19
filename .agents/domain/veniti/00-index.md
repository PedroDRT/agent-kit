# Veniti — Knowledge Base Index

> Sistema multi-portal de gestão de assistência veicular e pessoal (B2B2C).
> Source of truth: `application.json` → `webApp.path: ../veniti`
> Gerado por: QA Architect Agent

---

## Application Sources

| Source | Path | Stack |
|---|---|---|
| Web App | `../veniti/html/` | PHP 7.4+, Vanilla JS, Tailwind CSS |
| Business Logic | `../veniti/src/` | PHP — Domain, Models, UseCases, Integrations |
| Entry Point | `../veniti/html/index.php` | Subdomain routing para portais |

## Portals

| Portal | Audience | Path |
|---|---|---|
| **Assistência** | Operadores internos | `html/assistencia/` |
| **Cliente** | Seguradoras / empresas | `html/cliente/` |
| **Prestador** | Provedores de serviço | `html/prestador/` |
| **Beneficiário** | Cliente final (self-service) | `html/beneficiario/` |
| **Reembolso** | Cliente final (self-service) | `html/reembolso/` |
| **Extensão** | Integrações externas (API) | `html/extensao/` |

---

## Modules

### Atendimento
Core do sistema — representa a solicitação de serviço de assistência.

| Type | File |
|---|---|
| Feature | [[criacao-atendimento]] |
| Feature | [[gestao-status-atendimento]] |
| Flow | [[fluxo-atendimento-completo]] |

### Acionamento
Despacho e acompanhamento de prestadores em campo.

| Type | File |
|---|---|
| Feature | [[ciclo-vida-acionamento]] |
| Feature | [[dispatch-automatico]] |
| Flow | [[fluxo-despacho-prestador]] |

### Reembolso
Solicitação e processamento de reembolso financeiro ao beneficiário.

| Type | File |
|---|---|
| Feature | [[solicitacao-reembolso]] |
| Feature | [[processamento-pagamento-reembolso]] |
| Feature | `features/reembolso/link-reembolso.md` |
| Feature | `features/reembolso/status-personalizados-reembolso.md` |
| Feature | `features/reembolso/perguntas-reembolso.md` |
| Feature | `features/reembolso/tags-reembolso.md` |
| Feature | `features/reembolso/ocorrencias-reembolso.md` |
| Feature | `features/reembolso/badge-arquivos-reembolso.md` |
| Flow | [[fluxo-reembolso-completo]] |
| Flow | `flows/reembolso/status-personalizados-reembolso.md` |
| Flow | `flows/reembolso/perguntas-reembolso.md` |
| Flow | `flows/reembolso/tags-reembolso.md` |
| Selectors | `selectors/reembolso/status-personalizados-reembolso.json` |
| Integration | `integrations/reembolso/api-reembolsos.md` |

### Autenticação
Login e recuperação de acesso nos três portais.

| Type | File |
|---|---|
| Feature | [[login-multi-portal]] |
| Feature | [[recuperacao-senha]] |
| Flow | [[fluxo-login]] |

### Beneficiário
Cadastro e portal self-service do beneficiário.

| Type | File |
|---|---|
| Feature | [[cadastro-beneficiario]] |
| Feature | [[portal-self-service-beneficiario]] |
| Flow | [[fluxo-busca-prestador]] |

### Créditos
Cálculo e registro de créditos gerados por atendimentos finalizados. Controle de saldo do cliente (seguradora) com o Veniti. Módulo independente de Fechamento e Lote.

| Type | File |
|---|---|
| Feature | [[calculo-credito]] |
| Flow | [[fluxo-calculo-credito]] |
| Flow | [[fluxo-credito-para-pagamento]] |
| Business Rules | `business-rules/creditos.md` |
| Selectors | `selectors/creditos/creditos.json` |

### Evento
Incidente reportado pelo beneficiário — container pai de atendimentos.

| Type | File |
|---|---|
| Feature | [[gestao-evento]] |
| Feature | [[vinculo-plano-beneficiario]] |

### Prestador
Execução do serviço em campo pelo prestador.

| Type | File |
|---|---|
| Feature | [[execucao-servico]] |
| Flow | [[execucao-atendimento]] |

### Fechamento
Encerramento financeiro bilateral entre Assistência, Clientes e Prestadores.

| Type | File |
|---|---|
| Feature (cliente) | `features/fechamento/por-cliente/fechamento-por-cliente.md` |
| Feature (prestador) | `features/fechamento/por-prestador/fechamento-por-prestador.md` |
| Feature (parceiros) | `features/fechamento/por-cliente/servicos-prestadores-parceiros.md` |
| Flow (cliente) | `flows/fechamento/por-cliente/fechamento-por-cliente.md` |
| Flow (prestador) | `flows/fechamento/por-prestador/fechamento-por-prestador.md` |

### Lote (Faturamento)
Agrupamento de acionamentos finalizados em instrumento de pagamento ao prestador.

| Type | File |
|---|---|
| Feature | [[agrupamento-lote]] |
| Flow | [[faturamento-lote]] |

### Relatórios
Relatórios gerenciais no portal Assistência.

| Type | File | Módulo |
|---|---|---|
| Feature | `features/relatorios/contratos-por-cliente/contratos-por-cliente.md` | Contratos por Cliente |
| Feature | `features/relatorios/conectividade-por-prestador/conectividade-por-prestador.md` | Conectividade de Prestadores |
| Feature | `features/relatorios/atendimentos/relatorio-atendimentos.md` | Atendimentos |
| Flow | `flows/relatorios/contratos-por-cliente/fluxo-relatorio-contratos.md` | Contratos por Cliente |
| Flow | `flows/relatorios/conectividade-por-prestador/conectividade-por-prestador.md` | Conectividade de Prestadores |
| Flow | `flows/relatorios/atendimentos/fluxo-relatorio-atendimentos.md` | Atendimentos |
| Selectors | `selectors/relatorios/contratos-por-cliente.json` | Contratos por Cliente |
| Selectors | `selectors/relatorios/conectividade-por-prestador.json` | Conectividade de Prestadores |
| Selectors | `selectors/relatorios/atendimentos.json` | Atendimentos |

---

## Core Business Lifecycle (CRITICAL PATH)

```
FASE 1 — PRÉ-REQUISITOS
  [[cadastro-beneficiario]]
    └── [[vinculo-plano-beneficiario]]

FASE 2 — INCIDENTE
  [[gestao-evento]] (Evento criado)
    └── [[criacao-atendimento]] (Atendimento vinculado ao Evento)

FASE 3 — ACIONAMENTO
  [[ciclo-vida-acionamento]]
    ├── [[dispatch-automatico]] (acionamento automático)
    └── manual via /assistencia/operacao/acionamento.php

FASE 4 — EXECUÇÃO
  [[execucao-servico]] (Prestador em campo)
    └── [[execucao-atendimento]] (flow completo de execução)

FASE 5 — FINALIZAÇÃO
  [[gestao-status-atendimento]] → FINALIZADO
    ├── [[fluxo-calculo-credito]] (débito automático no saldo do cliente)
    └── eligível para lote do prestador

FASE 6 — FECHAMENTO FINANCEIRO (dois fluxos independentes)
  A. Crédito Cliente → [[fluxo-credito-para-pagamento]]
       └── clientes_contratos_creditos (débito) → saldo do contrato
  B. Lote Prestador → [[faturamento-lote]]
       └── fechamento_prestador → contas_pagar → Pagamento ao Prestador

CAMINHO ALTERNATIVO — REEMBOLSO
  [[fluxo-reembolso-completo]]
    └── [[portal-self-service-beneficiario]]
```

> **Flow mestre:** [[atendimento-lifecycle]] — descreve todas as fases end-to-end.

---

## Key Business Flows

```
Solicitação de Assistência (end-to-end)
  └── [[atendimento-lifecycle]]                 ← FLOW PRINCIPAL
        ├── [[fluxo-atendimento-completo]]      (lifecycle simplificado)
        ├── [[fluxo-despacho-prestador]]        (seleção e notificação do prestador)
        ├── [[execucao-atendimento]]             (execução em campo)
        ├── [[fluxo-calculo-credito]]            (crédito ao finalizar)
        ├── [[fechamento-por-cliente]]           (faturamento ao cliente)
        ├── [[fechamento-por-prestador]]         (negociação bilateral prestador)
        ├── [[faturamento-lote]]                 (lote de pagamento ao prestador)
        └── [[fluxo-reembolso-completo]]         (alternativa sem prestador)
              └── [[portal-self-service-beneficiario]]

Autenticação
  └── [[fluxo-login]]
        └── [[recuperacao-senha]]

Cadastro
  └── [[cadastro-beneficiario]]
        └── [[vinculo-plano-beneficiario]]
              └── [[gestao-evento]] → [[criacao-atendimento]]
```

---

## Architecture Summary

| Layer | Technology |
|---|---|
| Backend | PHP 7.4+, PDO (MySQL), custom MVC-light |
| Database | MySQL (primary), TimescaleDB (events), Redis (sessions/cache) |
| Frontend | Vanilla JS, DataTables, Leaflet.js, Tailwind CSS, Socket.io (Pusher) |
| AI | OpenAI (invoice processing via `src/CoreAI/`) |
| Storage | AWS S3 (files, logos, CSS) |
| Maps | Google Maps, HERE Maps, TomTom, OSRM |
| Infrastructure | Docker, Nginx, Filebeat, Kibana |
| Notifications | SMS, E-mail (PHPMailer), WebSocket (Pusher) |
| E-commerce | Tray integration (`src/Tray/`) |

---

## Key External Integrations

| Integration | Purpose |
|---|---|
| SGA | Sincronização de dados de serviço |
| MaxPar / CDF / Tato / TempoAssist | Busca de beneficiários em bases externas |
| Agiliza | Envio de notificações de alteração de atendimento |
| OpenAI | Processamento inteligente de notas fiscais |
| Google reCAPTCHA | Proteção de formulários de login |

---

## Folder Structure (this vault)

```
/knowledge/system/
  00-index.md                                      ← Este arquivo
  features/
    atendimento/
      criacao-atendimento.md
      gestao-status-atendimento.md
    acionamento/
      ciclo-vida-acionamento.md
      dispatch-automatico.md
    reembolso/
      solicitacao-reembolso.md
      processamento-pagamento-reembolso.md
      link-reembolso.md
      status-personalizados-reembolso.md
      perguntas-reembolso.md
      tags-reembolso.md
      ocorrencias-reembolso.md
      badge-arquivos-reembolso.md
    autenticacao/
      login-multi-portal.md
      recuperacao-senha.md
    beneficiario/
      cadastro-beneficiario.md
      portal-self-service-beneficiario.md
    creditos/
      calculo-credito.md
    evento/
      gestao-evento.md
      vinculo-plano-beneficiario.md
    prestador/
      execucao-servico.md
    fechamento/
      por-cliente/
        fechamento-por-cliente.md
        servicos-prestadores-parceiros.md
      por-prestador/
        fechamento-por-prestador.md
    lote/
      agrupamento-lote.md
    relatorios/
      contratos-por-cliente/
        contratos-por-cliente.md
      conectividade-por-prestador/
        conectividade-por-prestador.md
      atendimentos/
        relatorio-atendimentos.md
  flows/
    atendimento/
      fluxo-atendimento-completo.md
      atendimento-lifecycle.md
    acionamento/
      fluxo-despacho-prestador.md
      execucao-atendimento.md
    reembolso/
      fluxo-reembolso-completo.md
      status-personalizados-reembolso.md
      perguntas-reembolso.md
      tags-reembolso.md
    autenticacao/
      fluxo-login.md
    beneficiario/
      fluxo-busca-prestador.md
    creditos/
      fluxo-calculo-credito.md
      fluxo-credito-para-pagamento.md
    fechamento/
      por-cliente/
        fechamento-por-cliente.md
      por-prestador/
        fechamento-por-prestador.md
    faturamento/
      faturamento-lote.md
    relatorios/
      contratos-por-cliente/
        fluxo-relatorio-contratos.md
      conectividade-por-prestador/
        conectividade-por-prestador.md
      atendimentos/
        fluxo-relatorio-atendimentos.md
  selectors/
    autenticacao/
      login.json
    creditos/
      creditos.json
    reembolso/
      status-personalizados-reembolso.json
    relatorios/
      contratos-por-cliente.json
      conectividade-por-prestador.json
      atendimentos.json
  integrations/
    reembolso/
      api-reembolsos.md
```
