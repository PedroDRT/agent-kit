# Link do Beneficiário no Reembolso

## Resumo

O link do beneficiário funciona como o canal de interação entre o sistema e o beneficiário no fluxo de reembolso. O portal self-service (`/reembolso/`) foi redesenhado e passou a suportar sincronização bidirecional dos dados do recebedor entre Assistência e beneficiário. A Assistência pode pré-preencher os dados do recebedor no momento da criação; o link exibe esses dados já carregados. Alterações feitas pelo beneficiário (até o status `NOTA_RECEBIDA`) são refletidas na Assistência.

O link do beneficiário também concentra o fluxo de preenchimento dos dados do reembolso e o envio de arquivos — o link do reembolso separado foi descontinuado e seus campos foram migrados para o link do beneficiário.

---

## Comportamento do Link

Ao realizar um reembolso, o link do beneficiário exibe:
1. Formulário com os campos de preenchimento do reembolso (dados financeiros, documento, etc.)
2. Após o envio: possibilidade de **anexar arquivos** (múltiplos arquivos simultaneamente)
3. Botão **"Atualizar informações"**: sistema exibe o formulário novamente com os dados anteriores preenchidos e editáveis, permitindo também enviar novos arquivos

---

## Campos do Formulário

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

**Após envio:** exibe resumo com protocolo, data, motivo, tipo de serviço, valor solicitado e valor aprovado + botão "Atualizar informações"

---

## Critérios de Aceitação

1. O portal `/reembolso/` exibe o design reformulado
2. Se a Assistência preencher as informações do recebedor no cadastro, esses dados aparecem pré-preenchidos no formulário do link
3. Se a Assistência não preencher, o formulário é exibido vazio para o beneficiário
4. O beneficiário pode alterar os dados do recebedor até o status `NOTA_RECEBIDA` (inclusive); após esse status, o link exibe página de detalhes (forma de pagamento, dados bancários, timeline) — sem formulário de edição
5. Ao finalizar o envio, as alterações do beneficiário são refletidas na visão da Assistência
6. O campo "Tipo PIX" inclui a opção "CNPJ"
7. O fluxo no link é dividido em etapas (wizard); há uma página de confirmação antes do envio final
8. A notificação de sucesso é exibida ao beneficiário após o envio final
9. Campo "Chave PIX" aplica máscara e validação por tipo:
   - **Telefone**: aceita apenas dígitos, sem letras
   - **E-mail**: exige formato com domínio (ex: `@dominio.com`)
   - **CPF**: exige 11 dígitos (com máscara `XXX.XXX.XXX-XX`)
   - **CNPJ**: exige 14 dígitos (com máscara `XX.XXX.XXX/XXXX-XX`)
10. Validações de máscara impedem o avanço para a próxima etapa
11. Validações de máscara da chave PIX existem apenas no portal `/reembolso/`; o Veniti Web não as possui
12. Beneficiário pode anexar múltiplos arquivos simultaneamente
13. Ao clicar em "Atualizar informações", o formulário é exibido com os dados já preenchidos e editáveis

---

## Escopo de Teste

- Portal `/reembolso/` (link do beneficiário): pré-preenchimento, edição, validações de máscara, fluxo de etapas, confirmação, notificação de sucesso, envio de múltiplos arquivos, atualizar informações
- Portal `/assistencia/reembolso/`: pré-preenchimento dos dados do recebedor no cadastro, verificação da sincronização após alteração pelo beneficiário
- Campo Tipo PIX: opção CNPJ disponível e chave validada com máscara correta
- Cenários de status: edição permitida em `NOTA_PENDENTE` e `NOTA_RECEBIDA`; edição bloqueada em `AGENDADO`, `APROVADO`, `RECUSADO`, `PAGAMENTO_GERADO`, `PAGO`

---

## Fora do Escopo

- Validações de máscara da chave PIX no Veniti Web
- Fluxo financeiro posterior ao envio dos dados (aprovação, pagamento) — coberto por [[processamento-pagamento-reembolso]]

---

## Dependências e Premissas

- **Regras de negócio:** `knowledge/business-rules/reembolso.md`
  - RN-RBL-003 — Edição pelo beneficiário restrita até `NOTA_RECEBIDA`
  - RN-RBL-005 — Links de acesso expiram
  - RN-RBL-007 — Pipeline de status sequencial
- **Features relacionadas:** [[solicitacao-reembolso]], [[processamento-pagamento-reembolso]]
- **Flow relacionado:** [[fluxo-reembolso-completo]]
- **Pré-condição de dados:** É necessário um reembolso em status `NOTA_PENDENTE` para testar o link com dados pré-preenchidos e outro sem dados pré-preenchidos
- **Portal:** `/reembolso/` acesso via token em URL (sem autenticação por login); `/assistencia/reembolso/` acesso via credenciais de operador

---

## Pontos de Atenção

- **Validações de máscara:** Aplicadas apenas ao portal `/reembolso/`; Veniti Web está fora do escopo
- **Fluxo de etapas (wizard):** O link possui 3 etapas seguidas de uma página de confirmação final. As validações bloqueiam o avanço de etapa
- **Sincronização bidirecional:** A atualização dos dados do recebedor na Assistência ocorre ao finalizar o envio no link — não em tempo real durante o preenchimento
- **Máscara CNPJ:** Chave PIX do tipo CNPJ exibe formatação com pontuação (`XX.XXX.XXX/XXXX-XX`); validação exige 14 dígitos

---

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

## QA Notes

- **Validação CPF confirmada (formato apenas):** A máscara valida apenas formato (11 dígitos, `XXX.XXX.XXX-XX`); CPF com dígitos verificadores matematicamente inválidos é aceito
- **Status read-only confirmado:** Acesso ao link em `AGENDADO`, `APROVADO`, `RECUSADO`, `PAGAMENTO_GERADO` e `PAGO` exibe página de detalhes com timeline — sem formulário de edição. Não há redirecionamento
- **Sincronização bidirecional confirmada:** Sincronização ocorre no momento do submit final pelo beneficiário, não durante o preenchimento
- **Responsividade confirmada:** Layout funcional em 375px sem quebras visuais
- **Ambiguidade aberta:** Comportamento ao acessar link com token completamente inválido (não apenas expirado) — documentar diferença entre "expirado" e "inválido" se necessário
