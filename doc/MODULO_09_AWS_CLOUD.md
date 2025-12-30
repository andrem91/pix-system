# ☁️ Módulo 09: AWS Cloud para Desenvolvedores Java

> **Objetivo:** Dominar os serviços AWS mais exigidos em vagas Java, com foco prático usando LocalStack.

---

## 📌 Por que AWS?

- **85%+ das vagas** Java Pleno/Senior exigem conhecimento em cloud
- AWS é líder de mercado (~32% market share)
- Certificações AWS são altamente valorizadas
- Experiência prática diferencia candidatos

---

## 🎯 O que você vai aprender

| Seção | Tópico | Importância |
|-------|--------|-------------|
| 9.1 | Fundamentos AWS | 🔴 Crítico |
| 9.2 | IAM (Identity & Access) | 🔴 Crítico |
| 9.3 | RDS (Banco de Dados) | 🔴 Crítico |
| 9.4 | SQS/SNS (Mensageria) | 🔴 Crítico |
| 9.5 | S3 (Storage) | 🟡 Importante |
| 9.6 | Secrets Manager | 🟡 Importante |
| 9.7 | CloudWatch | 🟡 Importante |
| 9.8 | ECS/ECR (Containers) | 🟡 Importante |
| 9.9 | LocalStack | 🔴 Crítico |
| 9.10 | SDK AWS para Java | 🔴 Crítico |

---

## 9.1 Fundamentos AWS

### Conceitos Essenciais

```
┌─────────────────────────────────────────────────────────────────┐
│                         AWS GLOBAL                               │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐    ┌─────────────────────┐             │
│  │     REGION          │    │     REGION          │             │
│  │   (sa-east-1)       │    │   (us-east-1)       │             │
│  │  São Paulo          │    │   N. Virginia       │             │
│  │                     │    │                     │             │
│  │ ┌─────┐ ┌─────┐     │    │ ┌─────┐ ┌─────┐    │             │
│  │ │ AZ  │ │ AZ  │     │    │ │ AZ  │ │ AZ  │    │             │
│  │ │ 1a  │ │ 1b  │     │    │ │ 1a  │ │ 1b  │    │             │
│  │ └─────┘ └─────┘     │    │ └─────┘ └─────┘    │             │
│  └─────────────────────┘    └─────────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

| Conceito | Descrição | Exemplo |
|----------|-----------|---------|
| **Region** | Localização geográfica | sa-east-1 (São Paulo) |
| **Availability Zone** | Data center isolado | sa-east-1a, sa-east-1b |
| **VPC** | Rede virtual privada | Sua rede isolada na AWS |
| **Subnet** | Sub-rede dentro da VPC | Pública ou privada |
| **Security Group** | Firewall virtual | Regras de entrada/saída |

### VPC Básica

```
┌─────────────────────────────────────────────────────────────────┐
│                         VPC (10.0.0.0/16)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────┐    ┌─────────────────────┐            │
│   │   PUBLIC SUBNET     │    │   PRIVATE SUBNET    │            │
│   │   10.0.1.0/24       │    │   10.0.2.0/24       │            │
│   │                     │    │                     │            │
│   │  ┌───────────────┐  │    │  ┌───────────────┐  │            │
│   │  │ API Gateway   │  │    │  │ ECS Services  │  │            │
│   │  │ Load Balancer │  │    │  │ RDS Database  │  │            │
│   │  └───────────────┘  │    │  └───────────────┘  │            │
│   │         │           │    │         ▲          │            │
│   │         │           │    │         │          │            │
│   └─────────┼───────────┘    └─────────┼──────────┘            │
│             │                          │                        │
│             └──────────────────────────┘                        │
│                      NAT Gateway                                 │
└─────────────────────────────────────────────────────────────────┘
           │
           ▼
    ┌─────────────┐
    │  INTERNET   │
    │   GATEWAY   │
    └─────────────┘
```

---

## 9.2 IAM (Identity and Access Management)

### Conceitos

| Componente | Descrição | Uso |
|------------|-----------|-----|
| **User** | Identidade humana | Desenvolvedores, admins |
| **Group** | Conjunto de users | Team-developers |
| **Role** | Identidade assumível | EC2 acessando S3 |
| **Policy** | Permissões JSON | Definir o que pode fazer |

### Exemplo de Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::my-bucket/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:sa-east-1:123456789:pix-queue"
        }
    ]
}
```

### Princípio do Menor Privilégio

```java
// ❌ ERRADO: Permissões amplas demais
{
    "Effect": "Allow",
    "Action": "*",
    "Resource": "*"
}

// ✅ CORRETO: Apenas o necessário
{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::pix-documents/*"
}
```

---

## 9.3 RDS (Relational Database Service)

### Por que RDS ao invés de EC2 com banco?

| Aspecto | EC2 + Postgres | RDS Postgres |
|---------|----------------|--------------|
| **Backups** | Manual | Automático |
| **Patches** | Você aplica | AWS aplica |
| **Multi-AZ** | Configurar manualmente | Um clique |
| **Scaling** | Downtime | Read Replicas |
| **Monitoramento** | Instalar agentes | CloudWatch integrado |

### Configuração Spring Boot

```yaml
# application-aws.yml
spring:
  datasource:
    url: jdbc:postgresql://${RDS_HOSTNAME}:${RDS_PORT}/${RDS_DB_NAME}
    username: ${RDS_USERNAME}
    password: ${RDS_PASSWORD}
    
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
```

### Multi-AZ para Alta Disponibilidade

```
┌─────────────────────────────────────────────────────────────────┐
│                         RDS Multi-AZ                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────┐    ┌─────────────────────┐            │
│   │   AZ-1 (Primary)    │    │   AZ-2 (Standby)    │            │
│   │                     │    │                     │            │
│   │  ┌───────────────┐  │    │  ┌───────────────┐  │            │
│   │  │  PostgreSQL   │══════════│  PostgreSQL   │  │            │
│   │  │   Primary     │  │Sync│  │   Standby     │  │            │
│   │  │               │  │Repl│  │               │  │            │
│   │  └───────────────┘  │    │  └───────────────┘  │            │
│   └─────────────────────┘    └─────────────────────┘            │
│              │                          │                        │
│              │        FAILOVER          │                        │
│              │      (automático)        │                        │
│              └──────────────────────────┘                        │
│                         │                                        │
│                  ┌──────▼──────┐                                │
│                  │  Endpoint   │ (DNS não muda)                 │
│                  │   Único     │                                │
│                  └─────────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9.4 SQS/SNS (Mensageria)

### SQS (Simple Queue Service)

**Filas de mensagens totalmente gerenciadas.**

| Tipo | Garantia de Ordem | Throughput | Uso |
|------|-------------------|------------|-----|
| **Standard** | Best effort | Ilimitado | Alto volume |
| **FIFO** | Garantida | 300 msg/s | Ordens críticas |

```java
// Producer - Enviando mensagem para SQS
@Service
@RequiredArgsConstructor
public class SqsMessagePublisher {
    
    private final SqsTemplate sqsTemplate;
    
    public void publishTransferEvent(TransferCompletedEvent event) {
        sqsTemplate.send(to -> to
            .queue("pix-transfer-events")
            .payload(event)
            .header("messageType", "TRANSFER_COMPLETED")
        );
    }
}

// Consumer - Recebendo mensagem do SQS
@Component
@Slf4j
public class SqsTransferEventConsumer {
    
    @SqsListener("pix-transfer-events")
    public void handleTransferEvent(TransferCompletedEvent event) {
        log.info("Processando evento: {}", event.getTransferId());
        // Processar evento
    }
}
```

### SNS (Simple Notification Service)

**Pub/Sub para múltiplos subscribers.**

```
┌─────────────┐
│   Producer  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  SNS Topic  │
│ (pix-events)│
└──────┬──────┘
       │
       ├─────────────────┬─────────────────┐
       ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ SQS Queue   │   │ SQS Queue   │   │   Lambda    │
│ (notific.)  │   │ (analytics) │   │  (webhook)  │
└─────────────┘   └─────────────┘   └─────────────┘
       │                 │                 │
       ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│Notification │   │  Analytics  │   │  External   │
│  Service    │   │   Service   │   │   System    │
└─────────────┘   └─────────────┘   └─────────────┘
```

```java
// Publicando em SNS
@Service
@RequiredArgsConstructor
public class SnsEventPublisher implements EventPublisher {
    
    private final SnsTemplate snsTemplate;
    
    @Override
    public void publish(DomainEvent event) {
        snsTemplate.sendNotification("pix-events", 
            SnsNotification.builder()
                .message(event)
                .subject(event.getClass().getSimpleName())
                .build()
        );
    }
}
```

### SQS vs SNS vs Kafka

| Característica | SQS | SNS | Kafka |
|----------------|-----|-----|-------|
| **Modelo** | Queue (1:1) | Pub/Sub (1:N) | Pub/Sub + Replay |
| **Retenção** | 14 dias | Imediato | Configurável |
| **Ordem** | FIFO opcional | Não garantida | Por partição |
| **Replay** | Não | Não | Sim |
| **Gerenciamento** | Serverless | Serverless | Requer cluster |

---

## 9.5 S3 (Simple Storage Service)

### Casos de Uso

| Uso | Exemplo |
|-----|---------|
| **Arquivos** | Comprovantes de transferência PDF |
| **Logs** | Arquivamento de logs antigos |
| **Backup** | Backup de banco de dados |
| **Static** | Assets de frontend |

### Integração com Spring

```java
// Configuração
@Configuration
public class S3Config {
    
    @Bean
    public S3Client s3Client() {
        return S3Client.builder()
            .region(Region.SA_EAST_1)
            .build();
    }
}

// Service para upload
@Service
@RequiredArgsConstructor
public class S3StorageService implements StorageService {
    
    private final S3Client s3Client;
    
    @Value("${aws.s3.bucket}")
    private String bucketName;
    
    public String uploadTransferReceipt(TransferId transferId, byte[] pdfContent) {
        String key = "receipts/" + transferId.value() + ".pdf";
        
        PutObjectRequest request = PutObjectRequest.builder()
            .bucket(bucketName)
            .key(key)
            .contentType("application/pdf")
            .build();
        
        s3Client.putObject(request, RequestBody.fromBytes(pdfContent));
        
        return key;
    }
    
    public byte[] downloadFile(String key) {
        GetObjectRequest request = GetObjectRequest.builder()
            .bucket(bucketName)
            .key(key)
            .build();
        
        return s3Client.getObjectAsBytes(request).asByteArray();
    }
    
    public String generatePresignedUrl(String key, Duration expiration) {
        try (S3Presigner presigner = S3Presigner.create()) {
            GetObjectRequest getObjectRequest = GetObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
            
            GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
                .signatureDuration(expiration)
                .getObjectRequest(getObjectRequest)
                .build();
            
            return presigner.presignGetObject(presignRequest).url().toString();
        }
    }
}
```

---

## 9.6 Secrets Manager

### Por que usar?

| Problema | Solução |
|----------|---------|
| Senhas no código | Secrets Manager |
| Rotação manual | Rotação automática |
| Acesso auditado | CloudTrail integrado |
| Multi-ambiente | Mesmo código, secrets diferentes |

### Integração com Spring

```java
// Usando Spring Cloud AWS
// application.yml
spring:
  config:
    import: aws-secretsmanager:/pix-system/prod/

// Os secrets são automaticamente mapeados para properties
// Secret: /pix-system/prod/ com JSON:
// { "db.password": "xxx", "api.key": "yyy" }

// No código:
@Value("${db.password}")
private String dbPassword;
```

```java
// Programaticamente
@Service
@RequiredArgsConstructor
public class SecretsService {
    
    private final SecretsManagerClient secretsClient;
    
    public String getSecret(String secretName) {
        GetSecretValueRequest request = GetSecretValueRequest.builder()
            .secretId(secretName)
            .build();
        
        return secretsClient.getSecretValue(request).secretString();
    }
    
    @Cacheable("secrets")
    public DatabaseCredentials getDatabaseCredentials() {
        String secret = getSecret("/pix-system/database");
        return objectMapper.readValue(secret, DatabaseCredentials.class);
    }
}
```

---

## 9.7 CloudWatch

### Três Pilares

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLOUDWATCH                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│   │    LOGS     │   │   METRICS   │   │   ALARMS    │           │
│   │             │   │             │   │             │           │
│   │ - App logs  │   │ - CPU       │   │ - Thresholds│           │
│   │ - Access    │   │ - Memory    │   │ - Actions   │           │
│   │ - Errors    │   │ - Custom    │   │ - SNS notify│           │
│   └─────────────┘   └─────────────┘   └─────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Logs com Spring

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CLOUDWATCH" class="ca.pjer.logback.AwsLogsAppender">
        <logGroupName>/pix-system/account-service</logGroupName>
        <logStreamName>${HOSTNAME}</logStreamName>
        <logRegion>sa-east-1</logRegion>
        <layout>
            <pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </layout>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CLOUDWATCH"/>
    </root>
</configuration>
```

### Métricas Customizadas

```java
@Service
@RequiredArgsConstructor
public class TransferMetricsPublisher {
    
    private final CloudWatchClient cloudWatch;
    
    public void publishTransferMetric(Transfer transfer) {
        PutMetricDataRequest request = PutMetricDataRequest.builder()
            .namespace("PixSystem")
            .metricData(
                MetricDatum.builder()
                    .metricName("TransferAmount")
                    .value(transfer.getAmount().amount().doubleValue())
                    .unit(StandardUnit.COUNT)
                    .dimensions(
                        Dimension.builder()
                            .name("Status")
                            .value(transfer.getStatus().name())
                            .build()
                    )
                    .build()
            )
            .build();
        
        cloudWatch.putMetricData(request);
    }
}
```

---

## 9.8 ECS/ECR (Containers)

### ECR (Elastic Container Registry)

**Repositório de imagens Docker na AWS.**

```bash
# Login no ECR
aws ecr get-login-password --region sa-east-1 | \
    docker login --username AWS --password-stdin 123456789.dkr.ecr.sa-east-1.amazonaws.com

# Build e push
docker build -t pix-account-service .
docker tag pix-account-service:latest 123456789.dkr.ecr.sa-east-1.amazonaws.com/pix-account-service:latest
docker push 123456789.dkr.ecr.sa-east-1.amazonaws.com/pix-account-service:latest
```

### ECS (Elastic Container Service)

```
┌─────────────────────────────────────────────────────────────────┐
│                           ECS CLUSTER                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                      SERVICE                              │   │
│   │   (Account Service - desired: 3, running: 3)             │   │
│   │                                                           │   │
│   │   ┌─────────┐    ┌─────────┐    ┌─────────┐              │   │
│   │   │  TASK   │    │  TASK   │    │  TASK   │              │   │
│   │   │ (Pod)   │    │ (Pod)   │    │ (Pod)   │              │   │
│   │   │         │    │         │    │         │              │   │
│   │   │Container│    │Container│    │Container│              │   │
│   │   └─────────┘    └─────────┘    └─────────┘              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│                     ┌─────────────┐                             │
│                     │     ALB     │                             │
│                     │ (Load Bal.) │                             │
│                     └─────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9.9 LocalStack (Desenvolvimento Local)

### O que é?

**Emulador de serviços AWS que roda 100% local no Docker. Gratuito!**

### Configuração

```yaml
# docker-compose.localstack.yml
version: '3.8'
services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566"           # Gateway único
      - "4510-4559:4510-4559" # Serviços externos
    environment:
      - SERVICES=s3,sqs,sns,secretsmanager,ssm,dynamodb,rds
      - DEBUG=1
      - DATA_DIR=/var/lib/localstack/data
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "./localstack-data:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./init-aws:/etc/localstack/init/ready.d"
```

### Scripts de Inicialização

```bash
#!/bin/bash
# init-aws/init.sh

echo "Criando recursos AWS..."

# Criar bucket S3
awslocal s3 mb s3://pix-documents

# Criar filas SQS
awslocal sqs create-queue --queue-name pix-transfer-events
awslocal sqs create-queue --queue-name pix-notification-events

# Criar tópico SNS
awslocal sns create-topic --name pix-events

# Criar secret
awslocal secretsmanager create-secret \
    --name /pix-system/database \
    --secret-string '{"username":"pix","password":"pix123"}'

echo "Recursos AWS criados com sucesso!"
```

### Configuração Spring para LocalStack

```yaml
# application-local.yml
spring:
  profiles: local
  
cloud:
  aws:
    endpoint: http://localhost:4566
    region:
      static: sa-east-1
    credentials:
      access-key: test
      secret-key: test
    s3:
      endpoint: http://localhost:4566
      path-style-access-enabled: true
    sqs:
      endpoint: http://localhost:4566
    sns:
      endpoint: http://localhost:4566
```

```java
// Configuração programática
@Configuration
@Profile("local")
public class LocalStackConfig {
    
    @Bean
    public S3Client s3Client() {
        return S3Client.builder()
            .endpointOverride(URI.create("http://localhost:4566"))
            .region(Region.SA_EAST_1)
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create("test", "test")))
            .forcePathStyle(true)
            .build();
    }
    
    @Bean
    public SqsClient sqsClient() {
        return SqsClient.builder()
            .endpointOverride(URI.create("http://localhost:4566"))
            .region(Region.SA_EAST_1)
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create("test", "test")))
            .build();
    }
}
```

---

## 9.10 SDK AWS para Java

### Dependências (AWS SDK v2)

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bom</artifactId>
            <version>2.21.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- S3 -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
    </dependency>
    
    <!-- SQS -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sqs</artifactId>
    </dependency>
    
    <!-- SNS -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sns</artifactId>
    </dependency>
    
    <!-- Secrets Manager -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>secretsmanager</artifactId>
    </dependency>
    
    <!-- CloudWatch -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>cloudwatch</artifactId>
    </dependency>
    
    <!-- Spring Cloud AWS -->
    <dependency>
        <groupId>io.awspring.cloud</groupId>
        <artifactId>spring-cloud-aws-starter-sqs</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

---

## 🎓 Perguntas de Entrevista

### Nível Pleno

1. **"Qual a diferença entre SQS Standard e FIFO?"**
   - Standard: maior throughput, best-effort ordering
   - FIFO: ordenação garantida, exatamente uma entrega, 300 msg/s

2. **"Como você protege credenciais na AWS?"**
   - Secrets Manager ou Parameter Store
   - IAM Roles (nunca access keys em código)
   - Rotação automática de secrets

3. **"O que é uma VPC e por que usar?"**
   - Rede virtual isolada
   - Controle de tráfego (Security Groups, NACLs)
   - Separação público/privado

### Nível Senior

4. **"Como garantir alta disponibilidade no RDS?"**
   - Multi-AZ para failover automático
   - Read Replicas para leitura
   - Backups automáticos com point-in-time recovery

5. **"Descreva uma arquitetura event-driven na AWS"**
   - Eventos → SNS → SQS (fan-out)
   - Desacoplamento entre serviços
   - Dead letter queues para falhas
   - CloudWatch para monitoramento

---

## 📚 Recursos Adicionais

- [AWS SDK for Java 2.x Documentation](https://docs.aws.amazon.com/sdk-for-java/)
- [Spring Cloud AWS](https://awspring.io/)
- [LocalStack Documentation](https://docs.localstack.cloud/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

**Próximo:** [MODULO_10_KUBERNETES.md](MODULO_10_KUBERNETES.md) - Orquestração de containers
