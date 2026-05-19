# Fluxos — Perguntas de Reembolso

## Fluxo 1: Cadastrar Configuração de Perguntas

**Tela inicial:** `Configurações → Financeiro → Perguntas de reembolso`
**Pré-condição:** Usuário autenticado com permissão "Cadastrar Perguntas de reembolso"

### Passos

1. Acessar `Configurações → Financeiro → Perguntas de reembolso`
2. Clicar na aba **Cadastrar**
3. Preencher os campos de contexto:
   - **Cliente** (select, opção "TODOS" disponível)
   - **Tipo de evento** (select, opção "TODOS" disponível)
   - **Tipo de serviço** (select, opção "TODOS" disponível)
   - **Motivo do reembolso** (select, opção "TODOS" disponível)
4. Clicar em **+ Cadastrar** (botão de confirmar contexto)
5. Sistema abre etapa de cadastro das perguntas
6. Configurar a pergunta:
   - **Tipo de Pergunta:** OPCIONAL (único tipo disponível — não há obrigatória)
   - **Pergunta:** texto da pergunta
   - **Descrição/Orientações:** texto de orientação (opcional)
   - Opções de resposta (Sim/Não ou outras)
   - Lógicas condicionais: apenas "Exibir pergunta" ou "Ocultar pergunta"
7. Clicar em **Salvar**

**Estado final esperado:** Pergunta cadastrada, aparece no datatable do módulo com as colunas Cliente, Tipo de evento, Tipo de serviço, Motivo do reembolso, Status.

---

## Fluxo 2: Responder Perguntas no Modal de Cadastro do Reembolso

**Tela inicial:** Atendimento → Editar → Ação → Reembolso
**Pré-condição:** Existe pergunta configurada que corresponde ao contexto do atendimento

### Passos

1. Dentro do atendimento, clicar em **Editar**
2. Ao final da página, clicar em **Ação → Reembolso**
3. Confirmar no modal de confirmação
4. No modal de cadastro, a seção **PERGUNTAS** é exibida com as perguntas correspondentes
5. Responder cada pergunta (opcional — todas são opcionais)
   - Se a pergunta possui Descrição/Orientação: ao responder, modal de orientação é exibido
6. Preencher demais campos do reembolso
7. Clicar em **Realizar Reembolso**

**Estado final esperado:** Reembolso criado; respostas registradas nas ocorrências do reembolso.

---

## Fluxo 3: Gerenciar Perguntas Após Criação do Reembolso

**Tela inicial:** Reembolso aberto
**Pré-condição:** Reembolso criado com perguntas configuradas

### Passos

1. Abrir o reembolso
2. Clicar na aba **Perguntas**
3. Visualizar perguntas configuradas para o contexto
4. Editar/responder perguntas conforme necessário
5. Clicar em **Salvar**

**Estado final esperado:** Respostas atualizadas; alterações registradas nas ocorrências.

---

## Padrões de UI

- Seção PERGUNTAS no modal de cadastro do reembolso identificada por ícone de lista e label "PERGUNTAS"
- Ícone ↔ no canto direito da seção (colapsar/expandir)
- Perguntas do tipo radio button (Sim/Não ou múltipla escolha)
- Modal de orientação segue mesmo padrão do módulo de perguntas do atendimento
- Aba "Perguntas" visível apenas se existirem perguntas configuradas para o contexto

---

## Regras de Negócio Relevantes

- [[RN-RBL-014]] — Perguntas sempre opcionais
- [[RN-RBL-015]] — Unicidade de configuração
- [[RN-RBL-016]] — Registro automático nas ocorrências
- Sistema exibe a pergunta mais específica para o contexto (ex: configuração para cliente específico tem precedência sobre "TODOS")
