# Guide complet : Nouveautés Java 25 (LTS)

Ci-dessous les principales nouveautés introduites entre Java 21 et Java 25, avec des exemples pratiques pour votre migration.


## 1 - Scoped Values - Finalisé (JEP 487)

- **Utilité** : Alternative immuable et performante à ThreadLocal, optimisée pour les virtual threads.

```java
// Avant (Java 21) - ThreadLocal
private static final ThreadLocal<String> USER_ID = new ThreadLocal<>();

public void process() {
    USER_ID.set("user123");
    try {
        doWork();
    } finally {
        USER_ID.remove(); // Obligatoire pour éviter les fuites
    }
}

private void doWork() {
    String userId = USER_ID.get();
    // Traitement
}

// Après (Java 25) - Scoped Values
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

public void process() {
    ScopedValue.where(USER_ID, "user123")
               .run(() -> doWork());
    // Pas besoin de remove(), nettoyage automatique
}

private void doWork() {
    String userId = USER_ID.get();
    // Traitement
}
```

### Avec contexte multiple :

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
}
```
- **Impact** :
  - Performance : 10x plus rapide que ThreadLocal avec virtual threads
  - Sécurité : Immuable, impossible d'oublier le remove()
  - Mémoire : Pas de fuites mémoire possibles


## 2 - Module Import Declarations - Finalisé (JEP 511)

- **Utilité** : Importer tous les packages d'un module avec une seule déclaration.

```java
// Avant (Java 21)
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.stream.Collectors;
import java.util.stream.Stream;

// Après (Java 25)
import module java.base;

public class Example {
    public void demo() {
        List<String> list = new ArrayList<>();
        Map<String, Integer> map = new HashMap<>();
        Set<String> set = new HashSet<>();
        
        Stream<String> stream = list.stream();
        // Accès direct à tous les types exportés du module
    }
}
```

- **Impact** : Réduction drastique du nombre d'imports, particulièrement utile pour JShell et les fichiers compacts.


## 3 - Compact Source Files & Instance Main Methods - Finalisé (JEP 512)

- **Utilité** : Simplification radicale pour les débutants et les scripts simples.

```java
// Avant (Java 21) - Boilerplate complet
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// Après (Java 25) - Version ultra-simplifiée
void main() {
    println("Hello, World!"); // println disponible automatiquement
}

// Avec arguments
void main(String[] args) {
    println("Bonjour " + args[0]);
}
```

### Script avec imports implicites :

```java
// Fichier: WebFetch.java
void main(String[] args) throws Exception {
    var url = new URL(args[0]);
    var connection = url.openConnection();
    
    try (var in = new BufferedReader(
            new InputStreamReader(connection.getInputStream()))) {
        in.lines().forEach(System.out::println);
    }
}

// Exécution directe
// java WebFetch.java https://example.com

```
- **Impact** : Courbe d'apprentissage considérablement réduite, idéal pour l'enseignement et les scripts rapides.


## 4 - Flexible Constructor Bodies - Finalisé (JEP 513)

- **Utilité** : Permet d'exécuter du code avant l'appel au constructeur parent ou super().

```java
// Avant (Java 21) - super() doit être en premier
public class Rectangle extends Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        super(validateDimensions(width, height)); // Impossible!
        this.width = width;
        this.height = height;
    }
    
    // Solution avec méthode statique
    private static String validateDimensions(double w, double h) {
        if (w <= 0 || h <= 0) {
            throw new IllegalArgumentException("Invalid dimensions");
        }
        return "Rectangle";
    }
}

// Après (Java 25) - Code avant super()
public class Rectangle extends Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        // Validation avant super()
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Invalid dimensions");
        }
        
        // Calculs avant super()
        double area = width * height;
        
        super("Rectangle: " + area);
        
        this.width = width;
        this.height = height;
    }
}
```

### Exemple avec transformation de données :

```java
public class Person {
    private final String normalizedName;
    private final int validatedAge;
    
    public Person(String name, int age) {
        // Normalisation et validation avant super()
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name required");
        }
        
        String normalized = name.trim().toUpperCase();
        
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age");
        }
        
        super(); // Object constructor
        
        this.normalizedName = normalized;
        this.validatedAge = age;
    }
}
```

- **Impact** : Code plus naturel, moins de méthodes statiques auxiliaires, meilleure lisibilité.



## 5 - Primitive Types in Patterns - 3ème Preview (JEP 507)

- **Utilité** : Pattern matching avec types primitifs, réduisant le besoin de boxing.

```java
// Avant (Java 21)
Object obj = 42;

if (obj instanceof Integer) {
    Integer i = (Integer) obj;
    int value = i.intValue(); // Unboxing
    System.out.println(value);
}

// Après (Java 25)
Object obj = 42;

if (obj instanceof int i) {
    System.out.println(i); // Directement int
}

// Avec switch
String describe(Object obj) {
    return switch (obj) {
        case int i when i < 0    -> "Négatif: " + i;
        case int i when i == 0   -> "Zéro";
        case int i               -> "Positif: " + i;
        case long l              -> "Long: " + l;
        case double d            -> "Double: " + d;
        case String s            -> "String: " + s;
        default                  -> "Autre type";
    };
}
```

### Conversion avec range checking :

```java
// Conversion sécurisée avec vérification
void processNumber(Object obj) {
    switch (obj) {
        case byte b   -> println("Byte: " + b);
        case short s  -> println("Short: " + s);
        case int i    -> println("Int: " + i);
        case long l   -> println("Long: " + l);
        case float f  -> println("Float: " + f);
        case double d -> println("Double: " + d);
        default       -> println("Pas un nombre");
    }
}
```

- **Impact** : Élimination du boxing/unboxing, code plus performant et lisible.



## 6 - Structured Concurrency - 5ème Preview (JEP 505)

- **Utilité** : Gestion structurée de tâches concurrentes avec cancellation automatique.

```java
// Avant (Java 21)
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
Future<String> user = executor.submit(() -> fetchUser());
Future<String> orders = executor.submit(() -> fetchOrders());

try {
    String userData = user.get();
    String orderData = orders.get();
    return combine(userData, orderData);
} catch (Exception e) {
    user.cancel(true);
    orders.cancel(true);
    throw e;
} finally {
    executor.shutdown();
}

// Après (Java 25) - Améliorations dans la 5ème preview
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> user = scope.fork(() -> fetchUser());
    Subtask<String> orders = scope.fork(() -> fetchOrders());
    
    scope.join()
         .throwIfFailed();
    
    return combine(user.get(), orders.get());
}
// Annulation automatique en cas d'erreur ou d'exception
```

### Pattern "first to complete" :

```java
String fetchFromFastestServer(List<String> servers) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
        for (String server : servers) {
            scope.fork(() -> fetchFrom(server));
        }
        
        scope.join();
        
        return scope.result(); // Premier résultat disponible
    }
    // Toutes les autres tâches sont annulées automatiquement
}
```

- **Impact** : Gestion des erreurs simplifiée, pas de fuite de threads, cancellation automatique.



## 7 - Stable Values - Preview (JEP 502)

- **Utilité** : Alternative à final permettant l'initialisation différée tout en restant immuable.

```java
// Avant (Java 21) - Initialisation obligatoire
public class Service {
    private final Logger logger = createLogger(); // Créé au chargement
    
    private Logger createLogger() {
        // Initialisation coûteuse au chargement de la classe
        return LoggerFactory.getLogger(Service.class);
    }
}

// Après (Java 25) - Stable Values
public class Service {
    private final StableValue<Logger> logger = StableValue.of();
    
    private Logger getLogger() {
        return logger.orElseSet(() -> {
            // Initialisation différée, une seule fois
            return LoggerFactory.getLogger(Service.class);
        });
    }
    
    public void process() {
        getLogger().info("Processing...");
    }
}
```

### Avec dépendances complexes :

```java
public class Application {
    private final StableValue<Database> database = StableValue.of();
    private final StableValue<Cache> cache = StableValue.of();
    
    Database getDatabase() {
        return database.orElseSet(() -> new Database(config));
    }
    
    Cache getCache() {
        return cache.orElseSet(() -> new Cache(getDatabase()));
    }
    
    public void init() {
        // L'initialisation se fait à la demande
        // Une seule initialisation même en concurrent
    }
}
```

- **Impact** :
  - Performance : Initialisation différée sans synchronisation
  - Sécurité : Immuabilité garantie après initialisation
  - Optimisation JVM : Traité comme constante par le JIT



## 8 - Key Derivation Function API - Finalisé (JEP 510)

- **Utilité** : API standardisée pour dériver des clés cryptographiques à partir d'une clé secrète.

```java
// Avant (Java 21) - Implémentation manuelle complexe
public byte[] deriveKey(String password, byte[] salt) throws Exception {
    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 256);
    SecretKey tmp = factory.generateSecret(spec);
    return tmp.getEncoded();
}

// Après (Java 25) - KDF API
import javax.crypto.KDF;
import javax.crypto.SecretKey;

public SecretKey deriveKey(SecretKey masterKey, byte[] info) throws Exception {
    KDF kdf = KDF.getInstance("HKDF-SHA256");
    
    KDFParameters params = HKDFParametersSpec.ofExtract()
        .addIKM(masterKey)
        .addSalt(salt)
        .extractOnly();
    
    SecretKey prk = kdf.deriveKey("AES", params);
    
    params = HKDFParametersSpec.ofExpand()
        .addInfo(info)
        .thenExpand(32); // 256 bits
    
    return kdf.deriveKey("AES", params);
}
```

### Dérivation de multiples clés :

```java
// Dériver plusieurs clés pour différents usages
public class KeyDerivation {
    public Map<String, SecretKey> deriveKeys(SecretKey masterKey) 
            throws Exception {
        KDF kdf = KDF.getInstance("HKDF-SHA256");
        
        return Map.of(
            "encryption", deriveForPurpose(kdf, masterKey, "encryption"),
            "authentication", deriveForPurpose(kdf, masterKey, "auth"),
            "signing", deriveForPurpose(kdf, masterKey, "signing")
        );
    }
    
    private SecretKey deriveForPurpose(KDF kdf, SecretKey master, 
                                       String purpose) throws Exception {
        KDFParameters params = HKDFParametersSpec.ofExtract()
            .addIKM(master)
            .addInfo(purpose.getBytes())
            .thenExpand(32);
        
        return kdf.deriveKey("AES", params);
    }
}
```

- **Impact** : API standardisée pour la cryptographie moderne, meilleure sécurité, support post-quantique.



## 9 - Compact Object Headers - Finalisé (JEP 519)

- **Utilité** : Réduction de la taille des headers d'objets de 128 bits à 64 bits.

```bash
# Avant (Java 21) - Headers par défaut
# Object header: 96-128 bits par objet

# Après (Java 25) - Activation des compact headers
java -XX:+UseCompactObjectHeaders -jar application.jar
```

### Impact mesurable :

```java
// Exemple: Million d'objets
class SmallObject {
    private int value;
}

// Avant: ~24 bytes par objet (header + int + padding)
// Après:  ~16 bytes par objet (compact header + int)
// 
// Pour 1 million d'objets: économie de ~8 MB
// Pour 100 millions: économie de ~800 MB
```

- **Impact** :
  - Mémoire : Réduction de 30-40% de l'utilisation heap
  - Performance : Amélioration du cache CPU (data locality)
  - Scalabilité : Permet plus d'objets dans la même heap



## 10 - Generational Shenandoah - Finalisé (JEP 521)

- **Utilité** : Shenandoah GC avec support générationnel pour meilleures performances.

```bash
# Avant (Java 21) - Shenandoah classique
java -XX:+UseShenandoahGC -jar application.jar

# Après (Java 25) - Generational Shenandoah
java -XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational -jar application.jar
```

### Comparaison des modes :

```bash
# Mode classique (tous les objets)
-XX:+UseShenandoahGC

# Mode générationnel (jeune/vieux séparés)
-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational

# Mode IU (legacy)
-XX:+UseShenandoahGC -XX:ShenandoahGCMode=iu
```

- **Impact** :
  - Throughput : Amélioration de 20-30%
  - Latence : Pauses toujours < 10ms
  - Mémoire : Meilleure utilisation heap avec séparation générationnelle



## 11 - Ahead-of-Time (AOT) Enhancements

### AOT Command-Line Ergonomics - Finalisé (JEP 514)

- **Utilité** : Simplification de la création de caches AOT pour startup plus rapide.

```bash
# Avant (Java 21) - Process en 2 étapes
# 1. Enregistrer le profil
java -XX:CDS=record -jar app.jar
# 2. Créer l'archive
java -Xshare:dump -jar app.jar

# Après (Java 25) - Une seule étape
# Création automatique du cache AOT à la fermeture
java -XX:AOTCacheOutput=app.aot -jar app.jar

# Utilisation du cache
java -XX:AOTCache=app.aot -jar app.jar
```

- **Impact** : Réduction du temps de démarrage de 50-70%, idéal pour microservices et serverless.

### AOT Method Profiling - Finalisé (JEP 515)

- **Utilité** : Profilage des méthodes pour optimisation AOT du JIT.

```bash
# Création du cache avec profil de méthodes
java -XX:AOTCacheOutput=app.aot \
     -XX:AOTMethodProfiling=on \
     -jar app.jar

# Les méthodes chaudes sont pré-compilées
# Warm-up instantané au prochain démarrage
java -XX:AOTCache=app.aot -jar app.jar
```

- **Impact** :
  - Warm-up 5-10x plus rapide
  - Performance optimale dès le démarrage
  - Idéal pour applications cloud avec cold starts fréquents



## 12 - JFR Enhancements

### JFR Cooperative Sampling - Finalisé (JEP 518)

- **Utilité** : Amélioration de la stabilité du sampling de threads.

```bash
# Activation automatique, plus stable avec ZGC
java -XX:StartFlightRecording=filename=recording.jfr \
     -jar application.jar
```

- **Impact** : Profilage plus stable, moins d'impact sur les performances, meilleur support ZGC.

### JFR Method Timing & Tracing - Finalisé (JEP 520)

- **Utilité** : Timing et tracing précis des méthodes sans modification de code.

```bash
# Timer toutes les invocations d'une méthode
java -XX:StartFlightRecording:jdk.MethodTiming#enabled=true \
     -XX:FlightRecorderOptions:method-filter="com.example.*" \
     -jar app.jar

# Analyser avec jfr
jfr print --events jdk.MethodTiming recording.jfr
```

### Configuration programmatique :

```java
import jdk.jfr.FlightRecorder;
import jdk.jfr.Recording;

Recording recording = new Recording();
recording.enable("jdk.MethodTiming")
         .with("filter", "com.example.Service.*");
recording.start();

// Application s'exécute...

recording.stop();
recording.dump(Path.of("timing.jfr"));
```

- **Impact** : Identification précise des bottlenecks sans instrumentation manuelle.

### JFR CPU-Time Profiling - Expérimental (JEP 509)

- **Utilité** : Profilage CPU précis incluant le temps en code natif (Linux uniquement).

```bash
# Activation du profilage CPU-time
java -XX:StartFlightRecording:jdk.CPUTimeSample#enabled=true \
     -XX:+UnlockExperimentalVMOptions \
     filename=cpu-profile.jfr \
     -jar app.jar

# Analyse
jfr view cpu-time-hot-methods cpu-profile.jfr
```

- **Impact** : Mesure précise incluant le temps en JNI/FFM, meilleure identification des hotspots.



## 13 - PEM Encodings of Cryptographic Objects - Preview (JEP 470)

- **Utilité** : API concise pour encoder/décoder des objets cryptographiques en format PEM.

```java
// Avant (Java 21) - Manipulation manuelle complexe
import java.util.Base64;

public String keyToPEM(PublicKey key) {
    String encoded = Base64.getEncoder()
        .encodeToString(key.getEncoded());
    
    StringBuilder pem = new StringBuilder();
    pem.append("-----BEGIN PUBLIC KEY-----\n");
    // Ajouter des sauts de ligne tous les 64 caractères
    for (int i = 0; i < encoded.length(); i += 64) {
        pem.append(encoded, i, Math.min(i + 64, encoded.length()));
        pem.append("\n");
    }
    pem.append("-----END PUBLIC KEY-----\n");
    return pem.toString();
}

// Après (Java 25) - PEM API
import java.security.cert.PEM;

public String keyToPEM(PublicKey key) {
    return PEM.encode(key);
}

// Décodage
public PublicKey pemToKey(String pemString) throws Exception {
    return PEM.decode(pemString, PublicKey.class);
}
```

### Avec certificats :

```java
// Encoder un certificat
X509Certificate cert = ...;
String pemCert = PEM.encode(cert);

// Encoder plusieurs objets
String pemBundle = PEM.encode(cert, privateKey);

// Décoder
List<Object> objects = PEM.decodeAll(pemBundle);
```

- **Impact** : API simple et sûre pour manipulation PEM, moins d'erreurs de formatage.



## 14 - Vector API - 10ème Incubation (JEP 517)

- **Utilité** : SIMD explicite pour calculs vectoriels haute performance.

```java
// Avant (Java 21) - Boucle scalaire
void multiplyArrays(float[] a, float[] b, float[] result) {
    for (int i = 0; i < a.length; i++) {
        result[i] = a[i] * b[i];
    }
}

// Après (Java 25) - Vector API
import jdk.incubator.vector.*;

void multiplyArrays(float[] a, float[] b, float[] result) {
    var species = FloatVector.SPECIES_PREFERRED;
    int i = 0;
    int upperBound = species.loopBound(a.length);
    
    for (; i < upperBound; i += species.length()) {
        var va = FloatVector.fromArray(species, a, i);
        var vb = FloatVector.fromArray(species, b, i);
        var vc = va.mul(vb);
        vc.intoArray(result, i);
    }
    
    // Reste scalaire
    for (; i < a.length; i++) {
        result[i] = a[i] * b[i];
    }
}
```

### Opérations complexes :

```java
// Calcul de distance euclidienne vectorisée
double euclideanDistance(float[] a, float[] b) {
    var species = FloatVector.SPECIES_PREFERRED;
    var sumVec = FloatVector.zero(species);
    
    int i = 0;
    for (; i < species.loopBound(a.length); i += species.length()) {
        var va = FloatVector.fromArray(species, a, i);
        var vb = FloatVector.fromArray(species, b, i);
        var diff = va.sub(vb);
        sumVec = diff.fma(diff, sumVec); // diff * diff + sumVec
    }
    
    double sum = sumVec.reduceLanes(VectorOperators.ADD);
    
    // Reste scalaire
    for (; i < a.length; i++) {
        double diff = a[i] - b[i];
        sum += diff * diff;
    }
    
    return Math.sqrt(sum);
}
```

- **Impact** :
  - Performance : 4-8x plus rapide sur CPU supportant SIMD
  - Portabilité : Code portable, optimisé automatiquement pour l'architecture

