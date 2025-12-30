# 📖 Módulo 3: OOP, SOLID e DDD Básico (1 semana)

> Princípios que separam código amador de profissional. Muito cobrado em entrevistas.

---

## 📚 Índice

| Seção | Tópico | Relevância |
|-------|--------|------------|
| 3.1 | OOP e Encapsulamento | 🔴 Crítico |
| 3.2 | Herança vs Composição | 🔴 Crítico |
| 3.3 | Princípios SOLID | 🔴 Crítico |
| 3.4 | Design Patterns Essenciais | 🟡 Importante |
| 3.5 | DDD - Conceitos Básicos | 🔴 Crítico |
| 3.6 | Value Objects vs Entities | 🔴 Crítico |

---

## 📚 3.1 OOP e Encapsulamento

### Os 4 Pilares da OOP

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORIENTAÇÃO A OBJETOS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│   │ENCAPSULAMENTO│  │   HERANÇA    │  │POLIMORFISMO  │          │
│   │              │  │              │  │              │          │
│   │ Esconde      │  │ Reutiliza    │  │ Múltiplas    │          │
│   │ detalhes     │  │ código       │  │ formas       │          │
│   └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│                    ┌──────────────┐                             │
│                    │  ABSTRAÇÃO   │                             │
│                    │              │                             │
│                    │ Simplifica   │                             │
│                    │ complexidade │                             │
│                    └──────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

### Encapsulamento

**Conceito:** Esconder a implementação interna e expor apenas o necessário.

```java
// ❌ RUIM: Estado exposto
public class ContaBancaria {
    public BigDecimal saldo;  // Qualquer um pode alterar!
    public String titular;
}

// Uso perigoso:
conta.saldo = new BigDecimal("-1000"); // Saldo negativo!

// ✅ BOM: Estado protegido
public class ContaBancaria {
    private BigDecimal saldo;
    private final String titular;
    
    public ContaBancaria(String titular, BigDecimal saldoInicial) {
        this.titular = titular;
        this.saldo = saldoInicial;
    }
    
    public void depositar(BigDecimal valor) {
        if (valor.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Valor deve ser positivo");
        }
        this.saldo = this.saldo.add(valor);
    }
    
    public void sacar(BigDecimal valor) {
        if (valor.compareTo(saldo) > 0) {
            throw new SaldoInsuficienteException(saldo, valor);
        }
        this.saldo = this.saldo.subtract(valor);
    }
    
    public BigDecimal getSaldo() {
        return saldo;  // Retorna cópia, não referência
    }
}
```

### Tell, Don't Ask

```java
// ❌ RUIM: Pergunta o estado e decide fora
if (conta.getSaldo().compareTo(valor) >= 0) {
    conta.setSaldo(conta.getSaldo().subtract(valor));
}

// ✅ BOM: Diz o que fazer, objeto decide como
conta.sacar(valor); // Lança exceção se não puder
```

---

## 📚 3.2 Herança vs Composição

### Herança - Quando Usar

**Regra:** Use herança apenas quando existe relação **"é um"** verdadeira.

```java
// ✅ Correto: Cachorro É UM Animal
public abstract class Animal {
    public abstract void emitirSom();
}

public class Cachorro extends Animal {
    @Override
    public void emitirSom() {
        System.out.println("Au au!");
    }
}
```

### Problema da Herança

```java
// ❌ Problema: Quadrado NÃO É UM Retângulo (violação LSP)
public class Retangulo {
    protected int largura;
    protected int altura;
    
    public void setLargura(int l) { this.largura = l; }
    public void setAltura(int a) { this.altura = a; }
    public int getArea() { return largura * altura; }
}

public class Quadrado extends Retangulo {
    @Override
    public void setLargura(int l) {
        this.largura = l;
        this.altura = l; // Quebra expectativa!
    }
}

// Problema:
Retangulo r = new Quadrado();
r.setLargura(5);
r.setAltura(10);
r.getArea(); // Esperado: 50, Resultado: 100!
```

### Composição - Preferência

**Regra:** Prefira composição quando existe relação **"tem um"**.

```java
// ✅ Composição: Carro TEM UM Motor
public class Motor {
    private final int potencia;
    
    public void ligar() { }
    public void desligar() { }
}

public class Carro {
    private final Motor motor;  // Composição!
    
    public Carro(Motor motor) {
        this.motor = motor;
    }
    
    public void ligar() {
        motor.ligar();  // Delega
    }
}
```

### Resumo

| | Herança | Composição |
|---|---------|------------|
| **Relação** | "É um" | "Tem um" |
| **Acoplamento** | Alto | Baixo |
| **Flexibilidade** | Baixa | Alta |
| **Teste** | Difícil | Fácil |
| **Recomendação** | Evitar | Preferir |

---

## 📚 3.3 Princípios SOLID

### S - Single Responsibility Principle

**"Uma classe deve ter apenas um motivo para mudar"**

```java
// ❌ Múltiplas responsabilidades
public class UserService {
    public void save(User user) { }           // Persistência
    public void sendWelcomeEmail(User user) { }  // Email
    public String generateReport() { }        // Relatório
    public boolean validateCpf(String cpf) { }  // Validação
}
// 4 motivos para mudar essa classe!

// ✅ Separado por responsabilidade
public class UserService {
    private final UserRepository repository;
    public User save(User user) { return repository.save(user); }
}

public class EmailService {
    public void sendWelcome(User user) { }
}

public class UserReportService {
    public String generate(List<User> users) { }
}

public class CpfValidator {
    public boolean isValid(String cpf) { }
}
```

---

### O - Open/Closed Principle

**"Aberto para extensão, fechado para modificação"**

```java
// ❌ Precisa modificar para adicionar novo desconto
public class CalculadoraDesconto {
    public BigDecimal calcular(String tipo, BigDecimal valor) {
        if (tipo.equals("BLACKFRIDAY")) {
            return valor.multiply(new BigDecimal("0.7"));
        } else if (tipo.equals("NATAL")) {
            return valor.multiply(new BigDecimal("0.85"));
        } else if (tipo.equals("VERAO")) { // Mais um if!
            return valor.multiply(new BigDecimal("0.9"));
        }
        return valor;
    }
}

// ✅ Extensível sem modificar código existente
public interface DescontoStrategy {
    BigDecimal aplicar(BigDecimal valor);
    String getTipo();
}

@Component
public class BlackFridayDesconto implements DescontoStrategy {
    public BigDecimal aplicar(BigDecimal valor) {
        return valor.multiply(new BigDecimal("0.7"));
    }
    public String getTipo() { return "BLACKFRIDAY"; }
}

// Para adicionar novo desconto: apenas criar nova classe!
@Component
public class VeraoDesconto implements DescontoStrategy {
    public BigDecimal aplicar(BigDecimal valor) {
        return valor.multiply(new BigDecimal("0.9"));
    }
    public String getTipo() { return "VERAO"; }
}
```

---

### L - Liskov Substitution Principle

**"Subclasses devem ser substituíveis por suas classes base"**

```java
// ❌ Viola LSP (exemplo do Quadrado/Retângulo acima)

// ✅ Solução: Composição ou interfaces separadas
public interface Forma {
    int getArea();
}

public record Retangulo(int largura, int altura) implements Forma {
    public int getArea() { return largura * altura; }
}

public record Quadrado(int lado) implements Forma {
    public int getArea() { return lado * lado; }
}
```

---

### I - Interface Segregation Principle

**"Clientes não devem depender de interfaces que não usam"**

```java
// ❌ Interface muito ampla
public interface Trabalhador {
    void trabalhar();
    void comer();
    void dirigir();
}

// ✅ Interfaces segregadas
public interface Trabalhavel { void trabalhar(); }
public interface Alimentavel { void comer(); }
public interface Motorista { void dirigir(); }

public class Desenvolvedor implements Trabalhavel, Alimentavel {
    public void trabalhar() { }
    public void comer() { }
}
```

---

### D - Dependency Inversion Principle

**"Dependa de abstrações, não de implementações"**

```java
// ❌ Depende de implementação concreta
public class PedidoService {
    private final MySqlPedidoRepository repository; // Concreto!
    
    public PedidoService() {
        this.repository = new MySqlPedidoRepository();
    }
}

// ✅ Depende de abstrações
public class PedidoService {
    private final PedidoRepository repository;  // Interface!
    
    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}

public interface PedidoRepository {
    Pedido save(Pedido pedido);
    Optional<Pedido> findById(Long id);
}

// Implementações podem mudar sem afetar PedidoService
@Repository
public class JpaPedidoRepository implements PedidoRepository { }
```

---

## 📚 3.4 Design Patterns Essenciais

### Strategy Pattern

**Quando usar:** Algoritmos intercambiáveis em runtime

```java
public interface PaymentStrategy {
    PaymentResult process(BigDecimal amount);
}

@Component("CREDIT_CARD")
public class CreditCardPayment implements PaymentStrategy {
    public PaymentResult process(BigDecimal amount) {
        return new PaymentResult(true, "Cartão aprovado");
    }
}

@Component("PIX")
public class PixPayment implements PaymentStrategy {
    public PaymentResult process(BigDecimal amount) {
        return new PaymentResult(true, "PIX realizado");
    }
}

@Service
public class PaymentService {
    private final Map<String, PaymentStrategy> strategies;
    
    public PaymentService(Map<String, PaymentStrategy> strategies) {
        this.strategies = strategies;
    }
    
    public PaymentResult pay(String method, BigDecimal amount) {
        return strategies.get(method).process(amount);
    }
}
```

### Factory Pattern

**Quando usar:** Criação complexa de objetos

```java
public interface Notification {
    void send(String message, String recipient);
}

@Component
public class NotificationFactory {
    public Notification create(String type) {
        return switch (type.toUpperCase()) {
            case "EMAIL" -> new EmailNotification();
            case "SMS" -> new SmsNotification();
            case "PUSH" -> new PushNotification();
            default -> throw new IllegalArgumentException("Tipo desconhecido");
        };
    }
}
```

### Builder Pattern

**Quando usar:** Objetos com muitos parâmetros opcionais

```java
// Com Lombok - forma recomendada
@Builder
@Getter
public class Order {
    private String id;
    private Customer customer;
    @Singular private List<OrderItem> items;
    private Address shippingAddress;
}

// Uso
Order order = Order.builder()
    .id(UUID.randomUUID().toString())
    .customer(customer)
    .item(produto1)
    .item(produto2)
    .shippingAddress(endereco)
    .build();
```

---

## 📚 3.5 DDD - Conceitos Básicos

### O que é Domain-Driven Design?

**DDD** é uma abordagem de desenvolvimento que foca no **domínio do negócio** e na **linguagem ubíqua** (termos que desenvolvedores e especialistas de negócio usam em comum).

```
┌─────────────────────────────────────────────────────────────────┐
│                     DOMAIN-DRIVEN DESIGN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    CAMADA DE DOMÍNIO                        │ │
│  │                                                             │ │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │ │
│  │   │  ENTITIES   │  │   VALUE     │  │  AGGREGATES │        │ │
│  │   │             │  │  OBJECTS    │  │             │        │ │
│  │   │ Identidade  │  │ Sem ident.  │  │ Raiz + filhos│       │ │
│  │   │ própria     │  │ Imutáveis   │  │             │        │ │
│  │   └─────────────┘  └─────────────┘  └─────────────┘        │ │
│  │                                                             │ │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │ │
│  │   │  DOMAIN     │  │  DOMAIN     │  │ REPOSITORIES│        │ │
│  │   │  EVENTS     │  │  SERVICES   │  │             │        │ │
│  │   │             │  │             │  │ Persistência│        │ │
│  │   │ Coisas que  │  │ Lógica que  │  │ do Aggregate│        │ │
│  │   │ aconteceram │  │ não é de    │  │             │        │ │
│  │   │             │  │ uma entity  │  │             │        │ │
│  │   └─────────────┘  └─────────────┘  └─────────────┘        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Linguagem Ubíqua

```java
// ❌ Termos técnicos que negócio não entende
class UserDTO { }
class AccountDAO { }
void processData() { }

// ✅ Termos do negócio
class Correntista { }
class ContaCorrente { }
void realizarTransferenciaPix() { }
```

---

## 📚 3.6 Value Objects vs Entities

### Entity (Identidade)

**Características:**
- Tem **identidade única** (ID)
- Pode mudar de estado ao longo do tempo
- Dois objetos com mesmos dados mas IDs diferentes são DIFERENTES

```java
public class ContaCorrente {
    private final AccountId id;  // Identidade!
    private String titular;
    private BigDecimal saldo;
    
    // Pode mudar
    public void alterarTitular(String novoTitular) {
        this.titular = novoTitular;
    }
    
    // equals/hashCode baseado APENAS no ID
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ContaCorrente that)) return false;
        return id.equals(that.id);  // Compara só ID!
    }
}

// Duas contas diferentes, mesmo se tiverem mesmos dados
ContaCorrente conta1 = new ContaCorrente(id1, "João", BigDecimal.ZERO);
ContaCorrente conta2 = new ContaCorrente(id2, "João", BigDecimal.ZERO);
conta1.equals(conta2); // FALSE! IDs diferentes
```

### Value Object (Valor)

**Características:**
- **NÃO tem identidade** própria
- É **imutável** (não muda após criação)
- Dois objetos com mesmos dados são IGUAIS
- Pode usar **record** do Java

```java
// Value Object - CPF
public record CPF(String value) {
    
    public CPF {
        // Validação no construtor
        String cleaned = value.replaceAll("[^0-9]", "");
        if (cleaned.length() != 11) {
            throw new InvalidCPFException("CPF deve ter 11 dígitos");
        }
        if (!isValid(cleaned)) {
            throw new InvalidCPFException("CPF inválido: " + value);
        }
        value = cleaned;  // Normaliza
    }
    
    private static boolean isValid(String cpf) {
        // Algoritmo de validação (dígitos verificadores)
        if (cpf.matches("(\\d)\\1{10}")) return false; // 11111111111
        
        int[] pesos1 = {10, 9, 8, 7, 6, 5, 4, 3, 2};
        int[] pesos2 = {11, 10, 9, 8, 7, 6, 5, 4, 3, 2};
        
        int soma1 = 0;
        for (int i = 0; i < 9; i++) {
            soma1 += Character.getNumericValue(cpf.charAt(i)) * pesos1[i];
        }
        int digito1 = 11 - (soma1 % 11);
        if (digito1 > 9) digito1 = 0;
        
        int soma2 = 0;
        for (int i = 0; i < 10; i++) {
            soma2 += Character.getNumericValue(cpf.charAt(i)) * pesos2[i];
        }
        int digito2 = 11 - (soma2 % 11);
        if (digito2 > 9) digito2 = 0;
        
        return cpf.charAt(9) == Character.forDigit(digito1, 10)
            && cpf.charAt(10) == Character.forDigit(digito2, 10);
    }
    
    public String formatted() {
        return value.substring(0, 3) + "." +
               value.substring(3, 6) + "." +
               value.substring(6, 9) + "-" +
               value.substring(9);
    }
}

// Uso
CPF cpf1 = new CPF("529.982.247-25");
CPF cpf2 = new CPF("52998224725");
cpf1.equals(cpf2); // TRUE! Mesmo valor
```

### Value Object - Money

```java
public record Money(BigDecimal amount, Currency currency) {
    
    public static final Money ZERO = Money.of(BigDecimal.ZERO);
    
    public Money {
        if (amount == null) {
            throw new IllegalArgumentException("Amount cannot be null");
        }
        if (currency == null) {
            currency = Currency.getInstance("BRL");
        }
        // Normaliza para 2 casas decimais
        amount = amount.setScale(2, RoundingMode.HALF_UP);
    }
    
    public static Money of(BigDecimal amount) {
        return new Money(amount, Currency.getInstance("BRL"));
    }
    
    public static Money of(String amount) {
        return of(new BigDecimal(amount));
    }
    
    public Money add(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money subtract(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    public Money multiply(int quantity) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(quantity)), this.currency);
    }
    
    public boolean isGreaterThan(Money other) {
        validateSameCurrency(other);
        return this.amount.compareTo(other.amount) > 0;
    }
    
    public boolean isPositive() {
        return amount.compareTo(BigDecimal.ZERO) > 0;
    }
    
    public boolean isNegative() {
        return amount.compareTo(BigDecimal.ZERO) < 0;
    }
    
    private void validateSameCurrency(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new DifferentCurrencyException(this.currency, other.currency);
        }
    }
}
```

### Value Object - PixKey

```java
public record PixKey(String value, PixKeyType type) {
    
    public PixKey {
        Objects.requireNonNull(value, "Chave não pode ser nula");
        Objects.requireNonNull(type, "Tipo não pode ser nulo");
        
        value = value.trim().toLowerCase();
        
        if (!type.isValid(value)) {
            throw new InvalidPixKeyException(value, type);
        }
    }
    
    public static PixKey of(String value) {
        PixKeyType type = PixKeyType.detect(value);
        return new PixKey(value, type);
    }
}

public enum PixKeyType {
    CPF {
        @Override
        public boolean isValid(String value) {
            return value.matches("\\d{11}");
        }
    },
    CNPJ {
        @Override
        public boolean isValid(String value) {
            return value.matches("\\d{14}");
        }
    },
    EMAIL {
        @Override
        public boolean isValid(String value) {
            return value.matches("^[\\w-.]+@[\\w-]+\\.[a-z]{2,}$");
        }
    },
    PHONE {
        @Override
        public boolean isValid(String value) {
            return value.matches("\\+55\\d{10,11}");
        }
    },
    RANDOM {
        @Override
        public boolean isValid(String value) {
            return value.matches("[a-f0-9-]{36}");
        }
    };
    
    public abstract boolean isValid(String value);
    
    public static PixKeyType detect(String value) {
        for (PixKeyType type : values()) {
            if (type.isValid(value)) {
                return type;
            }
        }
        throw new InvalidPixKeyException("Formato de chave inválido: " + value);
    }
}
```

### Resumo: Entity vs Value Object

| Aspecto | Entity | Value Object |
|---------|--------|--------------|
| **Identidade** | Tem ID único | Não tem ID |
| **Igualdade** | Por ID | Por valores |
| **Mutabilidade** | Pode mudar | Imutável |
| **Ciclo de vida** | Longo | Efêmero |
| **Exemplo** | ContaCorrente, Usuario | CPF, Money, Email |
| **Java** | class | record |

---

## 🎯 Perguntas de Entrevista

1. **Explique os princípios SOLID**
2. **Quando usar Strategy Pattern?**
3. **Factory vs Builder - qual a diferença?**
4. **O que é um Value Object? Dê exemplos**
5. **Qual a diferença entre Entity e Value Object?**
6. **Por que Value Objects devem ser imutáveis?**
7. **Como você validaria um CPF em Java usando DDD?**

---

> **Próximo módulo:** [Módulo 4 - Spring Ecosystem](MODULO_04_SPRING.md)
