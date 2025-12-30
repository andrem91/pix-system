# 📖 Módulo 12: Segurança Avançada (1 semana)

> Autenticação e autorização são críticas em sistemas financeiros. OAuth2 e Keycloak são padrão de mercado.

---

## 📚 Índice

| Seção | Tópico | Relevância |
|-------|--------|------------|
| 12.1 | OAuth2 e OpenID Connect | 🔴 Crítico |
| 12.2 | Keycloak | 🔴 Crítico |
| 12.3 | Spring Security + OAuth2 | 🔴 Crítico |
| 12.4 | JWT Deep Dive | 🟡 Importante |
| 12.5 | Comunicação Service-to-Service | 🔴 Crítico |
| 12.6 | Fine-Grained Authorization | 🟡 Importante |

---

## 📚 12.1 OAuth2 e OpenID Connect

### O Problema

```
❌ Autenticação Básica (username/password)
- Senha trafega em toda requisição
- Cada serviço precisa validar credenciais
- Sem padrão para autorização

✅ OAuth2 + OIDC
- Token único (JWT) para todas as requisições
- Identity Provider centralizado
- Escopos para autorização granular
```

### Conceitos Fundamentais

```
┌─────────────────────────────────────────────────────────────────┐
│                        OAuth2 Actors                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     │
│  │   Resource   │     │Authorization │     │   Resource   │     │
│  │    Owner     │     │    Server    │     │    Server    │     │
│  │   (Usuário)  │     │  (Keycloak)  │     │  (Sua API)   │     │
│  └──────────────┘     └──────────────┘     └──────────────┘     │
│         │                    │                    │              │
│         │  1. Login          │                    │              │
│         │───────────────────►│                    │              │
│         │                    │                    │              │
│         │  2. Token (JWT)    │                    │              │
│         │◄───────────────────│                    │              │
│         │                    │                    │              │
│         │  3. Request + Token                     │              │
│         │────────────────────────────────────────►│              │
│         │                    │                    │              │
│         │                    │  4. Valida Token   │              │
│         │                    │◄───────────────────│              │
│         │                    │                    │              │
│         │  5. Response                            │              │
│         │◄────────────────────────────────────────│              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Fluxos OAuth2

| Fluxo | Quando Usar | Exemplo |
|-------|-------------|---------|
| **Authorization Code** | Apps web com backend | Portal do banco |
| **Authorization Code + PKCE** | SPAs e mobile | App do banco |
| **Client Credentials** | Máquina-a-máquina | Transfer → Account |
| **Resource Owner Password** | ❌ Legado, evitar | - |

### OpenID Connect (OIDC)

```
OAuth2    = Autorização ("O que você pode fazer?")
OIDC      = Autenticação ("Quem é você?")
OIDC      = OAuth2 + ID Token + UserInfo Endpoint
```

---

## 📚 12.2 Keycloak

### O que é?

Keycloak é um **Identity Provider (IdP)** open-source da Red Hat que implementa OAuth2 e OIDC.

### Por que Keycloak?

| Feature | Benefício |
|---------|-----------|
| Open Source | Grátis para estudar |
| Docker Ready | `docker run keycloak` |
| Multi-tenancy | Realms para ambientes |
| Federation | LDAP, Active Directory |
| Social Login | Google, GitHub, etc |
| Fine-Grained | Roles, Groups, Scopes |

### Docker Compose

```yaml
# docker-compose.yml
services:
  keycloak:
    image: quay.io/keycloak/keycloak:23.0
    command: start-dev
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8180:8080"  # Evitar conflito com app na 8080
    volumes:
      - keycloak_data:/opt/keycloak/data

volumes:
  keycloak_data:
```

### Configuração Inicial

**1. Criar Realm:**
```
Admin Console → Create Realm → "pix-system"
```

**2. Criar Client (API Gateway):**
```
Clients → Create Client
- Client ID: pix-gateway
- Client Authentication: ON
- Authorization: ON
```

**3. Criar Client (Service-to-Service):**
```
Clients → Create Client
- Client ID: pix-transfer-service
- Client Authentication: ON
- Service Account Roles: ON
```

**4. Criar Scopes:**
```
Client Scopes → Create
- pix:read   → Consultar saldo
- pix:write  → Realizar transferência
- account:admin → Gerenciar contas
```

**5. Criar Roles:**
```
Realm Roles → Create
- user      → Usuário comum
- admin     → Administrador
```

---

## 📚 12.3 Spring Security + OAuth2

### Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Configuração (Resource Server)

```yaml
# application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8180/realms/pix-system
          # Keycloak publica as chaves públicas em:
          # http://localhost:8180/realms/pix-system/protocol/openid-connect/certs
```

### Security Config

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/**").permitAll()
                .requestMatchers("/api/v1/public/**").permitAll()
                .requestMatchers("/api/v1/accounts/**").hasAuthority("SCOPE_account:read")
                .requestMatchers("/api/v1/transfers/**").hasAuthority("SCOPE_pix:write")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthConverter()))
            )
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthConverter.setAuthoritiesClaimName("realm_access.roles");
        grantedAuthConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(jwt -> {
            Collection<GrantedAuthority> authorities = new ArrayList<>();
            
            // Adiciona scopes
            authorities.addAll(grantedAuthConverter.convert(jwt));
            
            // Adiciona realm roles do Keycloak
            Map<String, Object> realmAccess = jwt.getClaim("realm_access");
            if (realmAccess != null) {
                List<String> roles = (List<String>) realmAccess.get("roles");
                roles.forEach(role -> 
                    authorities.add(new SimpleGrantedAuthority("ROLE_" + role))
                );
            }
            
            return authorities;
        });
        
        return converter;
    }
}
```

### Acessando Usuário Autenticado

```java
@RestController
@RequestMapping("/api/v1/accounts")
public class AccountController {

    @GetMapping("/me")
    public AccountResponse getMyAccount(@AuthenticationPrincipal Jwt jwt) {
        String userId = jwt.getSubject();       // UUID do Keycloak
        String email = jwt.getClaim("email");   // Email do usuário
        String name = jwt.getClaim("name");     // Nome completo
        
        return accountService.findByKeycloakId(userId);
    }
    
    @PreAuthorize("hasRole('admin')")
    @GetMapping
    public List<AccountResponse> getAllAccounts() {
        return accountService.findAll();
    }
    
    @PreAuthorize("hasAuthority('SCOPE_pix:write')")
    @PostMapping("/transfer")
    public TransferResponse transfer(@RequestBody TransferRequest request) {
        return transferService.execute(request);
    }
}
```

---

## 📚 12.4 JWT Deep Dive

### Estrutura do JWT

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvYW8ifQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

HEADER.PAYLOAD.SIGNATURE
```

### Claims do Keycloak

```json
{
  "exp": 1735600000,
  "iat": 1735596400,
  "jti": "uuid-do-token",
  "iss": "http://localhost:8180/realms/pix-system",
  "sub": "uuid-do-usuario",
  "typ": "Bearer",
  "azp": "pix-gateway",
  "scope": "openid pix:read pix:write",
  "realm_access": {
    "roles": ["user", "default-roles-pix-system"]
  },
  "resource_access": {
    "pix-gateway": {
      "roles": ["uma_protection"]
    }
  },
  "name": "João Silva",
  "email": "joao@email.com"
}
```

### Validação do JWT

```java
// Spring Security faz automaticamente:
// 1. Busca chaves públicas no Keycloak (JWKS)
// 2. Valida assinatura RS256
// 3. Verifica exp (expiração)
// 4. Verifica iss (issuer)
// 5. Extrai claims para Authentication
```

---

## 📚 12.5 Comunicação Service-to-Service

### O Problema

```
Transfer Service precisa chamar Account Service
Como autenticar sem usuário logado?

Solução: Client Credentials Flow
```

### Fluxo Client Credentials

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────────┐
│ Transfer Service│         │   Keycloak   │         │ Account Service │
└────────┬────────┘         └──────┬───────┘         └────────┬────────┘
         │                         │                          │
         │ 1. POST /token          │                          │
         │ (client_id, secret)     │                          │
         │────────────────────────►│                          │
         │                         │                          │
         │ 2. Access Token         │                          │
         │◄────────────────────────│                          │
         │                         │                          │
         │ 3. GET /accounts/123                               │
         │ Authorization: Bearer <token>                      │
         │───────────────────────────────────────────────────►│
         │                         │                          │
         │ 4. Response                                        │
         │◄───────────────────────────────────────────────────│
```

### Implementação com Feign

```java
// Configuração do OAuth2 Client
@Configuration
public class OAuth2ClientConfig {

    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientRepository authorizedClientRepository) {
        
        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();

        DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository, authorizedClientRepository);
        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

        return authorizedClientManager;
    }
}

// Interceptor para Feign
@Component
public class OAuth2FeignInterceptor implements RequestInterceptor {

    private final OAuth2AuthorizedClientManager clientManager;

    @Override
    public void apply(RequestTemplate template) {
        OAuth2AuthorizeRequest request = OAuth2AuthorizeRequest
            .withClientRegistrationId("keycloak")
            .principal("pix-transfer-service")
            .build();

        OAuth2AuthorizedClient client = clientManager.authorize(request);
        
        if (client != null) {
            String token = client.getAccessToken().getTokenValue();
            template.header("Authorization", "Bearer " + token);
        }
    }
}
```

### application.yml

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: pix-transfer-service
            client-secret: ${KEYCLOAK_CLIENT_SECRET}
            authorization-grant-type: client_credentials
            scope: account:read,pix:write
        provider:
          keycloak:
            token-uri: http://localhost:8180/realms/pix-system/protocol/openid-connect/token
```

---

## 📚 12.6 Fine-Grained Authorization

### Níveis de Autorização

```
┌─────────────────────────────────────────────────────────────────┐
│                    Authorization Levels                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Level 1: Authenticated                                         │
│  → Qualquer usuário logado                                      │
│                                                                  │
│  Level 2: Role-Based (RBAC)                                     │
│  → hasRole('admin'), hasRole('user')                            │
│                                                                  │
│  Level 3: Scope-Based                                           │
│  → hasAuthority('SCOPE_pix:write')                              │
│                                                                  │
│  Level 4: Resource-Based (ABAC)                                 │
│  → Usuário só pode ver SUAS contas                              │
│  → @PostAuthorize("returnObject.ownerId == principal.id")       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementação ABAC

```java
@Service
public class AccountService {

    @PreAuthorize("hasAuthority('SCOPE_account:read')")
    @PostAuthorize("returnObject.ownerId == authentication.name or hasRole('admin')")
    public Account findById(String accountId) {
        return accountRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException(accountId));
    }
    
    @PreAuthorize("#request.fromAccountId == authentication.name or hasRole('admin')")
    public TransferResult transfer(TransferRequest request) {
        // Só pode transferir DA PRÓPRIA conta (ou admin)
        return transferService.execute(request);
    }
}
```

### Custom Security Expression

```java
@Component("accountSecurity")
public class AccountSecurityEvaluator {

    private final AccountRepository accountRepository;

    public boolean isOwner(String accountId, Authentication auth) {
        return accountRepository.findById(accountId)
            .map(account -> account.getOwnerId().equals(auth.getName()))
            .orElse(false);
    }
}

// Uso
@PreAuthorize("@accountSecurity.isOwner(#accountId, authentication)")
public Account getAccount(String accountId) { ... }
```

---

## 🎯 Perguntas de Entrevista

1. **Qual a diferença entre OAuth2 e OpenID Connect?**
2. **Explique o fluxo Client Credentials**
3. **Como funciona a validação de um JWT?**
4. **O que é um Resource Server vs Authorization Server?**
5. **Como implementar comunicação segura entre microsserviços?**
6. **RBAC vs ABAC - quando usar cada um?**
7. **Por que usar Keycloak em vez de implementar autenticação própria?**

---

> **Próximo módulo:** [Módulo 13 - Spring Modulith](MODULO_13_SPRING_MODULITH.md) (opcional)
