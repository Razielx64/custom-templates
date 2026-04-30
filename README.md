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

> Templates customizados para o [Nuclei](https://github.com/projectdiscovery/nuclei) focados em detectar endpoints de gerenciamento expostos, informações sensíveis vazadas e misconfigurations em aplicações web modernas.

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

### `php_endpoints.yaml`
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

### `python_endpoints.yaml`
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

### `nodejs_endpoints.yaml`
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

### `ruby_endpoints.yaml`
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

### `api_endpoints.yaml`
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

### `swagger-paths.yaml`
Template dedicado à detecção de Swagger UI com lista abrangente de paths alternativos.

| Cobertura | O que detecta |
|-----------|---------------|
| Swagger UI | 50+ paths alternativos conhecidos |
| Versões legadas | `/v0.1/` → `/v0.12/index.html` |
| Paths por framework | Spring Boot, .NET, Django REST, FastAPI |

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
```

---

## Estrutura

```
custom/
├── README.md
├── php_endpoints.yaml       # PHP · Laravel · Symfony · WordPress
├── python_endpoints.yaml    # Django · Flask · FastAPI · Werkzeug
├── nodejs_endpoints.yaml    # Express · Fastify · NestJS
├── ruby_endpoints.yaml      # Ruby on Rails · Sidekiq
├── api_endpoints.yaml       # REST · GraphQL · OpenAPI
└── swagger-paths.yaml       # Swagger UI (paths extendidos)
```

---

## Aviso legal

> Estes templates devem ser utilizados **somente em sistemas para os quais você possui autorização explícita** para realizar testes de segurança. O uso não autorizado contra sistemas de terceiros é ilegal e antiético. O autor não se responsabiliza pelo uso indevido destas ferramentas.

---

*by [Razielx64](https://github.com/Razielx64)*
