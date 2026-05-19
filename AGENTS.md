# Agent Rules

@.agents/rules/php/RULES.md

---

## Domain Knowledge — Veniti (load on demand)

Índice mestre: `.agents/domain/veniti/00-index.md`

| Módulo | Features | Flows |
|--------|----------|-------|
| Atendimento | `features/atendimento/` | `flows/atendimento/` |
| Acionamento | `features/acionamento/` | `flows/acionamento/` |
| Reembolso | `features/reembolso/` | `flows/reembolso/` |
| Autenticação | `features/autenticacao/` | `flows/autenticacao/` |
| Beneficiário | `features/beneficiario/` | `flows/beneficiario/` |
| Créditos | `features/creditos/` | `flows/creditos/` |
| Evento | `features/evento/` | — |
| Prestador | `features/prestador/` | — |
| Fechamento | `features/fechamento/` | `flows/fechamento/` |
| Lote / Faturamento | `features/lote/` | `flows/faturamento/` |
| Relatórios | `features/relatorios/` | `flows/relatorios/` |
| Financeiro | `features/financeiro/` | `flows/financeiro/` |

> Todos os caminhos acima são relativos a `.agents/domain/veniti/`.
> Para carregar um módulo: "leia `.agents/domain/veniti/features/reembolso/` antes de implementar".