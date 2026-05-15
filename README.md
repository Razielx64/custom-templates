# nuclei-custom-templates

```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║    ███╗   ██╗██╗   ██╗ ██████╗██╗     ███████╗██╗                 ║
║    ████╗  ██║██║   ██║██╔════╝██║     ██╔════╝██║                 ║
║    ██╔██╗ ██║██║   ██║██║     ██║     █████╗  ██║                 ║
║    ██║╚██╗██║██║   ██║██║     ██║     ██╔══╝  ██║                 ║
║    ██║ ╚████║╚██████╔╝╚██████╗███████╗███████╗██║                 ║
║    ╚═╝  ╚═══╝ ╚═════╝  ╚═════╝╚══════╝╚══════╝╚═╝                 ║
║                                                                   ║
║          custom templates  ·  by Razielx64                        ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

> Templates customizados para o [Nuclei](https://github.com/projectdiscovery/nuclei) focados em detectar endpoints de gerenciamento expostos, informações sensíveis vazadas, injeções e misconfigurations em aplicações web modernas.

---

## Como funciona

```
                         ┌─────────────────────────────────┐
                         │         nuclei engine            │
                         └────────────────┬────────────────┘
                                          │  carrega templates
                    ┌─────────────────────▼──────────────────────┐
                    │                                             │
          ┌─────────▼──────┐                          ┌──────────▼──────┐
          │  matcher logic  │                          │  extractor logic │
          │                 │                          │                  │
          │  ✔ status: 200  │                          │  regex → versão  │
          │  ✔ body: words  │                          │  regex → creds   │
          │  ✗ negative     │                          │  regex → paths   │
          └─────────┬───────┘                          └──────────┬───────┘
                    │                                             │
                    └──────────────────┬──────────────────────────┘
                                       │
                           ┌───────────▼───────────┐
                           │       HTTP request      │
                           │   GET {{BaseURL}}/path  │
                           └───────────┬───────────┘
                                       │
              ┌────────────────────────▼────────────────────────┐
              │                   Target                         │
              │                                                  │
              │  PHP · Node.js · Python · Ruby · API · Swagger   │
              └──────────────────────────────────────────────────┘
```

---

## Templates

### Endpoints por Stack

#### `php_endpoints.yaml`
Detecta endpoints sensíveis expostos em aplicações PHP.

| Cobertura | O que detecta |
|-----------|---------------|
| `phpinfo.php` | Versão do PHP, paths internos, IP do servidor |
| `.env` / `.env.*` | `APP_KEY`, `DB_PASSWORD`, `AWS_SECRET`, `JWT_SECRET` |
| Laravel Telescope | Requests, queries SQL, emails sem autenticação |
| Laravel Horizon | Filas, job payloads, conexões Redis |
| PHP DebugBar | Queries SQL, bindings, paths internos |
| Adminer / phpMyAdmin | Versões expostas de gestores de banco |
| WordPress | Usuários via REST API, plugins, versão do WP, xmlrpc |
| Symfony Profiler | Tokens de sessão, DATABASE_URL, versão do Symfony |
| Xdebug | remote_host, remote_port, idekey expostos |
| `composer.json/lock` | Dependências, versão do Laravel, versão do PHP |
| Backups SQL / ZIP | Dumps de banco, password hashes |
| `laravel.log` | Stack traces de produção, senhas em logs |

---

#### `python_endpoints.yaml`
Detecta misconfigurations em aplicações Python.

| Cobertura | O que detecta |
|-----------|---------------|
| Werkzeug Debugger | Console interativo exposto, PIN do debugger |
| Django Debug Toolbar | Painel `__debug__` acessível |
| Django Admin | `/admin/` sem proteção adequada |
| FastAPI / OpenAPI | `/docs`, `/redoc`, `/openapi.json` |
| Flask Debug Mode | Traceback exposto publicamente |
| `.env` Python | `SECRET_KEY`, `DATABASE_URL`, `DJANGO_SECRET_KEY` |
| `requirements.txt` | Dependências expostas do projeto |

---

#### `nodejs_endpoints.yaml`
Detecta endpoints de debug e monitoramento em aplicações Node.js.

| Cobertura | O que detecta |
|-----------|---------------|
| `/metrics` | Prometheus/prom-client exposto |
| `/_health`, `/healthz` | Status interno, uptime, versões |
| Node.js Inspector | Porta de debug `9229` / `9230` |
| PM2 API | `/api/pm2` com info de processos |
| `.env` Node | `DATABASE_URL`, `JWT_SECRET`, `NODE_ENV` |
| `package.json` | Dependências e scripts expostos |

---

#### `ruby_endpoints.yaml`
Detecta endpoints de gerenciamento expostos em aplicações Ruby on Rails.

| Cobertura | O que detecta |
|-----------|---------------|
| `/rails/info/properties` | Versão do Rails, Ruby, ambiente |
| `/rails/info/routes` | Todas as rotas da aplicação |
| Rails Mailer Preview | Templates de e-mail via `/rails/mailers` |
| Sidekiq Web UI | Dashboard de filas sem autenticação |
| `Gemfile.lock` | Dependências expostas |
| `database.yml` | Configuração do banco de dados |

---

#### `api_endpoints.yaml`
Detecta documentação de API e endpoints sensíveis expostos.

| Cobertura | O que detecta |
|-----------|---------------|
| Swagger UI | `/swagger`, `/swagger-ui`, `/api-docs` |
| OpenAPI | `openapi.json`, `openapi.yaml`, `/api/schema` |
| GraphQL | `/graphql`, `/graphiql`, `/playground`, `/altair` |
| ReDoc | `/redoc`, `/api/redoc` |
| Admin routes | `/admin`, `/api/admin`, `/dashboard` |
| Versioned APIs | `/v1/` → `/v10/` com paths comuns |

---

#### `swagger-paths.yaml`
Template dedicado à detecção de Swagger UI com lista abrangente de paths alternativos.

| Cobertura | O que detecta |
|-----------|---------------|
| Swagger UI | 50+ paths alternativos conhecidos |
| Versões legadas | `/v0.1/` → `/v0.12/index.html` |
| Paths por framework | Spring Boot, .NET, Django REST, FastAPI |

---

### SQL Injection

#### `error-based-sql-injection.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta SQL injection baseado em erros para **29 engines de banco de dados** diferentes. Inspirado no banco de erros do sqlmap.

| Cobertura | O que detecta |
|-----------|---------------|
| MySQL / MariaDB / Drizzle / MemSQL | Mensagens de erro de sintaxe e exceções de driver |
| PostgreSQL / CockroachDB | `PG::SyntaxError`, `PSQLException`, erros de parser |
| Microsoft SQL Server / Access | `OLE DB`, `ODBC`, `SqlClient`, `JET Database Engine` |
| Oracle | `ORA-XXXXX`, `quoted string not properly terminated` |
| IBM DB2 / Informix / Sybase / Ingres | Exceções e mensagens de driver específicas |
| SQLite / H2 / HSQLDB / MonetDB | Erros de runtime e JDBC |
| Firebird / Vertica / Presto / CrateDB | Fingerprints por stack trace e JDBC |

---

#### `header-blind-sql-injection.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Blind SQL injection baseado em tempo via headers HTTP. Envia uma requisição de baseline primeiro, depois injeta payloads `sleep(5)` nos headers tipicamente logados ou processados em queries SQL.

| Cobertura | O que detecta |
|-----------|---------------|
| `User-Agent` / `Referer` | Injeção em headers comumente logados no banco |
| `X-Forwarded-For` / `X-Client-IP` | Injeção em headers de controle de acesso por IP |
| `X-Remote-IP` / `X-Originating-IP` | Headers de proxy frequentemente inseridos em queries |
| Comparação de tempo | Reduz falsos positivos comparando com o baseline |

---

#### `sqlInjection.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecção genérica de SQL injection via parâmetros de URL.

---

### SSRF

#### `header-blind-ssrf.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta SSRF cego via headers HTTP populares usando Interactsh (OOB). Faz fuzzing de headers como `X-Forwarded-For`, `X-Real-IP`, `Host`, entre outros.

---

#### `ssrf-by-proxy.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta SSRF via parâmetros de proxy em endpoints que aceitam URLs externas como input (`url=`, `proxy=`, `redirect=`, etc.).

---

#### `xmlrpc-pingback-ssrf.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta SSRF via `pingback.ping` no XML-RPC do WordPress/Liferay. Usa Interactsh (OOB) para confirmar a requisição de saída.

| Cobertura | O que detecta |
|-----------|---------------|
| `/xmlrpc/pingback` | Endpoint de pingback exposto |
| DNS / HTTP OOB | Confirmação via Interactsh |

---

### Injeção de Headers

#### `header-command-injection.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Faz fuzzing de headers HTTP para command injection. Testa headers frequentemente passados para comandos de sistema no backend.

---

#### `smtp-header-injection-relay-oob.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta SMTP header injection com confirmação OOB via relay de e-mail. Útil para detectar formulários de contato vulneráveis que injetam headers em chamadas SMTP.

---

### LFI (Local File Inclusion)

#### `lfi.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecção de LFI via path traversal em parâmetros comuns (`file=`, `path=`, `page=`, etc.).

---

#### `wordpressLFI.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
LFI específico para WordPress via plugins e temas vulneráveis.

---

### Autenticação / JWT

#### `jwt-algorithm-confusion.yaml` ![severity: critical](https://img.shields.io/badge/severity-critical-red)
Detecta implementações JWT vulneráveis a ataques de confusão de algoritmo.

| Cobertura | O que detecta |
|-----------|---------------|
| `alg:none` bypass | JWT aceito sem assinatura |
| RSA → HMAC downgrade | Token HS256 assinado com chave pública RSA |
| Endpoints testados | `/api/user`, `/api/profile`, `/api/admin`, `/api/me` |
| Extractor | Extrai `role`, `username`, `email` do body da resposta |

---

#### `fuzz-oauth.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Faz fuzzing em flows OAuth para identificar endpoints de autorização, tokens e misconfigurations.

---

### Secrets / Credenciais

#### `js_secrets.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta secrets, API keys, tokens e credenciais vazadas em arquivos JavaScript.

| Cobertura | O que detecta |
|-----------|---------------|
| API Keys | AWS, Google, Stripe, Twilio, SendGrid, etc. |
| Tokens | JWT, OAuth, Bearer tokens hardcoded |
| Credenciais | Senhas, connection strings em JS client-side |

---

#### `gmail-api-client-secrets.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta arquivos `client_secrets.json` da API do Gmail expostos publicamente.

---

#### `npmrc.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta arquivos `.npmrc` expostos que podem conter tokens de autenticação do npm registry.

---

#### `s3cfg.yaml` ![severity: high](https://img.shields.io/badge/severity-high-orange)
Detecta arquivos `.s3cfg` expostos contendo credenciais de autenticação do Amazon S3 (`access_key`, `secret_key`).

---

### Banco de Dados / Schema

#### `db-schema.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta arquivos `schema.rb` do ActiveRecord expostos publicamente, revelando a estrutura completa do banco de dados da aplicação.

| Cobertura | O que detecta |
|-----------|---------------|
| `/db/schema.rb` | Schema do banco Rails exposto |
| `/database/schema.rb` | Caminho alternativo do schema |
| Extractor | Extrai a versão do schema (`version: YYYYMMDDHHMMSS`) |

---

### Logs Expostos

#### `development-logs.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta arquivos `development.log` do Rails expostos, que podem conter queries SQL, stack traces e dados sensíveis de desenvolvimento.

---

#### `prod-logs.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta arquivos de log de produção expostos com potencial para vazar informações de erros, stack traces e dados de usuários.

---

### Information Disclosure

#### `public-documents.yaml` ![severity: info](https://img.shields.io/badge/severity-info-blue)
Detecta páginas que contêm links para documentos Excel, Word ou CSV públicos, indicando possível exposição de dados corporativos.

---

## Uso

```bash
# rodar um template específico
nuclei -u https://target.com -t php_endpoints.yaml

# rodar todos os templates da pasta
nuclei -u https://target.com -t .

# rodar com lista de alvos
nuclei -l targets.txt -t .

# salvar output em JSON
nuclei -u https://target.com -t . -json -o resultados.json

# rodar apenas templates com severidade media ou alta
nuclei -u https://target.com -t . -severity medium,high

# rodar apenas templates de SQL injection
nuclei -u https://target.com -t . -tags sqli,blind-sqli

# rodar apenas templates críticos e altos
nuclei -u https://target.com -t . -severity critical,high
```

---

## Estrutura

```
custom/
├── README.md
│
├── # Endpoints por Stack
├── php_endpoints.yaml               # PHP · Laravel · Symfony · WordPress
├── python_endpoints.yaml            # Django · Flask · FastAPI · Werkzeug
├── nodejs_endpoints.yaml            # Express · Fastify · NestJS
├── ruby_endpoints.yaml              # Ruby on Rails · Sidekiq
├── api_endpoints.yaml               # REST · GraphQL · OpenAPI
├── swagger-paths.yaml               # Swagger UI (paths extendidos)
│
├── # SQL Injection
├── error-based-sql-injection.yaml   # 29 engines · error-based
├── header-blind-sql-injection.yaml  # time-based · HTTP headers
├── sqlInjection.yaml                # genérico · parâmetros de URL
│
├── # SSRF
├── header-blind-ssrf.yaml           # OOB · headers populares
├── ssrf-by-proxy.yaml               # parâmetros de proxy/redirect
├── xmlrpc-pingback-ssrf.yaml        # WordPress/Liferay · pingback
│
├── # Injeção de Headers
├── header-command-injection.yaml    # command injection via headers
├── smtp-header-injection-relay-oob.yaml # SMTP · OOB relay
│
├── # LFI
├── lfi.yaml                         # path traversal · parâmetros comuns
├── wordpressLFI.yaml                # WordPress · plugins/temas
│
├── # Autenticação / JWT
├── jwt-algorithm-confusion.yaml     # alg:none · RSA→HMAC downgrade
├── fuzz-oauth.yaml                  # OAuth endpoints · fuzzing
│
├── # Secrets / Credenciais
├── js_secrets.yaml                  # API keys · tokens em JS
├── gmail-api-client-secrets.yaml    # client_secrets.json
├── npmrc.yaml                       # .npmrc · npm tokens
├── s3cfg.yaml                       # .s3cfg · AWS credentials
│
├── # Banco de Dados / Schema
├── db-schema.yaml                   # ActiveRecord schema.rb
│
├── # Logs Expostos
├── development-logs.yaml            # Rails development.log
├── prod-logs.yaml                   # logs de produção
│
└── # Information Disclosure
    └── public-documents.yaml        # Excel · Word · CSV públicos
```

---

## Aviso legal

> Estes templates devem ser utilizados **somente em sistemas para os quais você possui autorização explícita** para realizar testes de segurança. O uso não autorizado contra sistemas de terceiros é ilegal e antiético. O autor não se responsabiliza pelo uso indevido destas ferramentas.

---

*by [Razielx64](https://github.com/Razielx64)*
