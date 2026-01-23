# Microservices Proto

Este repositório contém as definições de **Protocol Buffers (protobuf)** para a comunicação entre os microsserviços via gRPC.

## Estrutura

```
microservices-proto/
├── order/
│   └── order.proto          # Definições de mensagens e serviço Order
├── payment/
│   └── payment.proto        # Definições de mensagens e serviço Payment
├── shipping/
│   └── shipping.proto       # Definições de mensagens e serviço Shipping
└── golang/
    ├── order/               # Código Go compilado para Order
    ├── payment/             # Código Go compilado para Payment
    └── shipping/            # Código Go compilado para Shipping
```

## Serviços Definidos

### Order Service

**Arquivo**: `order/order.proto`

```protobuf
service Order {
  rpc Create (CreateOrderRequest) returns (CreateOrderResponse) {}
}

message CreateOrderRequest {
  int32 customer_id = 1;
  repeated OrderItem order_items = 2;
}

message OrderItem {
  string product_code = 1;
  float unit_price = 2;
  int32 quantity = 3;
}

message CreateOrderResponse {
  int32 order_id = 1;
}
```

### Payment Service

**Arquivo**: `payment/payment.proto`

```protobuf
service Payment {
  rpc Create (CreatePaymentRequest) returns (CreatePaymentResponse) {}
}

message CreatePaymentRequest {
  int64 user_id = 1;
  int64 order_id = 2;
  float total_price = 3;
}

message CreatePaymentResponse {
  int64 payment_id = 1;
  int64 bill_id = 2;
}
```

### Shipping Service

**Arquivo**: `shipping/shipping.proto`

```protobuf
service Shipping {
  rpc Create (CreateShippingRequest) returns (CreateShippingResponse) {}
}

message ShippingItem {
  string product_code = 1;
  int32 quantity = 2;
}

message CreateShippingRequest {
  int64 order_id = 1;
  repeated ShippingItem items = 2;
}

message CreateShippingResponse {
  int64 shipping_id = 1;
  int32 delivery_days = 2;
}
```

## Como Compilar os Protos

### Pré-requisitos

- Protocol Buffer Compiler (`protoc`) instalado
- Plugins Go: `protoc-gen-go` e `protoc-gen-go-grpc`

### Instalação

```bash
# Instalar protoc
# No macOS:
brew install protobuf

# No Linux (Ubuntu/Debian):
sudo apt-get install protobuf-compiler

# Instalar plugins Go
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### Compilar

```bash
# Compilar todos os protos
protoc --go_out=./golang --go-grpc_out=./golang order/order.proto
protoc --go_out=./golang --go-grpc_out=./golang payment/payment.proto
protoc --go_out=./golang --go-grpc_out=./golang shipping/shipping.proto

# Ou usando Docker (se preferir não instalar protoc localmente)
docker run --rm -v $(pwd):/workspace -w /workspace \
  namely/protoc \
  --go_out=./golang --go-grpc_out=./golang order/order.proto
```

## Estrutura dos Arquivos Gerados

Após compilação, cada proto gera dois arquivos Go:

- `{service}.pb.go`: Definições de mensagens
- `{service}_grpc.pb.go`: Código do cliente e servidor gRPC

Exemplo para Order:
- `golang/order/order.pb.go`
- `golang/order/order_grpc.pb.go`

## Usando em um Microsserviço

```go
import (
	orderpb "github.com/nillocoelho/microservices-proto/golang/order"
	"google.golang.org/grpc"
)

// Cliente
conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())
client := orderpb.NewOrderClient(conn)
response, _ := client.Create(ctx, &orderpb.CreateOrderRequest{...})

// Servidor
s := grpc.NewServer()
orderpb.RegisterOrderServer(s, &MyOrderServer{})
```

## Versioning

As definições de proto seguem Semantic Versioning. Mudanças compatíveis com versões anteriores não incrementam a versão menor.

## Boas Práticas

1. **Números de Campo**: Nunca reutilize números de campo já usados
2. **Compatibilidade**: Sempre mantenha compatibilidade com versões anteriores
3. **Comentários**: Documente mensagens e serviços com comentários
4. **Nomenclatura**: Use `snake_case` em mensagens proto, que é convertido para `CamelCase` em Go

## Referências

- [Protocol Buffers Documentation](https://developers.google.com/protocol-buffers)
- [gRPC Go Documentation](https://grpc.io/docs/languages/go/)
- [Go Protocol Buffers API](https://pkg.go.dev/google.golang.org/protobuf)
