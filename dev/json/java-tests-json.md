# Java tests with json

## Why the hell would I like to use JSON in java?

Working with distributed systems where you need to send messages over the network, we often need to convert java object 
to send them through network or to store them to an event store. A Simple way to do that is to use Json format.  
For example: We can manipulate this kind of json object :

```json
{
  "eventType": "createUser",
  "firstName": "John",
  "lastName": "Doe",
  "age": 32
}
```

Using Java, we can represent this object with that [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) class: 
```java
public class UserCreation {
    private String eventType = "createUser";
    private String firstName;
    private String lastName;
    private int Age;
    
    // Getter & Setters
}
```

We can use libraries like [Gson](https://github.com/google/gson) or [Jackson](https://github.com/FasterXML/jackson) to 
convert automatically from one format to the other. But that's not the subject of this article and what I'm about to
talk works with both of these libraries.

Okay, we have the java representation of our json object. But how can we be sure that this class will keep representing
this json object with new functionality and refactoring coming to our codebase? Only one answer here : **TESTS !**  
Keep in mind that I don't want to check that Jackson or Gson does its job  (I'm not testing the library!), I want to 
check that my object matches the json format.

## How to tests json conversion?

The simplest way to test the conversion, is to convert firstly the object to its json format and then then to compare it
as a string.
Example :
```java
public class UserCreationTest {

    @Test
    public void should_be_able_to_convert_to_json() {
        // Given
        UserCreation userCreation = new UserCreation();
        [...] // Setup the object
        
        // When
        String json = new ObjectMapper().writeValueAsString(userCreation);
        
        // Then
        String expectedJson = "{\n" +
                "  \"eventType\": \"createUser\",\n" +
                "  \"firstName\": \"John\",\n" +
                "  \"lastName\": \"Doe\",\n" +
                "  \"age\": 32\n" +
                "}";
        assertEquals(expectedJson, json);
    }
}
```

This conversion from object to json is what we called serialization. The opposite way that convert json to object is 
called deserialization. So, we should also write a test for deserialization.

That test can work but it has a lot of problems :
* Jackson can write properties in any order.
* Error message are awful to read when the test fails
* `expectedJson` is really hard to read and to modify

## How can we improve that kind of tests?

The good news is that's a common problem: there is already someone that already solved it for you ! :smile:  
We can use [JsonAssert library](https://github.com/skyscreamer/JSONassert)
Another good news is that, the test above does not need to be changed a lot :
```java
public class UserCreationTest {

    @Test
    public void should_be_able_to_convert_to_json() {
        // Given
        UserCreation userCreation = new UserCreation();
        [...] // Setup the object
        
        // When
        String json = new ObjectMapper().writeValueAsString(userCreation);
        
        // Then
        String expectedJson = "{\n" +
                "  \"eventType\": \"createUser\",\n" +
                "  \"firstName\": \"John\",\n" +
                "  \"lastName\": \"Doe\",\n" +
                "  \"age\": 32\n" +
                "}";
        JSONAssert.assertEquals(expectedJson, json);
    }
}
```

JSONAssert is really a nice library and I invite you to read the readme which is almost the whole documentation you will
need.  If you work with Springboot, you don't need to add a dependency as [it is already included in the dependency tree
since version 1.4](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4) :wink:

Instead, switching to the JSONAssert implies only one and only one modification: adding `JSONAssert` before `assertEquals`?
With this change we fix the first two lines in our problems list:
* ~~Jackson can write properties in any order.~~
* ~~Error message are awful to read when the test fails~~

## Cool but I'm using [assertj](https://github.com/joel-costigliola/assertj-core) and I don't like `assertEquals`

I have a solution for you and the best part is that it will also fix the last point.  
[advertising]So if you don't know assertJ, go check it out, it is a great library to improve your tests :heart:[/advertising]

For this part, there is a big downside though, you need to be bound to Springboot... I'm a bit sad, but someday I think
I'll try to solve that problem.  
So yeah, for this part, we will be using [spring-boot-test](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test)
as there is some classes that are really interesting inside.  
As I'm a Jackson user, I'll write code for Jackson but there is the same for GSON.

Using [JacksonTester](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/json/JacksonTester.html), 
I can improve a lot my test above :
```java
public class UserCreationTest {
    private final JacksonTester<UserCreation> jacksonTester = new JacksonTester<>(UserCreation.class, ResolvableType.forType(UserCreation.class), new ObjectMapper());

    @Test
    public void should_be_able_to_serialize() throws IOException {
        UserCreation userCreation = new UserCreation();
        [...] // Setup the object
        
        assertThat(this.jacksonTester.write(userCreation))
            .isEqualToJson("userCreation.json");
   }
   
    @Test
    public void should_be_able_to_deserialize() throws IOException {
        UserCreation userCreation = new UserCreation();
        [...] // Setup the object
        
        assertThat(jacksonTester.readObject("userCreation.json"))
            .isEqualToJson(userCreation);
   }
}
```

*Note : I don't use the syntax with `JacksonTester.init` (as recommended in [documentation](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4#json-assertions))
in `setup` method because my IDE thinks that I did not initialized my jacksonTester variable and displays a warning that
I really dislike. It also uses tons of reflexion to determine types and I am not a fan of that, when I can simply put 
it.*

The file `userCreation.json` can be put in test/resources folder with that content :
```json
{
  "eventType": "createUser",
  "firstName": "John",
  "lastName": "Doe",
  "age": 32
}
```

Your IDE will be able to link the file, so no worries to lose hours looking for the file somewhere :wink:   
It also solve the last problem, as the test is really shorter, and the file is a json so your IDE can help you with
syntax highlighting and format validation.
