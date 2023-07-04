# Spring Boot 101
## Creating an E-Store API

### Topics
1. Input [validation](https://www.baeldung.com/spring-boot-bean-validation)
2. Handling Exceptions
3. JPA (Database)

---

### Input validations
Add the following dependency:
```xml
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-validation</artifactId> 
</dependency>
```

Create a `CreateUserRequest`:

```java
import lombok.Data;

@Data
public class CreateUserRequest {
    private String email;
    private String password;
    private String firstName;
    private String middleName;
    private String lastName;
    private String birthDate;
}
```

#### Annotations
`@JsonProperty` `@NotBlank` `@NotNull` `@NotEmpty` `@Pattern` `@Email`

#### Requirements
1. Request body must use valid keys and value format:
   ```json
      {
        "email": "jdoe@apper.com",
        "password": "123qwe",
        "first_name": "John",
        "last_name": "Doe",
        "birth_date": "2000-10-10"
      }
    ```
2. All fields are required except for middle name. Values must not be an empty string.
3. Email format must be valid.
4. Password must be at least 8 characters.
5. Birthdate format must be `yyyy-mm-dd`.
6. Give a clear error message for invalid fields.

---

### Handling Exceptions

The concept is to CONVERT caught exceptions to response.

Create an error response class/record:
```java
public record ServiceError(String message){}
```

Create the class to handle exception to error response conversation:

```java

import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;

@ControllerAdvice
public class ServiceExceptionHandler {

   @ResponseStatus(HttpStatus.BAD_REQUEST)
   @ExceptionHandler(MethodArgumentNotValidException.class)
   @ResponseBody
   public ServiceError handleInvalidInputFields(MethodArgumentNotValidException ex) {
      return ex.getBindingResult().getAllErrors().stream()
              .findFirst()
              .map(objectError -> new ServiceError(objectError.getDefaultMessage()))
              .orElse(new ServiceError("Unknown invalid argument encountered"));
   }

}
```

#### Laboratory
1. Create a validation that age must be at least 15 years old.
   ```java
      @RestController
      @RequestMapping("user")
      public class UserApi {
      
          @PostMapping
          public CreateUserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
              LocalDate birthDate = LocalDate.parse(request.getBirthDate());
      
              // Compute age. Review LocalDate
              // if age is below 15, throw an InvalidUserAgeException(you must create this exception)
              // create new method in your controller advice class that handles InvalidUserAgeException.class
      
              return new CreateUserResponse("RANDOM_STRING");
          }
      
      }
   ```

---

### JPA (Database)

Install docker and boot a MySql docker container.

https://hub.docker.com/_/mysql/