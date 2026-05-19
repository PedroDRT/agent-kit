---
type: feature
module: reembolso
layer: feature
related:
  - solicitacao-reembolso
  - processamento-pagamento-reembolso
  - fluxo-reembolso-completo
---

# Link do Beneficiário no Reembolso

> O link do beneficiário é o canal de interação entre o sistema e o beneficiário no fluxo de reembolso, com sincronização bidirecional dos dados do recebedor.

## Descrição

O link do beneficiário funciona como o canal de interação entre o sistema e o beneficiário no fluxo de reembolso. O portal self-service (`/reembolso/`) foi redesenhado e passou a suportar sincronização bidirecional dos dados do recebedor entre Assistência e beneficiário. A Assistência pode pré-preencher os dados do recebedor no momento da criação; o link exibe esses dados já carregados. Alterações feitas pelo beneficiário (até o status `NOTA_RECEBIDA`) são refletidas na Assistência.

O link do beneficiário também concentra o fluxo de preenchimento dos dados do reembolso e o envio de arquivos — o link do reembolso separado foi descontinuado e seus campos foram migrados para o link do beneficiário.

---

## Entradas

### Campos do Formulário

**Etapa de preenchimento:**
- CPF/CNPJ
- Nome
- E-mail
- Valor (R$)
- Forma de Pagamento (PIX, TED, DOC)
- Tipo PIX (CPF, CNPJ, email, telefone, aleatória)
- Chave PIX
- Número do Documento
- CPF/CNPJ emissor

## Saídas

- Dados do recebedor salvos no reembolso vinculado
- Arquivos enviados armazenados no AWS S3
- Sincronização dos dados na Assistência após submit final pelo beneficiário
- Exibição após envio: resumo com protocolo, data, motivo, tipo de serviço, valor solicitado e valor aprovado

---

## Regras de Negócio

- Se a Assistência preencher as informações do recebedor no cadastro, esses dados aparecem pré-preenchidos no formulário do link
- Se a Assistência não preencher, o formulário é exibido vazio para o beneficiário
- O beneficiário pode alterar os dados do recebedor até o status `NOTA_RECEBIDA` (inclusive)
- Após `NOTA_RECEBIDA`, o link exibe página de detalhes (forma de pagamento, dados bancários, timeline) — sem formulário de edição
- Ao finalizar o envio, as alterações do beneficiário são refletidas na visão da Assistência
- O campo "Tipo PIX" inclui a opção "CNPJ"
- O fluxo no link é dividido em etapas (wizard); há uma página de confirmação antes do envio final
- A notificação de sucesso é exibida ao beneficiário após o envio final
- Campo "Chave PIX" aplica máscara e validação por tipo:
  - **Telefone**: aceita apenas dígitos, sem letras
  - **E-mail**: exige formato com domínio (ex: `@dominio.com`)
  - **CPF**: exige 11 dígitos (com máscara `XXX.XXX.XXX-XX`)
  - **CNPJ**: exige 14 dígitos (com máscara `XX.XXX.XXX/XXXX-XX`)
- Validações de máscara impedem o avanço para a próxima etapa
- Validações de máscara da chave PIX existem apenas no portal `/reembolso/`; o Veniti Web não as possui
- Beneficiário pode anexar múltiplos arquivos simultaneamente
- Ao clicar em "Atualizar informações", o formulário é exibido com os dados já preenchidos e editáveis
- Sincronização bidirecional ocorre no momento do submit final pelo beneficiário, não em tempo real durante o preenchimento

## Cenários de Status

- Edição permitida em `NOTA_PENDENTE` e `NOTA_RECEBIDA`
- Edição bloqueada em `AGENDADO`, `APROVADO`, `RECUSADO`, `PAGAMENTO_GERADO`, `PAGO`

## Comportamentos Confirmados

| Comportamento | Confirmado | Observação |
|---|---|---|
| CPF com dígitos verificadores inválidos é aceito | ✅ Sim | Validação apenas de formato (11 dígitos + máscara) — validação matemática não existe no portal `/reembolso/` |
| Status AGENDADO/superior → exibe detalhes read-only | ✅ Sim | Página de detalhes (forma de pagamento, dados bancários, timeline) — sem formulário, sem redirecionamento |
| Sincronização bidirecional ao finalizar envio | ✅ Sim | Atualização na Assistência ocorre no submit final, não em tempo real |
| Dados parcialmente pré-preenchidos exibidos corretamente | ✅ Sim | Campos não preenchidos pela Assistência aparecem vazios no formulário do beneficiário |
| Responsividade em 375px (mobile) | ✅ Sim | Layout sem quebras visuais |
| Wizard: validações bloqueiam avanço de etapa | ✅ Sim | Submit não é realizado com chave PIX inválida — bloqueio ocorre na etapa, não apenas no envio final |

---

## Escopo de Teste

- Portal `/reembolso/` (link do beneficiário): pré-preenchimento, edição, validações de máscara, fluxo de etapas, confirmação, notificação de sucesso, envio de múltiplos arquivos, atualizar informações
- Portal `/assistencia/reembolso/`: pré-preenchimento dos dados do recebedor no cadastro, verificação da sincronização após alteração pelo beneficiário
- Campo Tipo PIX: opção CNPJ disponível e chave validada com máscara correta

## Fora do Escopo

- Validações de máscara da chave PIX no Veniti Web
- Fluxo financeiro posterior ao envio dos dados (aprovação, pagamento) — coberto por [[processamento-pagamento-reembolso]]

---

## Notas de QA

- **Validação CPF confirmada (formato apenas):** A máscara valida apenas formato (11 dígitos, `XXX.XXX.XXX-XX`); CPF com dígitos verificadores matematicamente inválidos é aceito
- **Status read-only confirmado:** Acesso ao link em `AGENDADO`, `APROVADO`, `RECUSADO`, `PAGAMENTO_GERADO` e `PAGO` exibe página de detalhes com timeline — sem formulário de edição. Não há redirecionamento
- **Sincronização bidirecional confirmada:** Sincronização ocorre no momento do submit final pelo beneficiário, não durante o preenchimento
- **Responsividade confirmada:** Layout funcional em 375px sem quebras visuais
- **Ambiguidade aberta:** Comportamento ao acessar link com token completamente inválido (não apenas expirado) — documentar diferença entre "expirado" e "inválido" se necessário

## Dependências

- **Regras de negócio:**
  - RN-RBL-003 — Edição pelo beneficiário restrita até `NOTA_RECEBIDA`
  - RN-RBL-005 — Links de acesso expiram
  - RN-RBL-007 — Pipeline de status sequencial
- **Portal:** `/reembolso/` acesso via token em URL (sem autenticação por login); `/assistencia/reembolso/` acesso via credenciais de operador

## Features Relacionadas

- [[solicitacao-reembolso]]
- [[processamento-pagamento-reembolso]]
