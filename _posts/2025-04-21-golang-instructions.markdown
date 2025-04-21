---
layout: post
title:  "Golang Instructions"
date:   2025-04-20 21:16:02 +0200
categories: Golang
---
### Golang Instructions
#### 1. create a new project
- open a terminal and run the following command to create a new web application
```bash
go mod init TodoListApp
go mod tidy
```
- navigate to the project directory
```bash
cd TodoListApp
```
#### 2. dependency management
```bash
go get github.com/gorilla/mux
go get github.com/gorilla/handlers
go get github.com/gorilla/sessions
```
#### 3. database with sqlc
- create a file named `sqlc.yaml` in the root directory and add the following content
```yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "query.sql"
    schema: "schema.sql"
    gen:
      go:
        package: "tutorial"
        out: "tutorial"
        sql_package: "pgx/v5"
        emit_json_tags: true
        overrides:
          - column: "devices.credentials"
            go_type:
              import: "github.com/wilbyang/ubiq-monitor/internal/dto"
              type: "Credentials"
```
- create a file named `schema.sql` in the root directory and add the following content
```sql
CREATE TABLE devices (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    credentials JSONB NOT NULL
);
```
- create a file named `query.sql` in the root directory and add the following content
```sql
-- name: CreateDevice :one
INSERT INTO devices (name, credentials) VALUES ($1, $2) RETURNING id;
-- name: GetDevice :one
SELECT * FROM devices WHERE id = $1;
-- name: ListDevices :many
SELECT * FROM devices;
```
- run the following command to generate the code
```bash
sqlc generate
```
#### 4. Project Structure
- create the following folders in the root directory
mkdir -p TodoListApp/internal/{config,controller,dto,repository,service}
```

#### 5. openapi spec
```bash
swag init -g main.go
```
