# Guide complet : Nouveautés Java 11 (LTS)
Ci-dessous les principales nouveautés introduites entre Java 8 et Java 11, avec des exemples pratiques.

## 1 - Nouvelles méthodes sur String
### `String.isBlank()`
- **Utilité** : Vérifie si une chaîne est vide ou ne contient que des espaces blancs.
 
```java
// Avant (Java 8)
String text = "   ";
boolean isEmpty = text.trim().isEmpty();

// Après (Java 11)
boolean isBlank = text.isBlank();
```

- **Impact** : Amélioration de la lisibilité et légère optimisation (pas de création d'objet temporaire avec `trim()`).

### `String.lines()`
- **Utilité** : Retourne un Stream des lignes d'une chaîne multilignes.

```java
// Avant (Java 8)
String multiline = "Ligne1\nLigne2\nLigne3";
String[] lines = multiline.split("\\r?\\n");
Arrays.stream(lines).forEach(System.out::println);

// Après (Java 11)
multiline.lines().forEach(System.out::println);
- **Impact** : Code plus expressif et gestion automatique des différents séparateurs de lignes (\n, \r\n).
```

### `String.repeat()`
- **Utilité** : Répète une chaîne n fois.

```java
// Avant (Java 8)
String repeated = new String(new char[3]).replace("\0", "Java");

// Après (Java 11)
String repeated = "Java".repeat(3); // "JavaJavaJava"
```

- **Impact** : Lisibilité considérablement améliorée et performance optimisée.

### `String.strip()`, `stripLeading()`, `stripTrailing()`
- **Utilité** : Supprime les espaces blancs Unicode (pas seulement ASCII comme trim()).

```java
// Avant (Java 8)
String text = " \u2000 Hello \u2000 ";
String trimmed = text.trim(); // Ne supprime pas tous les espaces Unicode

// Après (Java 11)
String stripped = text.strip(); // "Hello"
String leading = text.stripLeading();
String trailing = text.stripTrailing();
```

- **Impact** : Meilleure gestion des espaces Unicode, plus robuste pour l'internationalisation.

## 2 - Inférence de type avec var (Java 10, disponible en Java 11)
- **Utilité** : Réduit la verbosité en laissant le compilateur inférer le type des variables locales.

```java
// Avant (Java 8)
Map<String, List<Customer>> customersByCity = new HashMap<>();
List<String> names = Arrays.asList("Alice", "Bob");

// Après (Java 11)
var customersByCity = new HashMap<String, List<Customer>>();
var names = List.of("Alice", "Bob");
```

- **Impact** : Réduction de la verbosité tout en maintenant la sécurité des types. Attention à ne pas abuser pour garder la lisibilité.

## 3 - Collections Factory Methods (Java 9, disponible en Java 11)
- **Utilité** : Création simplifiée de collections immuables.

```java
// Avant (Java 8)
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");
List<String> immutableList = Collections.unmodifiableList(list);

Set<String> set = new HashSet<>();
set.add("X");
set.add("Y");
Set<String> immutableSet = Collections.unmodifiableSet(set);

Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
Map<String, Integer> immutableMap = Collections.unmodifiableMap(map);

// Après (Java 11)
List<String> immutableList = List.of("A", "B", "C");
Set<String> immutableSet = Set.of("X", "Y");
Map<String, Integer> immutableMap = Map.of("one", 1, "two", 2);
```

- **Impact** : Code beaucoup plus concis, performance améliorée (structures optimisées), collections immutables par défaut.

## 4 - Améliorations des Optionals
### `Optional.isEmpty()`
- **Utilité** : Méthode complémentaire à isPresent() pour mieux exprimer l'intention.

```java
// Avant (Java 8)
Optional<String> opt = Optional.empty();
if (!opt.isPresent()) {
    System.out.println("Vide");
}

// Après (Java 11)
if (opt.isEmpty()) {
    System.out.println("Vide");
}
```

- **Impact** : Amélioration de la lisibilité (logique positive vs négative).

## 5 - Nouveau client HTTP (Java 11)
- **Utilité** : API moderne et asynchrone pour les requêtes HTTP, remplaçant HttpURLConnection.

```java
// Avant (Java 8) - HttpURLConnection
URL url = new URL("https://api.example.com/data");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
BufferedReader in = new BufferedReader(
    new InputStreamReader(conn.getInputStream()));
String inputLine;
StringBuilder content = new StringBuilder();
while ((inputLine = in.readLine()) != null) {
    content.append(inputLine);
}
in.close();
conn.disconnect();

// Après (Java 11)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .GET()
    .build();

HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());
String content = response.body();
```

- **Impact** : API plus moderne, support HTTP/2, gestion asynchrone native, code plus maintenable.

## 6 - Amélioration des Streams
### `Stream.takeWhile()` et `dropWhile()`
- **Utilité** : Prendre ou ignorer des éléments selon une condition (introduit en Java 9).

```java
// Avant (Java 8) - pas d'équivalent simple
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 1, 2);
List<Integer> taken = new ArrayList<>();
for (Integer n : numbers) {
    if (n < 4) taken.add(n);
    else break;
}

// Après (Java 11)
List<Integer> taken = numbers.stream()
    .takeWhile(n -> n < 4)
    .collect(Collectors.toList()); // [1, 2, 3]

List<Integer> dropped = numbers.stream()
    .dropWhile(n -> n < 4)
    .collect(Collectors.toList()); // [4, 5, 1, 2]
```

- **Impact** : Opérations sur les streams plus expressives et fonctionnelles.

### `Stream.ofNullable()`
- **Utilité** : Crée un stream avec 0 ou 1 élément selon si la valeur est null.

```java
// Avant (Java 8)
String value = getValueOrNull();
Stream<String> stream = value != null ? 
    Stream.of(value) : Stream.empty();

// Après (Java 11)
Stream<String> stream = Stream.ofNullable(getValueOrNull());
```

- **Impact** : Code plus concis, évite les vérifications null explicites.

## 7 - Interface privée methods (Java 9)
- **Utilité** : Permet de factoriser du code dans les interfaces avec des méthodes privées.

```java
// Avant (Java 8) - code dupliqué dans les méthodes default
public interface Calculator {
    default int addAndLog(int a, int b) {
        System.out.println("Operation: addition");
        return a + b;
    }
    
    default int multiplyAndLog(int a, int b) {
        System.out.println("Operation: multiplication");
        return a * b;
    }
}

// Après (Java 11)
public interface Calculator {
    default int addAndLog(int a, int b) {
        log("addition");
        return a + b;
    }
    
    default int multiplyAndLog(int a, int b) {
        log("multiplication");
        return a * b;
    }
    
    private void log(String operation) {
        System.out.println("Operation: " + operation);
    }
}
```

- **Impact** : Meilleure factorisation du code dans les interfaces, moins de duplication.

## 8 - Try-with-resources amélioré (Java 9)
- **Utilité** : Permet d'utiliser des variables effectively final dans try-with-resources.

```java
// Avant (Java 8)
BufferedReader reader1 = new BufferedReader(new FileReader("file.txt"));
try (BufferedReader reader2 = reader1) {
    return reader2.readLine();
}

// Après (Java 11)
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
try (reader) {
    return reader.readLine();
}
```

- **Impact** : Code moins verbeux, plus lisible.

## 10 - Méthodes ajoutées à Files
Files.readString() et Files.writeString()
- **Utilité** : Lecture/écriture simplifiée de fichiers texte.

```java
// Avant (Java 8)
Path path = Paths.get("file.txt");
List<String> lines = Files.readAllLines(path);
String content = String.join("\n", lines);

Files.write(path, content.getBytes(StandardCharsets.UTF_8));

// Après (Java 11)
String content = Files.readString(path);
Files.writeString(path, content);
```

- **Impact** : API simplifiée, code plus lisible, gestion automatique de l'encoding.

## 11 - Collection.toArray() amélioré
- **Utilité** : Conversion collection vers array sans générateur de fonction.

```java
// Avant (Java 8)
List<String> list = Arrays.asList("A", "B", "C");
String[] array = list.toArray(new String[0]);

// Après (Java 11)
String[] array = list.toArray(String[]::new);
```

- **Impact** : Syntaxe plus moderne et expressive.

## 12 - Predicate.not()
- **Utilité** : Négation de prédicat plus lisible dans les streams.

```java
// Avant (Java 8)
List<String> nonEmpty = strings.stream()
    .filter(s -> !s.isEmpty())
    .collect(Collectors.toList());

// Après (Java 11)
List<String> nonEmpty = strings.stream()
    .filter(Predicate.not(String::isEmpty))
    .collect(Collectors.toList());
```

- **Impact** : Meilleure lisibilité avec les method references.

## 13 - Suppression de modules deprecated
- **Important** : Java 11 supprime plusieurs modules et APIs :
  - JavaFX (maintenant séparé)
  - CORBA
  - Java EE modules (JAX-WS, JAXB, etc.)

- **Action requise** : Si vous utilisez ces APIs, ajoutez les dépendances externes correspondantes.

```xml
<!-- Exemple pour JAXB -->
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>
```

## 14 - Launch Single-File Source-Code Programs
- **Utilité** : Exécuter directement un fichier .java sans compilation explicite.

```bash
# Avant (Java 8)
javac HelloWorld.java
java HelloWorld

# Après (Java 11)
java HelloWorld.java
```

- **Impact** : Idéal pour scripts et prototypage rapide.

## 9 - Nouveau garbage collector : ZGC (expérimental en Java 11)
- **Utilité** : Garbage collector à faible latence pour de grandes heaps (jusqu'à 16 TB).

```bash
# Activation
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -jar application.jar
```

- **Impact** : Pauses GC < 10ms, idéal pour applications nécessitant une faible latence.

---