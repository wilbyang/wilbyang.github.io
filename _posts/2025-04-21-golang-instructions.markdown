---
layout: post
title:  "Golang Instructions"
date:   2025-04-20 21:16:02 +0200
categories: Golang
---
### Golang Instructions
#### create a new project
- open a terminal and run the following command to create a new web application
```bash
go mod init TodoListApp
go mod tidy
```
- navigate to the project directory
```bash
cd TodoListApp
```
#### Project Structure 
- create the following folders in the root directory
```bash
mkdir -p TodoListApp/internal/{config,controller,dto,repository,service}
```

#### dependency injection

please refer to [this post]({% post_url 2025-04-21-golang-wire-dependency-injection %}).

#### database with sqlc
- create a file named `sqlc.yaml` in the root directory and add the following content

```yml
version: "2"
sql:
  - engine: "postgresql" # mysql, sqlite
    queries: "query.sql"
    schema: "schema.sql"
    gen:
      go:
        package: "entity"
        out: "internal/db"
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

### use the sqlc generated code

```go

package main

import (
	"context"
	"log"

	"github.com/jackc/pgx/v5/pgxpool"
	"github.com/wilbyang/gotmp/dto"
	"github.com/wilbyang/gotmp/tutorial"
)

func main() {
	ctx := context.Background()
	dbpool, err := pgxpool.New(ctx, "postgres://boya:@localhost:28813/device_monitor")
	if err != nil {
		panic(err)
	}
	defer dbpool.Close()
	queries := tutorial.New(dbpool)
	queries.CreateDevice(context.Background(), tutorial.CreateDeviceParams{
		Name: "test_device",
		Credentials: dto.Credentials{
			Username: "test_user",
			Password: "test_pass",
		},
	})
	if err != nil {
		log.Fatalf("failed to create device: %v", err)
	}
}

```


#### OpenAPI spec
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

#### Tracing & Metrics
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

#### GRPC
- create a folder named `proto` in the root directory and add the following content

```bash
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative proto/device.proto
```

- server 

```go
"google.golang.org/grpc/reflection"
reflection.Register(grpcServer)
```

- client test


```bash
grpcurl -plaintext -d '{"device_identifier": "device"}' localhost:8082 device.monitoring.DeviceMonitoringService/CheckHealth
```
