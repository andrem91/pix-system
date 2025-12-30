# ✅ Checklist de Preparação - Entrevista Java Pleno

Checklist de auto-avaliação antes de entrevistas. Marque os itens que você domina.

---

## 🔴 Obrigatório (Elimina se não souber)

### Java Core
- [ ] Explico ArrayList vs LinkedList em 30 segundos
- [ ] Sei como HashMap funciona (hash, bucket, colisão)
- [ ] Implemento equals/hashCode corretamente
- [ ] Uso Streams com fluência (filter, map, collect, groupingBy)
- [ ] Entendo Optional e seus anti-patterns
- [ ] Explico diferença entre Runnable e Callable

### Spring
- [ ] Explico por que injeção por construtor é melhor
- [ ] Conheço Bean Lifecycle (@PostConstruct, @PreDestroy)
- [ ] Sei por que @Transactional pode não funcionar
- [ ] Consigo explicar REQUIRED vs REQUIRES_NEW
- [ ] Implemento @ControllerAdvice para exceções

### JPA
- [ ] Identifico e resolvo problema N+1
- [ ] Conheço diferença LAZY vs EAGER
- [ ] Sei quando usar @Version (optimistic locking)
- [ ] Entendo flush() e clear()

### Testes
- [ ] Escrevo testes com JUnit 5 + Mockito
- [ ] Sei diferença @Mock vs @MockBean
- [ ] Já usei ou entendo TestContainers

---

## 🟡 Importante (Diferencial)

### Java Avançado
- [ ] Uso CompletableFuture adequadamente
- [ ] Entendo GC e tipos de coletores
- [ ] Conheço Records e Sealed Classes (Java 17+)
- [ ] Sei o que são Virtual Threads (Java 21)

### Spring Avançado
- [ ] Implemento JWT com Spring Security
- [ ] Conheço diferentes escopos de Bean
- [ ] Uso @Async corretamente

### Arquitetura
- [ ] Argumento monolito vs microsserviços com trade-offs
- [ ] Explico Circuit Breaker e quando usar
- [ ] Entendo Kafka (producer/consumer)
- [ ] Conheço conceitos de DDD (Aggregate, Value Object)

### DevOps
- [ ] Escrevo Dockerfile para Spring Boot
- [ ] Uso docker-compose para ambiente local
- [ ] Conheço Flyway/Liquibase

---

## 🟢 Bônus (Impressiona)

- [ ] Contribuí para projetos open source
- [ ] Tenho certificação Oracle/Spring
- [ ] Implementei Event Sourcing ou Saga
- [ ] Fiz performance tuning (profiling, conexões)
- [ ] Conheço Kubernetes básico

---

## 📋 Preparação Logística

### Uma semana antes
- [ ] Pesquisei sobre a empresa
- [ ] Revisei meu currículo
- [ ] Preparei 3-5 perguntas para fazer
- [ ] Revi projetos recentes com detalhes

### Um dia antes
- [ ] Testei equipamento (câmera, microfone)
- [ ] Escolhi ambiente silencioso
- [ ] Preparei água e papel para anotações
- [ ] Dormi bem

### No dia
- [ ] Entrei 5 minutos antes
- [ ] Câmera ligada, iluminação boa
- [ ] Celular no silencioso
- [ ] Respiro fundo, confio na preparação

---

## 🎯 Quick Review (15 min antes)

Revise mentalmente:

1. **Diferença ArrayList/LinkedList?** → Acesso vs inserção
2. **HashMap interno?** → hash → bucket → colisão
3. **N+1?** → JOIN FETCH, @EntityGraph
4. **@Transactional falha?** → checked, auto-invoke, private
5. **Injeção construtor?** → imutável, testável, falha rápida
6. **Circuit Breaker?** → closed → open → half-open

---

## 📊 Auto-avaliação

Conte quantos itens marcou em cada seção:

| Seção | Seu Score | Meta |
|-------|-----------|------|
| 🔴 Obrigatório | ___/17 | 17/17 |
| 🟡 Importante | ___/14 | 10+ |
| 🟢 Bônus | ___/5 | 2+ |

**Interpretação:**
- Obrigatório < 15: Foque aqui primeiro
- Importante < 8: Revise antes de entrevistas sênior
- Bônus: Diferencial competitivo

---

> **Lembrete:** Nervosismo é normal. Respire, organize o pensamento, e mostre seu raciocínio. Entrevistadores valorizam o processo, não só a resposta final.

**Boa sorte! 🍀**
