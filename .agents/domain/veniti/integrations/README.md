# Integrations

Documentação sobre sistemas externos e dependências de integração. Os agentes de QA usam este conhecimento para identificar dependências nos fluxos de teste e documentar bugs de integração com contexto adequado.

---

## Como usar

Crie um arquivo `.md` por integração dentro da pasta do módulo correspondente. Agentes que testam fluxos com dependências externas consultam esta pasta antes de criar cenários.

---

## Estrutura

```
/knowledge/system/integrations/
  reembolso/
    api-reembolsos.md
```

---

## Template de arquivo

```markdown
# Integração — [Nome do Sistema]

## Propósito
O que esta integração faz no contexto do sistema testado.

## Disponibilidade nos Ambientes
| Ambiente | Disponível | Observação |
|----------|-----------|------------|
| Local/Dev | Não | Usar mock configurado em /config |
| Staging | Sim | Ambiente de sandbox |
| Produção | Sim | Dados reais |

## Comportamento em Cenários de Teste

| Cenário | Comportamento Esperado | Impacto no Teste |
|---------|----------------------|-----------------|
| Integração disponível | Resposta normal | Testar fluxo completo |
| Integração indisponível | Mensagem de erro amigável | Cenário negativo |
| Timeout de resposta | Feedback de carregamento | Cenário de timing |
| Dados inválidos enviados | Resposta de erro específica | Cenário negativo |

## Notas para Testes
- Considerar: este fluxo funciona sem a integração?
- Fluxos que dependem desta integração devem ser marcados se falharem por indisponibilidade
```

---

## Regra para QA

Quando um teste falhar por indisponibilidade de integração externa:
- **Não classifique como bug da aplicação**
- Verifique se o sistema exibe mensagem de erro adequada (esse SIM pode ser um bug)
- Documente o contexto: "Falha por indisponibilidade de [sistema externo] no ambiente [X]"
