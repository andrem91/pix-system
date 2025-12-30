# 🏦 Projeto Integrador: Sistema Pix Completo

> Aplicação prática de **todos os conceitos** estudados em um sistema bancário real, incluindo **Cloud AWS**, **Kubernetes** e **Terraform**.

---

## 📌 Visão Geral

Um sistema de **Pix completo** que demonstra proficiência em:

- **Java Core:** Collections, Streams, Optional, Concorrência
- **Arquitetura:** DDD, Arquitetura Hexagonal, CQRS, Event Sourcing
- **Spring Ecosystem:** Boot, Data, Security, Cloud, Scheduling
- **Microsserviços:** API Gateway, Service Discovery, Comunicação síncrona/assíncrona
- **Qualidade:** TDD, Testes de Integração, TestContainers
- **DevOps:** Docker, CI/CD, Observabilidade
- **☁️ Cloud AWS:** RDS, SQS/SNS, S3, Secrets Manager, EKS, LocalStack
- **☸️ Kubernetes:** Deployments, Services, Ingress, HPA, Helm
- **🏗️ Terraform:** IaC, Módulos, State Management

### 🆕 Funcionalidades 2025

| Funcionalidade | Descrição | Status |
|----------------|-----------|--------|
| **Pix Instantâneo** | Transferência tradicional em tempo real | ✅ Core |
| **Pix Automático** | Pagamentos recorrentes sem autenticação (mandatos) | 🆕 Sprint 6 |
| **Pix Parcelado** | Parcelamento com crédito na interface do Pix | 🆕 Sprint 7 |

---

## 🏛️ Arquitetura do Sistema

### Visão Macro (Evolução)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FASE 1 (Sprint 0-2)                            │
│  ┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐        │
│  │ API Gateway  │────►│  Account Service │◄───►│ Transfer Service │        │
│  │   (Spring    │     │   (Contas, Pix   │     │  (Transferências)│        │
│  │   Cloud)     │     │    Keys, Saldo)  │     │                  │        │
│  └──────────────┘     └────────┬─────────┘     └────────┬─────────┘        │
│                                │                        │                   │
│                                └────────┬───────────────┘                   │
│                                         ▼                                   │
│                                ┌─────────────────┐                          │
│                                │     Kafka       │                          │
│                                │  (Eventos Pix)  │                          │
│                                └─────────────────┘                          │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              FASE 2 (Sprint 3-4)                            │
│  ┌──────────────┐                                                           │
│  │ API Gateway  │                                                           │
│  └──────┬───────┘                                                           │
│         │        ┌──────────────────┐                                       │
│         ├───────►│  Account Service │◄──────┐                               │
│         │        └────────┬─────────┘       │                               │
│         │                 │                 │                               │
│         │        ┌────────▼─────────┐       │                               │
│         ├───────►│ Transfer Service │       │                               │
│         │        └────────┬─────────┘       │                               │
│         │                 │                 │                               │
│         │        ┌────────▼─────────┐       │                               │
│         └───────►│  Notification    │◄──────┤                               │
│                  │    Service       │       │                               │
│                  └──────────────────┘       │                               │
│                                             │                               │
│         ┌───────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │     Kafka       │  │   Eureka        │  │   Jaeger        │             │
│  │  (Event Bus)    │  │ (Discovery)     │  │ (Tracing)       │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        FASE 3 (Sprint 6-7) - NOVIDADES 2025                  │
│  ┌──────────────┐                                                           │
│  │ API Gateway  │                                                           │
│  └──────┬───────┘                                                           │
│         │                                                                    │
│   ┌─────┴─────────────────────────────────────────────────────────────┐     │
│   │     │           │              │              │              │    │     │
│   ▼     ▼           ▼              ▼              ▼              ▼    │     │
│ ┌─────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│ │ Account │   │ Transfer │   │Recurring │   │Installmt │   │Notificat.│    │
│ │ Service │   │ Service  │   │ Service  │   │ Service  │   │ Service  │    │
│ │         │   │          │   │   🆕     │   │   🆕     │   │          │    │
│ │- Contas │   │- Pix     │   │- Pix     │   │- Pix     │   │- Email   │    │
│ │- Chaves │   │  Instant │   │Automático│   │Parcelado │   │- Push    │    │
│ │- Saldo  │   │- Events  │   │- Mandatos│   │- Crédito │   │- SMS     │    │
│ └────┬────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘    │
│      │             │              │              │              │          │
│      └─────────────┴──────────────┴──────────────┴──────────────┘          │
│                                   │                                         │
│                                   ▼                                         │
│      ┌───────────────────────────────────────────────────────────┐         │
│      │                    INFRAESTRUTURA                          │         │
│      │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌──────┐ │         │
│      │  │ Kafka  │  │ Eureka │  │ Jaeger │  │  Redis │  │Quartz│ │         │
│      │  │ Events │  │Registry│  │Tracing │  │ Cache  │  │ Jobs │ │         │
│      │  └────────┘  └────────┘  └────────┘  └────────┘  └──────┘ │         │
│      └───────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                   FASE 4 (Sprint 8-11) - CLOUD & INFRAESTRUTURA             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         AWS (ou LocalStack)                             │ │
│  ├────────────────────────────────────────────────────────────────────────┤ │
│  │                                                                         │ │
│  │   ┌───────────────┐                                                    │ │
│  │   │ AWS API GW    │ (ou ALB + Ingress)                                 │ │
│  │   │ + WAF         │                                                    │ │
│  │   └───────┬───────┘                                                    │ │
│  │           │                                                             │ │
│  │   ┌───────▼────────────────────────────────────────────────────────┐   │ │
│  │   │                    EKS (Kubernetes)                             │   │ │
│  │   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │   │ │
│  │   │  │Account  │ │Transfer │ │Recurring│ │Installmt│ │Notific. │  │   │ │
│  │   │  │Service  │ │Service  │ │Service  │ │Service  │ │Service  │  │   │ │
│  │   │  │(Pod x3) │ │(Pod x3) │ │(Pod x2) │ │(Pod x2) │ │(Pod x2) │  │   │ │
│  │   │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘  │   │ │
│  │   │       │           │           │           │           │       │   │ │
│  │   │  Helm Charts + HPA (Auto Scaling)                            │   │ │
│  │   └──────────────────────────┬────────────────────────────────────┘   │ │
│  │                              │                                         │ │
│  │   ┌──────────────────────────▼────────────────────────────────────┐   │ │
│  │   │                    AWS MANAGED SERVICES                        │   │ │
│  │   │                                                                │   │ │
│  │   │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │   │ │
│  │   │   │   RDS   │ │   SQS   │ │   S3    │ │ Secrets │ │CloudWat.│ │   │ │
│  │   │   │Postgres │ │   SNS   │ │Documents│ │ Manager │ │  Logs   │ │   │ │
│  │   │   │Multi-AZ │ │         │ │         │ │         │ │ Metrics │ │   │ │
│  │   │   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │   │ │
│  │   │                                                                │   │ │
│  │   │   ┌─────────┐ ┌─────────┐                                     │   │ │
│  │   │   │  ECR    │ │ElastiC. │                                     │   │ │
│  │   │   │ Images  │ │ (Redis) │                                     │   │ │
│  │   │   └─────────┘ └─────────┘                                     │   │ │
│  │   └────────────────────────────────────────────────────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         TERRAFORM (IaC)                                 │ │
│  │   Provisiona VPC, EKS, RDS, SQS, S3, IAM, Security Groups, etc.        │ │
│  │   State remoto no S3 + DynamoDB locking                                │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Arquitetura Hexagonal (Por Serviço)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TRANSFER SERVICE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐                              ┌─────────────┐       │
│  │   REST API  │                              │   Kafka     │       │
│  │  (Adapter)  │                              │  Consumer   │       │
│  └──────┬──────┘                              └──────┬──────┘       │
│         │            ADAPTERS (INBOUND)              │              │
│  ═══════╪════════════════════════════════════════════╪══════════    │
│         │                                            │              │
│         ▼                 PORTS                      ▼              │
│  ┌─────────────┐                              ┌─────────────┐       │
│  │  Transfer   │                              │   Event     │       │
│  │   Port      │                              │   Handler   │       │
│  └──────┬──────┘                              └──────┬──────┘       │
│         │                                            │              │
│         └──────────────┬─────────────────────────────┘              │
│                        ▼                                            │
│               ┌─────────────────┐                                   │
│               │     DOMAIN      │                                   │
│               │                 │                                   │
│               │  - Transfer     │                                   │
│               │  - PixKey       │                                   │
│               │  - Money (VO)   │                                   │
│               │  - Events       │                                   │
│               └────────┬────────┘                                   │
│                        │                                            │
│         ┌──────────────┴─────────────────────────────┐              │
│         ▼                 PORTS                      ▼              │
│  ┌─────────────┐                              ┌─────────────┐       │
│  │ Repository  │                              │  Account    │       │
│  │   Port      │                              │  Client     │       │
│  └──────┬──────┘                              └──────┬──────┘       │
│         │            ADAPTERS (OUTBOUND)             │              │
│  ═══════╪════════════════════════════════════════════╪══════════    │
│         ▼                                            ▼              │
│  ┌─────────────┐                              ┌─────────────┐       │
│  │   JPA       │                              │   Feign     │       │
│  │  Repository │                              │   Client    │       │
│  └─────────────┘                              └─────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Estrutura dos Microsserviços

### Estrutura Raiz
```
pix-system/
├── pix-account-service/          # Serviço de Contas e Chaves Pix
├── pix-transfer-service/         # Serviço de Transferências
├── pix-recurring-service/        # 🆕 Serviço de Pix Automático (Fase 3)
├── pix-installment-service/      # 🆕 Serviço de Pix Parcelado (Fase 3)
├── pix-notification-service/     # Serviço de Notificações (Fase 2)
├── pix-api-gateway/              # API Gateway
├── pix-service-discovery/        # Eureka Server
├── pix-commons/                  # Módulo compartilhado (DTOs, Events)
├── docker-compose.yml
├── docker-compose.infra.yml      # Kafka, Postgres, Redis, etc
└── README.md
```

### Account Service (Arquitetura Hexagonal)
```
pix-account-service/
├── src/main/java/com/bank/account/
│   │
│   ├── domain/                           # 🎯 CORE (Nenhuma dependência externa)
│   │   ├── model/
│   │   │   ├── Account.java              # Aggregate Root
│   │   │   ├── PixKey.java               # Entity
│   │   │   ├── PixKeyType.java           # Enum (CPF, EMAIL, PHONE, RANDOM)
│   │   │   └── AccountStatus.java        # Enum
│   │   ├── vo/                           # Value Objects (Imutáveis)
│   │   │   ├── Money.java
│   │   │   ├── Cpf.java
│   │   │   └── AccountId.java
│   │   ├── event/                        # Domain Events
│   │   │   ├── AccountCreatedEvent.java
│   │   │   ├── PixKeyRegisteredEvent.java
│   │   │   └── BalanceUpdatedEvent.java
│   │   ├── exception/
│   │   │   ├── InsufficientBalanceException.java
│   │   │   ├── PixKeyAlreadyExistsException.java
│   │   │   └── AccountNotFoundException.java
│   │   └── service/                      # Domain Services
│   │       └── PixKeyValidator.java
│   │
│   ├── application/                      # 🔄 USE CASES (Orquestração)
│   │   ├── port/
│   │   │   ├── in/                       # Ports de entrada (interfaces)
│   │   │   │   ├── CreateAccountUseCase.java
│   │   │   │   ├── RegisterPixKeyUseCase.java
│   │   │   │   ├── GetAccountUseCase.java
│   │   │   │   └── UpdateBalanceUseCase.java
│   │   │   └── out/                      # Ports de saída (interfaces)
│   │   │       ├── AccountRepository.java
│   │   │       ├── PixKeyRepository.java
│   │   │       └── EventPublisher.java
│   │   └── service/                      # Implementações dos Use Cases
│   │       ├── CreateAccountService.java
│   │       ├── RegisterPixKeyService.java
│   │       └── AccountQueryService.java
│   │
│   └── adapter/                          # 🔌 ADAPTERS (Infraestrutura)
│       ├── in/                           # Adapters de entrada
│       │   └── web/
│       │       ├── AccountController.java
│       │       ├── PixKeyController.java
│       │       ├── dto/
│       │       │   ├── request/
│       │       │   │   ├── CreateAccountRequest.java
│       │       │   │   └── RegisterPixKeyRequest.java
│       │       │   └── response/
│       │       │       ├── AccountResponse.java
│       │       │       └── PixKeyResponse.java
│       │       └── mapper/
│       │           └── AccountDtoMapper.java
│       │
│       └── out/                          # Adapters de saída
│           ├── persistence/
│           │   ├── entity/
│           │   │   ├── AccountJpaEntity.java
│           │   │   └── PixKeyJpaEntity.java
│           │   ├── repository/
│           │   │   ├── AccountJpaRepository.java
│           │   │   └── PixKeyJpaRepository.java
│           │   ├── mapper/
│           │   │   └── AccountPersistenceMapper.java
│           │   └── AccountRepositoryAdapter.java  # Implementa Port
│           │
│           └── messaging/
│               └── KafkaEventPublisher.java       # Implementa Port
│
├── src/main/resources/
│   ├── application.yml
│   ├── application-docker.yml
│   └── db/migration/                     # Flyway
│       ├── V1__create_accounts.sql
│       └── V2__create_pix_keys.sql
│
└── src/test/java/com/bank/account/
    ├── domain/                           # Testes de domínio (unitários puros)
    │   └── model/AccountTest.java
    ├── application/                      # Testes de use cases
    │   └── service/CreateAccountServiceTest.java
    └── adapter/                          # Testes de adapters
        ├── in/web/AccountControllerTest.java
        └── out/persistence/AccountRepositoryAdapterTest.java
```

### Transfer Service (Arquitetura Hexagonal + Event Sourcing)
```
pix-transfer-service/
├── src/main/java/com/bank/transfer/
│   │
│   ├── domain/
│   │   ├── model/
│   │   │   ├── Transfer.java             # Aggregate Root
│   │   │   ├── TransferStatus.java       # Enum
│   │   │   └── TransferType.java         # PIX_KEY, PIX_MANUAL
│   │   ├── vo/
│   │   │   ├── TransferId.java
│   │   │   ├── Money.java
│   │   │   └── PixKey.java
│   │   ├── event/                        # Event Sourcing Events
│   │   │   ├── TransferEvent.java        # Base
│   │   │   ├── TransferInitiatedEvent.java
│   │   │   ├── TransferValidatedEvent.java
│   │   │   ├── TransferCompletedEvent.java
│   │   │   └── TransferFailedEvent.java
│   │   └── exception/
│   │       ├── TransferNotFoundException.java
│   │       └── InvalidTransferException.java
│   │
│   ├── application/
│   │   ├── port/
│   │   │   ├── in/
│   │   │   │   ├── InitiateTransferUseCase.java
│   │   │   │   ├── ProcessTransferUseCase.java
│   │   │   │   └── GetTransferUseCase.java
│   │   │   └── out/
│   │   │       ├── TransferEventStore.java       # Event Sourcing
│   │   │       ├── AccountClient.java            # Feign
│   │   │       └── TransferEventPublisher.java
│   │   └── service/
│   │       ├── InitiateTransferService.java
│   │       └── TransferQueryService.java
│   │
│   └── adapter/
│       ├── in/
│       │   ├── web/
│       │   │   ├── TransferController.java
│       │   │   └── dto/
│       │   └── messaging/
│       │       └── AccountEventConsumer.java     # Consome eventos de Account
│       │
│       └── out/
│           ├── persistence/
│           │   ├── eventstore/
│           │   │   ├── TransferEventJpaEntity.java
│           │   │   └── JpaTransferEventStore.java
│           │   └── projection/                   # CQRS - Read Model
│           │       ├── TransferProjection.java
│           │       └── TransferProjectionRepository.java
│           ├── feign/
│           │   ├── AccountFeignClient.java
│           │   └── AccountClientAdapter.java
│           └── messaging/
│               └── KafkaTransferEventPublisher.java
│
└── src/test/                             # Estrutura similar
```

---

## 🔄 Fluxo de uma Transferência Pix

```
┌─────────┐          ┌───────────┐          ┌─────────────┐          ┌──────────┐
│ Cliente │          │API Gateway│          │  Transfer   │          │ Account  │
└────┬────┘          └─────┬─────┘          │   Service   │          │ Service  │
     │                     │                └──────┬──────┘          └────┬─────┘
     │  POST /pix/transfer │                       │                      │
     │────────────────────>│                       │                      │
     │                     │  Route to Transfer    │                      │
     │                     │──────────────────────>│                      │
     │                     │                       │                      │
     │                     │                       │ GET /accounts/{pixKey}
     │                     │                       │─────────────────────>│
     │                     │                       │                      │
     │                     │                       │   Account details    │
     │                     │                       │<─────────────────────│
     │                     │                       │                      │
     │                     │                       │ POST /accounts/debit │
     │                     │                       │─────────────────────>│
     │                     │                       │                      │
     │                     │                       │      Success         │
     │                     │                       │<─────────────────────│
     │                     │                       │                      │
     │                     │                       │ POST /accounts/credit│
     │                     │                       │─────────────────────>│
     │                     │                       │                      │
     │                     │                       │      Success         │
     │                     │                       │<─────────────────────│
     │                     │                       │                      │
     │                     │                       ├──┐ Publish           │
     │                     │                       │  │ TransferCompleted │
     │                     │                       │<─┘ to Kafka          │
     │                     │                       │                      │
     │                     │   Transfer Response   │                      │
     │                     │<──────────────────────│                      │
     │                     │                       │                      │
     │   201 Created       │                       │                      │
     │<────────────────────│                       │                      │
     │                     │                       │                      │
```

---

## 📋 Módulos do Estudo Aplicados

### Mapeamento Conceito → Implementação

| Conceito | Onde Aplicar | Arquivo/Componente |
|----------|--------------|-------------------|
| **Collections** | Chaves Pix de uma conta | `Account.pixKeys: Set<PixKey>` |
| **Streams** | Filtrar transferências, calcular totais | `TransferQueryService` |
| **Optional** | Busca de conta por Pix Key | `AccountRepository.findByPixKey()` |
| **Records** | DTOs de request/response | `CreateAccountRequest.java` |
| **DDD Aggregates** | Account, Transfer, Mandate, Contract | Aggregate Roots do domínio |
| **Value Objects** | Money, Cpf, AccountId | Imutáveis, sem identidade |
| **Domain Events** | TransferCompleted, MandateExecuted | Comunicação entre agregados |
| **Hexagonal** | Ports & Adapters | Toda estrutura de pacotes |
| **TDD** | Red-Green-Refactor | Todo desenvolvimento |
| **Event Sourcing** | Histórico de transferências | `TransferEventStore` |
| **CQRS** | Separar write/read | Command vs Query services |
| **Saga Pattern** | Transferência multi-step | Orquestração no TransferService |
| **Circuit Breaker** | Chamada ao AccountService | Resilience4j no FeignClient |
| **API Gateway** | Entry point único | Spring Cloud Gateway |
| **Service Discovery** | Registro de serviços | Eureka |
| **Distributed Tracing** | Rastrear requisições | Jaeger/Zipkin |
| **State Machine** 🆕 | Ciclo de vida do mandato | `RecurringMandate` |
| **Scheduler** 🆕 | Execuções automáticas | `@Scheduled`, Quartz |
| **Strategy Pattern** 🆕 | Cálculo de taxas de juros | `InterestRateStrategy` |
| **Domain Service** 🆕 | Cálculos financeiros complexos | `InstallmentCalculator` |

---

## ✅ Checklist de Funcionalidades

### Sprint 0: Setup & Domínio
- [ ] Estrutura do monorepo criada
- [ ] Account Service: entidades básicas
- [ ] Value Objects: Money, Cpf, AccountId
- [ ] Domain Events definidos
- [ ] Testes unitários do domínio

### Sprint 1: CRUD & Hexagonal
- [ ] Ports e Adapters implementados
- [ ] Account: criar, buscar, atualizar saldo
- [ ] Chaves Pix: registrar, listar, remover
- [ ] JPA Entities separadas das Domain Entities
- [ ] Mappers entre camadas

### Sprint 2: Transferências & Event Sourcing
- [ ] Transfer Service criado
- [ ] Event Store implementado
- [ ] Fluxo de transferência completo
- [ ] Comunicação síncrona (Feign)
- [ ] Circuit Breaker configurado

### Sprint 3: Mensageria & CQRS
- [ ] Kafka producer/consumer
- [ ] Read Model (projeções)
- [ ] Notification Service (básico)
- [ ] Eventos assíncronos funcionando

### Sprint 4: Infraestrutura & Observabilidade
- [ ] API Gateway configurado
- [ ] Eureka Server funcionando
- [ ] Docker Compose completo
- [ ] Distributed Tracing (Jaeger)
- [ ] GitHub Actions CI

### Sprint 5: Polimento & Produção
- [ ] Rate Limiting no Gateway
- [ ] Health Checks customizados
- [ ] Métricas com Micrometer
- [ ] Documentação OpenAPI
- [ ] Testes de carga básicos

### Sprint 6: 🆕 Pix Automático (2025)
- [ ] Recurring Service criado
- [ ] RecurringMandate (Aggregate Root)
- [ ] State Machine (PENDING → ACTIVE → CANCELLED)
- [ ] Scheduler para execuções automáticas
- [ ] Notificação prévia ao pagador (T-2 dias)
- [ ] Idempotência nas execuções
- [ ] Gestão de mandatos pelo usuário

### Sprint 7: 🆕 Pix Parcelado (2025)
- [ ] Installment Service criado
- [ ] InstallmentContract (Aggregate Root)
- [ ] Calculadora financeira (Price, IOF, CET)
- [ ] Strategy Pattern para taxas de juros
- [ ] Simulação de parcelas
- [ ] Contratação com análise de limite
- [ ] Gestão de parcelas (baixa, antecipação)
- [ ] Integração com Account Service para limites

### Sprint 8: ☸️ Kubernetes
- [ ] Minikube/Kind rodando localmente
- [ ] Dockerfiles multi-stage otimizados
- [ ] Deployment + Service para cada microsserviço
- [ ] ConfigMaps e Secrets
- [ ] Health checks (liveness/readiness)
- [ ] Ingress Controller configurado
- [ ] HPA (Horizontal Pod Autoscaler)
- [ ] Helm Charts criados

### Sprint 9: ☁️ AWS com LocalStack
- [ ] LocalStack rodando via Docker
- [ ] SQS substituindo Kafka (para alguns casos)
- [ ] SNS para pub/sub entre serviços
- [ ] S3 para armazenamento de comprovantes
- [ ] Secrets Manager para credenciais
- [ ] Configuração Spring Cloud AWS
- [ ] Testes de integração com LocalStack

### Sprint 10: 🏗️ Terraform
- [ ] Estrutura de módulos Terraform
- [ ] VPC, Subnets, Security Groups
- [ ] RDS PostgreSQL
- [ ] SQS, SNS, S3
- [ ] IAM Roles e Policies
- [ ] Módulo EKS (básico)
- [ ] State remoto no S3 + DynamoDB
- [ ] Terraform + LocalStack funcionando

### Sprint 11: 🚀 Integração Final
- [ ] CI/CD com deploy no Kubernetes
- [ ] Ambiente completo subindo com um comando
- [ ] Documentação de arquitetura
- [ ] Runbook de operações
- [ ] Testes end-to-end completos
- [ ] README profissional com badges

---

## 🧪 Abordagem TDD

### Ciclo Red-Green-Refactor

```java
// 1. RED: Escreva o teste PRIMEIRO (falha)
@Test
@DisplayName("Deve criar conta com saldo inicial zero")
void deveCriarContaComSaldoInicialZero() {
    // Given
    var command = new CreateAccountCommand("João", new Cpf("12345678900"));
    
    // When
    Account account = createAccountService.execute(command);
    
    // Then
    assertThat(account.getBalance()).isEqualTo(Money.ZERO);
    assertThat(account.getStatus()).isEqualTo(AccountStatus.ACTIVE);
}

// 2. GREEN: Implemente o MÍNIMO para passar
public Account execute(CreateAccountCommand command) {
    return new Account(command.name(), command.cpf());
}

// 3. REFACTOR: Melhore sem quebrar o teste
```

### Estrutura de Testes

| Tipo | O que testar | Framework |
|------|--------------|-----------|
| **Unitário Domínio** | Value Objects, Entities, Domain Services | JUnit 5, AssertJ |
| **Unitário Application** | Use Cases (mock ports) | Mockito |
| **Integração Adapter** | Repositories, Clients | TestContainers, WireMock |
| **Controller** | REST endpoints | @WebMvcTest |
| **End-to-End** | Fluxo completo | @SpringBootTest + TestContainers |

---

## 🚀 Como Começar

### Pré-requisitos
- Java 21+
- Docker & Docker Compose
- Maven 3.8+
- IDE com suporte a Lombok

### Ordem de Desenvolvimento (TDD)

```
1. Escreva teste para Value Object (Money)
   └─> Implemente Money
   
2. Escreva teste para Entity (Account)
   └─> Implemente Account
   
3. Escreva teste para Use Case (CreateAccountService)
   └─> Implemente Service + Ports
   
4. Escreva teste para Adapter (AccountRepositoryAdapter)
   └─> Implemente Adapter + JPA Entity
   
5. Escreva teste para Controller (AccountController)
   └─> Implemente Controller
```

---

## 📝 Endpoints Principais

### Account Service
```
POST   /api/v1/accounts                    # Criar conta
GET    /api/v1/accounts/{id}               # Buscar conta
GET    /api/v1/accounts?pixKey={key}       # Buscar por chave Pix
POST   /api/v1/accounts/{id}/pix-keys      # Registrar chave Pix
DELETE /api/v1/accounts/{id}/pix-keys/{keyId}  # Remover chave
POST   /api/v1/accounts/{id}/debit         # Debitar (interno)
POST   /api/v1/accounts/{id}/credit        # Creditar (interno)
```

### Transfer Service
```
POST   /api/v1/transfers                   # Iniciar transferência
GET    /api/v1/transfers/{id}              # Buscar transferência
GET    /api/v1/transfers/{id}/events       # Histórico de eventos
GET    /api/v1/transfers?accountId={id}    # Listar por conta
```

### 🆕 Recurring Service (Pix Automático)
```
POST   /api/v1/recurring/mandates                    # Criar mandato
GET    /api/v1/recurring/mandates/{id}               # Buscar mandato
GET    /api/v1/recurring/mandates?accountId={id}     # Listar mandatos da conta
PUT    /api/v1/recurring/mandates/{id}/confirm       # Confirmar mandato
PUT    /api/v1/recurring/mandates/{id}/suspend       # Suspender mandato
PUT    /api/v1/recurring/mandates/{id}/reactivate    # Reativar mandato
DELETE /api/v1/recurring/mandates/{id}               # Cancelar mandato
GET    /api/v1/recurring/mandates/{id}/executions    # Histórico de execuções
```

### 🆕 Installment Service (Pix Parcelado)
```
POST   /api/v1/installments/simulate                 # Simular parcelamento
POST   /api/v1/installments/contracts                # Contratar parcelamento
GET    /api/v1/installments/contracts/{id}           # Buscar contrato
GET    /api/v1/installments/contracts?accountId={id} # Listar contratos da conta
GET    /api/v1/installments/contracts/{id}/parcels   # Listar parcelas
POST   /api/v1/installments/contracts/{id}/anticipate # Antecipar parcelas
GET    /api/v1/installments/limit/{accountId}        # Consultar limite disponível
```

### API Gateway
```
/accounts/**      → pix-account-service
/transfers/**     → pix-transfer-service
/recurring/**     → pix-recurring-service
/installments/**  → pix-installment-service
/notifications/** → pix-notification-service
```

---

## 🎓 Dicas para Entrevistas

Ao discutir este projeto, prepare-se para explicar:

1. **"Por que separar Domain Entity de JPA Entity?"**
   - Domínio não deve depender de framework
   - Permite trocar banco sem alterar lógica

2. **"Qual a vantagem do Event Sourcing aqui?"**
   - Auditoria completa (regulatório bancário)
   - Histórico imutável de transferências
   - Permite rebuild do estado

3. **"Como você garante idempotência?"**
   - IdempotencyKey no request
   - Verificação antes de processar

4. **"O que acontece se o Account Service estiver fora?"**
   - Circuit Breaker abre
   - Fallback retorna erro gracioso
   - Retry com backoff exponencial

5. **"Como você implementou TDD?"**
   - Teste primeiro, código depois
   - Cobertura > 80%
   - Testes como documentação

6. **🆕 "Como funciona o Pix Automático?"**
   - Mandato = autorização do pagador
   - State Machine controla ciclo de vida
   - Scheduler executa diariamente
   - Notificação T-2 antes do débito
   - Idempotência: verifico se já executou no dia

7. **🆕 "Como você calculou as parcelas do Pix Parcelado?"**
   - Fórmula Price para parcelas fixas
   - IOF diário + fixo conforme regulação
   - CET (Custo Efetivo Total) calculado
   - Strategy Pattern para diferentes taxas por perfil

8. **🆕 "Por que usar State Machine no mandato?"**
   - Transições explícitas e validadas
   - Impossível ir de CANCELLED para ACTIVE
   - Facilita testes e auditoria
   - Documenta o comportamento do domínio

9. **🆕 "Como garantir que o scheduler não debite duas vezes?"**
   - Cada execução tem data única
   - Verifico `mandate.hasExecutionFor(today)` antes
   - Chave composta (mandateId + date) no banco
   - Transação garante atomicidade

---

**Bons estudos e boa implementação! 🚀🏦**
