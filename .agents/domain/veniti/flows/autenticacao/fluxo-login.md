# Fluxo de Login

## Description

Descreve o processo de autenticação em qualquer um dos três portais do sistema (Assistência, Cliente, Prestador). O fluxo é idêntico em estrutura mas executado em código independente por portal. O roteamento de portal é feito via subdomínio HTTP.

## Steps

### 1. Resolução de Portal por Subdomínio
- **Sistema**: `html/index.php` inspeciona `HTTP_HOST`
- **Lógica**: Redireciona para `/assistencia/login/`, `/cliente/login/` ou `/prestador/login/` conforme subdomínio
- **Branding**: Logo e CSS customizados carregados do AWS S3 para o tenant identificado

### 2. Verificação de Sessão Ativa
- **Sistema**: Verifica se existe sessão PHP válida com `AUTH_USER_ID_VENITI`
- **Se sim**: Redireciona imediatamente ao dashboard do portal (sem exibir formulário)
- **Se não**: Exibe formulário de login

### 3. Verificação de Auto-login (Opcional)
- **Sistema**: Verifica parâmetro `hash` na URL
- **Se válido**: Autentica automaticamente sem formulário
- **Se expirado**: Ignora e exibe formulário normal

### 4. Preenchimento do Formulário
- **Ator**: Usuário
- **Campos**: CNPJ/CPF da empresa (`login_codigo`), Nome de usuário (`login_usuario`), Senha (`login_senha`)
- **reCAPTCHA**: v3 invisível executado em background; v2 exibido se v3 falhar

### 5. Validação de Credenciais
- **Sistema**: `html/*/login/__acoes.php` processa `POST`
- **Verificações**:
  1. CNPJ/CPF válido e tenant ativo
  2. Usuário existe e está ativo no portal
  3. Senha correta (comparação bcrypt)
  4. reCAPTCHA score válido

### 6. Verificação de 2FA (Se habilitado)
- **Sistema**: Verifica se a conta tem 2FA ativo (Google Authenticator)
- **Se sim**: Exibe modal para inserção do código TOTP (6 dígitos)
- **Validação**: Token TOTP verificado com janela de 30 segundos

### 7. Verificação de Troca Obrigatória de Senha
- **Sistema**: Verifica `nova_senha_hash` e `nova_senha_tempo` na conta
- **Se obrigatória**: Redireciona para formulário de definição de nova senha antes do acesso

### 8. Criação de Sessão
- **Sistema**: Sessão PHP criada com dados do usuário:
  - `AUTH_USER_ID_VENITI` (ID do tenant)
  - ID do usuário, nome, perfil, permissões
- **Registro**: Último acesso atualizado no banco

### 9. Redirecionamento ao Dashboard
- **Sistema**: Redireciona para a página inicial do portal conforme perfil
  - Assistência → `/assistencia/operacao/eventos.php`
  - Cliente → `/cliente/operacao/eventos.php`
  - Prestador → `/prestador/dashboard/`

## Entry Points

- Acesso direto à URL do portal (sem sessão ativa)
- Link de auto-login recebido por e-mail
- Redirecionamento após logout

## Exit Points

- **Sucesso**: Dashboard do portal carregado
- **Falha de credencial**: Mensagem de erro, formulário reexibido
- **2FA inválido**: Erro exibido, usuário permanece na tela de código
- **Conta bloqueada**: Mensagem genérica, sem indicação de bloqueio específico
- **Tenant inativo**: Acesso negado com mensagem de erro

## Variations

### Recuperação de Senha
- Usuário clica em "Esqueci a senha" → [[recuperacao-senha]] iniciado
- Após redefinição: novo login necessário com a nova senha

### Portal do Prestador — Primeira Ativação
- Prestador recém-cadastrado acessa link de ativação de conta
- Modal `modal-activated-account.php` exibido
- Senha criada na primeira sessão

## Related Features

- [[login-multi-portal]]
- [[recuperacao-senha]]
