# 📖 Módulo 4: Spring Ecosystem (2 semanas)

> O coração do desenvolvimento Java empresarial. Dominar Spring é obrigatório.

---

## 📚 4.1 Dependency Injection

### Por que DI?
```java
// ❌ Sem DI - alto acoplamento
public class OrderService {
    private final OrderRepository repository = new MySqlOrderRepository();
    private final EmailService emailService = new SmtpEmailService();
    // Problemas: Impossível testar, trocar implementação = mudar código
}

// ✅ Com DI - baixo acoplamento
public class OrderService {
    private final OrderRepository repository;
    private final EmailService emailService;
    
    public OrderService(OrderRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
}
```

### Tipos de Injeção

#### Injeção por Construtor (Recomendado)
```java
@Service
public class UserService {
    private final UserRepository repository;
    private final PasswordEncoder encoder;
    
    // @Autowired opcional se único construtor
    public UserService(UserRepository repository, PasswordEncoder encoder) {
        this.repository = repository;
        this.encoder = encoder;
    }
}

// Vantagens:
// ✅ Campos podem ser final (imutabilidade)
// ✅ Dependências obrigatórias - NPE na inicialização, não em runtime
// ✅ Fácil de testar - basta passar mocks no construtor
// ✅ Dependências explícitas na assinatura
```

#### Injeção por Campo (Evitar)
```java
@Service
public class UserService {
    @Autowired
    private UserRepository repository;
    
    @Autowired
    private PasswordEncoder encoder;
}

// Problemas:
// ❌ Não pode ser final
// ❌ Difícil testar - precisa de reflexão
// ❌ Dependências ocultas
// ❌ NPE em runtime, não na inicialização
```

#### Injeção por Setter (Raramente usado)
```java
@Service
public class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}

// Uso: Dependências opcionais ou cíclicas
```

---

## 📚 4.2 Bean Lifecycle

### Ciclo de Vida Completo

```
┌─────────────────────────────────────────────────────────┐
│                   CRIAÇÃO                                │
├─────────────────────────────────────────────────────────┤
│ 1. Instanciação (new Bean())                            │
│ 2. Populate Properties (injeção de dependências)        │
│ 3. BeanNameAware.setBeanName()                          │
│ 4. BeanFactoryAware.setBeanFactory()                    │
│ 5. ApplicationContextAware.setApplicationContext()      │
│ 6. BeanPostProcessor.postProcessBeforeInitialization()  │
│ 7. @PostConstruct                                       │
│ 8. InitializingBean.afterPropertiesSet()                │
│ 9. Custom init-method                                   │
│ 10. BeanPostProcessor.postProcessAfterInitialization()  │
├─────────────────────────────────────────────────────────┤
│                   USO                                    │
├─────────────────────────────────────────────────────────┤
│                   DESTRUIÇÃO                             │
├─────────────────────────────────────────────────────────┤
│ 11. @PreDestroy                                         │
│ 12. DisposableBean.destroy()                            │
│ 13. Custom destroy-method                               │
└─────────────────────────────────────────────────────────┘
```

### Hooks Mais Usados
```java
@Component
public class CacheWarmer {
    
    private final ProductRepository repository;
    private Map<Long, Product> cache;
    
    public CacheWarmer(ProductRepository repository) {
        this.repository = repository;
    }
    
    @PostConstruct
    public void init() {
        // Executado após injeção de dependências
        this.cache = repository.findAllActive()
            .stream()
            .collect(Collectors.toMap(Product::getId, Function.identity()));
        log.info("Cache aquecido com {} produtos", cache.size());
    }
    
    @PreDestroy
    public void cleanup() {
        // Executado antes de destruir o bean
        cache.clear();
        log.info("Cache limpo");
    }
}
```

### Escopos de Bean

| Escopo | Descrição | Uso Comum |
|--------|-----------|-----------|
| `singleton` | Uma instância por container (default) | Services, Repositories |
| `prototype` | Nova instância a cada injeção | Builders, não-thread-safe |
| `request` | Uma por HTTP request | Request-scoped data |
| `session` | Uma por HTTP session | User session data |
| `websocket` | Uma por WebSocket session | WebSocket handlers |

```java
@Component
@Scope("prototype")
public class ExpensiveProcessor {
    // Nova instância a cada injeção
}

// ⚠️ CUIDADO: Singleton com dependência Prototype
@Service
public class OrderProcessor { // Singleton
    private final ExpensiveProcessor processor; // Injetado UMA vez!
    
    // Problema: processor é sempre a mesma instância
}

// ✅ Solução: @Lookup
@Service
public abstract class OrderProcessor {
    
    @Lookup
    protected abstract ExpensiveProcessor getProcessor();
    
    public void process(Order order) {
        ExpensiveProcessor processor = getProcessor(); // Nova instância
        processor.process(order);
    }
}

// ✅ Alternativa: ObjectProvider
@Service
public class OrderProcessor {
    private final ObjectProvider<ExpensiveProcessor> processorProvider;
    
    public void process(Order order) {
        ExpensiveProcessor processor = processorProvider.getObject();
        processor.process(order);
    }
}
```

---

## 📚 4.3 @Transactional

### Configuração Básica
```java
@Service
@Transactional(readOnly = true) // Otimização para leitura
public class OrderService {
    
    @Transactional // Override para escrita
    public Order create(CreateOrderRequest request) {
        // Transação de leitura/escrita
    }
    
    public Order findById(Long id) {
        // Usa transação readOnly da classe
    }
}
```

### Propagation Types

| Tipo | Comportamento | Uso |
|------|---------------|-----|
| `REQUIRED` | Usa existente ou cria nova (default) | Maioria dos casos |
| `REQUIRES_NEW` | Sempre cria nova, suspende existente | Logs independentes |
| `NESTED` | Cria savepoint na existente | Rollback parcial |
| `SUPPORTS` | Usa se existir, senão roda sem | Leitura opcional |
| `NOT_SUPPORTED` | Suspende se existir | Operação não-transacional |
| `MANDATORY` | Exige existente | Deve ser chamado com TX |
| `NEVER` | Não pode haver transação | Verificação |

```java
@Service
public class AuditService {
    
    // Sempre cria nova transação - persiste mesmo se método chamador falhar
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String action, String details) {
        auditRepository.save(new AuditLog(action, details));
    }
}

@Service
public class OrderService {
    private final AuditService auditService;
    
    @Transactional
    public Order create(CreateOrderRequest request) {
        Order order = orderRepository.save(toEntity(request));
        
        auditService.log("ORDER_CREATED", order.getId().toString());
        // Se falhar aqui, log já está commitado!
        
        processPayment(order); // Pode lançar exceção
        
        return order;
    }
}
```

### Armadilhas Comuns

#### 1. Checked Exception não faz Rollback
```java
// ❌ Checked exception - NÃO faz rollback por padrão
@Transactional
public void process() throws IOException {
    repository.save(entity);
    throw new IOException(); // Dados PERSISTEM!
}

// ✅ Solução
@Transactional(rollbackFor = Exception.class)
public void process() throws IOException {
    repository.save(entity);
    throw new IOException(); // Agora faz rollback
}
```

#### 2. Auto-Invocação
```java
// ❌ Auto-invocação - proxy não intercepta
@Service
public class OrderService {
    
    public void processAll(List<Order> orders) {
        for (Order order : orders) {
            processOne(order); // Chamada direta, @Transactional ignorado!
        }
    }
    
    @Transactional
    public void processOne(Order order) {
        // Não está em transação quando chamado de processAll!
    }
}

// ✅ Solução 1: Separar em outro service
@Service
public class OrderBatchService {
    private final OrderService orderService;
    
    public void processAll(List<Order> orders) {
        orders.forEach(orderService::processOne); // Passa pelo proxy
    }
}

// ✅ Solução 2: Self-injection
@Service
public class OrderService {
    @Lazy
    @Autowired
    private OrderService self;
    
    public void processAll(List<Order> orders) {
        orders.forEach(self::processOne); // Passa pelo proxy
    }
}
```

#### 3. Método Privado
```java
// ❌ Métodos privados não são interceptados
@Transactional
private void saveInternal(Entity entity) {
    // @Transactional NÃO FUNCIONA!
}

// ✅ Use métodos públicos ou protected
```

---

## 📚 4.4 Exception Handling

### @ControllerAdvice Global
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    // Entidade não encontrada
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Validação de campos
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        List<FieldError> fieldErrors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .toList();
            
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Erro de validação",
            fieldErrors,
            LocalDateTime.now()
        );
    }
    
    // Regra de negócio
    @ExceptionHandler(BusinessException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleBusiness(BusinessException ex) {
        return new ErrorResponse(
            HttpStatus.UNPROCESSABLE_ENTITY.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Erro genérico - sempre por último
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex) {
        log.error("Erro não tratado", ex);
        return new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Erro interno do servidor",
            LocalDateTime.now()
        );
    }
}

// DTOs de erro
public record ErrorResponse(
    int status,
    String message,
    LocalDateTime timestamp
) {
    public ErrorResponse(int status, String message, List<FieldError> errors, LocalDateTime timestamp) {
        // Overload com erros de campo
    }
}

public record FieldError(String field, String message) { }
```

---

## 📚 4.5 Spring Security

### Configuração JWT
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    private final JwtAuthFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE).hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(new JwtAuthenticationEntryPoint())
            )
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### JWT Filter
```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {
    
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain chain) throws ServletException, IOException {
        
        String authHeader = request.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }
        
        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);
        
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            if (jwtService.isTokenValid(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        chain.doFilter(request, response);
    }
}
```

### JWT Service
```java
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    @Value("${jwt.expiration}")
    private long jwtExpiration;
    
    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }
    
    private <T> T extractClaim(String token, Function<Claims, T> resolver) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
        return resolver.apply(claims);
    }
    
    private Key getSigningKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretKey));
    }
}
```

### Method Security
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public List<UserResponse> findAll() {
        // Apenas ADMIN
    }
    
    @GetMapping("/{id}")
    @PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
    public UserResponse findById(@PathVariable Long id) {
        // Apenas próprio usuário ou ADMIN
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void delete(@PathVariable Long id) {
        // Apenas ADMIN
    }
}
```

---

## 🎯 Perguntas de Entrevista

1. **Por que injeção por construtor é melhor?**
2. **O que é o Bean Lifecycle?**
3. **REQUIRED vs REQUIRES_NEW?**
4. **Por que @Transactional pode não funcionar?**
5. **Como funciona JWT com Spring Security?**

---

> **Próximo módulo:** [Módulo 5 - JPA e Persistência](MODULO_05_JPA_PERSISTENCIA.md)
