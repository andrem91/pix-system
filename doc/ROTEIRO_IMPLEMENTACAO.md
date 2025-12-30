# 🚀 Roteiro de Implementação: Sistema Pix (Teoria + Prática)

> **Abordagem:** Construir o sistema bancário do zero usando TDD, simulando um projeto real.

---

## 📌 Filosofia do Roteiro

Este roteiro segue a metodologia de **desenvolvimento profissional**:

1. **TDD:** Teste primeiro, código depois
2. **DDD:** Domínio no centro, infraestrutura nas bordas
3. **Incremental:** Comece simples, evolua com complexidade
4. **Real:** Mesmas decisões de um projeto bancário real

---

## 🗓️ Visão Geral das Sprints

```
Sprint 0  ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Setup & Value Objects
Sprint 1  ░░░░████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Account Service
Sprint 2  ░░░░░░░░████░░░░░░░░░░░░░░░░░░░░░░░░░░  Transfer Service
Sprint 3  ░░░░░░░░░░░░████░░░░░░░░░░░░░░░░░░░░░░  Mensageria & CQRS
Sprint 4  ░░░░░░░░░░░░░░░░████░░░░░░░░░░░░░░░░░░  Gateway & Discovery
Sprint 5  ░░░░░░░░░░░░░░░░░░░░████░░░░░░░░░░░░░░  DevOps
Sprint 6  ░░░░░░░░░░░░░░░░░░░░░░░░████░░░░░░░░░░  Pix Automático
Sprint 7  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░████░░░░░░  Pix Parcelado
Sprint 8  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░███░░░  ☸️ Kubernetes
Sprint 9  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░███  ☁️ AWS
Sprint 10 ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░██ 🏗️ Terraform
Sprint 11 ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░█ 🚀 Integração
```

| Sprint | Foco | Módulos Teóricos | Estimativa |
|--------|------|------------------|------------|
| 0 | Setup + DDD Básico | 3 (OOP/SOLID), Intro DDD | 3-4 dias |
| 1 | Account Service Completo | 4 (Spring), 5 (JPA), Hexagonal | 5-6 dias |
| 2 | Transfer + Event Sourcing | 1 (Streams), 6 (Arquitetura) | 5-6 dias |
| 3 | Kafka + CQRS | 6 (Kafka), 1 (Concorrência) | 4-5 dias |
| 4 | Gateway + Eureka | 6 (Microsserviços), 4 (Spring Cloud) | 3-4 dias |
| 5 | Docker + CI/CD + Tracing | 7 (Testes), 8 (DevOps) | 4-5 dias |
| 6 | 🆕 Pix Automático | State Machine, Scheduler | 4-5 dias |
| 7 | 🆕 Pix Parcelado | Strategy Pattern, Cálculos | 5-6 dias |
| 8 | ☸️ Kubernetes | 10 (Kubernetes) | 4-5 dias |
| 9 | ☁️ AWS + LocalStack | 9 (AWS Cloud) | 5-6 dias |
| 10 | 🏗️ Terraform | 11 (Terraform) | 4-5 dias |
| 11 | 🚀 Integração Final | Todos | 3-4 dias |

**Total Estimado:** 51-60 dias de estudo intensivo (~10-12 semanas)

---

## 🎯 Sprint 0: Setup & Fundamentos DDD

### Objetivo
Criar a estrutura do projeto e implementar os primeiros Value Objects usando TDD.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| OOP e Encapsulamento | `MODULO_03_OOP_SOLID.md` | 3.1, 3.2 |
| SOLID Principles | `MODULO_03_OOP_SOLID.md` | 3.3 |
| Value Objects | `MODULO_03_OOP_SOLID.md` | 3.5, 3.6 |
| Equals e HashCode | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.3 |
| Records | `MODULO_02_JAVA_MODERNO.md` | 2.1 |

**Leitura complementar (conceitos):**
- DDD: O que são Value Objects vs Entities
- Arquitetura Hexagonal: Conceito de Ports & Adapters
- TDD: Ciclo Red-Green-Refactor

### 🛠️ Implemente

#### Dia 1: Setup do Projeto

**1. Criar estrutura do monorepo:**
```bash
mkdir pix-system
cd pix-system

# Criar projeto parent (Maven)
# pom.xml com módulos
```

**2. Estrutura inicial:**
```
pix-system/
├── pix-account-service/
│   └── pom.xml
├── pix-commons/
│   └── pom.xml
├── pom.xml (parent)
└── docker-compose.yml
```

**3. Dependências do parent pom.xml:**
- Spring Boot 3.2+
- Java 21
- Lombok
- MapStruct

#### Dia 2-3: Value Objects com TDD

**Ordem de implementação (TDD):**

**1. Money (Value Object)**

```java
// ❌ RED: Escreva o teste PRIMEIRO
@Test
@DisplayName("Deve somar dois valores monetários")
void deveSomarDoisValores() {
    Money dez = Money.of(new BigDecimal("10.00"));
    Money cinco = Money.of(new BigDecimal("5.00"));
    
    Money resultado = dez.add(cinco);
    
    assertThat(resultado).isEqualTo(Money.of(new BigDecimal("15.00")));
}

@Test
@DisplayName("Não deve permitir valor negativo")
void naoDevePermitirValorNegativo() {
    assertThrows(IllegalArgumentException.class, 
        () -> Money.of(new BigDecimal("-10.00")));
}

@Test
@DisplayName("ZERO deve ser constante compartilhada")
void zeroDeveSerConstante() {
    assertThat(Money.ZERO.getAmount()).isEqualTo(BigDecimal.ZERO);
}
```

```java
// ✅ GREEN: Implemente o mínimo
public record Money(BigDecimal amount) {
    public static final Money ZERO = new Money(BigDecimal.ZERO);
    
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Valor não pode ser negativo");
        }
        amount = amount.setScale(2, RoundingMode.HALF_UP);
    }
    
    public static Money of(BigDecimal amount) {
        return new Money(amount);
    }
    
    public Money add(Money other) {
        return new Money(this.amount.add(other.amount));
    }
    
    public Money subtract(Money other) {
        return new Money(this.amount.subtract(other.amount));
    }
    
    public boolean isGreaterThan(Money other) {
        return this.amount.compareTo(other.amount) > 0;
    }
}
```

**2. Cpf (Value Object)**

```java
// Teste primeiro
@Test
@DisplayName("Deve validar CPF com 11 dígitos")
void deveValidarCpfCom11Digitos() {
    Cpf cpf = new Cpf("12345678901");
    assertThat(cpf.value()).isEqualTo("12345678901");
}

@Test
@DisplayName("Deve remover formatação do CPF")
void deveRemoverFormatacao() {
    Cpf cpf = new Cpf("123.456.789-01");
    assertThat(cpf.value()).isEqualTo("12345678901");
}

@Test
@DisplayName("Deve rejeitar CPF inválido")
void deveRejeitarCpfInvalido() {
    assertThrows(InvalidCpfException.class, () -> new Cpf("123"));
}
```

**3. AccountId (Value Object)**

```java
public record AccountId(UUID value) {
    public static AccountId generate() {
        return new AccountId(UUID.randomUUID());
    }
    
    public static AccountId of(String value) {
        return new AccountId(UUID.fromString(value));
    }
}
```

#### Dia 4: Entidades de Domínio

**1. PixKeyType (Enum)**
```java
public enum PixKeyType {
    CPF, CNPJ, EMAIL, PHONE, RANDOM
}
```

**2. Account (Aggregate Root) - TDD**

```java
// Teste primeiro
@Test
@DisplayName("Deve criar conta com saldo zero e status ativo")
void deveCriarContaComSaldoZero() {
    Account account = Account.create("João Silva", new Cpf("12345678901"));
    
    assertThat(account.getBalance()).isEqualTo(Money.ZERO);
    assertThat(account.getStatus()).isEqualTo(AccountStatus.ACTIVE);
    assertThat(account.getPixKeys()).isEmpty();
}

@Test
@DisplayName("Deve registrar chave Pix")
void deveRegistrarChavePix() {
    Account account = Account.create("João", new Cpf("12345678901"));
    
    PixKey key = account.registerPixKey(PixKeyType.EMAIL, "joao@email.com");
    
    assertThat(account.getPixKeys()).hasSize(1);
    assertThat(key.getType()).isEqualTo(PixKeyType.EMAIL);
}

@Test
@DisplayName("Não deve permitir mais de 5 chaves Pix")
void naoDevePermitirMaisDe5Chaves() {
    Account account = Account.create("João", new Cpf("12345678901"));
    
    // Registrar 5 chaves
    for (int i = 0; i < 5; i++) {
        account.registerPixKey(PixKeyType.RANDOM, UUID.randomUUID().toString());
    }
    
    assertThrows(MaxPixKeysExceededException.class, 
        () -> account.registerPixKey(PixKeyType.RANDOM, "nova-chave"));
}

@Test
@DisplayName("Deve debitar quando há saldo suficiente")
void deveDebitarComSaldoSuficiente() {
    Account account = Account.create("João", new Cpf("12345678901"));
    account.credit(Money.of(new BigDecimal("100.00")));
    
    account.debit(Money.of(new BigDecimal("30.00")));
    
    assertThat(account.getBalance()).isEqualTo(Money.of(new BigDecimal("70.00")));
}

@Test
@DisplayName("Não deve debitar sem saldo suficiente")
void naoDeveDebitarSemSaldo() {
    Account account = Account.create("João", new Cpf("12345678901"));
    
    assertThrows(InsufficientBalanceException.class, 
        () -> account.debit(Money.of(new BigDecimal("100.00"))));
}
```

```java
// Implementação
public class Account {
    private static final int MAX_PIX_KEYS = 5;
    
    private AccountId id;
    private String holderName;
    private Cpf cpf;
    private Money balance;
    private AccountStatus status;
    private Set<PixKey> pixKeys;
    private LocalDateTime createdAt;
    
    // Construtor privado - use factory method
    private Account(String holderName, Cpf cpf) {
        this.id = AccountId.generate();
        this.holderName = holderName;
        this.cpf = cpf;
        this.balance = Money.ZERO;
        this.status = AccountStatus.ACTIVE;
        this.pixKeys = new HashSet<>();
        this.createdAt = LocalDateTime.now();
    }
    
    // Factory method
    public static Account create(String holderName, Cpf cpf) {
        return new Account(holderName, cpf);
    }
    
    public PixKey registerPixKey(PixKeyType type, String keyValue) {
        if (pixKeys.size() >= MAX_PIX_KEYS) {
            throw new MaxPixKeysExceededException();
        }
        PixKey key = new PixKey(this, type, keyValue);
        pixKeys.add(key);
        return key;
    }
    
    public void debit(Money amount) {
        if (balance.isLessThan(amount)) {
            throw new InsufficientBalanceException(balance, amount);
        }
        this.balance = balance.subtract(amount);
    }
    
    public void credit(Money amount) {
        this.balance = balance.add(amount);
    }
    
    // Getters (sem setters - imutabilidade!)
}
```

### ✅ Critérios de Sucesso Sprint 0

- [ ] Projeto Maven multi-módulo criado
- [ ] Money, Cpf, AccountId implementados com TDD
- [ ] Account e PixKey com regras de negócio no domínio
- [ ] 100% dos testes passando
- [ ] Nenhuma dependência de framework no pacote `domain`

### 💡 Reflexão

> Por que as regras de negócio (máximo 5 chaves, saldo suficiente) estão na Entity e não em um Service?

---

## 🎯 Sprint 1: Account Service Completo (Hexagonal)

### Objetivo
Implementar o Account Service com arquitetura hexagonal, ports e adapters.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Spring DI | `MODULO_04_SPRING.md` | 4.1 Dependency Injection |
| @Transactional | `MODULO_04_SPRING.md` | 4.3 @Transactional |
| JPA Entities | `MODULO_05_JPA_PERSISTENCIA.md` | 5.1 Entity Lifecycle |
| N+1 Problem | `MODULO_05_JPA_PERSISTENCIA.md` | 5.4 Problema N+1 |
| Flyway | `MODULO_05_JPA_PERSISTENCIA.md` | 5.8 Flyway/Liquibase |

**Leitura complementar:**
- Arquitetura Hexagonal: Ports & Adapters em detalhe
- Por que separar Domain Entity de JPA Entity

### 🛠️ Implemente

#### Dia 1: Ports (Interfaces)

**1. Input Ports (Use Cases)**

```java
// application/port/in/CreateAccountUseCase.java
public interface CreateAccountUseCase {
    AccountId execute(CreateAccountCommand command);
}

// application/port/in/CreateAccountCommand.java
public record CreateAccountCommand(
    String holderName,
    String cpf
) {}
```

```java
// application/port/in/RegisterPixKeyUseCase.java
public interface RegisterPixKeyUseCase {
    PixKeyId execute(RegisterPixKeyCommand command);
}

public record RegisterPixKeyCommand(
    AccountId accountId,
    PixKeyType type,
    String keyValue
) {}
```

```java
// application/port/in/GetAccountUseCase.java
public interface GetAccountUseCase {
    Optional<Account> byId(AccountId id);
    Optional<Account> byPixKey(String pixKey);
}
```

**2. Output Ports (Repositories)**

```java
// application/port/out/AccountRepository.java
public interface AccountRepository {
    Account save(Account account);
    Optional<Account> findById(AccountId id);
    Optional<Account> findByPixKey(String pixKey);
    boolean existsByPixKey(String pixKey);
}
```

```java
// application/port/out/EventPublisher.java
public interface EventPublisher {
    void publish(DomainEvent event);
}
```

#### Dia 2: Application Services (Use Case Implementations)

```java
// application/service/CreateAccountService.java
@Service
@RequiredArgsConstructor
public class CreateAccountService implements CreateAccountUseCase {
    
    private final AccountRepository accountRepository;
    private final EventPublisher eventPublisher;
    
    @Override
    @Transactional
    public AccountId execute(CreateAccountCommand command) {
        Account account = Account.create(
            command.holderName(), 
            new Cpf(command.cpf())
        );
        
        Account saved = accountRepository.save(account);
        
        eventPublisher.publish(new AccountCreatedEvent(saved.getId()));
        
        return saved.getId();
    }
}
```

**Teste do Service (Mock dos Ports):**
```java
@ExtendWith(MockitoExtension.class)
class CreateAccountServiceTest {
    
    @Mock AccountRepository accountRepository;
    @Mock EventPublisher eventPublisher;
    @InjectMocks CreateAccountService service;
    
    @Test
    @DisplayName("Deve criar conta e publicar evento")
    void deveCriarContaEPublicarEvento() {
        // Given
        var command = new CreateAccountCommand("João", "12345678901");
        when(accountRepository.save(any())).thenAnswer(i -> i.getArgument(0));
        
        // When
        AccountId result = service.execute(command);
        
        // Then
        assertThat(result).isNotNull();
        verify(accountRepository).save(any(Account.class));
        verify(eventPublisher).publish(any(AccountCreatedEvent.class));
    }
}
```

#### Dia 3-4: Adapters de Saída (Persistence)

**1. JPA Entity (separada da Domain Entity!)**

```java
// adapter/out/persistence/entity/AccountJpaEntity.java
@Entity
@Table(name = "accounts")
@Getter @Setter
public class AccountJpaEntity {
    
    @Id
    private UUID id;
    
    @Column(name = "holder_name", nullable = false)
    private String holderName;
    
    @Column(nullable = false, unique = true)
    private String cpf;
    
    @Column(nullable = false, precision = 15, scale = 2)
    private BigDecimal balance;
    
    @Enumerated(EnumType.STRING)
    private AccountStatus status;
    
    @OneToMany(mappedBy = "account", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<PixKeyJpaEntity> pixKeys = new HashSet<>();
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    @Version
    private Long version;
}
```

**2. Mapper entre Domain e JPA**

```java
// adapter/out/persistence/mapper/AccountPersistenceMapper.java
@Component
public class AccountPersistenceMapper {
    
    public AccountJpaEntity toJpaEntity(Account domain) {
        var entity = new AccountJpaEntity();
        entity.setId(domain.getId().value());
        entity.setHolderName(domain.getHolderName());
        entity.setCpf(domain.getCpf().value());
        entity.setBalance(domain.getBalance().amount());
        entity.setStatus(domain.getStatus());
        // ... map pixKeys
        return entity;
    }
    
    public Account toDomain(AccountJpaEntity entity) {
        return Account.reconstitute(
            AccountId.of(entity.getId().toString()),
            entity.getHolderName(),
            new Cpf(entity.getCpf()),
            Money.of(entity.getBalance()),
            entity.getStatus(),
            mapPixKeys(entity.getPixKeys())
        );
    }
}
```

**3. Adapter implementando Port**

```java
// adapter/out/persistence/AccountRepositoryAdapter.java
@Repository
@RequiredArgsConstructor
public class AccountRepositoryAdapter implements AccountRepository {
    
    private final AccountJpaRepository jpaRepository;
    private final AccountPersistenceMapper mapper;
    
    @Override
    public Account save(Account account) {
        AccountJpaEntity entity = mapper.toJpaEntity(account);
        AccountJpaEntity saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }
    
    @Override
    public Optional<Account> findById(AccountId id) {
        return jpaRepository.findById(id.value())
            .map(mapper::toDomain);
    }
    
    @Override
    public Optional<Account> findByPixKey(String pixKey) {
        return jpaRepository.findByPixKeysKeyValue(pixKey)
            .map(mapper::toDomain);
    }
}
```

#### Dia 5: Adapters de Entrada (REST)

**1. Controller**

```java
// adapter/in/web/AccountController.java
@RestController
@RequestMapping("/api/v1/accounts")
@RequiredArgsConstructor
public class AccountController {
    
    private final CreateAccountUseCase createAccountUseCase;
    private final GetAccountUseCase getAccountUseCase;
    private final AccountDtoMapper mapper;
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public AccountResponse create(@Valid @RequestBody CreateAccountRequest request) {
        var command = new CreateAccountCommand(request.holderName(), request.cpf());
        AccountId id = createAccountUseCase.execute(command);
        
        Account account = getAccountUseCase.byId(id)
            .orElseThrow(() -> new IllegalStateException("Conta recém criada não encontrada"));
        
        return mapper.toResponse(account);
    }
    
    @GetMapping("/{id}")
    public AccountResponse getById(@PathVariable UUID id) {
        return getAccountUseCase.byId(AccountId.of(id.toString()))
            .map(mapper::toResponse)
            .orElseThrow(() -> new AccountNotFoundException(id));
    }
}
```

**2. DTOs**

```java
// adapter/in/web/dto/request/CreateAccountRequest.java
public record CreateAccountRequest(
    @NotBlank String holderName,
    @NotBlank @Pattern(regexp = "\\d{11}") String cpf
) {}

// adapter/in/web/dto/response/AccountResponse.java
public record AccountResponse(
    UUID id,
    String holderName,
    String cpf,
    BigDecimal balance,
    String status,
    List<PixKeyResponse> pixKeys,
    LocalDateTime createdAt
) {}
```

#### Dia 6: Flyway + Testes de Integração

**1. Migrations**

```sql
-- V1__create_accounts.sql
CREATE TABLE accounts (
    id UUID PRIMARY KEY,
    holder_name VARCHAR(255) NOT NULL,
    cpf VARCHAR(11) NOT NULL UNIQUE,
    balance DECIMAL(15, 2) NOT NULL DEFAULT 0,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP,
    version BIGINT NOT NULL DEFAULT 0
);

CREATE INDEX idx_accounts_cpf ON accounts(cpf);
```

```sql
-- V2__create_pix_keys.sql
CREATE TABLE pix_keys (
    id UUID PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(id),
    key_type VARCHAR(20) NOT NULL,
    key_value VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL,
    
    CONSTRAINT uk_pix_key_value UNIQUE (key_value)
);

CREATE INDEX idx_pix_keys_account ON pix_keys(account_id);
CREATE INDEX idx_pix_keys_value ON pix_keys(key_value);
```

**2. Teste de Integração**

```java
@SpringBootTest
@Testcontainers
class AccountIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    CreateAccountUseCase createAccountUseCase;
    
    @Autowired
    GetAccountUseCase getAccountUseCase;
    
    @Test
    void deveCriarERecuperarConta() {
        // Given
        var command = new CreateAccountCommand("Maria", "98765432100");
        
        // When
        AccountId id = createAccountUseCase.execute(command);
        
        // Then
        Optional<Account> found = getAccountUseCase.byId(id);
        assertThat(found).isPresent();
        assertThat(found.get().getHolderName()).isEqualTo("Maria");
    }
}
```

### ✅ Critérios de Sucesso Sprint 1

- [ ] Arquitetura hexagonal implementada (ports & adapters)
- [ ] Domain Entity separada de JPA Entity
- [ ] Mappers entre camadas
- [ ] CRUD completo funcionando
- [ ] Testes de controller com @WebMvcTest
- [ ] Testes de integração com TestContainers
- [ ] Flyway migrations aplicando

### 💡 Reflexão

> O que aconteceria se você usasse a JPA Entity diretamente no domínio? Quais problemas surgiram?

---

## 🎯 Sprint 2: Transfer Service + Event Sourcing

### Objetivo
Criar o segundo microsserviço com Event Sourcing para histórico de transferências.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Streams API | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.4 (completo) |
| Optional | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.5 |
| Feign Client | `MODULO_06_ARQUITETURA.md` | 6.2 Feign |
| Circuit Breaker | `MODULO_06_ARQUITETURA.md` | 6.3 Resilience4j |

**Leitura complementar:**
- Event Sourcing: conceitos e implementação
- Por que Event Sourcing em sistemas bancários

### 🛠️ Implemente

#### Dia 1: Domain Events

```java
// domain/event/TransferEvent.java (Base)
public abstract sealed class TransferEvent 
    permits TransferInitiatedEvent, TransferCompletedEvent, TransferFailedEvent {
    
    private final TransferId transferId;
    private final LocalDateTime occurredAt;
    private final int version;
    
    // Constructor, getters
}

// domain/event/TransferInitiatedEvent.java
public final class TransferInitiatedEvent extends TransferEvent {
    private final AccountId sourceAccountId;
    private final String destinationPixKey;
    private final Money amount;
    private final String description;
}

// domain/event/TransferCompletedEvent.java
public final class TransferCompletedEvent extends TransferEvent {
    private final AccountId destinationAccountId;
}

// domain/event/TransferFailedEvent.java
public final class TransferFailedEvent extends TransferEvent {
    private final String reason;
}
```

#### Dia 2: Transfer Aggregate com Event Sourcing

```java
// domain/model/Transfer.java
public class Transfer {
    private TransferId id;
    private AccountId sourceAccountId;
    private String destinationPixKey;
    private AccountId destinationAccountId;
    private Money amount;
    private String description;
    private TransferStatus status;
    private String failureReason;
    private LocalDateTime initiatedAt;
    private LocalDateTime completedAt;
    
    private final List<TransferEvent> uncommittedEvents = new ArrayList<>();
    
    // Private constructor - reconstituição via eventos
    private Transfer() {}
    
    // Factory method que gera evento
    public static Transfer initiate(
            AccountId sourceAccountId,
            String destinationPixKey, 
            Money amount,
            String description) {
        
        Transfer transfer = new Transfer();
        transfer.apply(new TransferInitiatedEvent(
            TransferId.generate(),
            sourceAccountId,
            destinationPixKey,
            amount,
            description,
            LocalDateTime.now(),
            1
        ));
        return transfer;
    }
    
    public void complete(AccountId destinationAccountId) {
        if (status != TransferStatus.PENDING) {
            throw new InvalidTransferStateException(status, TransferStatus.COMPLETED);
        }
        apply(new TransferCompletedEvent(id, destinationAccountId, LocalDateTime.now(), nextVersion()));
    }
    
    public void fail(String reason) {
        if (status != TransferStatus.PENDING) {
            throw new InvalidTransferStateException(status, TransferStatus.FAILED);
        }
        apply(new TransferFailedEvent(id, reason, LocalDateTime.now(), nextVersion()));
    }
    
    // Apply e mutate pattern (Event Sourcing)
    private void apply(TransferEvent event) {
        mutate(event);
        uncommittedEvents.add(event);
    }
    
    private void mutate(TransferEvent event) {
        switch (event) {
            case TransferInitiatedEvent e -> {
                this.id = e.getTransferId();
                this.sourceAccountId = e.getSourceAccountId();
                this.destinationPixKey = e.getDestinationPixKey();
                this.amount = e.getAmount();
                this.description = e.getDescription();
                this.status = TransferStatus.PENDING;
                this.initiatedAt = e.getOccurredAt();
            }
            case TransferCompletedEvent e -> {
                this.destinationAccountId = e.getDestinationAccountId();
                this.status = TransferStatus.COMPLETED;
                this.completedAt = e.getOccurredAt();
            }
            case TransferFailedEvent e -> {
                this.failureReason = e.getReason();
                this.status = TransferStatus.FAILED;
            }
        }
    }
    
    // Reconstituir do histórico de eventos
    public static Transfer reconstitute(List<TransferEvent> events) {
        Transfer transfer = new Transfer();
        events.forEach(transfer::mutate);
        return transfer;
    }
    
    public List<TransferEvent> getUncommittedEvents() {
        return Collections.unmodifiableList(uncommittedEvents);
    }
    
    public void markEventsAsCommitted() {
        uncommittedEvents.clear();
    }
}
```

#### Dia 3: Event Store

```java
// application/port/out/TransferEventStore.java
public interface TransferEventStore {
    void append(TransferId transferId, List<TransferEvent> events);
    List<TransferEvent> loadEvents(TransferId transferId);
    Optional<Transfer> load(TransferId transferId);
}

// adapter/out/persistence/eventstore/JpaTransferEventStore.java
@Repository
@RequiredArgsConstructor
public class JpaTransferEventStore implements TransferEventStore {
    
    private final TransferEventJpaRepository jpaRepository;
    private final ObjectMapper objectMapper;
    
    @Override
    @Transactional
    public void append(TransferId transferId, List<TransferEvent> events) {
        events.forEach(event -> {
            var entity = new TransferEventJpaEntity();
            entity.setId(UUID.randomUUID());
            entity.setTransferId(transferId.value());
            entity.setEventType(event.getClass().getSimpleName());
            entity.setPayload(serialize(event));
            entity.setVersion(event.getVersion());
            entity.setOccurredAt(event.getOccurredAt());
            jpaRepository.save(entity);
        });
    }
    
    @Override
    public Optional<Transfer> load(TransferId transferId) {
        List<TransferEvent> events = loadEvents(transferId);
        if (events.isEmpty()) {
            return Optional.empty();
        }
        return Optional.of(Transfer.reconstitute(events));
    }
}
```

#### Dia 4-5: Feign Client + Circuit Breaker

```java
// application/port/out/AccountClient.java
public interface AccountClient {
    Optional<AccountInfo> findByPixKey(String pixKey);
    void debit(AccountId accountId, Money amount, String description);
    void credit(AccountId accountId, Money amount, String description);
}

// adapter/out/feign/AccountFeignClient.java
@FeignClient(
    name = "pix-account-service",
    fallbackFactory = AccountClientFallbackFactory.class
)
public interface AccountFeignClient {
    
    @GetMapping("/api/v1/accounts")
    AccountResponse findByPixKey(@RequestParam("pixKey") String pixKey);
    
    @PostMapping("/api/v1/accounts/{id}/debit")
    void debit(@PathVariable UUID id, @RequestBody DebitRequest request);
    
    @PostMapping("/api/v1/accounts/{id}/credit")
    void credit(@PathVariable UUID id, @RequestBody CreditRequest request);
}

// adapter/out/feign/AccountClientAdapter.java
@Component
@RequiredArgsConstructor
public class AccountClientAdapter implements AccountClient {
    
    private final AccountFeignClient feignClient;
    
    @Override
    @CircuitBreaker(name = "accountService", fallbackMethod = "findByPixKeyFallback")
    @Retry(name = "accountService")
    public Optional<AccountInfo> findByPixKey(String pixKey) {
        try {
            AccountResponse response = feignClient.findByPixKey(pixKey);
            return Optional.of(toAccountInfo(response));
        } catch (FeignException.NotFound e) {
            return Optional.empty();
        }
    }
    
    private Optional<AccountInfo> findByPixKeyFallback(String pixKey, Exception e) {
        log.error("Fallback acionado para findByPixKey: {}", pixKey, e);
        throw new ServiceUnavailableException("Account Service indisponível");
    }
}
```

#### Dia 6: Use Case de Transferência

```java
// application/service/InitiateTransferService.java
@Service
@RequiredArgsConstructor
public class InitiateTransferService implements InitiateTransferUseCase {
    
    private final TransferEventStore eventStore;
    private final AccountClient accountClient;
    private final TransferEventPublisher eventPublisher;
    
    @Override
    @Transactional
    public TransferId execute(InitiateTransferCommand command) {
        // 1. Validar conta destino existe
        AccountInfo destinationAccount = accountClient.findByPixKey(command.destinationPixKey())
            .orElseThrow(() -> new PixKeyNotFoundException(command.destinationPixKey()));
        
        // 2. Criar transferência (gera evento)
        Transfer transfer = Transfer.initiate(
            command.sourceAccountId(),
            command.destinationPixKey(),
            command.amount(),
            command.description()
        );
        
        try {
            // 3. Debitar origem
            accountClient.debit(
                command.sourceAccountId(), 
                command.amount(),
                "PIX para " + command.destinationPixKey()
            );
            
            // 4. Creditar destino
            accountClient.credit(
                destinationAccount.accountId(),
                command.amount(),
                "PIX recebido de " + command.sourceAccountId()
            );
            
            // 5. Marcar como completada
            transfer.complete(destinationAccount.accountId());
            
        } catch (InsufficientBalanceException e) {
            transfer.fail("Saldo insuficiente");
        } catch (ServiceUnavailableException e) {
            transfer.fail("Serviço indisponível: " + e.getMessage());
        }
        
        // 6. Persistir eventos
        eventStore.append(transfer.getId(), transfer.getUncommittedEvents());
        
        // 7. Publicar para outros serviços
        transfer.getUncommittedEvents().forEach(eventPublisher::publish);
        
        transfer.markEventsAsCommitted();
        
        return transfer.getId();
    }
}
```

### ✅ Critérios de Sucesso Sprint 2

- [ ] Transfer Service criado com arquitetura hexagonal
- [ ] Event Sourcing implementado (append-only)
- [ ] Feign Client comunicando com Account Service
- [ ] Circuit Breaker funcionando (simule falha)
- [ ] Fluxo completo de transferência funcionando
- [ ] Histórico de eventos pode ser consultado

### 💡 Reflexão

> Se precisar reconstruir o estado de uma transferência, como você faria? Qual a vantagem do Event Sourcing aqui?

---

## 🎯 Sprint 3: Mensageria (Kafka) + CQRS

### Objetivo
Implementar comunicação assíncrona entre serviços e separar modelo de leitura/escrita.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Kafka | `MODULO_06_ARQUITETURA.md` | 6.4 Kafka |
| CompletableFuture | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.6 Concorrência |
| Collectors | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.4 Collectors Avançados |

### 🛠️ Implemente

#### Dia 1-2: Kafka Producer/Consumer

```java
// adapter/out/messaging/KafkaTransferEventPublisher.java
@Component
@RequiredArgsConstructor
public class KafkaTransferEventPublisher implements TransferEventPublisher {
    
    private final KafkaTemplate<String, TransferEvent> kafkaTemplate;
    
    @Override
    public void publish(TransferEvent event) {
        kafkaTemplate.send("transfer-events", event.getTransferId().toString(), event);
    }
}
```

```java
// adapter/in/messaging/TransferEventConsumer.java (no Notification Service)
@Component
@RequiredArgsConstructor
public class TransferEventConsumer {
    
    private final NotificationService notificationService;
    
    @KafkaListener(topics = "transfer-events", groupId = "notification-service")
    public void consume(TransferEvent event) {
        switch (event) {
            case TransferCompletedEvent e -> 
                notificationService.sendTransferConfirmation(e);
            case TransferFailedEvent e -> 
                notificationService.sendTransferFailure(e);
            default -> log.debug("Evento ignorado: {}", event.getClass());
        }
    }
}
```

#### Dia 3-4: CQRS - Read Model (Projeção)

```java
// adapter/out/persistence/projection/TransferProjection.java
@Entity
@Table(name = "transfer_projections")
public class TransferProjection {
    @Id
    private UUID id;
    private UUID sourceAccountId;
    private String sourceAccountName;  // Desnormalizado!
    private UUID destinationAccountId;
    private String destinationAccountName;  // Desnormalizado!
    private String destinationPixKey;
    private BigDecimal amount;
    private String status;
    private String failureReason;
    private LocalDateTime initiatedAt;
    private LocalDateTime completedAt;
}

// adapter/in/messaging/TransferProjectionUpdater.java
@Component
@RequiredArgsConstructor
public class TransferProjectionUpdater {
    
    private final TransferProjectionRepository projectionRepository;
    private final AccountClient accountClient;
    
    @KafkaListener(topics = "transfer-events", groupId = "transfer-projection")
    @Transactional
    public void updateProjection(TransferEvent event) {
        switch (event) {
            case TransferInitiatedEvent e -> createProjection(e);
            case TransferCompletedEvent e -> markCompleted(e);
            case TransferFailedEvent e -> markFailed(e);
        }
    }
    
    private void createProjection(TransferInitiatedEvent event) {
        // Enriquecer com nomes das contas
        AccountInfo source = accountClient.findById(event.getSourceAccountId()).orElseThrow();
        
        var projection = new TransferProjection();
        projection.setId(event.getTransferId().value());
        projection.setSourceAccountId(event.getSourceAccountId().value());
        projection.setSourceAccountName(source.holderName());
        projection.setAmount(event.getAmount().amount());
        projection.setStatus("PENDING");
        projection.setInitiatedAt(event.getOccurredAt());
        
        projectionRepository.save(projection);
    }
}
```

#### Dia 5: Query Service (Leitura)

```java
// application/service/TransferQueryService.java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class TransferQueryService implements GetTransferUseCase {
    
    private final TransferProjectionRepository projectionRepository;
    
    @Override
    public Optional<TransferDetails> findById(TransferId id) {
        return projectionRepository.findById(id.value())
            .map(this::toDetails);
    }
    
    @Override
    public Page<TransferSummary> findByAccount(AccountId accountId, Pageable pageable) {
        return projectionRepository
            .findBySourceAccountIdOrDestinationAccountId(
                accountId.value(), 
                accountId.value(), 
                pageable
            )
            .map(this::toSummary);
    }
    
    @Override
    public TransferStatistics getStatistics(AccountId accountId, LocalDate from, LocalDate to) {
        List<TransferProjection> transfers = projectionRepository
            .findByAccountAndPeriod(accountId.value(), from, to);
        
        return transfers.stream()
            .collect(Collectors.teeing(
                // Total enviado
                Collectors.filtering(
                    t -> t.getSourceAccountId().equals(accountId.value()),
                    Collectors.reducing(BigDecimal.ZERO, 
                        TransferProjection::getAmount, BigDecimal::add)
                ),
                // Total recebido
                Collectors.filtering(
                    t -> t.getDestinationAccountId().equals(accountId.value()),
                    Collectors.reducing(BigDecimal.ZERO,
                        TransferProjection::getAmount, BigDecimal::add)
                ),
                (sent, received) -> new TransferStatistics(sent, received)
            ));
    }
}
```

### ✅ Critérios de Sucesso Sprint 3

- [ ] Kafka producer publicando eventos
- [ ] Consumers processando eventos
- [ ] Projeção (read model) sendo atualizada
- [ ] Consultas usando projeção (não event store)
- [ ] Idempotência no consumer (verificar se já processou)
- [ ] Notification Service básico funcionando

### 💡 Reflexão

> Qual a vantagem de ter um modelo de leitura separado? O que acontece se o consumer ficar fora do ar por 1 hora?

---

## 🎯 Sprint 4: API Gateway + Service Discovery

### Objetivo
Implementar infraestrutura de microsserviços com Gateway e registro de serviços.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Spring Cloud Gateway | `MODULO_06_ARQUITETURA.md` | 6.10 API Gateway |
| Microsserviços | `MODULO_06_ARQUITETURA.md` | 6.1 |

### 🛠️ Implemente

#### Dia 1-2: Eureka Server

```java
// pix-service-discovery/src/main/java/.../EurekaServerApplication.java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

**Registrar serviços:**
```yaml
# account-service application.yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

#### Dia 3-4: API Gateway

```java
// pix-api-gateway/src/main/java/.../ApiGatewayApplication.java
@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: account-service
          uri: lb://pix-account-service
          predicates:
            - Path=/api/v1/accounts/**
          filters:
            - name: CircuitBreaker
              args:
                name: accountCircuitBreaker
                fallbackUri: forward:/fallback/accounts
                
        - id: transfer-service
          uri: lb://pix-transfer-service
          predicates:
            - Path=/api/v1/transfers/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### ✅ Critérios de Sucesso Sprint 4

- [ ] Eureka Server rodando
- [ ] Todos os serviços registrados
- [ ] API Gateway roteando requisições
- [ ] Load balancing funcionando (suba 2 instâncias)
- [ ] Rate limiting configurado
- [ ] Circuit breaker no gateway

---

## 🎯 Sprint 5: DevOps + Observabilidade

### Objetivo
Containerizar o sistema e adicionar observabilidade.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Docker | `MODULO_08_DEVOPS.md` | 8.1, 8.2 |
| CI/CD | `MODULO_08_DEVOPS.md` | 8.3 |
| Observabilidade | `MODULO_08_DEVOPS.md` | 8.4 |
| DevSecOps | `MODULO_08_DEVOPS.md` | 8.6 🆕 |
| Testes | `MODULO_07_TESTES.md` | Completo |

### 🛠️ Implemente

#### Dia 1-2: Docker Compose Completo

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: pix
      POSTGRES_USER: pix
      POSTGRES_PASSWORD: pix123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
  
  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  jaeger:
    image: jaegertracing/all-in-one:1.50
    ports:
      - "16686:16686"  # UI
      - "4317:4317"    # OTLP gRPC
  
  eureka:
    build: ./pix-service-discovery
    ports:
      - "8761:8761"
  
  account-service:
    build: ./pix-account-service
    depends_on:
      - postgres
      - kafka
      - eureka
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/pix
  
  transfer-service:
    build: ./pix-transfer-service
    depends_on:
      - postgres
      - kafka
      - eureka
      - account-service
    environment:
      SPRING_PROFILES_ACTIVE: docker
  
  api-gateway:
    build: ./pix-api-gateway
    ports:
      - "8080:8080"
    depends_on:
      - eureka
      - account-service
      - transfer-service

volumes:
  postgres_data:
```

#### Dia 3: Distributed Tracing

```yaml
# application.yml (cada serviço)
management:
  tracing:
    sampling:
      probability: 1.0
  otlp:
    tracing:
      endpoint: http://jaeger:4317
```

#### Dia 4-5: GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      
      - name: Cache Maven packages
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
      
      - name: Build and Test
        run: mvn clean verify
      
      - name: Build Docker Images
        run: |
          docker build -t pix-account-service ./pix-account-service
          docker build -t pix-transfer-service ./pix-transfer-service
```

#### Dia 3-4: 🔒 DevSecOps

**SonarCloud (Análise de Qualidade)**

```yaml
# .github/workflows/sonar.yml
name: SonarCloud Analysis

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better analysis
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      
      - name: Cache SonarCloud packages
        uses: actions/cache@v3
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
      
      - name: Build and Analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: |
          mvn clean verify sonar:sonar \
            -Dsonar.projectKey=seu-usuario_pix-system \
            -Dsonar.organization=seu-usuario \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.coverage.jacoco.xmlReportPaths=**/target/site/jacoco/jacoco.xml
```

**JaCoCo (Cobertura de Testes)**

```xml
<!-- pom.xml - Adicionar plugin JaCoCo -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum> <!-- 80% mínimo -->
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**Trivy (Scan de Vulnerabilidades em Containers)**

```yaml
# Adicionar ao ci.yml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'pix-account-service:latest'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'

- name: Upload Trivy scan results
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

**Dependabot (Atualizações Automáticas)**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
      - "security"
    
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

**Badges para README**

```markdown
<!-- Adicionar no README.md do projeto -->
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=seu-usuario_pix-system&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=seu-usuario_pix-system)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=seu-usuario_pix-system&metric=coverage)](https://sonarcloud.io/summary/new_code?id=seu-usuario_pix-system)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=seu-usuario_pix-system&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=seu-usuario_pix-system)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=seu-usuario_pix-system&metric=sqale_rating)](https://sonarcloud.io/summary/new_code?id=seu-usuario_pix-system)
```

### ✅ Critérios de Sucesso Sprint 5

**DevOps:**
- [ ] `docker-compose up` sobe todo o sistema
- [ ] Jaeger mostrando traces entre serviços
- [ ] Health checks funcionando `/actuator/health`
- [ ] Métricas Prometheus disponíveis `/actuator/prometheus`
- [ ] Documentação OpenAPI gerada `/swagger-ui.html`

**DevSecOps:**
- [ ] SonarCloud integrado ao repositório
- [ ] Quality Gate passando (sem bugs críticos)
- [ ] Cobertura de testes ≥ 80% (JaCoCo)
- [ ] Trivy scan sem vulnerabilidades CRITICAL
- [ ] Dependabot configurado e funcionando
- [ ] Badges no README do projeto

**CI/CD:**
- [ ] GitHub Actions rodando em cada PR
- [ ] Pipeline: Build → Test → SonarCloud → Docker → Trivy

---

## 🎯 Sprint 6: 🆕 Pix Automático (Recurring Service)

### Objetivo
Implementar o Pix Automático (lançado em 2025) com mandatos de autorização e execuções agendadas.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Concorrência | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.6 Concorrência |
| Design Patterns | `MODULO_03_OOP_SOLID.md` | 3.7 State Pattern |
| Spring Cloud | `MODULO_04_SPRING.md` | Scheduling |

**Leitura complementar:**
- State Machine: Padrão para ciclo de vida de entidades
- Idempotência: Como garantir execução única
- Scheduling: @Scheduled vs Quartz

### 🛠️ Implemente

#### Dia 1: Domain Model

**1. RecurringMandate (Aggregate Root)**

```java
// domain/model/RecurringMandate.java
public class RecurringMandate {
    private MandateId id;
    private AccountId payerAccountId;
    private String receiverPixKey;
    private String receiverName;
    private Money maxAmount;
    private RecurrenceType recurrenceType;  // WEEKLY, MONTHLY
    private Integer dayOfRecurrence;        // Dia do mês (1-28)
    private LocalDate startDate;
    private LocalDate endDate;              // Opcional
    private MandateStatus status;
    private List<RecurringExecution> executions;
    
    // State Machine - Transições válidas
    private static final Map<MandateStatus, Set<MandateStatus>> VALID_TRANSITIONS = Map.of(
        MandateStatus.PENDING, Set.of(MandateStatus.ACTIVE, MandateStatus.CANCELLED),
        MandateStatus.ACTIVE, Set.of(MandateStatus.SUSPENDED, MandateStatus.CANCELLED),
        MandateStatus.SUSPENDED, Set.of(MandateStatus.ACTIVE, MandateStatus.CANCELLED),
        MandateStatus.CANCELLED, Set.of()  // Estado final
    );
    
    public void confirm() {
        transition(MandateStatus.ACTIVE);
    }
    
    public void suspend() {
        transition(MandateStatus.SUSPENDED);
    }
    
    public void reactivate() {
        transition(MandateStatus.ACTIVE);
    }
    
    public void cancel() {
        transition(MandateStatus.CANCELLED);
    }
    
    private void transition(MandateStatus newStatus) {
        if (!VALID_TRANSITIONS.get(status).contains(newStatus)) {
            throw new InvalidMandateTransitionException(status, newStatus);
        }
        this.status = newStatus;
    }
    
    public boolean isScheduledFor(LocalDate date) {
        if (status != MandateStatus.ACTIVE) return false;
        if (date.isBefore(startDate)) return false;
        if (endDate != null && date.isAfter(endDate)) return false;
        
        return switch (recurrenceType) {
            case MONTHLY -> date.getDayOfMonth() == dayOfRecurrence;
            case WEEKLY -> date.getDayOfWeek().getValue() == dayOfRecurrence;
        };
    }
    
    public boolean hasExecutionFor(LocalDate date) {
        return executions.stream()
            .anyMatch(e -> e.getScheduledDate().equals(date));
    }
    
    public RecurringExecution recordExecution(LocalDate date, TransferId transferId) {
        var execution = RecurringExecution.success(id, date, maxAmount, transferId);
        executions.add(execution);
        return execution;
    }
    
    public RecurringExecution recordFailedExecution(LocalDate date, String reason) {
        var execution = RecurringExecution.failed(id, date, reason);
        executions.add(execution);
        return execution;
    }
}
```

**2. RecurringExecution (Entity)**

```java
public class RecurringExecution {
    private ExecutionId id;
    private MandateId mandateId;
    private LocalDate scheduledDate;
    private Money amount;
    private ExecutionStatus status;  // SCHEDULED, EXECUTED, FAILED, SKIPPED
    private TransferId transferId;
    private String failureReason;
    private LocalDateTime executedAt;
    
    public static RecurringExecution success(MandateId mandateId, LocalDate date, 
                                              Money amount, TransferId transferId) {
        var execution = new RecurringExecution();
        execution.id = ExecutionId.generate();
        execution.mandateId = mandateId;
        execution.scheduledDate = date;
        execution.amount = amount;
        execution.status = ExecutionStatus.EXECUTED;
        execution.transferId = transferId;
        execution.executedAt = LocalDateTime.now();
        return execution;
    }
    
    public static RecurringExecution failed(MandateId mandateId, LocalDate date, String reason) {
        var execution = new RecurringExecution();
        execution.id = ExecutionId.generate();
        execution.mandateId = mandateId;
        execution.scheduledDate = date;
        execution.status = ExecutionStatus.FAILED;
        execution.failureReason = reason;
        execution.executedAt = LocalDateTime.now();
        return execution;
    }
}
```

#### Dia 2: Ports & Use Cases

```java
// application/port/in/CreateMandateUseCase.java
public interface CreateMandateUseCase {
    MandateId execute(CreateMandateCommand command);
}

public record CreateMandateCommand(
    AccountId payerAccountId,
    String receiverPixKey,
    Money maxAmount,
    RecurrenceType recurrenceType,
    Integer dayOfRecurrence,
    LocalDate startDate,
    LocalDate endDate
) {}

// application/port/in/ManageMandateUseCase.java
public interface ManageMandateUseCase {
    void confirm(MandateId id);
    void suspend(MandateId id);
    void reactivate(MandateId id);
    void cancel(MandateId id);
}

// application/port/out/MandateRepository.java
public interface MandateRepository {
    RecurringMandate save(RecurringMandate mandate);
    Optional<RecurringMandate> findById(MandateId id);
    List<RecurringMandate> findActiveByScheduledDate(LocalDate date);
    List<RecurringMandate> findByPayerAccountId(AccountId accountId);
}
```

#### Dia 3: Scheduler

```java
// application/service/RecurringExecutionScheduler.java
@Service
@RequiredArgsConstructor
@Slf4j
public class RecurringExecutionScheduler {
    
    private final MandateRepository mandateRepository;
    private final TransferService transferService;
    private final NotificationService notificationService;
    private final TransactionTemplate transactionTemplate;
    
    // Executa todo dia às 6h
    @Scheduled(cron = "0 0 6 * * *")
    public void executeScheduledPayments() {
        LocalDate today = LocalDate.now();
        log.info("Iniciando execução de Pix Automático para {}", today);
        
        List<RecurringMandate> mandates = mandateRepository
            .findActiveByScheduledDate(today);
        
        log.info("Encontrados {} mandatos para execução", mandates.size());
        
        mandates.forEach(this::executeWithTransaction);
    }
    
    // Notifica 2 dias antes
    @Scheduled(cron = "0 0 10 * * *")
    public void notifyUpcomingPayments() {
        LocalDate twoDaysFromNow = LocalDate.now().plusDays(2);
        
        List<RecurringMandate> upcoming = mandateRepository
            .findActiveByScheduledDate(twoDaysFromNow);
        
        upcoming.forEach(m -> 
            notificationService.sendUpcomingPaymentNotification(m));
    }
    
    private void executeWithTransaction(RecurringMandate mandate) {
        transactionTemplate.execute(status -> {
            try {
                executeMandate(mandate);
            } catch (Exception e) {
                log.error("Erro ao executar mandato {}", mandate.getId(), e);
                status.setRollbackOnly();
            }
            return null;
        });
    }
    
    private void executeMandate(RecurringMandate mandate) {
        LocalDate today = LocalDate.now();
        
        // IDEMPOTÊNCIA: verificar se já executou hoje
        if (mandate.hasExecutionFor(today)) {
            log.info("Mandato {} já executado em {}", mandate.getId(), today);
            return;
        }
        
        try {
            TransferId transferId = transferService.initiateTransfer(
                new TransferCommand(
                    mandate.getPayerAccountId(),
                    mandate.getReceiverPixKey(),
                    mandate.getMaxAmount(),
                    "Pix Automático - " + mandate.getReceiverName()
                )
            );
            
            mandate.recordExecution(today, transferId);
            mandateRepository.save(mandate);
            
            notificationService.sendExecutionSuccess(mandate);
            
        } catch (InsufficientBalanceException e) {
            mandate.recordFailedExecution(today, "Saldo insuficiente");
            mandateRepository.save(mandate);
            
            notificationService.sendExecutionFailed(mandate, e.getMessage());
        }
    }
}
```

#### Dia 4: Controller & Integration

```java
// adapter/in/web/MandateController.java
@RestController
@RequestMapping("/api/v1/recurring/mandates")
@RequiredArgsConstructor
public class MandateController {
    
    private final CreateMandateUseCase createMandateUseCase;
    private final ManageMandateUseCase manageMandateUseCase;
    private final GetMandateUseCase getMandateUseCase;
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public MandateResponse create(@Valid @RequestBody CreateMandateRequest request) {
        var command = mapper.toCommand(request);
        MandateId id = createMandateUseCase.execute(command);
        return getMandateUseCase.byId(id).map(mapper::toResponse).orElseThrow();
    }
    
    @PutMapping("/{id}/confirm")
    public void confirm(@PathVariable UUID id) {
        manageMandateUseCase.confirm(MandateId.of(id));
    }
    
    @PutMapping("/{id}/suspend")
    public void suspend(@PathVariable UUID id) {
        manageMandateUseCase.suspend(MandateId.of(id));
    }
    
    @PutMapping("/{id}/reactivate")
    public void reactivate(@PathVariable UUID id) {
        manageMandateUseCase.reactivate(MandateId.of(id));
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void cancel(@PathVariable UUID id) {
        manageMandateUseCase.cancel(MandateId.of(id));
    }
    
    @GetMapping("/{id}/executions")
    public List<ExecutionResponse> getExecutions(@PathVariable UUID id) {
        return getMandateUseCase.getExecutions(MandateId.of(id))
            .stream()
            .map(mapper::toResponse)
            .toList();
    }
}
```

### ✅ Critérios de Sucesso Sprint 6

- [ ] Recurring Service criado com arquitetura hexagonal
- [ ] RecurringMandate com State Machine funcionando
- [ ] Scheduler executando diariamente
- [ ] Idempotência: não debita 2x no mesmo dia
- [ ] Notificação T-2 implementada
- [ ] Testes do scheduler com mocks
- [ ] Integração com Transfer Service

### 💡 Reflexão

> O que acontece se o scheduler estiver fora do ar por um dia? Como recuperar as execuções perdidas?

---

## 🎯 Sprint 7: 🆕 Pix Parcelado (Installment Service)

### Objetivo
Implementar o Pix Parcelado (lançado em 2025) com simulação, contratação e gestão de parcelas.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Design Patterns | `MODULO_03_OOP_SOLID.md` | 3.7 Strategy Pattern |
| Streams | `MODULO_01_FUNDAMENTOS_JAVA.md` | 1.4 Streams |
| BigDecimal | `MODULO_01_FUNDAMENTOS_JAVA.md` | Precisão decimal |

**Leitura complementar:**
- Matemática Financeira: Fórmula Price, CET, IOF
- Strategy Pattern: Diferentes políticas de taxas
- Domain Services: Lógica de negócio complexa

### 🛠️ Implemente

#### Dia 1: Value Objects e Domain Services

**1. Cálculos Financeiros (Value Objects)**

```java
// domain/vo/InterestRate.java
public record InterestRate(BigDecimal monthlyRate) {
    public InterestRate {
        if (monthlyRate.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Taxa não pode ser negativa");
        }
    }
    
    public BigDecimal getAnnualRate() {
        // (1 + i)^12 - 1
        return BigDecimal.ONE.add(monthlyRate)
            .pow(12)
            .subtract(BigDecimal.ONE)
            .setScale(4, RoundingMode.HALF_UP);
    }
}

// domain/vo/InstallmentSimulation.java
public record InstallmentSimulation(
    Money principalAmount,
    Money totalAmount,
    Money monthlyAmount,
    Money interestAmount,
    Money iofAmount,
    BigDecimal effectiveRate,  // CET
    int numberOfInstallments,
    List<InstallmentDetail> installments
) {
    public Money getTotalCost() {
        return totalAmount.subtract(principalAmount);
    }
}

public record InstallmentDetail(
    int number,
    LocalDate dueDate,
    Money amount,
    Money principal,
    Money interest
) {}
```

**2. Calculadora Financeira (Domain Service)**

```java
// domain/service/InstallmentCalculator.java
@Service
public class InstallmentCalculator {
    
    // IOF conforme regulação do Banco Central
    private static final BigDecimal IOF_DAILY_RATE = new BigDecimal("0.000082");
    private static final BigDecimal IOF_FIXED_RATE = new BigDecimal("0.0038");
    
    public InstallmentSimulation simulate(
            Money principalAmount, 
            int numberOfInstallments,
            InterestRate interestRate) {
        
        BigDecimal monthlyRate = interestRate.monthlyRate();
        
        // Fórmula Price: PMT = PV * [i * (1+i)^n] / [(1+i)^n - 1]
        BigDecimal pmt = calculatePMT(
            principalAmount.amount(), 
            monthlyRate, 
            numberOfInstallments
        );
        
        Money monthlyAmount = Money.of(pmt);
        Money totalPayments = monthlyAmount.multiply(numberOfInstallments);
        Money interestAmount = totalPayments.subtract(principalAmount);
        
        // IOF
        Money iofAmount = calculateIOF(principalAmount, numberOfInstallments);
        Money totalAmount = totalPayments.add(iofAmount);
        
        // Gerar detalhes das parcelas
        List<InstallmentDetail> installments = generateInstallmentDetails(
            principalAmount, monthlyAmount, monthlyRate, numberOfInstallments);
        
        // CET (Custo Efetivo Total)
        BigDecimal effectiveRate = calculateCET(
            principalAmount, totalAmount, numberOfInstallments);
        
        return new InstallmentSimulation(
            principalAmount,
            totalAmount,
            monthlyAmount,
            interestAmount,
            iofAmount,
            effectiveRate,
            numberOfInstallments,
            installments
        );
    }
    
    // Fórmula Price
    private BigDecimal calculatePMT(BigDecimal pv, BigDecimal i, int n) {
        if (i.compareTo(BigDecimal.ZERO) == 0) {
            return pv.divide(BigDecimal.valueOf(n), 2, RoundingMode.HALF_UP);
        }
        
        BigDecimal onePlusI = BigDecimal.ONE.add(i);
        BigDecimal onePlusIPowerN = onePlusI.pow(n);
        
        BigDecimal numerator = i.multiply(onePlusIPowerN);
        BigDecimal denominator = onePlusIPowerN.subtract(BigDecimal.ONE);
        
        return pv.multiply(numerator)
                 .divide(denominator, 10, RoundingMode.HALF_UP)
                 .setScale(2, RoundingMode.HALF_UP);
    }
    
    private Money calculateIOF(Money principal, int installments) {
        // IOF = Fixo (0.38%) + Diário (0.0082% por dia)
        int avgDays = installments * 15; // Média aproximada
        
        BigDecimal dailyIof = principal.amount()
            .multiply(IOF_DAILY_RATE)
            .multiply(BigDecimal.valueOf(avgDays));
        
        BigDecimal fixedIof = principal.amount().multiply(IOF_FIXED_RATE);
        
        return Money.of(dailyIof.add(fixedIof).setScale(2, RoundingMode.HALF_UP));
    }
    
    private List<InstallmentDetail> generateInstallmentDetails(
            Money principal, Money pmt, BigDecimal rate, int n) {
        
        List<InstallmentDetail> details = new ArrayList<>();
        BigDecimal balance = principal.amount();
        LocalDate dueDate = LocalDate.now().plusMonths(1);
        
        for (int i = 1; i <= n; i++) {
            BigDecimal interest = balance.multiply(rate).setScale(2, RoundingMode.HALF_UP);
            BigDecimal principalPart = pmt.amount().subtract(interest);
            balance = balance.subtract(principalPart);
            
            details.add(new InstallmentDetail(
                i,
                dueDate,
                pmt,
                Money.of(principalPart),
                Money.of(interest)
            ));
            
            dueDate = dueDate.plusMonths(1);
        }
        
        return details;
    }
}
```

#### Dia 2: Strategy Pattern para Taxas

```java
// domain/service/InterestRateStrategy.java
public interface InterestRateStrategy {
    InterestRate calculateRate(CreditProfile profile, int installments);
    String getName();
}

// domain/service/StandardRateStrategy.java
@Component
public class StandardRateStrategy implements InterestRateStrategy {
    
    private static final BigDecimal BASE_RATE = new BigDecimal("0.0199"); // 1.99% a.m.
    
    @Override
    public InterestRate calculateRate(CreditProfile profile, int installments) {
        BigDecimal riskSpread = calculateRiskSpread(profile);
        BigDecimal termSpread = calculateTermSpread(installments);
        
        return new InterestRate(BASE_RATE.add(riskSpread).add(termSpread));
    }
    
    private BigDecimal calculateRiskSpread(CreditProfile profile) {
        if (profile.score() >= 800) return BigDecimal.ZERO;
        if (profile.score() >= 700) return new BigDecimal("0.003");
        if (profile.score() >= 600) return new BigDecimal("0.006");
        return new BigDecimal("0.010");
    }
    
    private BigDecimal calculateTermSpread(int installments) {
        if (installments <= 3) return BigDecimal.ZERO;
        if (installments <= 6) return new BigDecimal("0.001");
        return new BigDecimal("0.002");
    }
    
    @Override
    public String getName() {
        return "STANDARD";
    }
}

// domain/service/PromotionalRateStrategy.java
@Component
public class PromotionalRateStrategy implements InterestRateStrategy {
    
    @Override
    public InterestRate calculateRate(CreditProfile profile, int installments) {
        // Taxa promocional para clientes premium
        if (profile.score() >= 750 && installments <= 6 && profile.isOldClient()) {
            return new InterestRate(new BigDecimal("0.0149")); // 1.49% a.m.
        }
        
        // Fallback para taxa padrão
        return new StandardRateStrategy().calculateRate(profile, installments);
    }
    
    @Override
    public String getName() {
        return "PROMOTIONAL";
    }
}

// Selector de estratégia
@Component
@RequiredArgsConstructor
public class InterestRateStrategySelector {
    
    private final List<InterestRateStrategy> strategies;
    
    public InterestRateStrategy selectBestFor(CreditProfile profile) {
        // Tenta promocional primeiro, depois padrão
        return strategies.stream()
            .filter(s -> s.getName().equals("PROMOTIONAL"))
            .findFirst()
            .filter(s -> isEligibleForPromotion(profile))
            .orElseGet(() -> strategies.stream()
                .filter(s -> s.getName().equals("STANDARD"))
                .findFirst()
                .orElseThrow());
    }
    
    private boolean isEligibleForPromotion(CreditProfile profile) {
        return profile.score() >= 750 && profile.isOldClient();
    }
}
```

#### Dia 3: Aggregates

```java
// domain/model/InstallmentContract.java
public class InstallmentContract {
    private ContractId id;
    private AccountId payerAccountId;
    private String receiverPixKey;
    private Money totalAmount;
    private Money principalAmount;
    private Money interestAmount;
    private Money iofAmount;
    private int numberOfInstallments;
    private InterestRate interestRate;
    private ContractStatus status;  // PENDING, ACTIVE, COMPLETED, DEFAULTED
    private List<Installment> installments;
    private LocalDate contractDate;
    
    public static InstallmentContract create(
            AccountId payerAccountId,
            String receiverPixKey,
            InstallmentSimulation simulation,
            InterestRate rate) {
        
        var contract = new InstallmentContract();
        contract.id = ContractId.generate();
        contract.payerAccountId = payerAccountId;
        contract.receiverPixKey = receiverPixKey;
        contract.totalAmount = simulation.totalAmount();
        contract.principalAmount = simulation.principalAmount();
        contract.interestAmount = simulation.interestAmount();
        contract.iofAmount = simulation.iofAmount();
        contract.numberOfInstallments = simulation.numberOfInstallments();
        contract.interestRate = rate;
        contract.status = ContractStatus.PENDING;
        contract.contractDate = LocalDate.now();
        
        contract.installments = simulation.installments().stream()
            .map(detail -> Installment.fromDetail(contract.id, detail))
            .toList();
        
        return contract;
    }
    
    public void activate(TransferId initialTransferId) {
        if (status != ContractStatus.PENDING) {
            throw new InvalidContractStateException(status, ContractStatus.ACTIVE);
        }
        this.status = ContractStatus.ACTIVE;
        // Registrar transferência inicial para o recebedor
    }
    
    public void payInstallment(int installmentNumber, TransferId transferId) {
        Installment installment = installments.stream()
            .filter(i -> i.getNumber() == installmentNumber)
            .findFirst()
            .orElseThrow(() -> new InstallmentNotFoundException(installmentNumber));
        
        installment.pay(transferId);
        
        checkCompletion();
    }
    
    private void checkCompletion() {
        boolean allPaid = installments.stream()
            .allMatch(i -> i.getStatus() == InstallmentStatus.PAID);
        
        if (allPaid) {
            this.status = ContractStatus.COMPLETED;
        }
    }
    
    public Money calculateAnticipationDiscount(List<Integer> installmentNumbers) {
        // Desconto proporcional aos juros "economizados"
        return installments.stream()
            .filter(i -> installmentNumbers.contains(i.getNumber()))
            .filter(i -> i.getStatus() == InstallmentStatus.PENDING)
            .map(Installment::getInterestPortion)
            .reduce(Money.ZERO, Money::add)
            .multiply(new BigDecimal("0.8")); // 80% do juros como desconto
    }
}

// domain/model/Installment.java
public class Installment {
    private InstallmentId id;
    private ContractId contractId;
    private int number;
    private Money amount;
    private Money principalPortion;
    private Money interestPortion;
    private LocalDate dueDate;
    private InstallmentStatus status;  // PENDING, PAID, OVERDUE
    private LocalDate paidAt;
    private TransferId transferId;
    
    public void pay(TransferId transferId) {
        if (status == InstallmentStatus.PAID) {
            throw new InstallmentAlreadyPaidException(id);
        }
        this.status = InstallmentStatus.PAID;
        this.paidAt = LocalDate.now();
        this.transferId = transferId;
    }
    
    public void markAsOverdue() {
        if (status == InstallmentStatus.PENDING && LocalDate.now().isAfter(dueDate)) {
            this.status = InstallmentStatus.OVERDUE;
        }
    }
}
```

#### Dia 4-5: Use Cases e Controller

```java
// application/service/SimulateInstallmentService.java
@Service
@RequiredArgsConstructor
public class SimulateInstallmentService implements SimulateInstallmentUseCase {
    
    private final InstallmentCalculator calculator;
    private final InterestRateStrategySelector strategySelector;
    private final AccountClient accountClient;
    
    @Override
    public InstallmentSimulation execute(SimulateCommand command) {
        // Buscar perfil de crédito do cliente
        CreditProfile profile = accountClient.getCreditProfile(command.accountId());
        
        // Selecionar melhor estratégia de taxa
        InterestRateStrategy strategy = strategySelector.selectBestFor(profile);
        InterestRate rate = strategy.calculateRate(profile, command.installments());
        
        // Calcular simulação
        return calculator.simulate(
            command.amount(),
            command.installments(),
            rate
        );
    }
}

// application/service/ContractInstallmentService.java
@Service
@RequiredArgsConstructor
public class ContractInstallmentService implements ContractInstallmentUseCase {
    
    private final SimulateInstallmentUseCase simulateUseCase;
    private final ContractRepository contractRepository;
    private final AccountClient accountClient;
    private final TransferService transferService;
    
    @Override
    @Transactional
    public ContractId execute(ContractCommand command) {
        // 1. Verificar limite disponível
        Money availableLimit = accountClient.getAvailableLimit(command.accountId());
        if (availableLimit.isLessThan(command.amount())) {
            throw new InsufficientCreditLimitException(availableLimit, command.amount());
        }
        
        // 2. Simular para obter detalhes
        InstallmentSimulation simulation = simulateUseCase.execute(
            new SimulateCommand(command.accountId(), command.amount(), command.installments())
        );
        
        // 3. Criar contrato
        InstallmentContract contract = InstallmentContract.create(
            command.accountId(),
            command.receiverPixKey(),
            simulation,
            // ... rate
        );
        
        // 4. Reservar limite
        accountClient.reserveLimit(command.accountId(), command.amount());
        
        // 5. Executar transferência inicial para recebedor
        TransferId transferId = transferService.initiateTransfer(
            new TransferCommand(
                command.accountId(),  // Será do banco/instituição na prática
                command.receiverPixKey(),
                command.amount(),
                "Pix Parcelado - " + command.installments() + "x"
            )
        );
        
        // 6. Ativar contrato
        contract.activate(transferId);
        
        return contractRepository.save(contract).getId();
    }
}

// Controller
@RestController
@RequestMapping("/api/v1/installments")
@RequiredArgsConstructor
public class InstallmentController {
    
    @PostMapping("/simulate")
    public SimulationResponse simulate(@Valid @RequestBody SimulateRequest request) {
        var simulation = simulateUseCase.execute(mapper.toCommand(request));
        return mapper.toResponse(simulation);
    }
    
    @PostMapping("/contracts")
    @ResponseStatus(HttpStatus.CREATED)
    public ContractResponse contract(@Valid @RequestBody ContractRequest request) {
        ContractId id = contractUseCase.execute(mapper.toCommand(request));
        return getContractUseCase.byId(id).map(mapper::toResponse).orElseThrow();
    }
    
    @GetMapping("/limit/{accountId}")
    public LimitResponse getLimit(@PathVariable UUID accountId) {
        Money limit = accountClient.getAvailableLimit(AccountId.of(accountId));
        return new LimitResponse(limit.amount());
    }
    
    @PostMapping("/contracts/{id}/anticipate")
    public AnticipationResponse anticipate(
            @PathVariable UUID id,
            @RequestBody AnticipateRequest request) {
        // Antecipar parcelas com desconto
        return anticipateUseCase.execute(ContractId.of(id), request.installmentNumbers());
    }
}
```

### ✅ Critérios de Sucesso Sprint 7

- [ ] Installment Service criado com arquitetura hexagonal
- [ ] Calculadora financeira implementada (Price, IOF, CET)
- [ ] Strategy Pattern para diferentes taxas
- [ ] Simulação retornando detalhes corretos
- [ ] Contratação com verificação de limite
- [ ] Gestão de parcelas (consulta, baixa)
- [ ] Antecipação com cálculo de desconto
- [ ] Testes unitários da calculadora financeira

### 💡 Reflexão

> Por que usamos Strategy Pattern para taxas ao invés de ifs no código? Qual a vantagem quando o banco decidir criar uma nova campanha promocional?

---

## 🎯 Sprint 8: ☸️ Kubernetes

### Objetivo
Deploy dos microsserviços em Kubernetes local com Minikube e Helm Charts.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Kubernetes | `MODULO_10_KUBERNETES.md` | 10.1 a 10.10 |
| Docker | `MODULO_08_DEVOPS.md` | Docker |

### 🛠️ Implemente

#### Dia 1: Setup Minikube

```bash
# Instalar Minikube (Windows)
choco install minikube

# Iniciar cluster
minikube start --driver=docker --memory=4096 --cpus=2

# Verificar
kubectl get nodes
minikube dashboard
```

#### Dia 2: Dockerfile Otimizado

```dockerfile
# Dockerfile (multi-stage)
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# Non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

EXPOSE 8080 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Dia 3: Manifests Kubernetes

```yaml
# k8s/account-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account-service
  namespace: pix-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: account-service
  template:
    metadata:
      labels:
        app: account-service
    spec:
      containers:
        - name: account-service
          image: pix-account-service:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
            - containerPort: 8081
          envFrom:
            - configMapRef:
                name: account-service-config
            - secretRef:
                name: account-service-secrets
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8081
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8081
            initialDelaySeconds: 10
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: account-service
  namespace: pix-system
spec:
  type: ClusterIP
  selector:
    app: account-service
  ports:
    - port: 80
      targetPort: 8080
```

#### Dia 4-5: Helm Chart

```yaml
# charts/pix-service/Chart.yaml
apiVersion: v2
name: pix-service
description: Generic Helm chart for Pix services
version: 1.0.0

# charts/pix-service/values.yaml
replicaCount: 2

image:
  repository: pix-account-service
  tag: latest

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    memory: "256Mi"
    cpu: "250m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilization: 70
```

```bash
# Deploy com Helm
helm install account-service ./charts/pix-service \
  -n pix-system \
  --set image.repository=pix-account-service
```

### ✅ Critérios de Sucesso Sprint 8

- [ ] Minikube rodando
- [ ] Dockerfiles otimizados (< 200MB)
- [ ] Todos os serviços rodando no K8s
- [ ] Health checks funcionando
- [ ] Ingress configurado
- [ ] Helm Charts criados
- [ ] HPA testado

---

## 🎯 Sprint 9: ☁️ AWS com LocalStack

### Objetivo
Integrar os microsserviços com serviços AWS usando LocalStack para desenvolvimento local.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| AWS | `MODULO_09_AWS_CLOUD.md` | 9.1 a 9.10 |
| SQS/SNS | `MODULO_09_AWS_CLOUD.md` | 9.4 |
| LocalStack | `MODULO_09_AWS_CLOUD.md` | 9.9 |

### 🛠️ Implemente

#### Dia 1: Setup LocalStack

```yaml
# docker-compose.localstack.yml
version: '3.8'
services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3,sqs,sns,secretsmanager
      - DEBUG=1
    volumes:
      - "./init-aws:/etc/localstack/init/ready.d"
```

```bash
# init-aws/init.sh
#!/bin/bash
awslocal sqs create-queue --queue-name pix-transfer-events
awslocal sns create-topic --name pix-events
awslocal s3 mb s3://pix-documents
awslocal secretsmanager create-secret \
  --name /pix/database \
  --secret-string '{"username":"pix","password":"pix123"}'
```

#### Dia 2-3: Integração SQS/SNS

```java
// Dependências
// spring-cloud-aws-starter-sqs
// spring-cloud-aws-starter-sns

// Producer
@Service
@RequiredArgsConstructor
public class SqsEventPublisher implements EventPublisher {
    private final SqsTemplate sqsTemplate;
    
    @Override
    public void publish(DomainEvent event) {
        sqsTemplate.send("pix-transfer-events", event);
    }
}

// Consumer
@SqsListener("pix-transfer-events")
public void handleEvent(TransferCompletedEvent event) {
    log.info("Processando: {}", event);
}
```

#### Dia 4: S3 para Comprovantes

```java
@Service
@RequiredArgsConstructor
public class S3ReceiptStorage implements ReceiptStorage {
    private final S3Client s3Client;
    
    public String store(TransferId id, byte[] pdf) {
        String key = "receipts/" + id.value() + ".pdf";
        s3Client.putObject(
            PutObjectRequest.builder()
                .bucket("pix-documents")
                .key(key)
                .build(),
            RequestBody.fromBytes(pdf)
        );
        return key;
    }
}
```

#### Dia 5: Secrets Manager

```yaml
# application-local.yml
spring:
  config:
    import: "aws-secretsmanager:/pix/"
    
cloud:
  aws:
    endpoint: http://localhost:4566
    region:
      static: sa-east-1
    credentials:
      access-key: test
      secret-key: test
```

### ✅ Critérios de Sucesso Sprint 9

- [ ] LocalStack funcionando
- [ ] SQS substituindo Kafka
- [ ] SNS para fan-out
- [ ] S3 para comprovantes
- [ ] Secrets Manager integrado
- [ ] Testes com LocalStack

---

## 🎯 Sprint 10: 🏗️ Terraform

### Objetivo
Provisionar toda a infraestrutura AWS como código usando Terraform + LocalStack.

### 📖 Estude Primeiro

| Tópico | Arquivo | Seções |
|--------|---------|--------|
| Terraform | `MODULO_11_TERRAFORM.md` | 11.1 a 11.10 |
| IaC | `MODULO_11_TERRAFORM.md` | 11.1 |

### 🛠️ Implemente

#### Dia 1-2: Estrutura de Módulos

```
terraform/
├── environments/
│   ├── local/          # LocalStack
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/           # AWS real (futuro)
├── modules/
│   ├── networking/     # VPC, Subnets
│   ├── database/       # RDS
│   ├── messaging/      # SQS, SNS
│   └── storage/        # S3
└── localstack.tf       # Provider LocalStack
```

#### Dia 3: Módulo de Mensageria

```hcl
# modules/messaging/main.tf
resource "aws_sqs_queue" "main" {
  name                       = "${var.project}-${var.queue_name}"
  visibility_timeout_seconds = 60
  message_retention_seconds  = 1209600
  
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.dlq.arn
    maxReceiveCount     = 3
  })
}

resource "aws_sqs_queue" "dlq" {
  name = "${var.project}-${var.queue_name}-dlq"
}

resource "aws_sns_topic" "main" {
  name = "${var.project}-events"
}

resource "aws_sns_topic_subscription" "sqs" {
  topic_arn = aws_sns_topic.main.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.main.arn
}
```

#### Dia 4-5: Ambiente Completo

```hcl
# environments/local/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region                      = "sa-east-1"
  access_key                  = "test"
  secret_key                  = "test"
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  
  endpoints {
    s3             = "http://localhost:4566"
    sqs            = "http://localhost:4566"
    sns            = "http://localhost:4566"
    secretsmanager = "http://localhost:4566"
  }
}

module "messaging" {
  source     = "../../modules/messaging"
  project    = "pix"
  queue_name = "transfer-events"
}

module "storage" {
  source  = "../../modules/storage"
  project = "pix"
  buckets = ["documents", "logs"]
}
```

```bash
# Deploy completo
cd terraform/environments/local
terraform init
terraform apply -auto-approve
```

### ✅ Critérios de Sucesso Sprint 10

- [ ] Módulos Terraform criados
- [ ] VPC, SQS, SNS, S3 provisionados
- [ ] Secrets Manager configurado
- [ ] terraform apply funciona com LocalStack
- [ ] State versionado
- [ ] Documentação dos módulos

---

## 🎯 Sprint 11: 🚀 Integração Final

### Objetivo
Integrar tudo: K8s + AWS + Terraform em um ambiente coeso e documentado.

### 🛠️ Implemente

#### Dia 1-2: Script de Setup Completo

```bash
#!/bin/bash
# setup-environment.sh

echo "🚀 Iniciando ambiente Pix System..."

# 1. Subir LocalStack
echo "📦 Subindo LocalStack..."
docker-compose -f docker-compose.localstack.yml up -d
sleep 10

# 2. Aplicar Terraform
echo "🏗️ Aplicando Terraform..."
cd terraform/environments/local
terraform init
terraform apply -auto-approve
cd -

# 3. Build imagens
echo "🐳 Building Docker images..."
./mvnw clean package -DskipTests
for service in account transfer recurring installment; do
  docker build -t pix-${service}-service ./pix-${service}-service
done

# 4. Deploy no Kubernetes
echo "☸️ Deploying no Kubernetes..."
kubectl create namespace pix-system --dry-run=client -o yaml | kubectl apply -f -
helm upgrade --install pix-system ./charts/pix-umbrella -n pix-system

echo "✅ Ambiente pronto!"
echo "📊 Dashboard: minikube dashboard"
```

#### Dia 3: CI/CD Completo

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      
      - name: Build
        run: ./mvnw clean package -DskipTests
      
      - name: Build Docker Images
        run: |
          for service in account transfer recurring installment; do
            docker build -t pix-${service}-service ./pix-${service}-service
          done
      
      - name: Run Tests
        run: ./mvnw test
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          helm upgrade --install pix-system ./charts/pix-umbrella
```

#### Dia 4: README Profissional

```markdown
# 🏦 Pix System

> Sistema completo de Pix com microsserviços, AWS e Kubernetes.

![Java](https://img.shields.io/badge/Java-21-orange)
![Spring](https://img.shields.io/badge/Spring%20Boot-3.2-green)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-blue)
![Terraform](https://img.shields.io/badge/Terraform-1.5-purple)

## 🚀 Quick Start

\`\`\`bash
./setup-environment.sh
\`\`\`

## 🏗️ Arquitetura

[Diagrama da arquitetura]

## 📦 Serviços

| Serviço | Descrição | Port |
|---------|-----------|------|
| Account | Gestão de contas | 8080 |
| Transfer | Transferências Pix | 8081 |
| Recurring | Pix Automático | 8082 |
| Installment | Pix Parcelado | 8083 |

## 🛠️ Stack

- Java 21, Spring Boot 3.2
- PostgreSQL, Redis
- Kafka → SQS/SNS (AWS)
- Kubernetes (EKS)
- Terraform

## 📚 Documentação

- [API Docs](./docs/api.md)
- [Arquitetura](./docs/architecture.md)
- [Runbook](./docs/runbook.md)
```

### ✅ Critérios de Sucesso Sprint 11

- [ ] Script de setup funcional
- [ ] Ambiente sobe com um comando
- [ ] CI/CD completo
- [ ] README profissional
- [ ] Documentação de arquitetura
- [ ] Runbook de operações
- [ ] Testes E2E passando

---

## 📊 Checklist Final Completo

### Java Core
- [ ] Value Objects imutáveis
- [ ] Streams para transformações
- [ ] Optional para buscas
- [ ] Records para DTOs
- [ ] Sealed classes para eventos

### DDD
- [ ] Aggregates bem definidos
- [ ] Entities com identidade
- [ ] Value Objects sem identidade
- [ ] Domain Events
- [ ] Domain Services

### Arquitetura
- [ ] Hexagonal (Ports & Adapters)
- [ ] Event Sourcing
- [ ] CQRS
- [ ] API Gateway
- [ ] Service Discovery

### Spring
- [ ] Injeção por construtor
- [ ] @Transactional correto
- [ ] Profiles
- [ ] Spring Cloud

### Testes
- [ ] TDD praticado
- [ ] Cobertura > 80%
- [ ] TestContainers
- [ ] Testes de contrato

### DevOps
- [ ] Docker multi-stage
- [ ] Docker Compose
- [ ] CI/CD
- [ ] Observabilidade

### ☁️ Cloud (NOVO)
- [ ] AWS (SQS, SNS, S3, Secrets)
- [ ] LocalStack para desenvolvimento
- [ ] Spring Cloud AWS configurado

### ☸️ Kubernetes (NOVO)
- [ ] Deployments e Services
- [ ] ConfigMaps e Secrets
- [ ] Health checks
- [ ] HPA configurado
- [ ] Helm Charts

### 🏗️ Terraform (NOVO)
- [ ] Módulos organizados
- [ ] State management
- [ ] Terraform + LocalStack
- [ ] Documentação IaC

---

**Bons estudos e boa implementação! 🚀🏦☁️**
