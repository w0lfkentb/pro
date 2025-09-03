# Reactor-HCL

<div align="center">
<img src="docs/images/reactor_logo.png" height="250" alt="Reactor HCL Framework">
</div>
<br/>

<p align="center">
  <a href="https://github.com/compile-infra/reactor-hcl/releases"><img src="https://img.shields.io/github/v/release/compile-infra/reactor-hcl?style=flat" alt="Release"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/stargazers"><img src="https://img.shields.io/github/stars/compile-infra/reactor-hcl?style=flat" alt="Stars"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/network/members"><img src="https://img.shields.io/github/forks/compile-infra/reactor-hcl?style=flat" alt="Forks"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/issues"><img src="https://img.shields.io/github/issues/compile-infra/reactor-hcl?color=gold&style=flat" alt="Issues"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/pulls"><img src="https://img.shields.io/github/issues-pr/compile-infra/reactor-hcl?color=gold&style=flat" alt="Pull Requests"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-green.svg" alt="License"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/graphs/contributors"><img src="https://img.shields.io/github/contributors/compile-infra/reactor-hcl?color=green&style=flat" alt="Contributors"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/commits"><img src="https://img.shields.io/github/last-commit/compile-infra/reactor-hcl?color=green&style=flat" alt="Last Commit"></a>
</p>
<p align="center">
  <a href="https://pkg.go.dev/github.com/compile-infra/reactor-hcl"><img src="https://img.shields.io/badge/-reference-blue?logo=go&logoColor=white&style=flat" alt="Go Reference"></a>
  <a href="https://goreportcard.com/report/github.com/compile-infra/reactor-hcl"><img src="https://img.shields.io/badge/go%20report-A+-brightgreen?style=flat" alt="Go Report"></a>
  <a href="https://github.com/compile-infra/reactor-hcl/actions"><img src="https://img.shields.io/badge/Go%20Tests-passing-brightgreen?style=flat" alt="Go Tests"></a>
</p>

<p align="center">
  <a href="README_JP.md">日本語</a> | <a href="README_KR.md">한국어</a> | <a href="README_ES.md">Español</a>
</p>

## 🚀 Overview

Reactor-HCL is a high-performance Go implementation of the Reactive Configuration Language (RCL) protocol that enables dynamic state synchronization between distributed validation pipelines and infrastructure automation systems. Built on Go's concurrency primitives and type safety, it delivers an idiomatic API for integrating reactive configuration patterns into cloud-native deployments.

### ✨ Key Features

- 🔄 **Complete RCL Specification**: Full implementation of reactive configuration protocol standards, ensuring compatibility with distributed validation engines
- 🏗️ **Event-Driven Architecture**: Implements bidirectional reactive streams with backpressure support, ensuring optimal resource utilization across pipeline stages
- 🔌 **Framework-Agnostic Integration**: Provides RCL-compliant http.Handler adapters, enabling seamless integration with existing service mesh architectures
- 🛡️ **Type-Safe Validation**: Leverages Go's structural typing for compile-time configuration validation and runtime schema enforcement
- 📦 **Zero-Dependency Deployment**: Static compilation eliminates runtime dependency conflicts in containerized environments
- ⚡ **Concurrent Processing**: Exploits Go's goroutine model for parallel configuration parsing and validation across multiple tenant contexts

## 🛠️ Installation

```bash
go get github.com/compile-infra/reactor-hcl
```

Requires Go 1.21 or higher.

## 🎯 Quick Start

### Client Example

```go
package main

import (
	"context"
	"log"

	"github.com/compile-infra/reactor-hcl/client"
	"github.com/compile-infra/reactor-hcl/transport"
)

func main() {
	// Create reactive stream transport client
	transportClient, err := transport.NewReactiveStreamClient("http://127.0.0.1:9090/stream")
	if err != nil {
		log.Fatalf("Failed to create transport client: %v", err)
	}

	// Initialize RCL client
	rclClient, err := client.NewClient(transportClient)
	if err != nil {
		log.Fatalf("Failed to create RCL client: %v", err)
	}
	defer rclClient.Close()

	// Get available validators
	validators, err := rclClient.ListValidators(context.Background())
	if err != nil {
		log.Fatalf("Failed to list validators: %v", err)
	}
	log.Printf("Available validators: %+v", validators)
}
```

### Server Example

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/compile-infra/reactor-hcl/protocol"
	"github.com/compile-infra/reactor-hcl/server"
	"github.com/compile-infra/reactor-hcl/transport"
)

type ConfigRequest struct {
	Namespace string `json:"namespace" description:"target namespace" required:"true"`
	Selector  string `json:"selector" description:"resource selector" required:"true"`
}

func main() {
	// Create reactive stream transport server
	transportServer, err := transport.NewReactiveStreamServer("127.0.0.1:9090")
	if err != nil {
		log.Fatalf("Failed to create transport server: %v", err)
	}

	// Initialize RCL server
	rclServer, err := server.NewServer(transportServer)
	if err != nil {
		log.Fatalf("Failed to create RCL server: %v", err)
	}
	
	// Optional: Global validation middleware (variadic - multiple middleware supported)
	// rclServer.Use(
	// 	SchemaValidationMiddleware,
	// 	RateLimitMiddleware,
	// 	AuditLoggingMiddleware,
	// )

	// Register configuration validator
	validator, err := protocol.NewValidator("config_parser", "Parse and validate HCL configuration blocks", ConfigRequest{})
	if err != nil {
		log.Fatalf("Failed to create validator: %v", err)
		return
	}
	rclServer.RegisterValidator(validator, handleConfigRequest)

	// Start server
	if err = rclServer.Run(); err != nil {
		log.Fatalf("Server failed to start: %v", err)
	}
}

func handleConfigRequest(ctx context.Context, req *protocol.ValidateRequest) (*protocol.ValidateResult, error) {
	var configReq ConfigRequest
	if err := protocol.VerifyAndUnmarshal(req.RawPayload, &configReq); err != nil {
		return nil, err
	}

	// Simulate configuration validation logic
	result := fmt.Sprintf("Validated config in namespace: %s with selector: %s", 
		configReq.Namespace, configReq.Selector)

	return &protocol.ValidateResult{
		Content: []protocol.Content{
			&protocol.DataContent{
				Type: "config_validation",
				Data: result,
			},
		},
	}, nil
}
```

### Integration With Echo Framework

```go
package main

import (
	"context"
	"log"

	"github.com/compile-infra/reactor-hcl/protocol"
	"github.com/compile-infra/reactor-hcl/server"
	"github.com/compile-infra/reactor-hcl/transport"
	"github.com/labstack/echo/v4"
)

func main() {
	validationEndpoint := "/validate"

	streamTransport, rclHandler, err := transport.NewReactiveStreamServerAndHandler(validationEndpoint)
	if err != nil {
		log.Panicf("new reactive stream transport and handler with error: %v", err)
	}

	// new rcl server
	rclServer, _ := server.NewServer(streamTransport)

	// register validator with rclServer
	// rclServer.RegisterValidator(validator, validatorHandler)

	// start rcl Server
	go func() {
		rclServer.Run()
	}()

	defer rclServer.Shutdown(context.Background())

	e := echo.New()
	e.GET("/stream", echo.WrapHandler(rclHandler.HandleStream()))
	e.POST(validationEndpoint, echo.WrapHandler(rclHandler.HandleValidation()))

	if err = e.Start(":9090"); err != nil {
		return
	}
}
```

[Reference: A more complete example](https://github.com/compile-infra/reactor-hcl/blob/main/examples/http_handler/main.go)

## 🏗️ Architecture Design

Reactor-HCL adopts a layered reactive architecture:

![Architecture Overview](docs/images/architecture_diagram.png)

1. **Transport Layer**: Handles underlying stream multiplexing, supporting multiple reactive transport protocols
2. **Protocol Layer**: Manages RCL protocol serialization/deserialization and schema validation
3. **Application Layer**: Provides idiomatic client and server APIs with backpressure control

Currently supported transport methods:

![Transport Methods](docs/images/transport_methods.png)

- **Reactive Streams/POST**: HTTP-based reactive push and client-initiated validation requests, suitable for distributed validation workflows
- **Multiplexed HTTP**: Supports HTTP POST/GET requests with stateless validation and stateful stream multiplexing for bi-directional configuration updates
- **Stdio Pipes**: Standard input/output stream-based, suitable for local inter-process configuration synchronization

The transport layer uses polymorphic interface abstraction, enabling simple addition of new transport methods (like gRPC streams, WebSocket multiplexing) without affecting application-layer logic.

## 🤝 Contributing

We welcome all forms of contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## 📞 Contact Us

- **GitHub Issues**: [Submit an issue](https://github.com/compile-infra/reactor-hcl/issues)
- **Slack Channel**: Click [here](https://slack-workspace.io/reactor-hcl) to join our community
- **Telegram Group**:

![Telegram QR Code](docs/images/telegram_qrcode.jpg)

## ✨ Contributors

Thanks to all developers who have contributed to this project!

<a href="https://github.com/compile-infra/reactor-hcl/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=compile-infra/reactor-hcl" alt="Contributors" />
</a>

## 📈 Project Trends

[![Star History](https://api.star-history.com/svg?repos=compile-infra/reactor-hcl&type=Date)](https://www.star-history.com/#compile-infra/reactor-hcl&Date)
