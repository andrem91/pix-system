# 🏦 Pix System

> Sistema de pagamentos instantâneos completo, construído com **Microsserviços**, **DDD**, **Arquitetura Hexagonal**, **AWS** e **Kubernetes**.

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=andrem91_pix-system&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=andrem91_pix-system)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=andrem91_pix-system&metric=coverage)](https://sonarcloud.io/summary/new_code?id=andrem91_pix-system)

---

## 📋 Sobre o Projeto

Este projeto é uma implementação completa de um sistema Pix, desenvolvido para fins de **estudo e portfólio**. Abrange desde conceitos fundamentais de Java até infraestrutura cloud com AWS e Kubernetes.

### 🎯 Objetivos

- Aplicar **DDD** (Domain-Driven Design) com Value Objects, Entities e Aggregates
- Implementar **Arquitetura Hexagonal** (Ports & Adapters)
- Desenvolver **Microsserviços** com Spring Boot 3.5
- Utilizar **Event Sourcing** e **CQRS** para auditoria
- Configurar **CI/CD** com GitHub Actions
- Deploy em **Kubernetes** com **Terraform**

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY                                 │
│                         (Spring Cloud Gateway)                          │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│    Account    │      │   Transfer    │      │  Notification │
│    Service    │◄────►│    Service    │─────►│    Service    │
│  (Contas Pix) │      │(Transferências)│      │    (Email)    │
└───────┬───────┘      └───────┬───────┘      └───────────────┘
        │                      │
        ▼                      ▼
   PostgreSQL              Event Store
                              │
                              ▼
                     ┌───────────────┐
                     │     Kafka     │
                     │    (Events)   │
                     └───────────────┘
```

---

## 🛠️ Tecnologias

| Categoria | Tecnologias |
|-----------|-------------|
| **Backend** | Java 21, Spring Boot 3.5, Spring Cloud |
| **Arquitetura** | DDD, Hexagonal, Event Sourcing, CQRS |
| **Banco de Dados** | PostgreSQL, Redis |
| **Mensageria** | Apache Kafka |
| **DevOps** | Docker, GitHub Actions, SonarCloud |
| **Cloud** | AWS (LocalStack), Kubernetes (Minikube), Terraform |
| **Testes** | JUnit 5, Mockito, TestContainers, WireMock |

---

## 📁 Estrutura do Projeto

```
pix-system/
├── pix-account-service/       # Gerenciamento de contas e chaves Pix
├── pix-transfer-service/      # Processamento de transferências
├── pix-notification-service/  # Envio de notificações
├── pix-api-gateway/           # Gateway com roteamento
├── pix-service-discovery/     # Eureka Server
├── infrastructure/
│   ├── docker/                # Docker Compose
│   ├── kubernetes/            # Manifests K8s + Helm
│   └── terraform/             # IaC para AWS
└── docs/                      # Documentação adicional
```

---

## 🚀 Como Executar

### Pré-requisitos

- Java 21
- Docker e Docker Compose
- Maven 3.9+

### Desenvolvimento Local

```bash
# Clone o repositório
git clone https://github.com/andrem91/pix-system.git
cd pix-system

# Suba a infraestrutura
docker-compose up -d postgres kafka redis

# Execute o serviço de contas
cd pix-account-service
./mvnw spring-boot:run
```

### Testes

```bash
# Testes unitários e integração
./mvnw verify

# Mutation testing
./mvnw test-compile org.pitest:pitest-maven:mutationCoverage

# Cobertura de código
./mvnw jacoco:report
```

---

## 📚 Módulos de Estudo

Este projeto faz parte de um plano de estudos completo para Java Pleno/Senior:

| Sprint | Foco |
|--------|------|
| 0 | Setup + Value Objects (CPF, Money, PixKey) |
| 1-2 | Account Service + Event Sourcing |
| 3-4 | Transfer Service + Saga Pattern |
| 5 | DevOps + DevSecOps |
| 6-7 | Pix Automático + Pix Parcelado |
| 8-11 | Kubernetes + AWS + Terraform |

---

## 🤝 Contribuição

Este é um projeto de estudo pessoal, mas sugestões são bem-vindas!

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.
