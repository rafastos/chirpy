
# Chirpy

Uma API simples para publicar e consumir mensagens curtas ("chirps").

- O que faz: fornece endpoints REST para usuários, autenticação por tokens, criação/consulta/remoção de chirps e suporte a refresh tokens e redefinição de senha.
- Por que se importaria: útil como exemplo leve de backend em Go com boas práticas (sqlc, migrations, refresh tokens) e pronto para servir uma SPA ou aplicativo móvel.
- Instalação e execução rápidas:

	Requisitos: Go, PostgreSQL.

	1. Configure a base de dados e variáveis de ambiente (ex.: `DATABASE_URL`, `PORT`, `JWT_SECRET`).
	2. Execute migrations em `sql/schema` (por exemplo via `psql`).
	3. Rode a aplicação:

	```bash
	go run .
	```

## Funcionalidades

- Autenticação (login + refresh tokens)
- CRUD parcial de usuários
- Publicação, listagem e remoção de "chirps"
- Reset de senha e webhooks básicos
- Endpoints de métricas e readiness

## Arquitetura

Projeto escrito em Go com acesso ao banco por código gerado (`sqlc`).

- Código principal: [main.go](main.go)
- Handlers: arquivos `handler_*.go` (rotas e lógica HTTP)
- Persistência: `internal/database` (ações geradas e modelos)
- Migrations: `sql/schema`

## Instalação detalhada

1. Instale o Go (versão compatível com o `go.mod`) e PostgreSQL.
2. Clone o repositório.
3. Configure a variável de ambiente `DATABASE_URL` apontando para seu PostgreSQL.
4. Aplique as migrations contidas em `sql/schema`:

```bash
psql "$DATABASE_URL" -f sql/schema/001_users.sql
psql "$DATABASE_URL" -f sql/schema/002_chirps.sql
psql "$DATABASE_URL" -f sql/schema/003_password.sql
psql "$DATABASE_URL" -f sql/schema/004_refresh_tokens.sql
psql "$DATABASE_URL" -f sql/schema/005_chirpy_red.sql
```

5. (Opcional) Gere artefatos do `sqlc` se você alterar queries:

```bash
sqlc generate
```

## Variáveis de ambiente sugeridas

- `DATABASE_URL` - string de conexão com PostgreSQL
- `PORT` - porta HTTP (ex.: `8080`)
- `JWT_SECRET` - segredo para assinatura de tokens

## Como rodar

No desenvolvimento:

```bash
DATABASE_URL="postgres://user:pass@localhost:5432/chirpy?sslmode=disable" \
JWT_SECRET="musecreto" \
go run .
```

Para compilar um binário:

```bash
go build -o chirpy
./chirpy
```

## Endpoints principais (resumo)

- `POST /login` — autentica usuário e retorna access + refresh tokens. (handler_login.go)
- `POST /refresh` — troca refresh token por novo access token. (handler_refresh_create.go)
- `POST /users` — cria usuário. (handler_users_create.go)
- `PUT /users/:id` — atualiza usuário. (handler_users_update.go)
- `POST /chirps` — cria um chirp autenticado. (handler_chirps_create.go)
- `GET /chirps` — lista chirps. (handler_chirps_get.go)
- `DELETE /chirps/:id` — remove um chirp. (handler_chirps_delete.go)
- `POST /reset` — inicia/reset de senha. (reset.go)
- `POST /webhooks` — endpoint de webhooks. (handler_webhooks.go)
- `GET /metrics` — métricas expostas. (metrics.go)
- `GET /ready` (ou similar) — readiness check. (readiness.go)

> Observação: os nomes e rotas exatos estão implementados nos arquivos `handler_*.go`.

## Banco de dados

As consultas e modelos ficam em `internal/database` e as queries SQL em `sql/queries`.
Use `sqlc` para regenerar o código caso altere as SQLs.

## Desenvolvimento

- Execute testes unitários:

```bash
go test ./...
```

- Linter / formatação: use `gofmt`/`golangci-lint` conforme sua preferência.

## Deploy

Este serviço é facilmente empacotado como binário e rodado em uma VPS ou container. Para produção:

- Execute migrations na infra de produção.
- Configure variáveis de ambiente seguras (`JWT_SECRET`, `DATABASE_URL`).
- Rode por trás de um proxy reverso / systemd / container.

## Contribuição

Contribuições bem-vindas: abra issues e pull requests. Siga padrões de commit claros e mantenha testes para mudanças comportamentais.

## Referências úteis no repositório

- [main.go](main.go) — entrada da aplicação
- [sql/schema](sql/schema) — migrations SQL
- [internal/database](internal/database) — código gerado e modelos

## Licença

Licença permissiva — escolha e adicione a licença desejada (ex.: MIT).

