# 🎯 Perguntas de Entrevista - Java Pleno

> 50+ perguntas frequentes com respostas estruturadas para preparação de entrevistas.

---

## 📚 Java Core

### 1. ArrayList vs LinkedList - quando usar cada?

**Resposta:**
- **ArrayList:** Acesso por índice O(1), inserção no final O(1) amortizado. Use quando precisa de acesso aleatório frequente.
- **LinkedList:** Inserção/remoção nas pontas O(1), acesso O(n). Use para filas (Queue) ou quando insere/remove muito no início.

**Na prática:** ArrayList em 95% dos casos. LinkedList raramente é a melhor escolha.

---

### 2. Como HashMap funciona internamente?

**Resposta:**
1. Calcula `hashCode()` da chave
2. Usa hash para encontrar o bucket (array index)
3. Se colisão: Java 7 usa LinkedList, Java 8+ usa árvore (Red-Black) após 8 elementos
4. `equals()` resolve colisões

**Load factor:** 0.75 - quando atinge, dobra o tamanho e rehash.

---

### 3. Por que sobrescrever equals() e hashCode() juntos?

**Resposta:**
- **Contrato:** Se `a.equals(b)` → `hashCode` deve ser igual
- **Problema:** Sem isso, HashMap/HashSet não funcionam corretamente
- Objetos iguais podem ir para buckets diferentes

---

### 4. Diferença entre map e flatMap em Streams?

**Resposta:**
- **map:** Transforma 1:1 (Stream<T> → Stream<R>)
- **flatMap:** Achata estruturas aninhadas (Stream<List<T>> → Stream<T>)

```java
// map: ["a,b", "c,d"] → [["a","b"], ["c","d"]]
// flatMap: ["a,b", "c,d"] → ["a", "b", "c", "d"]
```

---

### 5. O que é lazy evaluation em Streams?

**Resposta:**
Operações intermediárias (filter, map) não executam até haver uma operação terminal (collect, forEach). Permite otimizações como short-circuit.

---

### 6. Quando usar parallel streams?

**Resposta:**
- **Use:** Operações CPU-bound, grandes volumes, sem side effects
- **Evite:** I/O, coleções pequenas, operações com estado compartilhado

**Cuidado:** Overhead de paralelização pode ser maior que o ganho.

---

### 7. Diferença entre Runnable e Callable?

**Resposta:**
| Runnable | Callable |
|----------|----------|
| `void run()` | `V call()` |
| Não retorna valor | Retorna valor |
| Não lança checked exception | Lança Exception |

---

### 8. O que são Virtual Threads?

**Resposta:**
Threads leves gerenciadas pela JVM (não pelo OS). Permite milhares de threads sem overhead. Ideal para I/O-bound. Java 21+.

```java
Executors.newVirtualThreadPerTaskExecutor()
```

---

### 9. Heap vs Stack?

**Resposta:**
| Heap | Stack |
|------|-------|
| Objetos | Variáveis locais, referências |
| Compartilhado entre threads | Por thread |
| GC gerencia | Automático (escopo) |

---

### 10. Tipos de Garbage Collector?

**Resposta:**
- **G1:** Default, bom balanço latência/throughput
- **ZGC:** Ultra baixa latência (<10ms), Java 15+
- **Parallel:** Máximo throughput, pausas maiores

---

## 🍃 Spring Framework

### 11. Por que injeção por construtor é melhor?

**Resposta:**
1. **Imutabilidade:** Campos `final`
2. **Testabilidade:** Fácil criar mocks
3. **Falha rápida:** Erro na inicialização, não em runtime
4. **Dependências explícitas:** Visíveis na assinatura

---

### 12. Diferença entre @Component, @Service, @Repository?

**Resposta:**
Funcionalmente iguais, mas:
- **@Component:** Genérico
- **@Service:** Camada de serviço (semântica)
- **@Repository:** Acesso a dados, tradução de exceções SQL

---

### 13. O que é o Bean Lifecycle?

**Resposta:**
1. Instanciação
2. Injeção de dependências
3. `@PostConstruct`
4. Bean pronto
5. `@PreDestroy`
6. Destruição

---

### 14. REQUIRED vs REQUIRES_NEW em @Transactional?

**Resposta:**
- **REQUIRED:** Usa transação existente ou cria nova
- **REQUIRES_NEW:** Sempre cria nova, suspende a existente

**Use REQUIRES_NEW para:** Logs de auditoria que devem persistir mesmo se transação principal falhar.

---

### 15. Por que minha transação não fez rollback?

**Respostas possíveis:**
1. **Checked exception:** Por padrão só RuntimeException faz rollback
2. **Auto-invocação:** Método chamado internamente não passa pelo proxy
3. **Método privado:** Proxy não intercepta

```java
// Solução para exceções
@Transactional(rollbackFor = Exception.class)
```

---

### 16. O que é @ControllerAdvice?

**Resposta:**
Componente global para tratamento de exceções, binding, e model attributes. Centraliza lógica que seria repetida em cada controller.

---

### 17. Como funciona Spring Security com JWT?

**Resposta:**
1. Login → Gera token JWT com claims
2. Cada request envia token no header `Authorization: Bearer <token>`
3. Filtro extrai e valida token
4. Se válido, popula SecurityContext
5. Request prossegue autenticada

---

## 💾 JPA e Persistência

### 18. O que é o problema N+1?

**Resposta:**
1 query para buscar entidades + N queries para cada relacionamento lazy.

**Exemplo:** 100 pedidos = 1 + 100 queries (se acessar cliente de cada).

**Soluções:** JOIN FETCH, @EntityGraph, @BatchSize, Projections.

---

### 19. LAZY vs EAGER loading?

**Resposta:**
- **LAZY:** Carrega quando acessar (default para @OneToMany)
- **EAGER:** Carrega junto (default para @ManyToOne)

**Best practice:** Sempre LAZY, use fetch quando precisar.

---

### 20. Optimistic vs Pessimistic Locking?

**Resposta:**
| Optimistic | Pessimistic |
|------------|-------------|
| @Version, verifica no commit | Lock no banco |
| Melhor para poucos conflitos | Melhor para muitos conflitos |
| Pode falhar (retry) | Bloqueia outras transações |

---

### 21. O que é flush() e clear()?

**Resposta:**
- **flush():** Sincroniza mudanças com DB (sem commit)
- **clear():** Remove entidades do contexto de persistência

**Uso:** Batch processing para evitar OutOfMemoryError.

---

### 22. Por que usar Flyway/Liquibase?

**Resposta:**
- Versionamento de schema
- Reprodutibilidade entre ambientes
- Rollback controlado
- Histórico de mudanças
- CI/CD amigável

---

## 🏗️ Arquitetura

### 23. Quando usar microsserviços vs monolito?

**Resposta:**

| Monolito | Microsserviços |
|----------|----------------|
| MVP, time pequeno | Escala, times independentes |
| Transações ACID fáceis | Cada serviço seu banco |
| Deploy simples | Deploy independente |
| Menos complexidade operacional | Requer observabilidade madura |

---

### 24. O que é Circuit Breaker?

**Resposta:**
Pattern que "abre" após falhas repetidas, retornando fallback imediatamente. Evita cascata de falhas.

**Estados:** Closed → Open (após threshold) → Half-Open (testa recuperação)

---

### 25. O que é idempotência?

**Resposta:**
Operação que pode ser executada múltiplas vezes com mesmo resultado. Importante para retry em sistemas distribuídos.

**Exemplo:** PUT é idempotente, POST não necessariamente.

---

### 26. O que é Event Sourcing?

**Resposta:**
Armazena eventos em vez do estado atual. Estado é reconstruído reproduzindo eventos.

**Vantagens:** Histórico completo, auditoria, debug
**Desvantagens:** Complexidade, eventual consistency

---

### 27. O que é Saga Pattern?

**Resposta:**
Coordena transações distribuídas como sequência de transações locais. Cada etapa tem compensação se falhar.

**Tipos:**
- **Choreography:** Eventos entre serviços
- **Orchestration:** Coordenador central

---

### 28. O que são Aggregates em DDD?

**Resposta:**
Cluster de entidades tratadas como unidade. Aggregate Root é único ponto de entrada.

**Exemplo:** Order (root) contém OrderItems. Nunca acessar OrderItem diretamente.

---

## 🧪 Testes

### 29. @Mock vs @MockBean?

**Resposta:**
| @Mock | @MockBean |
|-------|-----------|
| Mockito puro | Spring Context |
| Testes unitários | Testes integração |
| Rápido | Mais lento |

---

### 30. O que são Test Slices?

**Resposta:**
Carregam apenas parte do contexto Spring:
- `@WebMvcTest` - Controllers
- `@DataJpaTest` - JPA
- `@JsonTest` - Serialização

Mais rápidos que `@SpringBootTest`.

---

### 31. Por que usar TestContainers?

**Resposta:**
- Banco real em container Docker
- Igual produção (sem H2)
- Reprodutível
- Isolado por teste

---

### 32. O que é WireMock?

**Resposta:**
Simula APIs externas para testes. Stub respostas HTTP. Evita depender de serviços externos em testes.

---

## 🐳 DevOps

### 33. O que é Dockerfile multi-stage?

**Resposta:**
Separa build de runtime. Imagem final menor (só JRE, não JDK).

```dockerfile
FROM jdk AS build    # Compila
FROM jre             # Só runtime
COPY --from=build    # Copia apenas JAR
```

---

### 34. O que são Health Checks?

**Resposta:**
Endpoints que indicam saúde da aplicação. Kubernetes/load balancers usam para routing.

```java
@Component
public class DatabaseHealth implements HealthIndicator { }
```

---

## 💡 Soft Skills

### 35. Me conta sobre um bug difícil que você resolveu

**Como responder:**
1. **Situação:** Contexto do problema
2. **Tarefa:** O que precisava resolver
3. **Ação:** Passos de investigação
4. **Resultado:** Solução e aprendizado

---

### 36. Como você decide entre tecnologias?

**Framework:**
1. Requisitos do problema
2. Experiência do time
3. Ecossistema/suporte
4. Performance/escalabilidade
5. Custo de manutenção

---

### 37. Como você lida com código legado?

**Resposta:**
1. Entender antes de mudar
2. Adicionar testes primeiro
3. Refatorar incrementalmente
4. Documentar decisões
5. Priorizar por impacto/risco

---

### 38. Como você se mantém atualizado?

**Sugestões:**
- Blogs técnicos (Baeldung, InfoQ)
- Conferências (JavaOne, QCon)
- Open source
- Side projects
- Comunidades

---

## ⚠️ Red Flags que Entrevistadores Buscam

1. **Não admite que não sabe**
2. **Respostas decoradas sem entendimento**
3. **Não conhece trade-offs**
4. **Não testa código**
5. **Blame no time anterior**
6. **Não faz perguntas**

---

## ✅ Checklist Pré-Entrevista

- [ ] Revise seu currículo - saiba explicar cada item
- [ ] Pesquise sobre a empresa
- [ ] Prepare 3-5 perguntas para fazer
- [ ] Revise projetos recentes com detalhes técnicos
- [ ] Teste equipamento (se remota)
- [ ] Durma bem na noite anterior

---

> **Dica Final:** Pratique explicar conceitos em voz alta. A fluência vem com repetição.
