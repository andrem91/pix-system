# 📖 Módulo 6: Arquitetura (2 semanas)

> Decisões arquiteturais que definem o sucesso de sistemas em produção.

---

## 📚 6.1 REST Best Practices

### Estrutura de Controller
```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {
    
    private final OrderService orderService;
    
    @GetMapping
    public Page<OrderResponse> findAll(Pageable pageable) {
        return orderService.findAll(pageable);
    }
    
    @GetMapping("/{id}")
    public OrderResponse findById(@PathVariable Long id) {
        return orderService.findById(id);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse create(@Valid @RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }
    
    @PutMapping("/{id}")
    public OrderResponse update(
            @PathVariable Long id,
            @Valid @RequestBody UpdateOrderRequest request) {
        return orderService.update(id, request);
    }
    
    @PatchMapping("/{id}/status")
    public OrderResponse updateStatus(
            @PathVariable Long id,
            @Valid @RequestBody UpdateStatusRequest request) {
        return orderService.updateStatus(id, request);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        orderService.delete(id);
    }
}
```

### HTTP Methods Corretos

| Método | Uso | Idempotente |
|--------|-----|-------------|
| GET | Buscar dados | Sim |
| POST | Criar recurso | Não |
| PUT | Substituir completamente | Sim |
| PATCH | Atualização parcial | Sim |
| DELETE | Remover | Sim |

### Status Codes

| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | GET, PUT, PATCH com sucesso |
| 201 | Created | POST com sucesso |
| 204 | No Content | DELETE com sucesso |
| 400 | Bad Request | Validação falhou |
| 401 | Unauthorized | Não autenticado |
| 403 | Forbidden | Sem permissão |
| 404 | Not Found | Recurso não existe |
| 409 | Conflict | Conflito de estado |
| 422 | Unprocessable Entity | Regra de negócio |
| 500 | Internal Server Error | Erro não tratado |

---

## 📚 6.2 Feign Client

### Declaração
```java
@FeignClient(
    name = "payment-service",
    url = "${services.payment.url}",
    configuration = FeignConfig.class
)
public interface PaymentClient {
    
    @PostMapping("/api/v1/payments")
    PaymentResponse process(@RequestBody PaymentRequest request);
    
    @GetMapping("/api/v1/payments/{id}")
    Optional<PaymentResponse> findById(@PathVariable("id") String id);
    
    @GetMapping("/api/v1/payments")
    List<PaymentResponse> findByCustomer(
        @RequestParam("customerId") Long customerId,
        @RequestHeader("X-Request-Id") String requestId
    );
}
```

### Configuração
```java
@Configuration
public class FeignConfig {
    
    @Bean
    public ErrorDecoder errorDecoder() {
        return new CustomErrorDecoder();
    }
    
    @Bean
    public Retryer retryer() {
        return new Retryer.Default(100, 1000, 3);
    }
    
    @Bean
    public RequestInterceptor authInterceptor() {
        return template -> {
            template.header("Authorization", "Bearer " + getToken());
        };
    }
}

public class CustomErrorDecoder implements ErrorDecoder {
    
    @Override
    public Exception decode(String methodKey, Response response) {
        return switch (response.status()) {
            case 400 -> new BadRequestException("Requisição inválida");
            case 404 -> new NotFoundException("Recurso não encontrado");
            case 500 -> new ServiceUnavailableException("Serviço indisponível");
            default -> new FeignException.InternalServerError("Erro desconhecido", response.request(), null, null);
        };
    }
}
```

---

## 📚 6.3 Circuit Breaker (Resilience4j)

### Configuração
```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
        slowCallRateThreshold: 100
        slowCallDurationThreshold: 2s
        
  retry:
    instances:
      payment:
        maxAttempts: 3
        waitDuration: 500ms
        retryExceptions:
          - java.io.IOException
          - feign.FeignException.ServiceUnavailable
          
  timelimiter:
    instances:
      payment:
        timeoutDuration: 3s
```

### Uso
```java
@Service
@RequiredArgsConstructor
public class PaymentService {
    
    private final PaymentClient paymentClient;
    
    @CircuitBreaker(name = "payment", fallbackMethod = "processPaymentFallback")
    @Retry(name = "payment")
    @TimeLimiter(name = "payment")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> paymentClient.process(request));
    }
    
    // Fallback - MESMO nome + parâmetros + Exception
    public CompletableFuture<PaymentResponse> processPaymentFallback(
            PaymentRequest request, Exception ex) {
        log.warn("Fallback triggered for payment: {}", ex.getMessage());
        return CompletableFuture.completedFuture(
            new PaymentResponse(Status.PENDING, "Processamento offline")
        );
    }
}
```

### Estados do Circuit Breaker

```
     ┌───────────────────────────────────────┐
     │              CLOSED                    │
     │  (funcionando normalmente)             │
     │                                        │
     │  Contando falhas...                    │
     │  Se failureRate > threshold:           │
     └────────────────┬──────────────────────┘
                      │ ABRE
                      ▼
     ┌───────────────────────────────────────┐
     │               OPEN                     │
     │  (rejeitando requisições)              │
     │                                        │
     │  Retorna fallback imediatamente        │
     │  Após waitDuration:                    │
     └────────────────┬──────────────────────┘
                      │ TENTATIVA
                      ▼
     ┌───────────────────────────────────────┐
     │            HALF_OPEN                   │
     │  (testando recuperação)                │
     │                                        │
     │  Permite N requisições de teste        │
     │  Se sucesso > threshold: FECHA         │
     │  Se falha: ABRE novamente              │
     └───────────────────────────────────────┘
```

---

## 📚 6.4 Kafka

### Producer
```java
@Service
@RequiredArgsConstructor
public class OrderEventProducer {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void publish(OrderEvent event) {
        kafkaTemplate.send("orders-topic", event.getOrderId(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Falha ao publicar evento: {}", ex.getMessage());
                } else {
                    log.info("Evento publicado: partition={}, offset={}", 
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                }
            });
    }
}
```

### Consumer
```java
@Service
@Slf4j
public class OrderEventConsumer {
    
    @KafkaListener(
        topics = "orders-topic",
        groupId = "order-processor",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consume(
            @Payload OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment ack) {
        
        try {
            log.info("Recebido: key={}, partition={}, offset={}", key, partition, offset);
            processOrder(event);
            ack.acknowledge();
        } catch (RetryableException e) {
            log.warn("Erro retryable, não confirma: {}", e.getMessage());
            // Não chama ack - será reprocessado
        } catch (Exception e) {
            log.error("Erro não retryable, enviando para DLQ: {}", e.getMessage());
            sendToDlq(event, e);
            ack.acknowledge();
        }
    }
}
```

### Configuração
```java
@Configuration
public class KafkaConfig {
    
    @Bean
    public ProducerFactory<String, OrderEvent> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        config.put(ProducerConfig.RETRIES_CONFIG, 3);
        return new DefaultKafkaProducerFactory<>(config);
    }
    
    @Bean
    public ConsumerFactory<String, OrderEvent> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        return new DefaultKafkaConsumerFactory<>(config,
            new StringDeserializer(),
            new JsonDeserializer<>(OrderEvent.class));
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> 
            kafkaListenerContainerFactory() {
        var factory = new ConcurrentKafkaListenerContainerFactory<String, OrderEvent>();
        factory.setConsumerFactory(consumerFactory());
        factory.getContainerProperties().setAckMode(AckMode.MANUAL);
        return factory;
    }
}
```

---

## 📚 6.5 DDD Básico

### Aggregate Root
```java
// Order é o Aggregate Root
// Entidades internas só são acessadas através dele
public class Order {
    private OrderId id;
    private OrderStatus status;
    private List<OrderItem> items; // Entidade interna
    private Money total;           // Value Object
    
    // Construtor que garante consistência
    public Order(Customer customer, List<OrderItem> items) {
        this.id = OrderId.generate();
        this.status = OrderStatus.CREATED;
        this.items = new ArrayList<>(items);
        this.total = calculateTotal();
        validate();
    }
    
    // Operações de negócio
    public void addItem(Product product, int quantity) {
        items.add(new OrderItem(product, quantity));
        recalculateTotal();
    }
    
    public void confirm() {
        if (status != OrderStatus.CREATED) {
            throw new IllegalStateException("Pedido não pode ser confirmado");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    private void validate() {
        if (items.isEmpty()) {
            throw new IllegalArgumentException("Pedido deve ter itens");
        }
    }
}
```

### Value Objects
```java
// Imutável, comparado por valor
public record Money(BigDecimal amount, Currency currency) {
    
    public Money {
        Objects.requireNonNull(amount);
        Objects.requireNonNull(currency);
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
    }
    
    public Money add(Money other) {
        assertSameCurrency(other);
        return new Money(amount.add(other.amount), currency);
    }
    
    public Money multiply(int factor) {
        return new Money(amount.multiply(BigDecimal.valueOf(factor)), currency);
    }
    
    private void assertSameCurrency(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currency mismatch");
        }
    }
}

public record OrderId(UUID value) {
    public static OrderId generate() {
        return new OrderId(UUID.randomUUID());
    }
    
    public static OrderId from(String value) {
        return new OrderId(UUID.fromString(value));
    }
}
```

---

## 📚 6.6 Saga Pattern

### Conceito
```
    Pedido          Pagamento       Estoque         Entrega
       │                │              │               │
       │    Criar       │              │               │
       ├───────────────►│              │               │
       │                │   Reservar   │               │
       │                ├─────────────►│               │
       │                │              │   Agendar     │
       │                │              ├──────────────►│
       │                │              │               │
       │    OK/FALHA    │              │               │
       │◄───────────────┼──────────────┼───────────────┤
       │                │              │               │
   Se FALHA:            │              │               │
   Compensação ◄────────┼──────────────┼───────────────┤
```

### Implementação Simples
```java
@Service
@RequiredArgsConstructor
public class OrderSaga {
    
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final ShippingService shippingService;
    
    @Transactional
    public Order execute(CreateOrderCommand command) {
        Order order = Order.create(command);
        
        try {
            // Etapa 1: Pagamento
            PaymentResult payment = paymentService.charge(order);
            if (!payment.isSuccess()) {
                throw new PaymentException(payment.getReason());
            }
            
            // Etapa 2: Reservar estoque
            inventoryService.reserve(order.getItems());
            
            // Etapa 3: Agendar entrega
            shippingService.schedule(order);
            
            order.confirm();
            return order;
            
        } catch (InventoryException e) {
            // Compensação: estornar pagamento
            paymentService.refund(order);
            throw e;
            
        } catch (ShippingException e) {
            // Compensação: liberar estoque e estornar
            inventoryService.release(order.getItems());
            paymentService.refund(order);
            throw e;
        }
    }
}
```

---

## 📚 6.7 Monolito vs Microsserviços

| Aspecto | Monolito | Microsserviços |
|---------|----------|----------------|
| **Complexidade inicial** | Simples | Alta |
| **Deploy** | Único | Múltiplos independentes |
| **Escalabilidade** | Vertical (mais recursos) | Horizontal (mais instâncias) |
| **Transações** | ACID fácil | Eventual consistency |
| **Comunicação** | In-process (rápido) | Network (latência) |
| **Observabilidade** | Simples | Requer ferramentas |
| **Time** | Pode ser pequeno | Idealmente independentes |
| **Quando usar** | MVP, time pequeno | Alta escala, múltiplos times |

### Decisão
```
Comece com monolito modular → Extraia microsserviços quando necessário
```

---

## 📚 6.8 Arquitetura Hexagonal (Ports & Adapters) ⭐ NOVO

### Conceito
```
                    ┌─────────────────────────────────────┐
                    │         INFRASTRUCTURE              │
                    │  (Adapters - implementações)        │
                    │                                     │
   HTTP Request ──► │ ┌─────────────────────────────────┐ │
                    │ │        APPLICATION               │ │
                    │ │  (Ports - interfaces)            │ │
                    │ │                                  │ │
   Database ◄────── │ │ ┌─────────────────────────────┐ │ │
                    │ │ │         DOMAIN               │ │ │
   Kafka ◄───────── │ │ │  (Regras de negócio puras)  │ │ │
                    │ │ │  Sem dependências externas   │ │ │
                    │ │ └─────────────────────────────┘ │ │
                    │ └─────────────────────────────────┘ │
                    └─────────────────────────────────────┘
```

### Estrutura de Pastas
```
src/main/java/com/example/order/
├── domain/                      # Núcleo - SEM dependências externas
│   ├── model/
│   │   ├── Order.java           # Entidade
│   │   ├── OrderItem.java
│   │   └── OrderStatus.java
│   ├── service/
│   │   └── OrderDomainService.java
│   └── port/                    # Interfaces (contratos)
│       ├── input/               # Driven by (use cases)
│       │   └── CreateOrderUseCase.java
│       └── output/              # Driving (repositórios, serviços externos)
│           ├── OrderRepository.java
│           └── PaymentGateway.java
│
├── application/                 # Implementação dos use cases
│   └── service/
│       └── OrderApplicationService.java
│
└── infrastructure/              # Adapters (implementações)
    ├── web/                     # Adapter HTTP
    │   └── OrderController.java
    ├── persistence/             # Adapter banco
    │   ├── JpaOrderRepository.java
    │   └── OrderEntity.java
    └── external/                # Adapter serviços externos
        └── PaymentGatewayAdapter.java
```

### Exemplo de Ports
```java
// Port de entrada (use case)
public interface CreateOrderUseCase {
    Order execute(CreateOrderCommand command);
}

// Port de saída (repositório)
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId id);
}

// Port de saída (serviço externo)
public interface PaymentGateway {
    PaymentResult charge(Order order);
}
```

### Exemplo de Adapters
```java
// Adapter primário (entrada)
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final CreateOrderUseCase createOrder;
    
    @PostMapping
    public OrderResponse create(@RequestBody CreateOrderRequest request) {
        Order order = createOrder.execute(toCommand(request));
        return toResponse(order);
    }
}

// Adapter secundário (saída)
@Repository
public class JpaOrderRepository implements OrderRepository {
    private final SpringDataOrderRepository springRepo;
    
    @Override
    public Order save(Order order) {
        OrderEntity entity = toEntity(order);
        return toDomain(springRepo.save(entity));
    }
}
```

### Benefícios
- **Testabilidade:** Domínio testável sem frameworks
- **Flexibilidade:** Trocar banco/framework sem alterar negócio
- **Clareza:** Separação clara de responsabilidades

---

## 📚 6.9 State Machine ⭐ NOVO

### Conceito
Máquina de estados controla transições válidas entre estados de uma entidade.

```
          ┌──────────────────────────────────────────────┐
          │                                              │
          ▼                                              │
  ┌───────────────┐    PAYMENT     ┌───────────────┐    │
  │    CREATED    │──────────────►│     PAID      │    │
  └───────┬───────┘                └───────┬───────┘    │
          │                                │            │
          │ CANCEL                   SHIP  │            │
          ▼                                ▼            │
  ┌───────────────┐                ┌───────────────┐    │
  │   CANCELLED   │                │    SHIPPED    │    │
  └───────────────┘                └───────┬───────┘    │
                                           │            │
                                   DELIVER │            │
                                           ▼            │
                                   ┌───────────────┐    │
                                   │   DELIVERED   │    │
                                   └───────────────┘    │
```

### Implementação com Spring State Machine
```java
@Configuration
@EnableStateMachine
public class OrderStateMachineConfig 
        extends StateMachineConfigurerAdapter<OrderState, OrderEvent> {
    
    @Override
    public void configure(StateMachineStateConfigurer<OrderState, OrderEvent> states) 
            throws Exception {
        states.withStates()
            .initial(OrderState.CREATED)
            .state(OrderState.PAID)
            .state(OrderState.SHIPPED)
            .end(OrderState.DELIVERED)
            .end(OrderState.CANCELLED);
    }
    
    @Override
    public void configure(StateMachineTransitionConfigurer<OrderState, OrderEvent> transitions) 
            throws Exception {
        transitions
            .withExternal()
                .source(OrderState.CREATED)
                .target(OrderState.PAID)
                .event(OrderEvent.PAYMENT_RECEIVED)
            .and()
            .withExternal()
                .source(OrderState.PAID)
                .target(OrderState.SHIPPED)
                .event(OrderEvent.SHIP)
            .and()
            .withExternal()
                .source(OrderState.SHIPPED)
                .target(OrderState.DELIVERED)
                .event(OrderEvent.DELIVER)
            .and()
            .withExternal()
                .source(OrderState.CREATED)
                .target(OrderState.CANCELLED)
                .event(OrderEvent.CANCEL);
    }
}
```

### Enums
```java
public enum OrderState {
    CREATED, PAID, SHIPPED, DELIVERED, CANCELLED
}

public enum OrderEvent {
    PAYMENT_RECEIVED, SHIP, DELIVER, CANCEL
}
```

### Uso
```java
@Service
public class OrderStateMachineService {
    
    private final StateMachineFactory<OrderState, OrderEvent> factory;
    
    public void processPayment(Order order) {
        StateMachine<OrderState, OrderEvent> sm = factory.getStateMachine(order.getId());
        sm.start();
        sm.sendEvent(OrderEvent.PAYMENT_RECEIVED);
        
        if (sm.getState().getId() == OrderState.PAID) {
            order.setStatus(OrderState.PAID);
            orderRepository.save(order);
        }
    }
}
```

### Alternativa: Enum Simples
```java
public enum OrderStatus {
    CREATED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return Set.of(PAID, CANCELLED);
        }
    },
    PAID {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return Set.of(SHIPPED, CANCELLED);
        }
    },
    SHIPPED {
        @Override
        public Set<OrderStatus> allowedTransitions() {
            return Set.of(DELIVERED);
        }
    },
    DELIVERED, CANCELLED;
    
    public Set<OrderStatus> allowedTransitions() {
        return Set.of(); // Estados finais
    }
    
    public void transitionTo(OrderStatus newStatus) {
        if (!allowedTransitions().contains(newStatus)) {
            throw new IllegalStateException(
                "Transição inválida: " + this + " -> " + newStatus);
        }
    }
}
```

---

## 📚 6.10 API Gateway ⭐ NOVO

### Conceito
Ponto de entrada único para microsserviços. Faz roteamento, autenticação, rate limiting.

```
              ┌─────────────────────────────────────┐
              │            API GATEWAY              │
  Client ──►  │  - Routing                          │
              │  - Authentication                   │
              │  - Rate Limiting                    │
              │  - Load Balancing                   │
              └──────────────┬──────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
  │   Order     │    │   Payment   │    │    User     │
  │   Service   │    │   Service   │    │   Service   │
  └─────────────┘    └─────────────┘    └─────────────┘
```

### Spring Cloud Gateway
```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: orderCircuitBreaker
                fallbackUri: forward:/fallback/orders
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Request-Source, gateway
            
        - id: auth-service
          uri: lb://AUTH-SERVICE
          predicates:
            - Path=/api/auth/**
```

### Filtros Customizados
```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest()
            .getHeaders()
            .getFirst("Authorization");
            
        if (token == null || !isValid(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
    
    @Override
    public int getOrder() {
        return -1; // Executar primeiro
    }
}
```

### Fallback Controller
```java
@RestController
@RequestMapping("/fallback")
public class FallbackController {
    
    @GetMapping("/orders")
    public ResponseEntity<Map<String, String>> ordersFallback() {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(Map.of(
                "message", "Order service temporarily unavailable",
                "suggestion", "Please try again later"
            ));
    }
}
```

---

## 🎯 Perguntas de Entrevista

1. **Quando usar microsserviços vs monolito?**
2. **O que é Circuit Breaker?**
3. **O que é idempotência?**
4. **Como funciona o Kafka?**
5. **O que é Saga Pattern?**
6. **O que é Arquitetura Hexagonal?**
7. **Como funciona uma State Machine?**
8. **Para que serve um API Gateway?**

---

> **Próximo módulo:** [Módulo 7 - Testes](MODULO_07_TESTES.md)
