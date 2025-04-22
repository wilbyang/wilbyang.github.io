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
#### 2. dependency injection

please refer to [this post]({% post_url 2025-04-20-golang-wire-denpendency-injection %}).

#### 3. database with sqlc
- create a file named `sqlc.yaml` in the root directory and add the following content

```yml
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

#### 5. OpenAPI spec
```bash
swag init -g main.go
```

```go
// @Summary Add a new law document
// @Description Create a new law document with the provided details
// @Tags documents
// @Accept json
// @Param document body repository.CreateDocumentParams true "Document details"
// @Produce json
// @Success 201 {object} map[string]interface{} "Document created"
// @Failure 400 {object} map[string]interface{} "Invalid input"
// @Failure 500 {object} map[string]interface{} "Failed to create document"
// @Router /api/v1/docs [post]
import (
_ "github.com/wilbyang/ubiq-monitor/docs"
swaggerFiles "github.com/swaggo/files"
ginSwagger "github.com/swaggo/gin-swagger"
)
r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```

#### 6. Tracing & Metrics
```go
"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
  var (
	counter = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "file_upload_total",
			Help: "Total number of file uploads",
		},
		[]string{"doctype", "priority"},
	)
)

func init() {
	prometheus.MustRegister(counter)
}
api.router.GET("/metrics", gin.WrapH(promhttp.Handler()))
```