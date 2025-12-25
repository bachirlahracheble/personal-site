# Guide complet : Nouveautés Java 8 (LTS)

Ci-dessous les principales nouveautés introduites entre Java 7 et Java 8, avec des exemples pratiques pour votre migration.

## 1 - Expressions Lambda

- **Utilité** : Syntaxe concise pour implémenter des interfaces fonctionnelles (interfaces avec une seule méthode abstraite).

```java
// Avant (Java 7) - Classe anonyme
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

// Après (Java 8) - Lambda
Collections.sort(names, (a, b) -> a.compareTo(b));

// Encore plus court avec method reference
Collections.sort(names, String::compareTo);
```

### Différentes syntaxes lambda :

```java
// Sans paramètre
Runnable r = () -> System.out.println("Hello");

// Un paramètre (parenthèses optionnelles)
Consumer<String> printer = s -> System.out.println(s);
Consumer<String> printer2 = (s) -> System.out.println(s);

// Plusieurs paramètres
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Avec bloc de code
BiFunction<Integer, Integer, Integer> calculate = (a, b) -> {
    int sum = a + b;
    System.out.println("Somme: " + sum);
    return sum;
};

// Avec types explicites
BiFunction<Integer, Integer, Integer> multiply = 
    (Integer a, Integer b) -> a * b;
```

- **Impact** : Code beaucoup plus concis et lisible, encourage la programmation fonctionnelle.



## 2 - Stream API

- **Utilité** : Traitement déclaratif des collections avec opérations fonctionnelles (`filter`, `map`, `reduce`, etc.).

```java
// Avant (Java 7) - Style impératif
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> filtered = new ArrayList<>();
for (String name : names) {
    if (name.length() > 3) {
        filtered.add(name.toUpperCase());
    }
}
Collections.sort(filtered);

// Après (Java 8) - Style déclaratif avec Streams
List<String> filtered = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

### Opérations courantes :

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Map
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// Reduce
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// Count
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();

// Any/All/None Match
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);

boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);

boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);

// Find
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();

Optional<Integer> any = numbers.stream()
    .filter(n -> n > 5)
    .findAny();
```

### Opérations avancées :

```java
// Distinct
List<Integer> unique = Arrays.asList(1, 2, 2, 3, 3, 3)
    .stream()
    .distinct()
    .collect(Collectors.toList());

// Limit et Skip
List<Integer> limited = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());

List<Integer> skipped = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());

// FlatMap - Aplatir une structure
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);

List<Integer> flat = nested.stream()
    .flatMap(list -> list.stream())
    .collect(Collectors.toList());

// Grouping
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));

// Partitioning
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// Joining
String joined = names.stream()
    .collect(Collectors.joining(", "));

// Statistics
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
System.out.println("Max: " + stats.getMax());
System.out.println("Min: " + stats.getMin());
System.out.println("Average: " + stats.getAverage());
```

### Streams parallèles :

```java
// Avant (Java 7)
List<Integer> results = new ArrayList<>();
for (Integer number : largeList) {
    results.add(expensiveComputation(number));
}

// Après (Java 8) - Parallèle automatique
List<Integer> results = largeList.parallelStream()
    .map(this::expensiveComputation)
    .collect(Collectors.toList());
```

- **Impact** :
   - Lisibilité : Code déclaratif plus facile à comprendre
    - Performance : Parallélisation simple avec parallelStream()
    - Maintenabilité : Moins de boucles explicites, moins d'erreurs



## 3 - Optional

- **Utilité** : Conteneur pour valeurs potentiellement nulles, évite les `NullPointerException`.

```java
// Avant (Java 7) - Gestion manuelle de null
public String getUserCity(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            City city = address.getCity();
            if (city != null) {
                return city.getName();
            }
        }
    }
    return "Unknown";
}

// Après (Java 8) - Optional
public String getUserCity(User user) {
    return Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .map(City::getName)
        .orElse("Unknown");
}
```

### Méthodes courantes d'Optional :

```java
// Création
Optional<String> empty = Optional.empty();
Optional<String> value = Optional.of("Hello"); // Lance exception si null
Optional<String> nullable = Optional.ofNullable(getValue()); // Accepte null

// Vérification
if (optional.isPresent()) {
    System.out.println(optional.get());
}

// ifPresent avec lambda
optional.ifPresent(val -> System.out.println(val));
optional.ifPresent(System.out::println);

// orElse - Valeur par défaut
String result = optional.orElse("default");

// orElseGet - Supplier pour valeur par défaut
String result2 = optional.orElseGet(() -> computeDefault());

// orElseThrow - Lance exception si absent
String result3 = optional.orElseThrow(() -> 
    new IllegalStateException("Value not found"));

// filter
Optional<String> filtered = optional.filter(s -> s.length() > 5);

// map
Optional<Integer> length = optional.map(String::length);

// flatMap - Pour éviter Optional<Optional<T>>
Optional<String> result4 = optional.flatMap(s -> findRelated(s));
```

### Exemple pratique :

```java
// Avant (Java 7)
public User findUser(String id) {
    User user = database.getUser(id);
    if (user != null && user.isActive()) {
        return user;
    }
    return null;
}

public void processUser(String id) {
    User user = findUser(id);
    if (user != null) {
        String email = user.getEmail();
        if (email != null) {
            sendEmail(email);
        }
    }
}

// Après (Java 8)
public Optional<User> findUser(String id) {
    return Optional.ofNullable(database.getUser(id))
        .filter(User::isActive);
}

public void processUser(String id) {
    findUser(id)
        .map(User::getEmail)
        .ifPresent(this::sendEmail);
}
```

- **Impact** : Gestion explicite de l'absence de valeur, code plus sûr et expressif.



## 4 - Interfaces par défaut (Default Methods)
- **Utilité** : Permet d'ajouter des méthodes à des interfaces sans casser la compatibilité.

```java
// Avant (Java 7) - Impossible d'ajouter méthodes sans casser les implémentations
public interface Vehicle {
    void start();
    void stop();
}

// Après (Java 8) - Méthodes par défaut
public interface Vehicle {
    void start();
    void stop();
    
    // Méthode par défaut
    default void honk() {
        System.out.println("Beep beep!");
    }
    
    // Autre méthode par défaut
    default String getType() {
        return "Generic Vehicle";
    }
}

// Les implémentations existantes continuent de fonctionner
public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopping");
    }
    
    // honk() et getType() héritées automatiquement
}

// Possibilité de surcharger
public class Truck implements Vehicle {
    @Override
    public void start() {
        System.out.println("Truck starting");
    }
    
    @Override
    public void stop() {
        System.out.println("Truck stopping");
    }
    
    @Override
    public void honk() {
        System.out.println("HOOOONK!"); // Klaxon de camion
    }
}
```

### Méthodes statiques dans les interfaces :

```java
public interface MathUtils {
    // Méthode statique
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
}

// Utilisation
int sum = MathUtils.add(5, 3);
```

- **Impact** : Évolution des API sans casser la compatibilité, factorisation de code dans les interfaces.



## 5 - Method References
- **Utilité** : Raccourci syntaxique pour les lambdas qui appellent une méthode existante.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Lambda
names.forEach(name -> System.out.println(name));

// Method reference
names.forEach(System.out::println);
```

### Types de method references :

```java
// 1. Référence à une méthode statique
// ClassName::staticMethodName
Function<String, Integer> parser = Integer::parseInt;
int number = parser.apply("123");

// 2. Référence à une méthode d'instance d'un objet particulier
// instance::instanceMethodName
String prefix = "Hello ";
Function<String, String> greeter = prefix::concat;
String greeting = greeter.apply("World");

// 3. Référence à une méthode d'instance d'un type arbitraire
// ClassName::instanceMethodName
Function<String, String> upperCaser = String::toUpperCase;
String upper = upperCaser.apply("hello");

// Avec streams
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// 4. Référence à un constructeur
// ClassName::new
Supplier<ArrayList<String>> listSupplier = ArrayList::new;
ArrayList<String> list = listSupplier.get();

Function<Integer, ArrayList<String>> listCreator = ArrayList::new;
ArrayList<String> listWithCapacity = listCreator.apply(10);
```

### Exemples pratiques :

```java
// Avant (Java 7)
List<User> users = getUsers();
Collections.sort(users, new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) {
        return u1.getName().compareTo(u2.getName());
    }
});

// Après (Java 8) - Avec lambda
users.sort((u1, u2) -> u1.getName().compareTo(u2.getName()));

// Après (Java 8) - Avec method reference
users.sort(Comparator.comparing(User::getName));
```

- **Impact** : Code encore plus concis que les lambdas pour les cas simples.



## 6 - Date and Time API (`java.time`)
- **Utilité** : API moderne et thread-safe pour manipuler dates et heures, remplace `Date` et `Calendar`.

```java
// Avant (Java 7) - Date et Calendar complexes
Date now = new Date();
Calendar cal = Calendar.getInstance();
cal.setTime(now);
cal.add(Calendar.DAY_OF_MONTH, 5);
Date futureDate = cal.getTime();

SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
String formatted = sdf.format(now);

// Après (Java 8) - API moderne
LocalDate today = LocalDate.now();
LocalDate futureDate = today.plusDays(5);
String formatted = today.format(DateTimeFormatter.ISO_LOCAL_DATE);
```

### Types principaux :

```java
// LocalDate - Date sans heure
LocalDate date = LocalDate.now();
LocalDate birthday = LocalDate.of(1990, 5, 15);
LocalDate parsed = LocalDate.parse("2024-01-15");

// LocalTime - Heure sans date
LocalTime time = LocalTime.now();
LocalTime lunch = LocalTime.of(12, 30);
LocalTime parsed2 = LocalTime.parse("14:30:00");

// LocalDateTime - Date + Heure
LocalDateTime dateTime = LocalDateTime.now();
LocalDateTime meeting = LocalDateTime.of(2024, 1, 15, 14, 30);
LocalDateTime parsed3 = LocalDateTime.parse("2024-01-15T14:30:00");

// ZonedDateTime - Date + Heure + Fuseau
ZonedDateTime zoned = ZonedDateTime.now();
ZonedDateTime paris = ZonedDateTime.now(ZoneId.of("Europe/Paris"));
ZonedDateTime tokyo = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));

// Instant - Point dans le temps (timestamp)
Instant instant = Instant.now();
Instant epoch = Instant.ofEpochMilli(System.currentTimeMillis());

// Duration - Durée entre deux instants
Duration duration = Duration.between(time, lunch);
Duration twoHours = Duration.ofHours(2);

// Period - Période entre deux dates
Period period = Period.between(birthday, today);
Period oneYear = Period.ofYears(1);
```

### Opérations courantes :

```java
LocalDate today = LocalDate.now();

// Addition/Soustraction
LocalDate tomorrow = today.plusDays(1);
LocalDate nextWeek = today.plusWeeks(1);
LocalDate nextMonth = today.plusMonths(1);
LocalDate nextYear = today.plusYears(1);
LocalDate yesterday = today.minusDays(1);

// Extraction de composants
int year = today.getYear();
Month month = today.getMonth();
int monthValue = today.getMonthValue();
int day = today.getDayOfMonth();
DayOfWeek dayOfWeek = today.getDayOfWeek();

// Comparaisons
boolean isBefore = date1.isBefore(date2);
boolean isAfter = date1.isAfter(date2);
boolean isEqual = date1.isEqual(date2);

// Modification
LocalDate modified = today
    .withYear(2025)
    .withMonth(12)
    .withDayOfMonth(25);

// Formatage
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String formatted = today.format(formatter);

DateTimeFormatter isoFormatter = DateTimeFormatter.ISO_LOCAL_DATE;
String iso = today.format(isoFormatter);

// Parsing
LocalDate parsed = LocalDate.parse("15/01/2024", formatter);
```

### Exemples pratiques :

```java
// Calculer l'âge
public int calculateAge(LocalDate birthDate) {
    return Period.between(birthDate, LocalDate.now()).getYears();
}

// Vérifier si c'est un jour ouvré
public boolean isWorkDay(LocalDate date) {
    DayOfWeek day = date.getDayOfWeek();
    return day != DayOfWeek.SATURDAY && day != DayOfWeek.SUNDAY;
}

// Trouver le prochain vendredi
public LocalDate nextFriday(LocalDate date) {
    return date.with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
}

// Calculer temps écoulé
public String timeSince(LocalDateTime past) {
    Duration duration = Duration.between(past, LocalDateTime.now());
    long hours = duration.toHours();
    long minutes = duration.toMinutes() % 60;
    return hours + "h " + minutes + "m";
}
```

- **Impact** :
  - Thread-safety : Toutes les classes sont immuables
  - Clarté : API intuitive et cohérente
  - Fonctionnalités : Plus riche que `Date`/`Calendar`



## 7 - Annotations répétables et sur types
- **Utilité** : Permet d'appliquer la même annotation plusieurs fois et sur plus d'éléments.

```java
// Avant (Java 7) - Annotation conteneur nécessaire
@Schedules({
    @Schedule(day = "Monday", time = "9:00"),
    @Schedule(day = "Friday", time = "17:00")
})
public class Task {
}

// Après (Java 8) - Annotations répétables
@Schedule(day = "Monday", time = "9:00")
@Schedule(day = "Friday", time = "17:00")
public class Task {
}

// Définition de l'annotation répétable
@Repeatable(Schedules.class)
public @interface Schedule {
    String day();
    String time();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Schedules {
    Schedule[] value();
}
```

### Annotations sur types :

```java
// Annotations sur types pour analyse statique
@NonNull String name = "Alice";
List<@NonNull String> names = new ArrayList<>();

public @NonNull String getName(@NonNull User user) {
    return user.getName();
}
```

- **Impact** : Code plus lisible pour annotations multiples, meilleure analyse statique.



## 8 - Interfaces fonctionnelles
- **Utilité** : Interfaces avec une seule méthode abstraite, utilisables avec lambdas.

```java
// Annotation @FunctionalInterface (optionnelle mais recommandée)
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Méthodes par défaut et statiques autorisées
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }
}

// Utilisation
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

int sum = add.calculate(5, 3);
int product = multiply.calculate(5, 3);
```

### Interfaces fonctionnelles standard (`java.util.function`) :

```java
// Predicate<T> - Test (T -> boolean)
Predicate<String> isEmpty = String::isEmpty;
Predicate<Integer> isPositive = n -> n > 0;

boolean result = isEmpty.test("");
boolean result2 = isPositive.test(5);

// Function<T, R> - Transformation (T -> R)
Function<String, Integer> length = String::length;
Function<Integer, String> toString = Object::toString;

int len = length.apply("Hello");
String str = toString.apply(42);

// Consumer<T> - Action (T -> void)
Consumer<String> printer = System.out::println;
Consumer<User> logger = user -> log(user.getName());

printer.accept("Hello");

// Supplier<T> - Fourniture (() -> T)
Supplier<String> uuidGenerator = () -> UUID.randomUUID().toString();
Supplier<LocalDate> today = LocalDate::now;

String uuid = uuidGenerator.get();
LocalDate date = today.get();

// BiFunction<T, U, R> - Transformation 2 args ((T, U) -> R)
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<String, String, String> concat = (a, b) -> a + b;

int sum = add.apply(5, 3);

// BiConsumer<T, U> - Action 2 args ((T, U) -> void)
BiConsumer<String, Integer> logger = (name, count) -> 
    System.out.println(name + ": " + count);

logger.accept("Items", 10);

// UnaryOperator<T> - Transformation même type (T -> T)
UnaryOperator<String> upper = String::toUpperCase;
UnaryOperator<Integer> square = n -> n * n;

String result3 = upper.apply("hello");

// BinaryOperator<T> - Opération 2 args même type ((T, T) -> T)
BinaryOperator<Integer> max = Math::max;
BinaryOperator<String> concat2 = String::concat;

int maximum = max.apply(5, 10);
```

### Combinaison d'interfaces fonctionnelles :

```java
// Predicate - and, or, negate
Predicate<String> isLong = s -> s.length() > 5;
Predicate<String> startsWithA = s -> s.startsWith("A");

Predicate<String> combined = isLong.and(startsWithA);
Predicate<String> either = isLong.or(startsWithA);
Predicate<String> opposite = isLong.negate();

// Function - compose, andThen
Function<String, String> trim = String::trim;
Function<String, String> upper = String::toUpperCase;

Function<String, String> trimThenUpper = trim.andThen(upper);
Function<String, String> upperThenTrim = upper.compose(trim);

String result4 = trimThenUpper.apply("  hello  ");
```

- **Impact** : Programmation fonctionnelle facilitée, code plus expressif et réutilisable.



## 9 - Nashorn JavaScript Engine
- **Utilité** : Moteur JavaScript intégré à la JVM pour exécuter du code JavaScript.

```java
// Exécuter du JavaScript depuis Java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn");

// Exécuter du code JavaScript
engine.eval("print('Hello from JavaScript')");

// Évaluer une expression
Object result = engine.eval("10 + 20");
System.out.println(result); // 30

// Passer des objets Java au JavaScript
engine.put("name", "Alice");
engine.eval("print('Hello ' + name)");

// Appeler des fonctions JavaScript
engine.eval("function greet(name) { return 'Hello ' + name; }");
Invocable invocable = (Invocable) engine;
Object greeting = invocable.invokeFunction("greet", "Bob");
```

- **Impact** : Interopérabilité Java/JavaScript, scripting dynamique.



## 10 - Améliorations des Collections

### Nouvelles méthodes sur Map

```java
Map<String, Integer> map = new HashMap<>();

// Avant (Java 7)
if (!map.containsKey("key")) {
    map.put("key", 1);
}

// Après (Java 8) - putIfAbsent
map.putIfAbsent("key", 1);

// compute - Calculer nouvelle valeur
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// computeIfAbsent - Calculer si absent
map.computeIfAbsent("key", k -> computeValue(k));

// computeIfPresent - Calculer si présent
map.computeIfPresent("key", (k, v) -> v + 1);

// merge - Fusionner valeurs
map.merge("key", 1, Integer::sum);

// forEach
map.forEach((key, value) -> System.out.println(key + " = " + value));

// replaceAll
map.replaceAll((key, value) -> value * 2);

// getOrDefault
int value = map.getOrDefault("missing", 0);
```

### Nouvelles méthodes sur `List`

```java
List<String> list = new ArrayList<>();

// replaceAll
list.replaceAll(String::toUpperCase);

// sort
list.sort(Comparator.naturalOrder());
list.sort(Comparator.reverseOrder());

// removeIf
list.removeIf(s -> s.isEmpty());
Nouvelles méthodes sur Collection
javaCollection<String> collection = new ArrayList<>();

// removeIf
collection.removeIf(s -> s.length() < 3);

// stream
Stream<String> stream = collection.stream();

// parallelStream
Stream<String> parallelStream = collection.parallelStream();
```

- **Impact** : Opérations courantes simplifiées, code plus expressif.



## 11 - Comparator amélioré

- **Utilité** : Construction de comparateurs complexes de manière fluide.

```java
class Person {
    String name;
    int age;
    String city;
    
    // constructeur, getters...
}

List<Person> people = getPeople();

// Avant (Java 7)
Collections.sort(people, new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        int nameComp = p1.getName().compareTo(p2.getName());
        if (nameComp != 0) return nameComp;
        return Integer.compare(p1.getAge(), p2.getAge());
    }
});

// Après (Java 8)
people.sort(Comparator.comparing(Person::getName)
                      .thenComparing(Person::getAge));

// Autres exemples
people.sort(Comparator.comparing(Person::getAge).reversed());

people.sort(Comparator.comparing(Person::getName)
                      .thenComparing(Person::getAge)
                      .thenComparing(Person::getCity));

people.sort(Comparator.comparing(Person::getName, 
                                 String.CASE_INSENSITIVE_ORDER));

people.sort(Comparator.comparingInt(Person::getAge));

// Null-safe
people.sort(Comparator.nullsFirst(
    Comparator.comparing(Person::getName)));

people.sort(Comparator.nullsLast(
    Comparator.comparing(Person::getName)));
```

- **Impact** : Tri complexe beaucoup plus lisible et maintenable.



## 12 - `StringJoiner` et `String.join()`

- **Utilité** : Joindre des chaînes avec un délimiteur de manière simple.

```java
// Avant (Java 7)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < items.size(); i++) {
    sb.append(items.get(i));
    if (i < items.size() - 1) {
        sb.append(", ");
    }
}
String result = sb.toString();

// Après (Java 8) - String.join()
String result = String.join(", ", items);

String result2 = String.join(" - ", "A", "B", "C");

// StringJoiner pour plus de contrôle
StringJoiner joiner = new StringJoiner(", ", "[", "]");
joiner.add("A");
joiner.add("B");
joiner.add("C");
String result3 = joiner.toString(); // [A, B, C]

// Avec Collectors.joining()
String result4 = items.stream()
    .collect(Collectors.joining(", "));

String result5 = items.stream()
    .collect(Collectors.joining(", ", "[", "]"));
```

- **Impact** : Code plus concis pour joindre des chaînes, moins d'erreurs.



## 13 - Base64
- **Utilité** : API standard pour encoder/décoder en Base64.

```java
// Avant (Java 7) - javax.xml.bind.DatatypeConverter ou bibliothèque externe
import javax.xml.bind.DatatypeConverter;
String encoded = DatatypeConverter.printBase64Binary(data);
byte[] decoded = DatatypeConverter.parseBase64Binary(encoded);

// Après (Java 8) - API standard
import java.util.Base64;

// Encodage
String original = "Hello World";
String encoded = Base64.getEncoder()
    .encodeToString(original.getBytes());

// Décodage
byte[] decoded = Base64.getDecoder().decode(encoded);
String decodedStr = new String(decoded);

// URL-safe
String urlEncoded = Base64.getUrlEncoder()
    .encodeToString(original.getBytes());
byte[] urlDecoded = Base64.getUrlDecoder().decode(urlEncoded);

// MIME (avec retours à la ligne)
String mimeEncoded = Base64.getMimeEncoder()
    .encodeToString(largeData);
```

- **Impact** : API standard unifiée, plus besoin de dépendances externes.

