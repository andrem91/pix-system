# 📖 Módulo 5: JPA e Persistência (2 semanas)

> JPA é onde muitos problemas de performance têm origem. Dominar é essencial.

---

## 📚 5.1 Entity Lifecycle

### Estados de uma Entidade

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│         ┌─────────┐                                      │
│         │TRANSIENT│  (new Entity())                      │
│         └────┬────┘                                      │
│              │ persist()                                 │
│              ▼                                           │
│         ┌─────────┐  ◄──── find()/query                  │
│         │ MANAGED │                                      │
│         └────┬────┘                                      │
│              │                                           │
│    ┌─────────┼─────────┐                                 │
│    │ detach()│ remove()│                                 │
│    ▼         │         ▼                                 │
│ ┌────────┐   │     ┌───────┐                             │
│ │DETACHED│   │     │REMOVED│                             │
│ └────┬───┘   │     └───────┘                             │
│      │       │                                           │
│      │merge()│                                           │
│      └───────┴──► MANAGED                                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

| Estado | Descrição | No Banco? |
|--------|-----------|-----------|
| **Transient** | Recém criado, não persistido | Não |
| **Managed** | Gerenciado pelo EntityManager | Sim, sincronizado |
| **Detached** | Foi managed, sessão fechou | Sim, não sincronizado |
| **Removed** | Marcado para deleção | Será deletado |

```java
// TRANSIENT
User user = new User("João"); // Não existe no banco

// MANAGED
entityManager.persist(user); // Agora gerenciado
user.setEmail("joao@email.com"); // Mudanças serão persistidas!

// DETACHED
entityManager.detach(user); // Não mais gerenciado
user.setEmail("outro@email.com"); // Mudança NÃO será persistida

// Re-attach (merge)
User managedUser = entityManager.merge(user); // Retorna instância managed

// REMOVED
entityManager.remove(user); // Será deletado no flush/commit
```

---

## 📚 5.2 Flush e Clear

### EntityManager.flush()
```java
// flush() sincroniza mudanças com o banco SEM commit
@Transactional
public void process() {
    User user = new User("João");
    entityManager.persist(user);
    
    // Neste ponto, o INSERT ainda não foi para o banco
    
    entityManager.flush(); // INSERT executado, mas transação não commitada
    
    Long id = user.getId(); // ID disponível após flush
    
    // Se exceção aqui, rollback desfaz o INSERT
}
```

### EntityManager.clear()
```java
// clear() desanexa TODAS as entidades do contexto
entityManager.clear();

// Útil para batch processing
@Transactional
public void importarEmMassa(List<User> users) {
    int batchSize = 50;
    
    for (int i = 0; i < users.size(); i++) {
        entityManager.persist(users.get(i));
        
        if (i > 0 && i % batchSize == 0) {
            entityManager.flush(); // Envia para o banco
            entityManager.clear(); // Libera memória
        }
    }
}
// Sem flush/clear: OutOfMemoryError para grandes volumes
```

---

## 📚 5.3 Relacionamentos

### @ManyToOne (Mais comum)
```java
@Entity
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY) // SEMPRE LAZY!
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;
}

// ⚠️ Default é EAGER - mude para LAZY!
// EAGER causa N+1 e carregamento desnecessário
```

### @OneToMany
```java
@Entity
public class Cliente {
    @Id
    private Long id;
    
    @OneToMany(
        mappedBy = "cliente",          // Lado inverso
        cascade = CascadeType.ALL,     // Propaga operações
        orphanRemoval = true           // Remove órfãos
    )
    private List<Pedido> pedidos = new ArrayList<>();
    
    // Métodos auxiliares para manter consistência
    public void addPedido(Pedido pedido) {
        pedidos.add(pedido);
        pedido.setCliente(this);
    }
    
    public void removePedido(Pedido pedido) {
        pedidos.remove(pedido);
        pedido.setCliente(null);
    }
}
```

### @ManyToMany
```java
@Entity
public class Estudante {
    @Id
    private Long id;
    
    @ManyToMany
    @JoinTable(
        name = "estudante_curso",
        joinColumns = @JoinColumn(name = "estudante_id"),
        inverseJoinColumns = @JoinColumn(name = "curso_id")
    )
    private Set<Curso> cursos = new HashSet<>();
}

@Entity
public class Curso {
    @Id
    private Long id;
    
    @ManyToMany(mappedBy = "cursos")
    private Set<Estudante> estudantes = new HashSet<>();
}
```

---

## 📚 5.4 Problema N+1

### O Problema
```java
// 1 query para buscar pedidos
List<Pedido> pedidos = pedidoRepository.findAll();
// SELECT * FROM pedido

// N queries para buscar cliente de cada pedido
for (Pedido p : pedidos) {
    System.out.println(p.getCliente().getNome()); // Query para CADA cliente!
    // SELECT * FROM cliente WHERE id = ?
}
// Total: 1 + N queries! 😱
```

### Solução 1: JOIN FETCH
```java
@Query("SELECT p FROM Pedido p JOIN FETCH p.cliente")
List<Pedido> findAllWithCliente();

// Gera: SELECT p.*, c.* FROM pedido p JOIN cliente c ON p.cliente_id = c.id
// UMA única query!

// Múltiplos relacionamentos
@Query("SELECT p FROM Pedido p JOIN FETCH p.cliente JOIN FETCH p.itens")
List<Pedido> findAllComplete();

// ⚠️ CUIDADO: JOIN FETCH com paginação não funciona bem
// Hibernate faz paginação em memória
```

### Solução 2: @EntityGraph
```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {
    
    @EntityGraph(attributePaths = {"cliente"})
    List<Pedido> findAll();
    
    @EntityGraph(attributePaths = {"cliente", "itens"})
    @Query("SELECT p FROM Pedido p WHERE p.status = :status")
    List<Pedido> findByStatus(@Param("status") Status status);
}
```

### Solução 3: @BatchSize
```java
@Entity
public class Cliente {
    
    @OneToMany(mappedBy = "cliente")
    @BatchSize(size = 25) // Carrega 25 por vez
    private List<Pedido> pedidos;
}

// Ao invés de N queries: ceil(N/25) queries
// SELECT * FROM pedido WHERE cliente_id IN (?, ?, ?, ...)
```

### Solução 4: Projection/DTO
```java
// Busca apenas o necessário
public record PedidoResumo(Long id, LocalDate data, String clienteNome) { }

@Query("""
    SELECT new com.example.PedidoResumo(p.id, p.data, c.nome)
    FROM Pedido p JOIN p.cliente c
    """)
List<PedidoResumo> findResumo();

// Vantagens: Menos dados, sem problemas de N+1
```

---

## 📚 5.5 Locking

### Optimistic Locking
```java
@Entity
public class Produto {
    @Id
    private Long id;
    
    private String nome;
    private BigDecimal preco;
    private int estoque;
    
    @Version
    private Long version; // Incrementa automaticamente
}

// Funcionamento:
// UPDATE produto SET nome=?, preco=?, estoque=?, version=version+1
// WHERE id=? AND version=?

// Se versão mudou: OptimisticLockException
// → Transação deve ser refeita

// Quando usar: Poucos conflitos esperados, melhor performance
```

### Pessimistic Locking
```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Produto p WHERE p.id = :id")
    Optional<Produto> findByIdForUpdate(@Param("id") Long id);
}

// Gera: SELECT ... FOR UPDATE
// Bloqueia a linha no banco até commit

// Quando usar: Muitos conflitos esperados, operações críticas
```

---

## 📚 5.6 Auditing

### Configuração
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
            .map(SecurityContext::getAuthentication)
            .filter(Authentication::isAuthenticated)
            .map(Authentication::getName);
    }
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {
    
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    private String updatedBy;
}

@Entity
public class Pedido extends BaseEntity {
    // Herda todos os campos de auditoria
}
```

---

## 📚 5.7 Cache

### First Level Cache (L1)
```java
// Automático, por EntityManager/Session
// Dentro da mesma transação, mesma entidade não é buscada 2x

@Transactional
public void process(Long id) {
    User user1 = repository.findById(id); // Query no banco
    User user2 = repository.findById(id); // Retorna do cache!
    
    System.out.println(user1 == user2); // true - mesma instância
}
```

### Second Level Cache (L2)
```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          region.factory_class: org.hibernate.cache.jcache.JCacheRegionFactory
```

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Configuracao {
    // Entidades pouco alteradas são boas candidatas
}
```

### Query Cache
```java
// Cache de resultados de queries
@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
List<Produto> findByCategoria(String categoria);
```

---

## 📚 5.8 Flyway/Liquibase

### Flyway
```sql
-- src/main/resources/db/migration/V1__create_users.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- V2__add_role_column.sql
ALTER TABLE users ADD COLUMN role VARCHAR(50) DEFAULT 'USER';

-- V3__create_orders.sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

### Boas Práticas de Migration

1. **Nunca edite migrations já aplicadas**
2. **Nomes descritivos:** `V3__add_email_index.sql`
3. **Migrations pequenas e focadas**
4. **Teste rollback quando possível**
5. **Use transações**

---

## 🎯 Perguntas de Entrevista

1. **O que é o problema N+1? Como resolver?**
2. **LAZY vs EAGER loading?**
3. **Optimistic vs Pessimistic Locking?**
4. **O que é flush() e clear()?**
5. **Por que usar Flyway?**

---

> **Próximo módulo:** [Módulo 6 - Arquitetura](MODULO_06_ARQUITETURA.md)
