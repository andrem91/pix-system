# 📖 Módulo 8: DevOps (1 semana)

> Entendimento básico de deploy e observabilidade é esperado de desenvolvedores plenos.

---

## 📚 8.1 Docker

### Dockerfile Multi-Stage
```dockerfile
# Stage 1: Build
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app

# Cache de dependências
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline -B

# Build da aplicação
COPY src ./src
RUN ./mvnw clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:21-jre
WORKDIR /app

# Usuário não-root
RUN addgroup --system appgroup && adduser --system appuser --ingroup appgroup
USER appuser

# Copia apenas o JAR
COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Otimizações
```dockerfile
# JVM otimizada para containers
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:InitialRAMPercentage=50.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
```

---

## 📚 8.2 Docker Compose

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/orders
      - SPRING_DATASOURCE_USERNAME=app
      - SPRING_DATASOURCE_PASSWORD=secret
      - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d orders"]
      interval: 10s
      timeout: 5s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

---

## 📚 8.3 GitHub Actions

### CI Pipeline
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
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
          cache: maven
      
      - name: Build and Test
        run: ./mvnw verify
      
      - name: Upload Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: target/surefire-reports/

  quality:
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven
      
      - name: SonarQube Scan
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: ./mvnw sonar:sonar -Dsonar.projectKey=my-project

  docker:
    runs-on: ubuntu-latest
    needs: [build, quality]
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker Image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to Registry
        env:
          DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
        run: |
          echo $DOCKER_TOKEN | docker login -u ${{ secrets.DOCKER_USER }} --password-stdin
          docker push myapp:${{ github.sha }}
```

---

## 📚 8.4 Observabilidade

### Health Checks
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(1)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("validationQuery", "SELECT 1")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withException(e)
                .build();
        }
        return Health.unknown().build();
    }
}

@Component
public class KafkaHealthIndicator implements HealthIndicator {
    
    private final KafkaAdmin kafkaAdmin;
    
    @Override
    public Health health() {
        try {
            Map<String, Object> config = kafkaAdmin.getConfigurationProperties();
            return Health.up()
                .withDetail("bootstrapServers", config.get("bootstrap.servers"))
                .build();
        } catch (Exception e) {
            return Health.down()
                .withException(e)
                .build();
        }
    }
}
```

### application.yml
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when_authorized
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

### Métricas Customizadas
```java
@Service
public class OrderService {
    
    private final Counter ordersCreated;
    private final Timer orderProcessingTime;
    private final AtomicInteger activeOrders;
    
    public OrderService(MeterRegistry registry) {
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total de pedidos criados")
            .tag("service", "order-service")
            .register(registry);
            
        this.orderProcessingTime = Timer.builder("orders.processing.time")
            .description("Tempo de processamento de pedidos")
            .register(registry);
            
        this.activeOrders = registry.gauge("orders.active", new AtomicInteger(0));
    }
    
    public Order create(CreateOrderRequest request) {
        return orderProcessingTime.record(() -> {
            activeOrders.incrementAndGet();
            try {
                Order order = processOrder(request);
                ordersCreated.increment();
                return order;
            } finally {
                activeOrders.decrementAndGet();
            }
        });
    }
}
```

### Logging Estruturado
```yaml
# application.yml
logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{traceId:-}] [%X{spanId:-}] %-5level %logger{36} - %msg%n"
```

```java
@Slf4j
@Service
public class OrderService {
    
    public Order create(CreateOrderRequest request) {
        MDC.put("customerId", request.getCustomerId().toString());
        MDC.put("requestId", UUID.randomUUID().toString());
        
        try {
            log.info("Criando pedido para cliente: {}", request.getCustomerId());
            // ...
            log.info("Pedido criado com sucesso: {}", order.getId());
            return order;
        } catch (Exception e) {
            log.error("Erro ao criar pedido: {}", e.getMessage(), e);
            throw e;
        } finally {
            MDC.clear();
        }
    }
}
```

---

## 📚 8.5 Kubernetes Básico

### Conceitos
```
┌─────────────────────────────────────────────────────────┐
│                      CLUSTER                             │
│  ┌──────────────────────────────────────────────────┐   │
│  │                    NODE                           │   │
│  │  ┌─────────────────────────────────────────────┐ │   │
│  │  │                    POD                      │ │   │
│  │  │  ┌─────────────┐  ┌─────────────┐          │ │   │
│  │  │  │ Container 1 │  │ Container 2 │          │ │   │
│  │  │  │  (app)      │  │  (sidecar)  │          │ │   │
│  │  │  └─────────────┘  └─────────────┘          │ │   │
│  │  └─────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

| Recurso | Descrição |
|---------|-----------|
| Pod | Menor unidade, 1+ containers |
| Deployment | Gerencia réplicas de pods |
| Service | Expõe pods na rede |
| ConfigMap | Configurações |
| Secret | Dados sensíveis |
| Ingress | Roteamento HTTP externo |

### Manifests Básicos
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: app
          image: myapp:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "kubernetes"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: password
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

---

## 📚 8.6 DevSecOps

### O que é DevSecOps?

```
┌─────────────────────────────────────────────────────────────────┐
│                        DevSecOps                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   DEV ───────> SEC ───────> OPS                                 │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              INTEGRAÇÃO CONTÍNUA (CI)                    │   │
│   │                                                          │   │
│   │  Code → Build → Test → SAST → SCA → Container Scan      │   │
│   │                                                          │   │
│   │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │   │
│   │  │ Commit │→│ Build  │→│JaCoCo  │→│ Sonar  │→│ Trivy  │ │   │
│   │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Sigla | Significado | Ferramenta |
|-------|-------------|------------|
| **SAST** | Static Application Security Testing | SonarQube, Checkmarx |
| **SCA** | Software Composition Analysis | Dependabot, Snyk |
| **DAST** | Dynamic Application Security Testing | OWASP ZAP |

---

### SonarQube/SonarCloud

**O que analisa:**
- 🐛 **Bugs** - Problemas que causam comportamento inesperado
- 🔒 **Vulnerabilidades** - Falhas de segurança
- 🧹 **Code Smells** - Código que dificulta manutenção
- 📊 **Cobertura** - % de código testado
- 📏 **Duplicação** - Código repetido

**Quality Gate:**
```
Condições para código passar:
✅ 0 bugs críticos
✅ 0 vulnerabilidades críticas
✅ Cobertura >= 80%
✅ Duplicação < 3%
✅ Debt ratio < 5%
```

**Configuração Maven:**
```xml
<!-- pom.xml -->
<properties>
    <sonar.organization>seu-usuario</sonar.organization>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
    <sonar.projectKey>seu-usuario_pix-system</sonar.projectKey>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.10.0.2594</version>
        </plugin>
    </plugins>
</build>
```

---

### JaCoCo (Code Coverage)

**Por que 80%?**
- Menos de 70%: Código arriscado
- 70-80%: Aceitável
- 80%+: Bom padrão
- 100%: Diminishing returns

**Excluir do coverage:**
```xml
<configuration>
    <excludes>
        <exclude>**/config/**</exclude>
        <exclude>**/*Application.class</exclude>
        <exclude>**/dto/**</exclude>
    </excludes>
</configuration>
```

---

### Trivy (Container Security)

**O que escaneia:**
- 🐳 Imagens Docker
- 📦 Dependências (Maven, npm, etc)
- 📄 Arquivos de configuração
- 🔐 Secrets expostos

```bash
# Scan local
trivy image pix-account-service:latest

# Apenas CRITICAL e HIGH
trivy image --severity CRITICAL,HIGH myapp:latest

# Formato JSON para CI
trivy image --format json --output results.json myapp:latest
```

---

### Dependabot

**Benefícios:**
- ✅ Atualiza dependências automaticamente
- ✅ Cria PRs com changelogs
- ✅ Identifica vulnerabilidades conhecidas
- ✅ Grátis para repos públicos e privados

**Labels úteis:**
```yaml
labels:
  - "dependencies"
  - "security"      # Se for security patch
  - "auto-merge"    # Para minor/patch versions
```

---

### OWASP Top 10 (Conhecer!)

| # | Vulnerabilidade | Maven |
|---|-----------------|-------|
| 1 | Broken Access Control | Spring Security |
| 2 | Cryptographic Failures | Jasypt, Vault |
| 3 | Injection | Prepared Statements, JPA |
| 4 | Insecure Design | Threat Modeling |
| 5 | Security Misconfiguration | Spring Profiles |
| 6 | Vulnerable Components | Dependabot, Snyk |
| 7 | Auth Failures | Spring Security |
| 8 | Data Integrity | Checksums, Signatures |
| 9 | Logging Failures | Logback + SIEM |
| 10 | SSRF | URL Validation |

---

### Secrets Management

```java
// ❌ NUNCA faça isso
public static final String API_KEY = "sk-1234567890";

// ❌ Nem isso
application.yml:
  api-key: sk-1234567890

// ✅ Use variáveis de ambiente
application.yml:
  api-key: ${API_KEY}

// ✅ Ou AWS Secrets Manager / Vault
@Value("${/pix/api-key}")
private String apiKey;
```

---

## 🎯 Perguntas de Entrevista

1. **O que é Dockerfile multi-stage?**
2. **Como funciona health check?**
3. **O que são métricas? Como implementar?**
4. **Qual a diferença entre Pod, Deployment e Service?**
5. **Como fazer logging estruturado?**
6. **O que é SAST vs DAST?** 🆕
7. **Por que cobertura de 80%?** 🆕
8. **Como gerenciar secrets em produção?** 🆕

---

> **Próximo:** [Módulo 9 - AWS Cloud](MODULO_09_AWS_CLOUD.md)

