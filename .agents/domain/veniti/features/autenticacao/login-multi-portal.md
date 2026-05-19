---
type: feature
module: autenticacao
layer: feature
related:
  - fluxo-login
  - recuperacao-senha
  - cadastro-beneficiario
---

# Login Multi-Portal

> O sistema Veniti opera com três portais distintos, cada um com seu próprio processo de autenticação e contexto de sessão.

## Descrição

O sistema Veniti opera com três portais distintos, cada um com seu próprio processo de autenticação e contexto de sessão. O roteamento entre portais é feito por subdomínio (tenant routing). Cada portal possui identidade visual própria carregada dinamicamente do tenant.

---

## Entradas

### Portais e URLs

| Portal | Subdomínio | Path | Usuário alvo |
|---|---|---|---|
| **Assistência** | `assistencia.{tenant}.veniti.com.br` | `/assistencia/login/` | Operadores internos, admins |
| **Cliente** | `{tenant}.veniti.com.br` | `/cliente/login/` | Empresas clientes (seguradoras) |
| **Prestador** | `prestador.{tenant}.veniti.com.br` | `/prestador/login/` | Prestadores de serviço |

### Login Principal

| Campo | Identificador HTML | Portal |
|---|---|---|
| CNPJ/CPF da empresa | `login_codigo` | Todos |
| Nome de usuário | `login_usuario` | Todos |
| Senha | `login_senha` | Todos |

### Autenticação 2FA (Google Authenticator)

| Campo | Descrição |
|---|---|
| Token TOTP | Código de 6 dígitos gerado pelo app |

### Auto-login

| Parâmetro | Descrição |
|---|---|
| `hash` URL | Hash de sessão para login automático (gerado internamente) |

## Saídas

- Sessão PHP criada com variáveis:
  - `AUTH_USER_ID_VENITI` — ID do tenant (empresa)
  - Dados do usuário autenticado (ID, nome, perfil, permissões)
- Redirecionamento para dashboard do portal correspondente
- Logo e CSS customizados carregados do AWS S3 para o tenant
- Registro de último acesso no banco

---

## Regras de Negócio

- O CNPJ/CPF identifica o tenant — deve estar cadastrado como cliente ativo
- O subdomínio do HTTP_HOST determina qual portal é exibido
- Senhas são verificadas com hash seguro (bcrypt)
- 2FA com Google Authenticator é obrigatório para contas com esse fator habilitado
- Usuários com senha temporária são forçados a trocar antes de prosseguir
- `nova_senha_hash` e `nova_senha_tempo` controlam o fluxo de redefinição obrigatória
- Auto-login via hash é válido por tempo limitado — expirado, redireciona para login normal
- reCAPTCHA v3 (invisível) e v2 (visível como fallback) estão integrados
- Sessões ativas são verificadas no carregamento — usuário já logado é redirecionado direto
- Cada portal tem arquivos de login independentes (`index.php`, `index2.php`, `index3.php`) para variações de layout
- Contas inativas ou bloqueadas são rejeitadas com mensagem genérica

## Casos de Borda

- Usuário tenta acessar portal incorreto para seu perfil
- Tenant com subdomínio não configurado (host não reconhecido)
- Hash de auto-login reutilizado após uso único
- Token 2FA inserido com atraso (janela de 30s expirada)
- Usuário tenta login em paralelo em múltiplas abas (gestão de sessão)
- CNPJ válido mas empresa sem usuários ativos cadastrados
- reCAPTCHA falha em ambiente de teste (token inválido)
- Sessão corrompida após timeout do servidor

---

## Dependências

- **Auth handler**: `html/assistencia/login/__acoes.php`, `html/cliente/login/__acoes.php`, `html/prestador/login/__acoes.php`
- **Sessão**: PHP sessions + Redis (caching/sessões)
- **2FA**: Google Authenticator TOTP integration
- **reCAPTCHA**: Google reCAPTCHA v2/v3
- **Storage**: AWS S3 (logos e CSS customizados por tenant)
- **Banco**: tabelas de usuários por portal (`clientes_usuarios`, `prestadores_usuarios`, etc.)

## Flows Relacionados

- [[fluxo-login]]

## Features Relacionadas

- [[recuperacao-senha]]
- [[cadastro-beneficiario]]
