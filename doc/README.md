# 📚 Plano de Estudos Java Pleno - Guia Principal

> **Objetivo:** Preparação completa para entrevistas de Java Pleno/Senior em **20+ semanas**.

---

## 📁 Estrutura dos Arquivos

### Módulos de Estudo

| Arquivo | Conteúdo | Duração |
|---------|----------|---------|
| [MODULO_01_FUNDAMENTOS_JAVA.md](MODULO_01_FUNDAMENTOS_JAVA.md) | Collections, Streams, Concorrência, Memória | 3 semanas |
| [MODULO_02_JAVA_MODERNO.md](MODULO_02_JAVA_MODERNO.md) | Records, Sealed Classes, Pattern Matching | 1 semana |
| [MODULO_03_OOP_SOLID.md](MODULO_03_OOP_SOLID.md) | Princípios SOLID, Design Patterns, DDD | 1 semana |
| [MODULO_04_SPRING.md](MODULO_04_SPRING.md) | DI, Transações, Security, Exception Handling | 2 semanas |
| [MODULO_05_JPA_PERSISTENCIA.md](MODULO_05_JPA_PERSISTENCIA.md) | Entity Lifecycle, N+1, Locking, Flyway | 2 semanas |
| [MODULO_06_ARQUITETURA.md](MODULO_06_ARQUITETURA.md) | REST, Feign, Kafka, Circuit Breaker, DDD | 2 semanas |
| [MODULO_07_TESTES.md](MODULO_07_TESTES.md) | JUnit 5, Mockito, TestContainers, Contract | 1 semana |
| [MODULO_08_DEVOPS.md](MODULO_08_DEVOPS.md) | Docker, CI/CD, Observabilidade, DevSecOps | 1 semana |
| [MODULO_09_AWS_CLOUD.md](MODULO_09_AWS_CLOUD.md) | ☁️ IAM, RDS, SQS/SNS, S3, LocalStack | 2 semanas |
| [MODULO_10_KUBERNETES.md](MODULO_10_KUBERNETES.md) | ☸️ Pods, Deployments, Services, Helm | 2 semanas |
| [MODULO_11_TERRAFORM.md](MODULO_11_TERRAFORM.md) | 🏗️ IaC, HCL, Modules, State Management | 1 semana |
| [MODULO_12_SEGURANCA.md](MODULO_12_SEGURANCA.md) | 🔐 OAuth2, Keycloak, JWT, Service-to-Service | 1 semana |
| [MODULO_13_SPRING_MODULITH.md](MODULO_13_SPRING_MODULITH.md) | 🧩 Modulith, Arquitetura, Idempotência AOP | 1 semana |

### Projeto Prático

| Arquivo | Conteúdo | Duração |
|---------|----------|---------|
| [PROJETO_INTEGRADOR.md](PROJETO_INTEGRADOR.md) | 🏦 Sistema Pix Completo (AWS + K8s + Terraform) | 8-10 semanas |

---

## 📋 Arquivos Complementares

| Arquivo | Descrição |
|---------|-----------|
| [PERGUNTAS_ENTREVISTA.md](PERGUNTAS_ENTREVISTA.md) | 38+ perguntas com respostas modelo |
| [CHECKLIST_PREPARACAO.md](CHECKLIST_PREPARACAO.md) | Checklist de auto-avaliação |
| [ROTEIRO_IMPLEMENTACAO.md](ROTEIRO_IMPLEMENTACAO.md) | 🌟 Roteiro para estudo integrado (teoria + prática) |

---

## 📅 Cronograma Visual

```
Semana 1-3   ████████████████████░░░░░░░░░░░░░░░░░░░░░░░  Módulo 1: Fundamentos
Semana 4     ░░░░░░░░░░░░░░░░░░░░████░░░░░░░░░░░░░░░░░░░  Módulo 2: Java Moderno
Semana 5     ░░░░░░░░░░░░░░░░░░░░░░░░████░░░░░░░░░░░░░░░  Módulo 3: OOP/SOLID
Semana 6-7   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░████████░░░░░░░  Módulo 4: Spring
Semana 8-9   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░████████  Módulo 5: JPA
Semana 10-11 ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Módulo 6: Arquitetura
Semana 12    ░░░░░░░░████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Módulo 7: Testes
Semana 13    ░░░░░░░░░░░░████░░░░░░░░░░░░░░░░░░░░░░░░░░░  Módulo 8: DevOps
Semana 14-16 ░░░░░░░░░░░░░░░░████████████░░░░░░░░░░░░░░░  Projeto Integrador
```

---

## ⏱️ Expectativa de Horas

| Situação | Horas/Semana | Tempo Total |
|----------|--------------|-------------|
| Estudando full-time | 40h | 16 semanas |
| Trabalhando + 2-3h/dia | 15h | 20-24 semanas |
| Apenas finais de semana | 10h | 6-8 meses |
| **Integrado (teoria + prática)** | 20-25h | 4-5 semanas |

---

## 🔄 Abordagens de Estudo

### Opção 1: Sequencial (Tradicional)
Estudar toda a teoria primeiro (Módulos 1-8) e depois implementar o Projeto Integrador.
- ✅ Base teórica sólida antes da prática
- ❌ Pode esquecer conceitos quando for implementar

### Opção 2: Integrada (Recomendada) 🌟
Estudar teoria e implementar no mesmo dia, seguindo o [ROTEIRO_IMPLEMENTACAO.md](ROTEIRO_IMPLEMENTACAO.md).
- ✅ Fixação imediata dos conceitos
- ✅ Simula projeto real
- ✅ Portfolio construído durante o estudo
- ❌ Requer mais foco e organização diária

**Para a abordagem integrada, siga as Sprints do roteiro:**

| Sprint | Foco | Módulos Teóricos |
|--------|------|------------------|
| 0 | Setup + DDD (Value Objects) + Modulith | 3 (OOP/SOLID), 13 (Modulith) |
| 1 | Account Service (Hexagonal) | 4 (Spring), 5 (JPA), 13 (Modulith) |
| 2 | Transfer Service (Event Sourcing) + Idempotência | 1 (Streams), 6 (Arquitetura), 13 |
| 3 | Kafka + CQRS + Contract Testing | 6 (Kafka), 7 (Contract) |
| 4 | API Gateway + Keycloak OAuth2 | 6 (Microsserviços), 12 (Segurança) |
| 5 | Docker + CI/CD + DevSecOps | 7 (Testes), 8 (DevOps) |
| 6 | 🆕 Pix Automático (2025) | State Machine, Scheduler |
| 7 | 🆕 Pix Parcelado (2025) | Strategy Pattern, Cálculos |
| 8 | ☸️ Kubernetes | 10 (Kubernetes) |
| 9 | ☁️ AWS + LocalStack | 9 (AWS Cloud) |
| 10 | 🏗️ Terraform | 11 (Terraform) |
| 11 | 🚀 Integração Final | Deploy completo na AWS |

---

## 🎯 Priorização por Importância

### 🔴 Crítico (Elimina se não souber)
1. Collections Framework
2. Streams API
3. Spring DI e @Transactional
4. Problema N+1
5. Testes com Mockito

### 🟡 Importante (Diferencial)
1. Concorrência (CompletableFuture)
2. Spring Security
3. Design Patterns
4. Circuit Breaker
5. TestContainers

### 🟢 Bônus (Impressiona)
1. Virtual Threads (Java 21)
2. DDD / Event Sourcing
3. Kubernetes
4. Contribuições Open Source

---

## 📖 Ordem de Estudo Recomendada

```
┌─────────────────────────────────────────────────────────┐
│  1. FUNDAMENTOS (obrigatório antes de tudo)             │
│     Collections → Streams → Optional → Concorrência     │
└─────────────────────────────┬───────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│  2. JAVA MODERNO + OOP (base para Spring)               │
│     Records → SOLID → Design Patterns                   │
└─────────────────────────────┬───────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│  3. SPRING + JPA (core do trabalho diário)              │
│     DI → Transações → Security → N+1 → Locking          │
└─────────────────────────────┬───────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│  4. ARQUITETURA + TESTES (maturidade profissional)      │
│     REST → Kafka → Testes → Docker                      │
└─────────────────────────────┬───────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│  5. PROJETO INTEGRADOR (consolidação + portfolio)       │
│     Aplicar tudo em um projeto real                     │
└─────────────────────────────────────────────────────────┘
```

---

## 📚 Recursos Complementares

### Livros
- **Effective Java** - Joshua Bloch (Bible do Java)
- **Clean Code** - Robert Martin (Código limpo)
- **Domain-Driven Design** - Eric Evans (DDD)

### Cursos
- Alura: Formação Java
- Baeldung: Tutoriais Spring
- YouTube: Java Brains, Amigoscode

### Prática
- LeetCode: Algoritmos
- GitHub: Projetos open source
- HackerRank: Desafios Java

---

## ✅ Checklist Pré-Entrevista

Antes de qualquer entrevista, revise:

- [ ] ArrayList vs LinkedList (30 segundos)
- [ ] Como HashMap funciona (colisões)
- [ ] Por que injeção por construtor
- [ ] O que é N+1 e como resolver
- [ ] REQUIRED vs REQUIRES_NEW
- [ ] Por que @Transactional pode falhar
- [ ] @Mock vs @MockBean

---

## 💡 Dica Final

> O diferencial de um desenvolvedor **Pleno** não é decorar conceitos, mas saber **quando e por que** aplicar cada um. Foque em entender **trade-offs** e pratique explicar suas decisões em voz alta.

**Bons estudos! 🚀**
