# 📖 Módulo 1: Fundamentos Java Core (3 semanas)

> Dominar a base do Java é essencial. Este módulo cobre Collections, algoritmos, Streams, Optional, Concorrência e Memória.

---

## 📚 1.1 Collections Framework

### Hierarquia Completa

```
                    Iterable
                       │
                   Collection
          ┌────────────┼────────────┐
         List         Set         Queue
          │            │            │
    ┌─────┼─────┐  ┌───┼───┐    ┌───┼───┐
ArrayList LinkedList HashSet TreeSet PriorityQueue
                   LinkedHashSet     Deque
```

### List Implementations

#### ArrayList
```java
// Array dinâmico - redimensiona quando cheio
ArrayList<String> lista = new ArrayList<>();

// Características:
// - Acesso por índice: O(1)
// - Inserção no final: O(1) amortizado
// - Inserção no meio: O(n) - precisa mover elementos
// - Busca por valor: O(n)

// Quando usar: Acesso aleatório frequente, maioria dos casos
```

#### LinkedList
```java
// Lista duplamente encadeada
LinkedList<String> lista = new LinkedList<>();

// Características:
// - Acesso por índice: O(n) - percorre nós
// - Inserção nas pontas: O(1)
// - Inserção no meio: O(n) para encontrar + O(1) para inserir
// - Implementa Deque - pode ser usada como pilha ou fila

// Quando usar: Filas (Queue), inserção/remoção frequente nas pontas
```

#### Vector (Legado)
```java
// Versão thread-safe de ArrayList - EVITE
Vector<String> vector = new Vector<>();

// Problema: Sincronização em cada operação = lento
// Alternativa moderna:
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
// Ou melhor: CopyOnWriteArrayList para leituras frequentes
```

### Set Implementations

#### HashSet
```java
HashSet<String> set = new HashSet<>();

// Internamente usa HashMap (value é dummy)
// Características:
// - add/remove/contains: O(1) médio
// - Sem ordem garantida
// - Permite null
// - Requer equals() e hashCode() corretos

// Como funciona:
// 1. Calcula hash do elemento
// 2. Encontra bucket no array interno
// 3. Se colisão: Java 8+ usa árvore após 8 elementos
```

#### LinkedHashSet
```java
LinkedHashSet<String> set = new LinkedHashSet<>();

// Mantém ordem de inserção
// Usa lista duplamente encadeada internamente
// Ligeiramente mais lento que HashSet
// Útil quando precisa de unicidade + ordem
```

#### TreeSet
```java
TreeSet<String> set = new TreeSet<>();

// Ordenado naturalmente ou por Comparator
// Internamente usa Red-Black Tree
// Características:
// - add/remove/contains: O(log n)
// - Navegação: first(), last(), lower(), higher()
// - Requer Comparable ou Comparator

// Exemplo com Comparator:
TreeSet<Pessoa> pessoas = new TreeSet<>(Comparator.comparing(Pessoa::getNome));
```

### Map Implementations

#### HashMap
```java
HashMap<String, Integer> map = new HashMap<>();

// O mais usado
// Características:
// - get/put/remove: O(1) médio
// - Permite 1 null key, múltiplos null values
// - Não thread-safe
// - Load factor: 0.75 (rehash quando 75% cheio)

// Estrutura interna:
// - Array de buckets
// - Cada bucket: LinkedList ou Tree (8+ elementos)
// - Capacidade inicial: 16

// Colisão:
// Quando dois objetos têm mesmo hashCode % capacidade
// Resolvido com encadeamento (lista/árvore no bucket)
```

#### LinkedHashMap
```java
LinkedHashMap<String, Integer> map = new LinkedHashMap<>();

// Mantém ordem de inserção
// Pode manter ordem de acesso (útil para LRU Cache)

// LRU Cache implementation:
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxEntries;
    
    public LRUCache(int maxEntries) {
        super(maxEntries, 0.75f, true); // true = access order
        this.maxEntries = maxEntries;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxEntries;
    }
}
```

#### TreeMap
```java
TreeMap<String, Integer> map = new TreeMap<>();

// Ordenado por chave
// Internamente usa Red-Black Tree
// Características:
// - get/put/remove: O(log n)
// - Navegação: firstKey(), lastKey(), lowerKey(), higherKey()
// - Não permite null key (pode ter null values)
```

#### ConcurrentHashMap
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Thread-safe sem sincronização global
// Usa locks por segmento (segment locking)
// Características:
// - Operações atômicas: putIfAbsent, computeIfAbsent
// - Não permite null key ou value
// - Melhor performance que Collections.synchronizedMap

// Operações atômicas:
map.computeIfAbsent("key", k -> expensiveComputation());
map.merge("key", 1, Integer::sum); // Incrementa atomicamente
```

### Perguntas de Entrevista

1. **"Qual a diferença entre ArrayList e LinkedList?"**
   - ArrayList: array dinâmico, O(1) acesso, O(n) inserção no meio
   - LinkedList: nós encadeados, O(n) acesso, O(1) inserção nas pontas
   - Na prática: ArrayList 95% dos casos

2. **"Como HashMap resolve colisões?"**
   - Encadeamento: lista/árvore no bucket
   - Java 8+: converte para árvore após 8 elementos
   - equals() determina qual elemento retornar

3. **"Por que ConcurrentHashMap não permite null?"**
   - Ambiguidade: null significa "não existe" ou "valor é null"?
   - get() retorna null = key não existe ou value = null?

---

## 📚 1.2 Big O Notation

### Complexidades Comuns

| O(?) | Nome | Descrição | Exemplo |
|------|------|-----------|---------|
| O(1) | Constante | Sempre mesmo tempo | HashMap.get(), array[i] |
| O(log n) | Logarítmica | Divide problema pela metade | Binary search, TreeMap |
| O(n) | Linear | Proporcional ao input | Loop simples, ArrayList.contains() |
| O(n log n) | Linearítmica | Ordenação eficiente | Arrays.sort(), merge sort |
| O(n²) | Quadrática | Loops aninhados | Bubble sort, comparar todos pares |
| O(2^n) | Exponencial | Dobra a cada elemento | Subsets, Fibonacci recursivo |

### Análise de Algoritmos Comuns

```java
// O(1) - Constante
public int getFirst(int[] arr) {
    return arr[0]; // Sempre 1 operação
}

// O(n) - Linear
public boolean contains(int[] arr, int target) {
    for (int num : arr) { // Percorre todos
        if (num == target) return true;
    }
    return false;
}

// O(n²) - Quadrática
public void printPairs(int[] arr) {
    for (int i = 0; i < arr.length; i++) {       // n vezes
        for (int j = 0; j < arr.length; j++) {   // n vezes cada
            System.out.println(arr[i] + ", " + arr[j]);
        }
    }
} // Total: n * n = n²

// O(log n) - Logarítmica
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
} // Divide pela metade a cada iteração
```

### Complexidade de Espaço

```java
// O(1) espaço - in-place
public void reverse(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left++] = arr[right];
        arr[right--] = temp;
    }
}

// O(n) espaço - cria nova estrutura
public int[] doubled(int[] arr) {
    int[] result = new int[arr.length]; // Aloca n elementos
    for (int i = 0; i < arr.length; i++) {
        result[i] = arr[i] * 2;
    }
    return result;
}
```

---

## 📚 1.3 Equals e HashCode

### Contrato

```java
// REGRA OBRIGATÓRIA:
// Se a.equals(b) == true → a.hashCode() == b.hashCode()

// NÃO É OBRIGATÓRIO (mas ideal):
// Se a.hashCode() == b.hashCode() → a.equals(b) pode ser true ou false
```

### Implementação Correta

```java
public class Pessoa {
    private String cpf;
    private String nome;
    private int idade;
    
    @Override
    public boolean equals(Object o) {
        // 1. Mesma referência? Retorna true
        if (this == o) return true;
        
        // 2. Null ou classe diferente? Retorna false
        if (o == null || getClass() != o.getClass()) return false;
        
        // 3. Cast e compara campos relevantes
        Pessoa pessoa = (Pessoa) o;
        return Objects.equals(cpf, pessoa.cpf);
        // NOTA: Apenas CPF! É a identidade de negócio
    }
    
    @Override
    public int hashCode() {
        // DEVE usar OS MESMOS campos do equals
        return Objects.hash(cpf);
    }
}
```

### Problemas Comuns

```java
// ❌ PROBLEMA 1: hashCode usa campo diferente de equals
@Override
public boolean equals(Object o) {
    // Compara cpf
    return Objects.equals(cpf, ((Pessoa) o).cpf);
}

@Override
public int hashCode() {
    return Objects.hash(nome); // USA NOME! Quebra o contrato
}
// Resultado: HashSet/HashMap não funcionam corretamente

// ❌ PROBLEMA 2: Usar campos mutáveis
public class Produto {
    private String codigo;
    private BigDecimal preco; // Mutável!
    
    public void setPreco(BigDecimal preco) { this.preco = preco; }
    
    @Override
    public int hashCode() {
        return Objects.hash(codigo, preco); // Muda se preço mudar!
    }
}
// Resultado: Objeto "desaparece" do HashSet após modificação

// ✅ SOLUÇÃO: Use apenas campos imutáveis no hashCode
```

### Com Lombok
```java
@Data // Gera equals/hashCode com TODOS os campos
public class Pessoa { }

@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Pessoa {
    @EqualsAndHashCode.Include
    private String cpf; // Apenas CPF usado
    
    private String nome; // Ignorado
}
```

---

## 📚 1.4 Streams API

### Operações Intermediárias (Lazy)

```java
List<String> nomes = Arrays.asList("Ana", "Bruno", "Carlos", "Ana");

// filter - filtra elementos
nomes.stream()
    .filter(n -> n.startsWith("A"))
    .forEach(System.out::println); // Ana, Ana

// map - transforma elementos
nomes.stream()
    .map(String::toUpperCase)
    .forEach(System.out::println); // ANA, BRUNO, CARLOS, ANA

// flatMap - achata estruturas aninhadas
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
nested.stream()
    .flatMap(List::stream)
    .forEach(System.out::println); // 1, 2, 3, 4

// distinct - remove duplicados
nomes.stream()
    .distinct()
    .forEach(System.out::println); // Ana, Bruno, Carlos

// sorted - ordena
nomes.stream()
    .sorted()
    .forEach(System.out::println); // Ana, Ana, Bruno, Carlos

// limit/skip - paginação
nomes.stream()
    .skip(1)
    .limit(2)
    .forEach(System.out::println); // Bruno, Carlos

// peek - debug (não altera)
nomes.stream()
    .peek(n -> System.out.println("Processando: " + n))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

### Operações Terminais (Executam)

```java
// collect - coleta em coleção
List<String> lista = nomes.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());

// forEach - itera
nomes.stream().forEach(System.out::println);

// reduce - reduz a um valor
int soma = numeros.stream()
    .reduce(0, Integer::sum);

BigDecimal total = pedidos.stream()
    .map(Pedido::getValor)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// count - conta elementos
long count = nomes.stream()
    .filter(n -> n.startsWith("A"))
    .count();

// findFirst/findAny - encontra elemento
Optional<String> primeiro = nomes.stream()
    .filter(n -> n.startsWith("B"))
    .findFirst();

// anyMatch/allMatch/noneMatch - predicados
boolean temAna = nomes.stream().anyMatch(n -> n.equals("Ana"));
boolean todosComA = nomes.stream().allMatch(n -> n.startsWith("A"));
boolean nenhumVazio = nomes.stream().noneMatch(String::isEmpty);

// min/max - extremos
Optional<String> menor = nomes.stream()
    .min(Comparator.naturalOrder());
```

### Collectors Avançados

```java
// toList, toSet
List<String> lista = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());

// toMap
Map<Long, Usuario> porId = usuarios.stream()
    .collect(Collectors.toMap(
        Usuario::getId,
        Function.identity()
    ));

// groupingBy - agrupa por chave
Map<String, List<Produto>> porCategoria = produtos.stream()
    .collect(Collectors.groupingBy(Produto::getCategoria));

// groupingBy com downstream
Map<String, Long> contagemPorCategoria = produtos.stream()
    .collect(Collectors.groupingBy(
        Produto::getCategoria,
        Collectors.counting()
    ));

Map<String, BigDecimal> totalPorCategoria = produtos.stream()
    .collect(Collectors.groupingBy(
        Produto::getCategoria,
        Collectors.reducing(BigDecimal.ZERO, Produto::getPreco, BigDecimal::add)
    ));

// partitioningBy - divide em 2 grupos
Map<Boolean, List<Produto>> carosEBaratos = produtos.stream()
    .collect(Collectors.partitioningBy(p -> p.getPreco().compareTo(new BigDecimal("100")) > 0));

// joining - concatena strings
String nomesConcatenados = nomes.stream()
    .collect(Collectors.joining(", ", "[", "]")); // [Ana, Bruno, Carlos]
```

### Parallel Streams

```java
// Quando usar:
// ✅ Operações CPU-bound (cálculos pesados)
// ✅ Grandes volumes de dados (10k+ elementos)
// ✅ Operações stateless e sem side effects

// Quando NÃO usar:
// ❌ I/O operations (bloqueiam threads do pool)
// ❌ Operações com estado compartilhado
// ❌ Coleções pequenas (overhead > benefício)

// Exemplo correto:
long sum = IntStream.range(0, 1_000_000)
    .parallel()
    .mapToLong(i -> heavyComputation(i))
    .sum();

// ❌ ERRADO: Modifica estado compartilhado
List<Integer> results = new ArrayList<>(); // NÃO THREAD-SAFE!
stream.parallel().forEach(n -> results.add(n)); // Race condition!

// ✅ CORRETO
List<Integer> results = stream.parallel()
    .collect(Collectors.toList()); // Thread-safe
```

---

## 📚 1.5 Optional

### Criação

```java
// Valor presente
Optional<String> nome = Optional.of("João"); // NPE se null!
Optional<String> nome = Optional.ofNullable(valor); // null → empty

// Valor ausente
Optional<String> vazio = Optional.empty();
```

### Uso Correto

```java
// ✅ Encadeamento fluente
return repository.findById(id)
    .map(this::toDto)
    .orElseThrow(() -> new NotFoundException("Não encontrado"));

// ✅ Valor default
String nome = optional.orElse("Anônimo");

// ✅ Supplier para valor default (lazy)
String nome = optional.orElseGet(() -> buscarNomePadrao());

// ✅ ifPresent para efeitos colaterais
optional.ifPresent(valor -> log.info("Encontrado: {}", valor));

// ✅ ifPresentOrElse (Java 9+)
optional.ifPresentOrElse(
    valor -> processarValor(valor),
    () -> log.warn("Valor não encontrado")
);

// ✅ or - fallback para outro Optional (Java 9+)
Optional<User> user = primaryRepo.findById(id)
    .or(() -> secondaryRepo.findById(id));
```

### Anti-Patterns

```java
// ❌ NUNCA: get() sem verificação
String nome = optional.get(); // NoSuchElementException se vazio!

// ❌ NUNCA: Optional como parâmetro
public void process(Optional<String> nome) { } // Confuso, use @Nullable

// ❌ NUNCA: Optional em campos de classe
public class Pessoa {
    private Optional<String> apelido; // NÃO! Use @Nullable ou valor padrão
}

// ❌ EVITAR: isPresent() + get()
if (optional.isPresent()) {
    return optional.get(); // Use orElse/map/flatMap
}

// ❌ EVITAR: Optional com collections
Optional<List<String>> nomes; // Use lista vazia: Collections.emptyList()
```

---

## 📚 1.6 Concorrência

### Threads Básico

```java
// Runnable - não retorna valor
Runnable task = () -> System.out.println("Executando");
Thread thread = new Thread(task);
thread.start();

// Callable - retorna valor
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};
```

### ExecutorService

```java
// Pool de threads fixo
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submeter tarefas
Future<String> future = executor.submit(() -> {
    return processarDados();
});

// Aguardar resultado (bloqueia)
try {
    String resultado = future.get(5, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    future.cancel(true);
}

// SEMPRE encerrar o executor
executor.shutdown();
try {
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
}
```

### CompletableFuture (Essencial!)

```java
// Assíncrono sem bloqueio
CompletableFuture<User> userFuture = CompletableFuture
    .supplyAsync(() -> userService.findById(id));

// Encadeamento
CompletableFuture<OrderDto> orderFuture = CompletableFuture
    .supplyAsync(() -> orderRepository.findById(id))
    .thenApply(order -> enrichWithCustomer(order))
    .thenApply(order -> convertToDto(order))
    .exceptionally(ex -> {
        log.error("Erro ao processar pedido", ex);
        return OrderDto.empty();
    });

// Combinar múltiplos futures
CompletableFuture<User> userFuture = fetchUser(id);
CompletableFuture<List<Order>> ordersFuture = fetchOrders(id);
CompletableFuture<Address> addressFuture = fetchAddress(id);

CompletableFuture<UserProfile> profileFuture = userFuture
    .thenCombine(ordersFuture, (user, orders) -> new UserWithOrders(user, orders))
    .thenCombine(addressFuture, (uo, address) -> new UserProfile(uo.user, uo.orders, address));

// Aguardar todos
CompletableFuture.allOf(future1, future2, future3)
    .thenRun(() -> log.info("Todos concluídos"));

// Primeiro que completar
CompletableFuture.anyOf(future1, future2, future3)
    .thenAccept(result -> log.info("Primeiro resultado: {}", result));
```

### Virtual Threads (Java 21+)

```java
// Threads leves gerenciadas pela JVM (não pelo OS)
// Permite milhares de threads simultâneas sem overhead

// Executor de virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i ->
        executor.submit(() -> {
            // Cada tarefa em sua virtual thread
            processarTarefa(i);
        })
    );
}

// Thread.startVirtualThread
Thread.startVirtualThread(() -> {
    // Executado em virtual thread
    processarDados();
});

// Quando usar:
// ✅ I/O-bound tasks (HTTP calls, DB queries)
// ✅ Alta concorrência (milhares de requisições)
// ❌ CPU-bound tasks (use platform threads)
```

### Sincronização

```java
// synchronized - lock implícito
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Lock explícito - mais controle
public class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}

// AtomicInteger - operações atômicas sem lock
public class Counter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}
```

---

## 📚 1.7 Memória e Garbage Collection

### Java Memory Model

```
┌──────────────────────────────────────────────────────────┐
│                        JVM                                │
├────────────────────────┬─────────────────────────────────┤
│         HEAP           │          NON-HEAP               │
│   (objetos e arrays)   │                                 │
├────────────────────────┤   ┌─────────────────────────┐   │
│      Young Generation  │   │      Metaspace          │   │
│   ┌──────┬──────┬────┐ │   │   (class metadata,      │   │
│   │ Eden │  S0  │ S1 │ │   │    method info)         │   │
│   └──────┴──────┴────┘ │   └─────────────────────────┘   │
├────────────────────────┤   ┌─────────────────────────┐   │
│      Old Generation    │   │      Code Cache         │   │
│   (objetos de longa    │   │   (compiled code)       │   │
│    duração)            │   └─────────────────────────┘   │
└────────────────────────┴─────────────────────────────────┤
│                    Thread Stacks                          │
│   (variáveis locais, referências, call stack)            │
└──────────────────────────────────────────────────────────┘
```

### Heap vs Stack

| Heap | Stack |
|------|-------|
| Objetos, arrays | Variáveis locais primitivas |
| Compartilhado entre threads | Cada thread tem sua stack |
| GC gerencia | Automático (escopo) |
| OutOfMemoryError | StackOverflowError |
| -Xmx, -Xms | -Xss |

### Garbage Collectors

| GC | Uso | Características |
|----|-----|-----------------|
| **G1** | Default (Java 11+) | Bom balanço latência/throughput |
| **ZGC** | Ultra baixa latência | Pause < 10ms, heaps grandes |
| **Shenandoah** | Baixa latência | Alternativa ao ZGC |
| **Parallel** | Throughput | Pausas maiores, mais throughput |

```bash
# Escolher GC
java -XX:+UseG1GC -jar app.jar
java -XX:+UseZGC -jar app.jar
```

### Memory Leaks Comuns

```java
// ❌ Static collections que crescem indefinidamente
public class Cache {
    private static final Map<String, Object> cache = new HashMap<>();
    // Nunca remove! Leak!
}

// ❌ Listeners não removidos
button.addActionListener(listener);
// Esqueceu: button.removeActionListener(listener);

// ❌ Conexões não fechadas
Connection conn = dataSource.getConnection();
// Esqueceu: conn.close();

// ✅ Use try-with-resources
try (Connection conn = dataSource.getConnection()) {
    // usa conexão
} // Fechada automaticamente

// ❌ Threads não encerradas
ExecutorService executor = Executors.newFixedThreadPool(10);
// Esqueceu: executor.shutdown();
```

---

## 📝 Exercícios Práticos

1. **LRU Cache:** Implemente usando LinkedHashMap
2. **Anagramas:** Agrupe palavras por anagramas usando Streams
3. **Producer/Consumer:** Use BlockingQueue com múltiplas threads
4. **Rate Limiter:** Implemente com Semaphore ou AtomicInteger

---

## 🎯 Perguntas de Entrevista

1. ArrayList vs LinkedList - quando usar?
2. Como HashMap resolve colisões?
3. Por que equals e hashCode juntos?
4. map vs flatMap em Streams?
5. Quando usar parallel streams?
6. Runnable vs Callable?
7. O que são Virtual Threads?
8. Como evitar memory leaks?

---

> **Próximo módulo:** [Módulo 2 - Java Moderno 17+](MODULO_02_JAVA_MODERNO.md)
