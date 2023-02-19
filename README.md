## Vérification des valeurs nulles Java | Spring Boot
Dans ce repo,  je vous partage une article qui explique quelques exemples des différents types de vérifications nulles ou des techniques d'évitement NPE (NullPointerException) en Java - Spring Boot.

*Notez que je suis toujours dans le processus d'apprentissage - probablement sans fin :)-*

**java.util.Optional**
Un objet conteneur qui peut contenir ou non une valeur non nulle. Si une valeur est présente, `isPresent()` renverra `true` et `get()` renverra la valeur.
	
**Optional.ofNullable()** - *Renvoie un Optional décrivant la valeur donnée, si non null, sinon renvoie un Optional vide.*

```
@Repository
@RequiredArgsConstructor
public class StudentDaoImpl implements StudentDao {
    private final MongoClient mongoClient;

    @Override
    public Optional<Student> findStudentsById(Long id) {
        return Optional.ofNullable(mongoClient.getDatabase("dbName")
                .getCollection("collectionName", Student.class)
                .find(eq("id", id))
                .first());
    }
}
```

La valeur renvoyée par cette méthode ne sera jamais nulle. S'il est nul, la valeur renvoyée sera `Optional.empty()`. De cette façon, si le résultat de cette méthode est utilisé ailleurs, il n'y aura aucune chance d'obtenir un NPE.

Optional.orElseThrow() - *Si une valeur est présente, renvoie la valeur, sinon lève NoSuchElementException. Dans le cas de null, si vous souhaitez lever une exception, vous pouvez simplement utiliser `orElseThrow()`.

```
@Service
@RequiredArgsConstructor
public class StudentServiceImpl implements StudentService{
    private final StudentDao studentDao;

    @Override
    public Student getStudentById(Long id) {
        return studentDao.findStudentsById(id).orElseThrow(()-> new CustomException(STUDENT_NOT_FOUND));
    }
}
```

**Optional.orElse()** - *Si une valeur est présente, renvoie la valeur, sinon renvoie autre.*

```
@Override 
public Student getMockStudent(Long id) { 
    final var student = Student. constructeur () 
            .id(1L) 
            .name("ege") 
            .build(); 

    return studentDao.findStudentsById(id).orElse(student); 
}
```

**Optional.get()** - *Si une valeur est présente, renvoie la valeur, sinon lève NoSuchElementException.*

```
@Override 
public Student getStudentDirectly(Long id) { 
    return studentDao.findStudentsById(id).get(); 
}
```

Equivalent à :

```
// manière correcte de le faire 
@Override 
public Student getStudentDirectly(Long id) { 
    final var studentOptional = studentDao.findStudentsById(id); 
    if(studentOptional.isPresent()){ 
        // faites vos affaires ici
    } 
}
```

Un autre exemple :

```
@Override 
public String getStudentName(Long id) { 
    return studentDao.findStudentsById(id) 
            .map(Student::getName) 
            .orElse("Hayley"); 
}
```

**Classes utilitaires de apache.commons**
- Pour les instances Collection : `CollectionUtils.isEmpty()`
- Pour les instances Map : `MapUtils.isEmpty()` ou `MapUtils.isNotEmpty()`
- Pour Chaîne : `StringUtils.isEmpty()` ou `StringUtils.isNotEmpty()`
	
Dans le cas de listes, de Map, etc., `isEmpty()` vérifie si la collection/map est nulle ou a une taille de 0. De même String, elle vérifie si la String est nulle ou a une longueur de 0.

```
// Check for list
final var list = List.of();

if (CollectionUtils.isEmpty(list)) {
    System.out.print("list is empty or null");
}

// Does the same thing as previous if statement
if (list == null || list.size() == 0) {
    System.out.print("list is empty or null");
}

// Check for string
final var str = "";

if (StringUtils.isEmpty(str)) {
    System.out.println("string is empty or null");
}

// Does the same thing as previous if statement
if (str == null || str.equals("")) {
    System.out.println("string is empty or null");
}
```

**Objects::nonNull dans Streams** - *Renvoie `true` si la référence fournie n'est pas nulle sinon renvoie `false`.*

Exemple :

```
final var list = Arrays.asList(1, 2, null, 3, null, 4);

list.stream()
        .filter(Objects::nonNull)
        .forEach(System.out::print);
```

Le résultat : `1234`

**requireNonNull methods de java.util.Objects**
**Lombok’s Builder.Default**

```
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Student {

    private Long id;
    @Builder.Default
    private String name = "hayley";
    @Builder.Default
    private List<SchoolClass> classes = new ArrayList<>();
}
```

**NotNull, NotEmpty, NotBlank Annotations** :
- `@NotNull` : L'élément annoté ne doit pas être nul. Accepte tout type.
- `@NotEmpty` : L'élément annoté ne doit pas être nul ou vide. Les types pris en charge sont CharSequence, Collection, Map, Array.
- `@NotBlank` : L'élément annoté ne doit pas être nul et doit contenir au moins un caractère non blanc. Accepte CharSequence”

Exemple :

```
@NotNull
private Long id;
@NotEmpty
private String name;
@NotEmpty
private List<SchoolClass> classes;
private Map<Integer, Teacher> teacherMap;
```

*Bon apprentissage... Bon codage...*

[Source d'article](https://betterprogramming.pub/checking-for-nulls-in-java-minimize-using-if-else-edae27016474)
