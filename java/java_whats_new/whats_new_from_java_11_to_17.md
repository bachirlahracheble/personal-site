# Guide complet : Nouveautés Java 17 (LTS)

Ci-dessous les principales nouveautés introduites entre Java 11 et Java 17, avec des exemples.

## 1 - Records (Java 14-16, finalisé en Java 16)

- **Utilité** : Classes immuables compactes pour transporter des données, avec `equals()`, `hashCode()`, `toString()` générés automatiquement.

```java
// Avant (Java 11)
public final class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String name() { return name; }
    public int age() { return age; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "Person[name=" + name + ", age=" + age + "]";
    }
}

// Après (Java 17)
public record Person(String name, int age) {}
- **Impact** : Réduction drastique du boilerplate (30+ lignes → 1 ligne), immutabilité garantie, code plus maintenable.
Records avec validations
javapublic record Person(String name, int age) {
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age ne peut pas être négatif");
        }
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Le nom est requis");
        }
    }
}
```

## 2 - Sealed Classes (Java 17)
- **Utilité** : Contrôle explicite des classes qui peuvent étendre/implémenter une classe/interface.

```java
// Avant (Java 11) - impossible de restreindre
public abstract class Shape {
    public abstract double area();
}
// N'importe qui peut étendre Shape

// Après (Java 17)
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    public abstract double area();
}

public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) { this.radius = radius; }
    
    @Override
    public double area() { return Math.PI * radius * radius; }
}

public final class Rectangle extends Shape {
    private final double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() { return width * height; }
}

public non-sealed class Triangle extends Shape {
    // non-sealed permet l'extension future
    @Override
    public double area() { return 0; }
}
```

- **Impact** : Hiérarchies de types plus sûres, meilleure exhaustivité dans le pattern matching, documentation explicite des sous-types possibles.


## 3 - Pattern Matching for instanceof (Java 16)
- **Utilité** : Élimine le cast explicite après un `instanceof`.

```java
// Avant (Java 11)
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println(str.length());
}

// Après (Java 17)
if (obj instanceof String str) {
    System.out.println(str.length());
}
Exemple plus complexe :

```java
// Avant (Java 11)
public String format(Object obj) {
    if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return String.format("int %d", i);
    } else if (obj instanceof Long) {
        Long l = (Long) obj;
        return String.format("long %d", l);
    } else if (obj instanceof Double) {
        Double d = (Double) obj;
        return String.format("double %f", d);
    } else {
        return obj.toString();
    }
}

// Après (Java 17)
public String format(Object obj) {
    if (obj instanceof Integer i) {
        return String.format("int %d", i);
    } else if (obj instanceof Long l) {
        return String.format("long %d", l);
    } else if (obj instanceof Double d) {
        return String.format("double %f", d);
    } else {
        return obj.toString();
    }
}
```

- **Impact** : Code plus concis, moins d'erreurs de cast, meilleure lisibilité.


## 4 - `Switch` Expressions (Java 14)
- **Utilité** : `Switch` comme expression avec syntaxe moderne et exhaustivité vérifiée.

```java
// Avant (Java 11) - switch statement
String result;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        result = "6 lettres";
        break;
    case TUESDAY:
        result = "7 lettres";
        break;
    case THURSDAY:
    case SATURDAY:
        result = "8 lettres";
        break;
    case WEDNESDAY:
        result = "9 lettres";
        break;
    default:
        throw new IllegalStateException("Jour invalide: " + day);
}

// Après (Java 17) - switch expression
String result = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> "6 lettres";
    case TUESDAY -> "7 lettres";
    case THURSDAY, SATURDAY -> "8 lettres";
    case WEDNESDAY -> "9 lettres";
};
Avec blocs de code :
javaint numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    case THURSDAY, SATURDAY -> 8;
    case WEDNESDAY -> 9;
};

// Avec yield pour blocs complexes
String result = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> {
        System.out.println("Court");
        yield "6 lettres";
    }
    case TUESDAY -> {
        System.out.println("Moyen");
        yield "7 lettres";
    }
    default -> "Autre";
};
```

- **Impact** : Pas de `break` oubliés, valeur de retour directe, exhaustivité vérifiée à la compilation, code plus fonctionnel.


## 5 - Text Blocks (Java 15)
- **Utilité** : Chaînes multiligne lisibles sans échappement excessif.

```java
// Avant (Java 11)
String json = "{\n" +
              "  \"name\": \"John\",\n" +
              "  \"age\": 30,\n" +
              "  \"city\": \"New York\"\n" +
              "}";

String html = "<html>\n" +
              "  <body>\n" +
              "    <p>Hello, World!</p>\n" +
              "  </body>\n" +
              "</html>";

String sql = "SELECT id, name, email\n" +
             "FROM users\n" +
             "WHERE status = 'ACTIVE'\n" +
             "ORDER BY name";

// Après (Java 17)
String json = """
    {
      "name": "John",
      "age": 30,
      "city": "New York"
    }
    """;

String html = """
    <html>
      <body>
        <p>Hello, World!</p>
      </body>
    </html>
    """;

String sql = """
    SELECT id, name, email
    FROM users
    WHERE status = 'ACTIVE'
    ORDER BY name
    """;
Avec interpolation :
javaString name = "Alice";
int age = 25;

String message = """
    Bonjour %s,
    Vous avez %d ans.
    """.formatted(name, age);
```

- **Impact** : Lisibilité grandement améliorée pour `JSON`, `SQL`, `HTML`, `XML`. Moins d'erreurs d'échappement.


## 6 - Helpful NullPointerExceptions (Java 14)
- **Utilité** : Messages d'erreur détaillés indiquant exactement quelle variable est `null`.

```java
// Avant (Java 11)
person.getAddress().getCity().toLowerCase();
// Exception: NullPointerException
// Laquelle est null ? person, address, ou city ?

// Après (Java 17)
// Exception: NullPointerException: 
// Cannot invoke "String.toLowerCase()" because the return value of 
// "Address.getCity()" is null
```

- **Impact** : Débogage beaucoup plus rapide, identification immédiate de la source du problème.


## 7 - Améliorations des Streams
### `Stream.toList()` (Java 16)
- **Utilité** : Raccourci pour collecter un stream en `List` immuable.

```java
// Avant (Java 11)
List<String> list = stream
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// Après (Java 17)
List<String> list = stream
    .filter(s -> s.length() > 3)
    .toList();
```

- **Impact** : Code plus concis, list immuable par défaut (plus sûr).

### `Stream.mapMulti()` (Java 16)
- **Utilité** : Alternative plus performante à `flatMap` pour certains cas.

```java
// Avant (Java 11) - avec flatMap
List<Integer> result = list.stream()
    .flatMap(item -> {
        List<Integer> subList = new ArrayList<>();
        if (item > 0) subList.add(item);
        if (item > 10) subList.add(item * 2);
        return subList.stream();
    })
    .toList();

// Après (Java 17) - avec mapMulti
List<Integer> result = list.stream()
    .<Integer>mapMulti((item, consumer) -> {
        if (item > 0) consumer.accept(item);
        if (item > 10) consumer.accept(item * 2);
    })
    .toList();
```

- **Impact** : Performance améliorée (évite la création de streams intermédiaires), moins d'allocations mémoire.


## 8 - Amélioration des Random Generators (Java 17)
- **Utilité** : Nouveaux algorithmes de génération aléatoire plus performants et API unifiée.

```java
// Avant (Java 11)
Random random = new Random();
int value = random.nextInt(100);
SecureRandom secureRandom = new SecureRandom();

// Après (Java 17)
RandomGenerator random = RandomGenerator.of("L64X128MixRandom");
int value = random.nextInt(100);

// Pour des streams
random.ints(10, 0, 100)
    .forEach(System.out::println);

// Différents algorithmes disponibles
RandomGenerator xoroshiro = RandomGenerator.of("Xoroshiro128PlusPlus");
RandomGenerator l64x128 = RandomGenerator.of("L64X128MixRandom");
```

- **Impact** : Meilleure performance, plus de choix d'algorithmes selon les besoins, API plus cohérente.


## 9 - Enhanced Pseudo-Random Number Generators
- **Utilité** : Nouveaux générateurs pour différents cas d'usage.

```java
// Pour des streams parallèles
RandomGeneratorFactory<RandomGenerator> factory = 
    RandomGeneratorFactory.of("L128X256MixRandom");

List<Integer> randomNumbers = IntStream.range(0, 1000)
    .parallel()
    .mapToObj(i -> factory.create(i).nextInt(100))
    .toList();
```

- **Impact** : Meilleure performance en parallèle, générateurs adaptés à chaque contexte.


## 10 - ZGC et G1GC Améliorations
- **Utilité** : Améliorations significatives des garbage collectors.

```bash
# ZGC (production-ready en Java 17)
java -XX:+UseZGC -jar application.jar

# G1GC avec retour dynamique de mémoire
java -XX:+UseG1GC -XX:G1PeriodicGCInterval=30000 -jar application.jar
- **Impact** :

ZGC : Pauses < 1ms même pour heaps très larges
G1GC : Retour automatique de mémoire au système d'exploitation
```

## 11 - macOS/AArch64 Support (Java 17)
- **Utilité** : Support natif pour les Mac Apple Silicon (M1/M2/M3).
- **Impact** : Performance native sur architecture ARM, pas besoin de Rosetta.


## 12 - Strong Encapsulation of JDK Internals
- **Utilité** : Encapsulation stricte des APIs internes du JDK.

```java
// Ne fonctionne plus en Java 17
// sun.misc.Unsafe unsafe = sun.misc.Unsafe.getUnsafe();

// Solution : utiliser les APIs publiques
// java.lang.invoke.VarHandle
// java.util.concurrent.atomic
```

- **Impact** : Code plus robuste et maintenable, migration nécessaire si vous utilisiez des APIs internes.


## 13 - Context-Specific Deserialization Filters (Java 17)
- **Utilité** : Sécurité renforcée contre les attaques de désérialisation.

```java
// Configuration d'un filtre de désérialisation
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "java.base/*;!*"
);

ObjectInputStream ois = new ObjectInputStream(inputStream);
ois.setObjectInputFilter(filter);
```

- **Impact** : Protection contre les vulnérabilités de désérialisation, meilleure sécurité.


## 14 - Deprecation et Suppression
APIs supprimées ou deprecated :

RMI Activation (JEP 407)
Security Manager (deprecated for removal)
Applet API (supprimée)

Action requise : Vérifier l'utilisation de ces APIs et migrer vers des alternatives modernes.

## 15 - Nouveaux Méthodes Utilitaires
`Objects.checkIndex()`, `checkFromToIndex()`, `checkFromIndexSize()`

```java
// Avant (Java 11)
if (index < 0 || index >= list.size()) {
    throw new IndexOutOfBoundsException("Index: " + index);
}

// Après (Java 17)
Objects.checkIndex(index, list.size());
Objects.checkFromToIndex(fromIndex, toIndex, list.size());
```

- **Impact** : Validation d'index standardisée, messages d'erreur cohérents.


## 16 - Day Period Support (Java 17)
- **Utilité** : Support des périodes de la journée (matin, après-midi, soir) dans le formatage de dates.

```java
javaDateTimeFormatter formatter = DateTimeFormatter.ofPattern(
    "B", Locale.FRENCH
);
String period = LocalTime.of(14, 0).format(formatter); // "après-midi"
```

- **Impact** : Meilleur support de l'internationalisation pour les heures.

