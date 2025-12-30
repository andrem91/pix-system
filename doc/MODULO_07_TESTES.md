# 📖 Módulo 7: Testes (1 semana)

> Código sem testes é débito técnico. Testes bem escritos são documentação viva.

---

## 📚 7.1 JUnit 5

### Estrutura Básica
```java
import org.junit.jupiter.api.*;
import static org.assertj.core.api.Assertions.*;

class CalculadoraTest {
    
    private Calculadora calculadora;
    
    @BeforeEach
    void setUp() {
        calculadora = new Calculadora();
    }
    
    @Test
    @DisplayName("Deve somar dois números positivos")
    void deveSomarNumerosPositivos() {
        // Given
        int a = 5, b = 3;
        
        // When
        int resultado = calculadora.somar(a, b);
        
        // Then
        assertThat(resultado).isEqualTo(8);
    }
    
    @Test
    @DisplayName("Deve lançar exceção ao dividir por zero")
    void deveLancarExcecaoAoDividirPorZero() {
        assertThatThrownBy(() -> calculadora.dividir(10, 0))
            .isInstanceOf(ArithmeticException.class)
            .hasMessageContaining("zero");
    }
}
```

### Testes Parametrizados
```java
class ValidadorCpfTest {
    
    @ParameterizedTest
    @ValueSource(strings = {"123.456.789-09", "111.444.777-35"})
    @DisplayName("Deve validar CPFs válidos")
    void deveValidarCpfsValidos(String cpf) {
        assertThat(ValidadorCpf.isValid(cpf)).isTrue();
    }
    
    @ParameterizedTest
    @NullAndEmptySource
    @ValueSource(strings = {"123", "abc", "111.111.111-11"})
    @DisplayName("Deve rejeitar CPFs inválidos")
    void deveRejeitarCpfsInvalidos(String cpf) {
        assertThat(ValidadorCpf.isValid(cpf)).isFalse();
    }
    
    @ParameterizedTest
    @CsvSource({
        "100, 10, 90",
        "50, 50, 0",
        "200, 0, 200"
    })
    @DisplayName("Deve calcular desconto")
    void deveCalcularDesconto(int valor, int desconto, int esperado) {
        assertThat(calculadora.desconto(valor, desconto)).isEqualTo(esperado);
    }
    
    @ParameterizedTest
    @MethodSource("provideStringsForIsBlank")
    void deveTratarStringsVazias(String input, boolean expected) {
        assertThat(StringUtils.isBlank(input)).isEqualTo(expected);
    }
    
    private static Stream<Arguments> provideStringsForIsBlank() {
        return Stream.of(
            Arguments.of("", true),
            Arguments.of("  ", true),
            Arguments.of(null, true),
            Arguments.of("not blank", false)
        );
    }
}
```

---

## 📚 7.2 Mockito

### Mocking Básico
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private OrderRepository orderRepository;
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    @DisplayName("Deve criar pedido com sucesso")
    void deveCriarPedidoComSucesso() {
        // Given
        CreateOrderRequest request = new CreateOrderRequest(/*...*/);
        Order expectedOrder = new Order(/*...*/);
        
        when(orderRepository.save(any(Order.class)))
            .thenAnswer(invocation -> {
                Order order = invocation.getArgument(0);
                order.setId(1L);
                return order;
            });
        
        // When
        Order result = orderService.create(request);
        
        // Then
        assertThat(result.getId()).isNotNull();
        assertThat(result.getStatus()).isEqualTo(OrderStatus.CREATED);
        
        verify(orderRepository).save(any(Order.class));
        verify(emailService).sendOrderConfirmation(any(Order.class));
        verifyNoInteractions(paymentService);
    }
    
    @Test
    @DisplayName("Deve lançar exceção quando cliente não existe")
    void deveLancarExcecaoQuandoClienteNaoExiste() {
        // Given
        when(customerRepository.findById(anyLong()))
            .thenReturn(Optional.empty());
        
        CreateOrderRequest request = new CreateOrderRequest(999L, /*...*/);
        
        // When/Then
        assertThatThrownBy(() -> orderService.create(request))
            .isInstanceOf(CustomerNotFoundException.class)
            .hasMessage("Cliente não encontrado: 999");
        
        verify(orderRepository, never()).save(any());
    }
}
```

### Argument Captors
```java
@Test
void deveEnviarEmailComDadosCorretos() {
    // Given
    ArgumentCaptor<EmailMessage> emailCaptor = ArgumentCaptor.forClass(EmailMessage.class);
    
    // When
    orderService.create(request);
    
    // Then
    verify(emailService).send(emailCaptor.capture());
    
    EmailMessage email = emailCaptor.getValue();
    assertThat(email.getTo()).isEqualTo("customer@email.com");
    assertThat(email.getSubject()).contains("Pedido confirmado");
}
```

---

## 📚 7.3 @Mock vs @MockBean

| @Mock (Mockito) | @MockBean (Spring) |
|-----------------|--------------------| 
| Testes unitários | Testes de integração |
| Não carrega contexto Spring | Substitui bean no contexto |
| Muito rápido | Mais lento |
| Use com @InjectMocks | Use com @Autowired |

```java
// Teste UNITÁRIO - @Mock
@ExtendWith(MockitoExtension.class)
class OrderServiceUnitTest {
    @Mock
    private OrderRepository repository;
    
    @InjectMocks
    private OrderService service;
    
    @Test
    void test() { /* sem Spring */ }
}

// Teste de INTEGRAÇÃO - @MockBean
@SpringBootTest
class OrderServiceIntegrationTest {
    @MockBean
    private PaymentClient paymentClient; // Substitui bean real
    
    @Autowired
    private OrderService service; // Bean real com dependência mockada
    
    @Test
    void test() { /* com contexto Spring */ }
}
```

---

## 📚 7.4 Test Slices

### @WebMvcTest - Controllers
```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private OrderService orderService;
    
    @Test
    void deveRetornarPedidoPorId() throws Exception {
        // Given
        OrderResponse response = new OrderResponse(1L, "CREATED", BigDecimal.TEN);
        when(orderService.findById(1L)).thenReturn(response);
        
        // When/Then
        mockMvc.perform(get("/api/v1/orders/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.status").value("CREATED"));
    }
    
    @Test
    void deveRetornar400ParaRequestInvalido() throws Exception {
        // Given
        String invalidRequest = """
            {
                "customerId": null,
                "items": []
            }
            """;
        
        // When/Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(invalidRequest))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").isArray());
    }
}
```

### @DataJpaTest - Repositórios
```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
class OrderRepositoryTest {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void deveBuscarPedidosPorStatus() {
        // Given
        Order order1 = entityManager.persist(new Order(Status.CREATED));
        Order order2 = entityManager.persist(new Order(Status.CONFIRMED));
        entityManager.flush();
        
        // When
        List<Order> result = orderRepository.findByStatus(Status.CREATED);
        
        // Then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getId()).isEqualTo(order1.getId());
    }
    
    @Test
    @Sql("/test-data/orders.sql")
    void deveBuscarPedidosComDadosPreCarregados() {
        List<Order> orders = orderRepository.findAll();
        assertThat(orders).hasSize(5);
    }
}
```

### @JsonTest - Serialização
```java
@JsonTest
class OrderResponseJsonTest {
    
    @Autowired
    private JacksonTester<OrderResponse> json;
    
    @Test
    void deveSerializarCorretamente() throws Exception {
        OrderResponse response = new OrderResponse(1L, "CREATED", new BigDecimal("99.90"));
        
        JsonContent<OrderResponse> result = json.write(response);
        
        assertThat(result).extractingJsonPathNumberValue("$.id").isEqualTo(1);
        assertThat(result).extractingJsonPathStringValue("$.status").isEqualTo("CREATED");
        assertThat(result).extractingJsonPathNumberValue("$.total").isEqualTo(99.90);
    }
}
```

---

## 📚 7.5 TestContainers

```java
@SpringBootTest
@Testcontainers
class OrderIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.4.0"));
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private OrderService orderService;
    
    @Test
    void deveCriarPedidoEPersistir() {
        // Given
        CreateOrderRequest request = createValidRequest();
        
        // When
        Order result = orderService.create(request);
        
        // Then
        Order saved = orderRepository.findById(result.getId()).orElseThrow();
        assertThat(saved.getStatus()).isEqualTo(OrderStatus.CREATED);
    }
    
    @Test
    void devePublicarEventoNoKafka() {
        // Teste de integração com Kafka real
    }
}
```

---

## 📚 7.6 WireMock

```java
@SpringBootTest
@AutoConfigureWireMock(port = 0) // Porta dinâmica
class PaymentClientTest {
    
    @Autowired
    private PaymentClient paymentClient;
    
    @Test
    void deveProcessarPagamentoComSucesso() {
        // Given
        stubFor(post(urlEqualTo("/api/v1/payments"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {
                        "id": "pay_123",
                        "status": "APPROVED",
                        "amount": 100.00
                    }
                    """)));
        
        // When
        PaymentResponse response = paymentClient.process(new PaymentRequest(100.00));
        
        // Then
        assertThat(response.status()).isEqualTo("APPROVED");
        
        verify(postRequestedFor(urlEqualTo("/api/v1/payments"))
            .withHeader("Content-Type", equalTo("application/json")));
    }
    
    @Test
    void deveTratarTimeoutDoServicoExterno() {
        // Given - simula timeout
        stubFor(post(urlEqualTo("/api/v1/payments"))
            .willReturn(aResponse()
                .withStatus(200)
                .withFixedDelay(5000))); // 5 segundos de delay
        
        // When/Then
        assertThatThrownBy(() -> paymentClient.process(new PaymentRequest(100.00)))
            .isInstanceOf(FeignException.class);
    }
}
```

---

## 📚 7.7 Pirâmide de Testes

```
            /\
           /  \
          /    \         E2E (poucos)
         /______\        - Fluxos críticos
        /        \       - Lentos, frágeis
       /          \
      /   Integr   \     Integração (alguns)
     /______________\    - TestContainers, @SpringBootTest
    /                \   - Componentes juntos
   /                  \
  /     Unitários      \ Unitários (muitos)
 /______________________\- Mockito, rápidos
                         - Lógica isolada
```

### O Que Testar

| Tipo | O Que Testar | Ferramentas |
|------|--------------|-------------|
| Unit | Lógica de negócio, validações | JUnit, Mockito |
| Integration | Repository + DB, API + Service | TestContainers, @SpringBootTest |
| E2E | Fluxos críticos completos | REST Assured, Selenium |
| Contract | Contratos entre serviços | Spring Cloud Contract, Pact |

---

## 📚 7.8 Mutation Testing (PIT)

### O Problema
> "Tenho 90% de cobertura, mas meus testes são fracos"

Cobertura alta **não significa** testes eficazes. Seus testes podem estar apenas executando o código sem realmente validar.

### Como Funciona

```
┌───────────────────────────────────────────────────────────────┐
│                    MUTATION TESTING                            │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│  1. PIT pega seu código                                       │
│     if (idade >= 18) return true;                             │
│                                                                │
│  2. Cria MUTANTES (pequenas alterações)                       │
│     if (idade > 18) return true;    ← Mutante 1 (>= → >)      │
│     if (idade >= 18) return false;  ← Mutante 2 (true→false)  │
│                                                                │
│  3. Roda seus testes contra cada mutante                      │
│                                                                │
│  4. Se o teste FALHAR → Mutante MORTO ✅ (teste é bom!)       │
│     Se o teste PASSAR → Mutante VIVO ❌ (teste é fraco!)      │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Configuração Maven

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.15.3</version>
    <dependencies>
        <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>
    <configuration>
        <targetClasses>
            <param>com.pixsystem.account.domain.*</param>
        </targetClasses>
        <targetTests>
            <param>com.pixsystem.account.domain.*Test</param>
        </targetTests>
        <mutationThreshold>80</mutationThreshold>
        <coverageThreshold>80</coverageThreshold>
    </configuration>
</plugin>
```

### Executar

```bash
mvn test-compile org.pitest:pitest-maven:mutationCoverage
```

### Interpretando Resultados

```
>> Mutations: 50
>> Killed: 45 (90%)        ← Testes mataram 90% dos mutantes
>> Survived: 5 (10%)       ← 5 mutantes sobreviveram!
```

**Mutantes que sobrevivem = testes que precisam melhorar!**

### Exemplo Prático

```java
// Código
public boolean isAdult(int age) {
    return age >= 18;  // Limite exato
}

// ❌ Teste FRACO (mutante sobrevive)
@Test
void testIsAdult() {
    assertTrue(isAdult(20));  // Não testa o limite!
}

// ✅ Teste FORTE (mata o mutante >= → >)
@Test
void testIsAdultBoundary() {
    assertFalse(isAdult(17)); // Abaixo do limite
    assertTrue(isAdult(18));  // Exatamente no limite
    assertTrue(isAdult(19));  // Acima do limite
}
```

---

## 📚 7.9 Contract Testing

### O Problema em Microsserviços

```
Account Service         Transfer Service
      │                        │
      │  GET /accounts/{id}    │
      │◄───────────────────────│
      │                        │
      │  { "balance": 100.00 } │
      │────────────────────────►
      │                        │
      
⚠️ Se Account mudar para { "saldo": 100.00 }
   Transfer quebra em PRODUÇÃO!
```

### Solução: Consumer-Driven Contracts

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONTRACT TESTING                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONSUMER (Transfer Service)                                    │
│  ┌──────────────────────────────────────────────┐               │
│  │ "Eu preciso de { balance: number }"          │               │
│  │ "Campo DEVE existir e ser numérico"          │               │
│  └──────────────────────────────────────────────┘               │
│           │                                                      │
│           │ Contrato                                            │
│           ▼                                                      │
│  PROVIDER (Account Service)                                     │
│  ┌──────────────────────────────────────────────┐               │
│  │ Build FALHA se contrato for violado          │               │
│  │ "Você vai quebrar o Transfer Service!"       │               │
│  └──────────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

### Spring Cloud Contract

**1. Consumer define o contrato (Groovy DSL):**

```groovy
// contracts/shouldReturnAccountBalance.groovy
Contract.make {
    description "Should return account balance"
    
    request {
        method GET()
        url "/api/v1/accounts/123"
        headers {
            contentType applicationJson()
        }
    }
    
    response {
        status OK()
        headers {
            contentType applicationJson()
        }
        body([
            id: 123,
            balance: 1000.00,
            status: "ACTIVE"
        ])
    }
}
```

**2. Provider gera testes automaticamente:**

```java
// Gerado automaticamente pelo Spring Cloud Contract
public class ContractVerifierTest extends AccountServiceBase {
    
    @Test
    public void validate_shouldReturnAccountBalance() {
        // Request
        given()
            .header("Content-Type", "application/json")
        .when()
            .get("/api/v1/accounts/123")
        .then()
            .statusCode(200)
            .body("id", equalTo(123))
            .body("balance", equalTo(1000.00))
            .body("status", equalTo("ACTIVE"));
    }
}
```

**3. Consumer usa stub gerado:**

```java
@SpringBootTest
@AutoConfigureStubRunner(
    stubsMode = StubRunnerProperties.StubsMode.LOCAL,
    ids = "com.pixsystem:account-service:+:stubs:8080"
)
class TransferServiceTest {
    
    @Autowired
    private AccountClient accountClient;
    
    @Test
    void deveObterSaldoDaConta() {
        // Stub gerado pelo contrato responde automaticamente
        AccountResponse response = accountClient.getBalance("123");
        
        assertThat(response.balance()).isEqualTo(new BigDecimal("1000.00"));
    }
}
```

### Dependências

```xml
<!-- Provider (Account Service) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <scope>test</scope>
</dependency>

<!-- Consumer (Transfer Service) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
    <scope>test</scope>
</dependency>
```

### Alternativa: Pact

```java
// Consumer side
@Pact(consumer = "transfer-service")
public RequestResponsePact accountBalancePact(PactDslWithProvider builder) {
    return builder
        .given("account 123 exists with balance 1000")
        .uponReceiving("a request for account balance")
            .path("/api/v1/accounts/123")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .integerType("id", 123)
                .decimalType("balance", 1000.00))
        .toPact();
}
```

---

## 🎯 Perguntas de Entrevista

1. **@Mock vs @MockBean?**
2. **O que são Test Slices?**
3. **Por que usar TestContainers?**
4. **Como testar APIs externas?**
5. **Qual a pirâmide de testes ideal?**
6. **O que é Mutation Testing? Por que usar?** 🆕
7. **Como garantir compatibilidade entre microsserviços?** 🆕
8. **Spring Cloud Contract vs Pact - quando usar cada um?** 🆕

---

> **Próximo módulo:** [Módulo 8 - DevOps](MODULO_08_DEVOPS.md)

