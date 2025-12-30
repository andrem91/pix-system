# 📖 Módulo 2: Java Moderno 17+ (1 semana)

> Features modernas do Java que demonstram que você está atualizado. Essencial para entrevistas.

---

## 📚 2.1 Records

### O Problema
```java
// Classe tradicional para dados imutáveis - VERBOSE
public class Usuario {
    private final Long id;
    private final String nome;
    private final String email;
    
    public Usuario(Long id, String nome, String email) {
        this.id = id;
        this.nome = nome;
        this.email = email;
    }
    
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public String getEmail() { return email; }
    
    @Override
    public boolean equals(Object o) { /* ... */ }
    
    @Override
    public int hashCode() { /* ... */ }
    
    @Override
    public String toString() { /* ... */ }
}
// 30+ linhas para APENAS armazenar dados!
```

### A Solução: Records
```java
// Record - 1 linha!
public record Usuario(Long id, String nome, String email) { }

// Automaticamente gera:
// - Construtor canônico
// - Getters (nome(), não getNome())
// - equals() baseado em TODOS os campos
// - hashCode() baseado em TODOS os campos
// - toString() descritivo
```

### Construtor Compacto
```java
public record Usuario(Long id, String nome, String email) {
    
    // Construtor compacto para validação
    public Usuario {
        Objects.requireNonNull(email, "Email é obrigatório");
        email = email.toLowerCase().trim();
    }
}
```

### Métodos Adicionais
```java
public record Retangulo(double largura, double altura) {
    
    // Método adicional
    public double area() {
        return largura * altura;
    }
    
    // Método estático factory
    public static Retangulo quadrado(double lado) {
        return new Retangulo(lado, lado);
    }
}
```

### Records com JPA
```java
// ✅ Records são ótimos para DTOs e Projections
public record UsuarioResumo(Long id, String nome) { }

// No repository:
@Query("SELECT new com.example.UsuarioResumo(u.id, u.nome) FROM Usuario u")
List<UsuarioResumo> findAllResumo();

// ❌ Records NÃO devem ser usados como Entities JPA
// (são imutáveis, JPA precisa de mutabilidade)
```

### Quando Usar

| ✅ Use Records | ❌ Não Use |
|----------------|-----------|
| DTOs | JPA Entities |
| Value Objects | Classes com herança complexa |
| Projections | Classes que precisam ser mutáveis |
| Eventos | Classes com muito comportamento |

---

## 📚 2.2 Sealed Classes

### O Problema
```java
// Como garantir que só certas classes estendam?
public abstract class Forma { }

public class Circulo extends Forma { }  // OK
public class MalwareClass extends Forma { } // Também pode! 😱
```

### A Solução
```java
// Sealed class - define quem pode estender
public sealed class Pagamento 
    permits CartaoCredito, Pix, Boleto {
    
    protected BigDecimal valor;
}

// Subclasses devem ser: final, sealed, ou non-sealed
public final class CartaoCredito extends Pagamento {
    private String numeroCartao;
}

public final class Pix extends Pagamento {
    private String chave;
}

public final class Boleto extends Pagamento {
    private String codigoBarras;
}
```

### Sealed Interfaces
```java
public sealed interface Resultado 
    permits Sucesso, Erro, EmProcessamento {
}

public record Sucesso(Object valor) implements Resultado { }
public record Erro(String mensagem) implements Resultado { }
public record EmProcessamento(String status) implements Resultado { }
```

### Benefícios
1. **Exhaustiveness:** Compilador sabe todas as subclasses
2. **Segurança:** Código externo não pode estender
3. **Pattern Matching:** Combinação perfeita com switch

---

## 📚 2.3 Pattern Matching

### Pattern Matching para instanceof
```java
// ❌ Antes do Java 16
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// ✅ Java 16+
if (obj instanceof String s) {
    System.out.println(s.length());
}

// Com condição adicional
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s.toUpperCase());
}
```

### Pattern Matching para switch
```java
// Switch com tipos
String descricao = switch (obj) {
    case Integer i -> "Número inteiro: " + i;
    case Long l -> "Número longo: " + l;
    case String s -> "Texto: " + s;
    case null -> "Valor nulo";
    default -> "Tipo desconhecido";
};

// Switch com sealed classes - EXAUSTIVO!
String processarPagamento(Pagamento p) {
    return switch (p) {
        case CartaoCredito cc -> processarCartao(cc);
        case Pix pix -> processarPix(pix);
        case Boleto b -> processarBoleto(b);
        // Não precisa de default - compilador sabe que é exaustivo!
    };
}

// Switch com Records (deconstruction)
record Ponto(int x, int y) { }

String analisarPonto(Object obj) {
    return switch (obj) {
        case Ponto(int x, int y) when x == 0 && y == 0 -> "Origem";
        case Ponto(int x, int y) when x == y -> "Diagonal";
        case Ponto(int x, int y) -> "Ponto(%d, %d)".formatted(x, y);
        default -> "Não é um ponto";
    };
}
```

### Guarded Patterns
```java
String classificar(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Negativo";
        case Integer i when i == 0 -> "Zero";
        case Integer i when i > 0 -> "Positivo";
        case String s when s.isEmpty() -> "String vazia";
        case String s -> "String: " + s;
        default -> "Outro";
    };
}
```

---

## 📚 2.4 Text Blocks

### Strings Multi-linha
```java
// ❌ Antes
String json = "{\n" +
    "    \"nome\": \"João\",\n" +
    "    \"idade\": 30\n" +
    "}";

// ✅ Com Text Blocks
String json = """
    {
        "nome": "João",
        "idade": 30
    }
    """;

// SQL legível
String sql = """
    SELECT u.id, u.nome, u.email
    FROM usuarios u
    WHERE u.ativo = true
      AND u.created_at > :data
    ORDER BY u.nome
    """;

// HTML
String html = """
    <html>
        <body>
            <h1>Olá, %s!</h1>
        </body>
    </html>
    """.formatted(nome);
```

### Métodos Úteis
```java
String texto = """
    Linha 1
    Linha 2
    """;
    
// stripIndent() - remove indentação comum (automático)
// translateEscapes() - processa \n, \t, etc.

// formatted() - substitui %s, %d
String saudacao = """
    Olá, %s!
    Você tem %d mensagens.
    """.formatted("João", 5);
```

---

## 📚 2.5 Outras Features Úteis

### var (Java 10+)
```java
// Inferência de tipo local
var lista = new ArrayList<String>(); // ArrayList<String>
var mapa = Map.of("a", 1, "b", 2);   // Map<String, Integer>

// Útil para tipos complexos
var resultado = stream
    .collect(Collectors.groupingBy(
        Pessoa::getCidade,
        Collectors.mapping(Pessoa::getNome, Collectors.toList())
    )); // Evita: Map<String, List<String>>

// ❌ Evite quando obscurece o tipo
var x = getData(); // O que retorna?
```

### switch Expression (Java 14+)
```java
// Retorna valor
int diasNoMes = switch (mes) {
    case 1, 3, 5, 7, 8, 10, 12 -> 31;
    case 4, 6, 9, 11 -> 30;
    case 2 -> 28;
    default -> throw new IllegalArgumentException("Mês inválido");
};

// Com bloco e yield
String resultado = switch (status) {
    case PENDING -> {
        log.info("Processando...");
        yield "Em andamento";
    }
    case COMPLETED -> "Concluído";
    case FAILED -> "Falhou";
};
```

### Helpful NullPointerExceptions (Java 14+)
```java
// Antes: java.lang.NullPointerException
// Depois: java.lang.NullPointerException: 
//         Cannot invoke "String.toUpperCase()" because "user.getAddress().getCity()" is null

// Ativar (default em Java 15+):
// java -XX:+ShowCodeDetailsInExceptionMessages
```

### SequencedCollections (Java 21+)
```java
SequencedCollection<String> lista = new ArrayList<>();
lista.addFirst("primeiro");
lista.addLast("último");
String primeiro = lista.getFirst();
String ultimo = lista.getLast();
lista.reversed(); // Vista reversa
```

---

## 🎯 Perguntas de Entrevista

1. **O que são Records? Quando usar?**
   - Classes imutáveis para dados
   - DTOs, Value Objects, Projections

2. **O que são Sealed Classes?**
   - Classes com subclasses restritas
   - Combinam bem com pattern matching

3. **Qual a vantagem de Text Blocks?**
   - Legibilidade de strings multi-linha
   - Menos escapes, formatação preservada

4. **Quando usar var?**
   - Tipos óbvios ou muito longos
   - Não usar quando obscurece o tipo

---

> **Próximo módulo:** [Módulo 3 - OOP e SOLID](MODULO_03_OOP_SOLID.md)
