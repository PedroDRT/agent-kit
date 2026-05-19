---
type: feature
module: autenticacao
layer: feature
related:
  - fluxo-login
  - login-multi-portal
---

# Recuperação de Senha

> Permite que usuários dos portais Assistência, Cliente e Prestador recuperem acesso à conta quando esquecem a senha ou o nome de usuário.

## Descrição

Permite que usuários dos portais Assistência, Cliente e Prestador recuperem acesso à conta quando esquecem a senha ou o nome de usuário. O fluxo é inteiramente por e-mail, com link de redefinição com prazo de validade.

---

## Entradas

### Esqueci a Senha

| Campo | Descrição |
|---|---|
| `login_codigo` | CNPJ/CPF da empresa |
| `login_email` | E-mail cadastrado na conta |

### Esqueci o Usuário

| Campo | Descrição |
|---|---|
| `login_codigo` | CNPJ/CPF da empresa |
| `login_email` | E-mail cadastrado na conta |

### Redefinição de Senha

| Campo | Descrição |
|---|---|
| `nova_senha` | Nova senha desejada |
| `nova_senha_confirmacao` | Confirmação da nova senha |
| `nova_senha_hash` | Token de validação (URL param) |

## Saídas

- E-mail enviado com link de recuperação (PHPMailer)
- Link contém `nova_senha_hash` único e `nova_senha_tempo` (expiração)
- Ao acessar o link válido: formulário de nova senha exibido
- Senha atualizada no banco com novo hash bcrypt
- Hash invalidado após uso
- Para recuperação de usuário: e-mail com o nome de usuário cadastrado

---

## Regras de Negócio

- O e-mail deve corresponder exatamente ao cadastrado para o CNPJ/CPF informado
- O link de recuperação tem prazo de validade configurado (`nova_senha_tempo`)
- Links expirados redirecionam para o formulário de login com mensagem de erro
- O hash de recuperação é de uso único — nova solicitação gera novo hash e invalida o anterior
- A nova senha deve atender aos critérios mínimos de segurança do sistema
- O processo de "esqueci usuário" envia o nome de usuário por e-mail (não a senha)
- Todos os portais (Assistência, Cliente, Prestador) têm fluxo equivalente, mas fluxos independentes

## Casos de Borda

- E-mail não cadastrado para o CNPJ informado
- Usuário solicita recuperação múltiplas vezes em sequência (múltiplos links ativos)
- Link de recuperação acessado por outro dispositivo/browser
- E-mail de recuperação vai para spam
- Conta bloqueada — recuperação de senha não reativa a conta

---

## Dependências

- **E-mail**: PHPMailer (`html/__inc/PHPMailer/`)
- **Banco**: tabelas de usuários de cada portal
- **Auth actions**: `html/*/login/__acoes.php` de cada portal

## Flows Relacionados

- [[fluxo-login]]

## Features Relacionadas

- [[login-multi-portal]]
