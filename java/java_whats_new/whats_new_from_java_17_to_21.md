# Guide complet : Nouveautés Java 21 (LTS)

Ci-dessous les principales nouveautés introduites entre Java 17 et Java 21, avec des exemples pratiques pour votre migration.

## 1 - Pattern Matching for switch (Java 21)

- **Utilité** : Combinaison de pattern matching, guards et switch expressions pour un code plus expressif.

```java
// Avant (Java 17)
static String format(Object obj) {
    String formatted;
    if (obj instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (obj instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (obj instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (obj instanceof String s) {
        formatted = String.format("String %s", s);
    } else {
        formatted = obj.toString();
    }
    return formatted;
}

// Après (Java 21)
static String format(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}
```

### Avec guards (when) :

```java
// Avant (Java 17)
static String classify(String s) {
    if (s == null) {
        return "null";
    } else if (s.isEmpty()) {
        return "vide";
    } else if (s.length() < 5) {
        return "court";
    } else {
        return "long";
    }
}

// Après (Java 21)
static String classify(String s) {
    return switch (s) {
        case null              -> "null";
        case String str when str.isEmpty() -> "vide";
        case String str when str.length() < 5 -> "court";
        default                -> "long";
    };
}
```

### Pattern matching avec records :

```java
record Point(int x, int y) {}

// Avant (Java 17)
static void printPoint(Object obj) {
    if (obj instanceof Point) {
        Point p = (Point) obj;
        System.out.println("x: " + p.x() + ", y: " + p.y());
    }
}

// Après (Java 21) - Record Patterns
static void printPoint(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println("x: " + x + ", y: " + y);
    }
}

// Avec switch
static String describe(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == y -> "Point sur diagonale";
        case Point(int x, int y) when x == 0 -> "Point sur axe Y";
        case Point(int x, int y) when y == 0 -> "Point sur axe X";
        case Point(int x, int y)              -> "Point (%d, %d)".formatted(x, y);
        default                               -> "Pas un point";
    };
}
```

- **Impact** : Code beaucoup plus concis et expressif, exhaustivité vérifiée à la compilation, moins d'erreurs de cast.


## 2 - Record Patterns (Java 21)

- **Utilité** : Décomposition de records directement dans les patterns.

```java
record Person(String name, int age) {}
record Employee(Person person, String department) {}

// Avant (Java 17)
static void process(Object obj) {
    if (obj instanceof Employee) {
        Employee emp = (Employee) obj;
        Person person = emp.person();
        String name = person.name();
        int age = person.age();
        String dept = emp.department();
        System.out.println(name + ", " + age + ", " + dept);
    }
}

// Après (Java 21) - Nested Record Patterns
static void process(Object obj) {
    if (obj instanceof Employee(Person(String name, int age), String dept)) {
        System.out.println(name + ", " + age + ", " + dept);
    }
}

// Avec switch
static String format(Object obj) {
    return switch (obj) {
        case Employee(Person(String name, int age), String dept) 
            when age > 50 -> "%s (senior) - %s".formatted(name, dept);
        case Employee(Person(String name, int age), String dept) 
            -> "%s (%d ans) - %s".formatted(name, age, dept);
        default -> "Unknown";
    };
}
```

- **Impact** : Décomposition élégante des structures de données, code très lisible, moins de variables temporaires.


## 3 - Virtual Threads (Project Loom - Java 21)

- **Utilité** : Threads légers gérés par la JVM permettant des millions de threads concurrents avec peu de ressources.

```java
// Avant (Java 17) - Platform Threads
ExecutorService executor = Executors.newFixedThreadPool(100);

for (int i = 0; i < 10000; i++) {
    executor.submit(() -> {
        // Tâche bloquante
        try {
            Thread.sleep(1000);
            // Traitement...
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}
executor.shutdown();

// Après (Java 21) - Virtual Threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10000; i++) {
        executor.submit(() -> {
            // Tâche bloquante
            try {
                Thread.sleep(Duration.ofSeconds(1));
                // Traitement...
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
} // Auto-shutdown
```

### Création directe de virtual threads :

```java
// Créer un virtual thread
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread");
});
vThread.join();

// Avec factory
Thread.Builder builder = Thread.ofVirtual().name("worker-", 0);
Thread t1 = builder.start(() -> System.out.println("Task 1"));
Thread t2 = builder.start(() -> System.out.println("Task 2"));
```

### Serveur HTTP avec virtual threads :

```java
// Avant (Java 17) - Limité par le nombre de threads
ServerSocket serverSocket = new ServerSocket(8080);
ExecutorService executor = Executors.newFixedThreadPool(200);

while (true) {
    Socket client = serverSocket.accept();
    executor.submit(() -> handleClient(client));
}

// Après (Java 21) - Millions de connexions possibles
ServerSocket serverSocket = new ServerSocket(8080);

while (true) {
    Socket client = serverSocket.accept();
    Thread.startVirtualThread(() -> handleClient(client));
}
```

- **Impact** :
  - Performance : Gestion de millions de threads concurrents avec peu de mémoire
  - Scalabilité : Modèle thread-per-request viable même à grande échelle
  - Simplicité : Code synchrone simple, pas besoin de `CompletableFuture` complexes


## 4 - Structured Concurrency (Preview - Java 21)

- **Utilité** : Gestion structurée de tâches concurrentes avec cycle de vie clair.

```java
// Avant (Java 17) - Gestion manuelle difficile
ExecutorService executor = Executors.newCachedThreadPool();
Future<String> user = executor.submit(() -> fetchUser());
Future<String> orders = executor.submit(() -> fetchOrders());

try {
    String userData = user.get();
    String orderData = orders.get();
    return combine(userData, orderData);
} finally {
    executor.shutdown();
}

// Après (Java 21) - Structured Concurrency
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Supplier<String> user = scope.fork(() -> fetchUser());
    Supplier<String> orders = scope.fork(() -> fetchOrders());
    
    scope.join()           // Attend toutes les tâches
         .throwIfFailed(); // Lance exception si échec
    
    return combine(user.get(), orders.get());
}
```

### Avec timeout et gestion d'erreurs :

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Supplier<String> task1 = scope.fork(() -> longRunningTask1());
    Supplier<String> task2 = scope.fork(() -> longRunningTask2());
    Supplier<String> task3 = scope.fork(() -> longRunningTask3());
    
    scope.join(Duration.ofSeconds(5))
         .throwIfFailed();
    
    // Toutes les tâches ont réussi
    return List.of(task1.get(), task2.get(), task3.get());
} catch (TimeoutException e) {
    // Toutes les tâches sont automatiquement annulées
    return Collections.emptyList();
}
```

- **Impact** : Gestion plus sûre de la concurrence, pas de fuite de threads, annulation automatique, code plus maintenable.


## 5 - Scoped Values (Preview - Java 21)

- **Utilité** : Alternative immuable et performante aux ThreadLocal, idéal pour virtual threads.

```java
// Avant (Java 17) - ThreadLocal
private static final ThreadLocal<String> USER_ID = new ThreadLocal<>();

public void process() {
    USER_ID.set("user123");
    try {
        doWork();
    } finally {
        USER_ID.remove(); // Important pour éviter les fuites
    }
}

private void doWork() {
    String userId = USER_ID.get();
    // Utiliser userId
}

// Après (Java 21) - Scoped Values
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

public void process() {
    ScopedValue.where(USER_ID, "user123")
               .run(() -> doWork());
}

private void doWork() {
    String userId = USER_ID.get();
    // Utiliser userId
}
```

### Avec multiple valeurs :

```java
public class RequestContext {
    private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    private static final ScopedValue<Locale> LOCALE = ScopedValue.newInstance();
    
    public void handleRequest(String userId, String requestId, Locale locale) {
        ScopedValue.where(USER_ID, userId)
                   .where(REQUEST_ID, requestId)
                   .where(LOCALE, locale)
                   .run(() -> processRequest());
    }
    
    private void processRequest() {
        log("User: " + USER_ID.get());
        log("Request: " + REQUEST_ID.get());
        String message = getLocalizedMessage(LOCALE.get());
    }
}
```

- **Impact** :
  - Performance : Plus efficace que ThreadLocal avec virtual threads
  - Sécurité : Immuable, pas de remove() à oublier
  - Lisibilité : Scope explicite et limité


## 6 - Sequenced Collections (Java 21)

- **Utilité** : Interfaces pour collections avec ordre défini et accès aux premiers/derniers éléments.

```java
// Avant (Java 17) - APIs incohérentes
List<String> list = new ArrayList<>();
String first = list.get(0);
String last = list.get(list.size() - 1);

Deque<String> deque = new ArrayDeque<>();
String firstDeque = deque.getFirst();
String lastDeque = deque.getLast();

// Après (Java 21) - API unifiée
SequencedCollection<String> collection = new ArrayList<>();
String first = collection.getFirst();
String last = collection.getLast();

// Ajouter au début/fin
collection.addFirst("début");
collection.addLast("fin");

// Collection inversée
SequencedCollection<String> reversed = collection.reversed();
```

### Avec `SequencedSet` et `SequencedMap` :

```java
// SequencedSet
SequencedSet<String> set = new LinkedHashSet<>();
set.add("A");
set.add("B");
set.add("C");

String first = set.getFirst();  // "A"
String last = set.getLast();    // "C"
SequencedSet<String> reversed = set.reversed();

// SequencedMap
SequencedMap<String, Integer> map = new LinkedHashMap<>();
map.put("first", 1);
map.put("second", 2);
map.put("third", 3);

var firstEntry = map.firstEntry();  // first=1
var lastEntry = map.lastEntry();    // third=3
SequencedMap<String, Integer> reversedMap = map.reversed();

// Polling
map.pollFirstEntry();  // Retire et retourne first=1
map.pollLastEntry();   // Retire et retourne third=3
```

- **Impact** : API cohérente pour toutes les collections ordonnées, code plus lisible, moins d'erreurs off-by-one.


## 7 - String Templates (Preview - Java 21)

- **Utilité** : Interpolation de chaînes sécurisée et expressive.

```java
// Avant (Java 17)
String name = "Alice";
int age = 30;
String message = String.format("Bonjour %s, vous avez %d ans", name, age);

String json = """
    {
      "name": "%s",
      "age": %d
    }
    """.formatted(name, age);

// Après (Java 21) - String Templates
String message = STR."Bonjour \{name}, vous avez \{age} ans";

String json = STR."""
    {
      "name": "\{name}",
      "age": \{age}
    }
    """;
Avec expressions complexes :
javarecord Rectangle(double length, double width) {
    double area() { return length * width; }
}

Rectangle rect = new Rectangle(5, 3);

// Expressions dans les templates
String info = STR."Rectangle: \{rect.length()} x \{rect.width()} = \{rect.area()}";

// Calculs
int x = 10, y = 20;
String calc = STR."La somme de \{x} et \{y} est \{x + y}";

// Appels de méthodes
String upper = STR."En majuscules: \{name.toUpperCase()}";
```

### Templates personnalisés (FMT, RAW) :

```java
// FMT - Formatage avec printf
double value = 123.456;
String formatted = FMT."Valeur: %.2f\{value}";  // "Valeur: 123.46"

// RAW - Accès aux parties du template
StringTemplate st = RAW."Le résultat est \{x + y}";
List<String> fragments = st.fragments();  // ["Le résultat est ", ""]
List<Object> values = st.values();        // [30]
```

- **Impact** :
  - Sécurité : Prévention des injections SQL/XSS par design
  - Lisibilité : Plus naturel que String.format()
  - Puissance : Templates personnalisés pour validation


## 8 - Unnamed Patterns and Variables (Preview - Java 21)

- **Utilité** : Utilisation de _ pour les variables non utilisées.

```java
// Avant (Java 17)
if (obj instanceof Point p) {
    // p pas utilisé mais doit être nommé
}

try {
    riskyOperation();
} catch (Exception e) {
    // e pas utilisé
    log("Une erreur est survenue");
}

// Après (Java 21)
if (obj instanceof Point _) {
    // Pattern matching sans variable inutile
}

try {
    riskyOperation();
} catch (Exception _) {
    log("Une erreur est survenue");
}

// Dans les lambdas
list.forEach(_ -> System.out.println("Item"));

// Dans les records patterns
record Pair(int first, int second) {}

if (obj instanceof Pair(int x, int _)) {
    // Seulement x est utilisé
    System.out.println(x);
}
```

- **Impact** : Code plus clair en montrant explicitement ce qui n'est pas utilisé, moins de warnings du compilateur.


## 9 - Unnamed Classes and Instance Main Methods (Preview - Java 21)

- **Utilité** : Simplification pour les débutants et les scripts simples.

```java
// Avant (Java 17)
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// Après (Java 21) - Unnamed class
void main() {
    System.out.println("Hello, World!");
}

// Avec arguments
void main(String[] args) {
    System.out.println("Arguments: " + String.join(", ", args));
}
```

- **Impact** : Réduction drastique du boilerplate pour scripts et apprentissage, plus accessible aux débutants.


## 10 - Generational ZGC (Java 21)

- **Utilité** : ZGC avec génération pour améliorer les performances.

```bash
# Avant (Java 17) - ZGC classique
java -XX:+UseZGC -jar application.jar

# Après (Java 21) - Generational ZGC (par défaut)
java -XX:+UseZGC -XX:+ZGenerational -jar application.jar
```

- **Impact** :
  - Réduction de la consommation mémoire
  - Amélioration du débit
  - Pauses toujours < 1ms


## 11 - Key Encapsulation Mechanism API (Java 21)

- **Utilité** : Support pour KEM (cryptographie post-quantique).

```java
// Génération de clés pour KEM
KeyPairGenerator kpg = KeyPairGenerator.getInstance("X25519");
KeyPair keyPair = kpg.generateKeyPair();

// Encapsulation
KEM kem = KEM.getInstance("DHKEM");
KEM.Encapsulator encapsulator = kem.newEncapsulator(keyPair.getPublic());
KEM.Encapsulated encapsulated = encapsulator.encapsulate();

byte[] sharedSecret = encapsulated.key();
byte[] encapsulation = encapsulated.encapsulation();

// Décapsulation
KEM.Decapsulator decapsulator = kem.newDecapsulator(keyPair.getPrivate());
SecretKey recoveredKey = decapsulator.decapsulate(encapsulation);
```

- **Impact** : Préparation pour la cryptographie post-quantique, API standardisée pour KEM.


## 12 - Améliorations des performances

### Foreign Function & Memory API (FFM - Java 21)

- **Utilité** : Interaction native avec le code C et gestion de mémoire off-heap sécurisée.

```java
// Avant (Java 17) - JNI complexe
// Nécessite code C et configuration

// Après (Java 21) - FFM API
Linker linker = Linker.nativeLinker();
SymbolLookup stdlib = linker.defaultLookup();

// Appel à strlen de la libc
MethodHandle strlen = linker.downcallHandle(
    stdlib.find("strlen").orElseThrow(),
    FunctionDescriptor.of(JAVA_LONG, ADDRESS)
);

try (Arena arena = Arena.ofConfined()) {
    MemorySegment str = arena.allocateUtf8String("Hello");
    long length = (long) strlen.invoke(str);
    System.out.println("Length: " + length);  // 5
}
```

### Gestion de mémoire native :

```java
// Allocation de mémoire off-heap
try (Arena arena = Arena.ofConfined()) {
    MemorySegment segment = arena.allocate(1024);
    
    // Écriture
    segment.set(ValueLayout.JAVA_INT, 0, 42);
    
    // Lecture
    int value = segment.get(ValueLayout.JAVA_INT, 0);
    
    // Libération automatique à la fin du try
}
```

- **Impact** :
  - Performance native sans JNI
  - Sécurité mémoire améliorée
  - Interopérabilité facilitée avec code natif


## 13 - Nouveaux collecteurs (Collectors)

```java
// Avant (Java 17)
Map<Boolean, List<String>> partitioned = list.stream()
    .collect(Collectors.partitioningBy(s -> s.length() > 5));

// Après (Java 21) - teeing collector amélioré
record Stats(long count, double average) {}

Stats stats = list.stream()
    .collect(Collectors.teeing(
        Collectors.counting(),
        Collectors.averagingInt(String::length),
        Stats::new
    ));
```

- **Impact** : Collecteurs plus expressifs pour analyses complexes.

