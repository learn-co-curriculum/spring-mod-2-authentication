# Code Along: Authentication

## Learning Goals

- Briefly explain how authentication works with Spring Security.
- Set up authentication using users we define.
- Practice and test using Postman with authentication.

## Introduction

In the last lesson, we saw how we can use authentication to access our API
when using Spring Security at its default. Remember, authentication is the
process of validating that a user is who they claim to be.

But what if we want to authenticate our own users? How can we do that in Spring?

Let's explore that option more in this lesson.

## Spring Security Authentication

Up until this point, we have used the built-in default `user` user generated for
us by the Spring Framework.

We can, however, get users from any source we want by creating a custom
`UserDetailsService` that can either return users it creates itself, or use an
external data source to fetch the users from a database or another service. We
will first look at how to implement the former option.

As we saw in the last lesson, all the endpoints are now protected under Spring
Security by default. So what exactly is happening when we authenticate a user?
When we send a request to the server, it can be intercepted by an authentication
filter. The authentication filter will hand off the request to the
authentication manager and authentication provider to check for a
**user details service** and a **password encoder**. The `UserDetailsService`
interface loads user-specific data, such as a username and password. The
`PasswordEncoder` interface implements password management and is used in
conjunction with the `UserDetailsService` when authenticating. Below is a
diagram to illustrate the flow of Spring authentication.

![spring-authentication-diagram](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/spring-authentication-diagram.png)

We'll create our own `UserDetailsService` and `PasswordEncoder` beans to add our
own user!

Open up the `SecurityConfiguration` class we created in the
`spring-security-demo` project that we used in the last lesson. This is where we
will customize our application's authentication. Go ahead and add the following
code:

```java
package com.example.springsecuritydemo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
public class SecurityConfiguration {

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager userDetailsService = new InMemoryUserDetailsManager();

        UserDetails user = User.withUsername("mary")
                .password(passwordEncoder().encode("test"))
                .authorities("read")
                .build();

        userDetailsService.createUser(user);

        return userDetailsService;
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Let's break down this code:

- We will first create two bean methods: `userDetailsService()` and
  `passwordEncoder()`.
- The `userDetailsService()` method will return a `UserDetailsService` instance.
  The `UserDetailsService` has a handful of implementing classes. We'll use the
  `InMemoryUserDetailsManager` implementation to manage all the users we add to
  it "in memory" - this means these users will not be saved to a file or in a
  database anywhere, and will cease to exist when the application is stopped.
  This would not work for a production-level application, but it is a good way
  to demonstrate the mechanism of `UserDetailsService` without having to worry
  about persistence or service integration.
- Next we create a test user using the `User` class that is part of the
  `org.springframework.security.core.userdetails` package. To create a user, we
  need to provide that user a username and a password.
  - Note that in Spring 5, we are required to secure the password. To do so, we
    will have the `passwordEncoder()` method return a new `PasswordEncoder`
    instance. `BCryptPasswordEncoder` is the recommended password encoder to
    use, so we'll return an instance of that implementation. We will address
    password handling and security in a separate lesson later on to better
    explain this.
- Even though we have not yet covered authorization, we do have to give our
  user a default "authority". We'll explain what that means in a later lesson.
- Finally, we'll add our new `user` to the `userDetailsService` and then return
  the `userDetailService`.

Let's restart the application and see what happens.

![intellij-console](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/intellij-spring-security-console.png)

Notice when we start the application this time that we do not see the
auto-generated password anymore. This means we now will have to pass in the
username/password combination that we defined in our custom `UserDetailsService`
bean.

![basic-auth-credentials](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/postman-basic-auth-mary-test-credentials.png)

If we send the request now with these credentials, we'll hit our `/hello`
endpoint again!

## User Authentication and PostgreSQL

Now that we know how to authenticate using a user "in-memory", let's talk about
updating our application to use a user stored in a database.

For this, we'll need to add some dependencies to our project. Open up the
`pom.xml` file to add the Spring Data JPA dependency and the PostgreSQL Driver
dependency:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.6</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.example</groupId>
  <artifactId>spring-security-demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>spring-security-demo</name>
  <description>spring-security-demo</description>
  <properties>
    <java.version>11</java.version>
  </properties>
  <dependencies>
    <!-- Add the Spring Data JPA dependency -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Add the PostgreSQL Driver dependency -->
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>${project.parent.version}</version>
        <configuration>
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

Once the dependencies have been added to the `pom.xml`, click the little
Maven icon in the upper-right hand corner to reload the changes:

![load-maven-changes](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/load-maven-changes.png)

Now let's create a database that we can connect to in order to access user data!
Open up pgAdmin4 and create a new database. Let's call it "security_demo".

![create-security-demo-db](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/create-sercurity-demo-db.png)

Once the database has been created, open up the Query Tool and copy in the
following to create the `users` table and a user with the same credentials we
saw previously:

```postgresql
DROP TABLE IF EXISTS users;

CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT NOT NULL,
  password TEXT NOT NULL
);

INSERT INTO users(id, username, password) VALUES(1, 'mary', 'test');
```

Execute the query and then run a `SELECT * FROM users;` query to ensure the
user, "mary", has been added to the database table.

Now we'll need to add some packages and classes to our application. Go ahead and
create a `repository`, `entity`, and `service` package with `UserRepository`,
`User`, and `UserService` classes. The following should be the new project
structure:

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springsecuritydemo
    │   │               ├── SpringSecurityDemoApplication.java
    │   │               ├── config
    │   │               │   └── SecurityConfiguration.java
    │   │               ├── controller
    │   │               │   └── DemoController.java
    │   │               ├── entity
    │   │               │   └── User.java
    │   │               ├── repository
    │   │               │   └── UserRepository.java
    │   │               └── service
    │   │                   └── UserService.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springsecuritydemo
                        └── SpringSecurityDemoApplicationTests.java
```

In the `User` class, add the following code to match the `users` table we just
created:

```java
package com.example.springsecuritydemo.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private int id;

    private String username;

    private String password;
}
```

Let's add the code for the `UserRepository` class as well:

```java
package com.example.springsecuritydemo.repository;

// Make sure you import the correct User class
import com.example.springsecuritydemo.entity.User;
import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface UserRepository extends CrudRepository<User, Integer> {

    Optional<User> findUserByUsername(String username);
}
```

We'll also add the following to the `application.properties` file, so we can
connect to our `security_demo` database:

```properties
spring.datasource.url= jdbc:postgresql://localhost:5432/security_demo
spring.datasource.username= postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate ddl auto (create, create-drop, validate, update)
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

Now for the `UserService` class, we want to implement the `UserDetailsService`
that we used in the last example. This will allow our service to become an
implementation of the `UserDetailsService` that the Authentication Provider will
call upon to check the user details.

```java
package com.example.springsecuritydemo.service;

import com.example.springsecuritydemo.entity.User;
import com.example.springsecuritydemo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
public class UserService implements UserDetailsService {

    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<User> user = userRepository.findUserByUsername(username);

        // How do we turn our Optional<User> into UserDetails?
    }
}
```

Let's look at the code above.

We'll have the `UserService` implement the `UserDetailsService`, which means we
will need to override the `loadUserByUsername()` method. In the method, we'll
call the `findUserByUsername()` method we created in our repository class. This
will return an `Optional<User>`.

But uh-oh. We aren't supposed to return an `Optional<User>` or even just the
`User` entity. We need to return a `UserDetails` instance. So how do we do that?

Let's create a wrapper class that encompasses a `User` instance and implements
the `UserDetails` interface. Within the `entity` package, create a
`UserWrapper.java` file that implements the `UserDetails` interface. We'll also
have to override several methods. Consider the following implementation:

```java
package com.example.springsecuritydemo.entity;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.util.Collection;
import java.util.List;

public class UserWrapper implements UserDetails {

  private final User user;

  public UserWrapper(User user) {
    this.user = user;
  }

  @Override
  public String getUsername() {
    return user.getUsername();
  }

  @Override
  public String getPassword() {
    // We'll need to encode the user's password before we return it
    return new BCryptPasswordEncoder().encode(user.getPassword());
  }

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return List.of(() -> "read");
  }

  @Override
  public boolean isAccountNonExpired() {
    return true;
  }

  @Override
  public boolean isAccountNonLocked() {
    return true;
  }

  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }

  @Override
  public boolean isEnabled() {
    return true;
  }
}
```

Let's break down this code a little bit more:

- Create a private final `User` instance that will be passed into a
  `UserWrapper` constructor.
- Override the `getUsername()` method by returning the user's username:
  `return user.getUsername();`
- Override the `getPassword()` method by returning the encoded user's password:
  `return new BCryptPasswordEncoder().encode(user.getPassword());`
- Override the `getAuthorities()` method by returning a list with an authority
  of "read". We'll elaborate more on this in the next lesson.
- Have the `isAccountNonExpired()`, `isAccountNonLocked()`,
  `isCredentialsNonExpired()`, and `isEnabled()` methods all return true.
  - These methods have to do with looking to see if the user is locked out,
    expired, and enabled. For our purposes, we'll be leaving these set to true
    so we won't have to worry about expired or locked credentials.

A question we might still have is "why does the password need to be encoded
here?" Since Spring requires the password to be protected and secured, and we
are using the `BCryptPasswordEncoder` in the `SecurityConfiguration` class, we
must be consistent in how we are returning the password. Notice in our database,
the password is currently being stored as plain text. This is obviously bad, but
we will cover password hashing in another lesson later on. In the meantime, if
we were to just return `user.getPassword()` in the `UserWrapper` class, we might
run into a warning like this:

`WARN 23827 --- [nio-8080-exec-2] o.s.s.c.bcrypt.BCryptPasswordEncoder     : Encoded password does not look like BCrypt`

To have the `UserDetailsService` return an "encoded" password to the
authentication provider and authentication manager, we need to use the
`BCryptPasswordEncoder` here in the `UserWrapper` class like so:
`return new BCryptPasswordEncoder().encode(user.getPassword());`

We'll now go back to the `UserService` class and finish overriding the
`loadUserByUsername()` method:

```java
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<User> user = userRepository.findUserByUsername(username);

        return user.map(UserWrapper::new).orElseThrow(() -> new UsernameNotFoundException("Username not found"));
    }
```

We're almost ready to run our application! Let's go back into our
`SecurityConfiguration` class and remove the `userDetailsService()` bean
method:

```java
package com.example.springsecuritydemo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class SecurityConfiguration {

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Now run the application! We should be able to enter the same credentials as
before and have it still work - except this time we're making use of our
database!

![basic-auth-credentials](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/postman-basic-auth-mary-test-credentials.png)

## Specifying Authentication on Certain Endpoints

Let's consider that we may not want all our endpoints to require a user to be
authenticated. With our current configuration any endpoint that we create will
require the user to be authenticated. Let's validate this by adding a new
endpoint in our `DemoController` class:

```java
    @GetMapping("/status")
    public String status() {
        return "Congratulations - you must be an admin since you can see the application's status information";
    }
```

As we can imagine, the status of our application shouldn't be something that
all users can access. Sure enough, based on our current configuration, this
endpoint will be protected just like our `/hello` endpoint. Go ahead and restart
the application to validate this in Postman by making a new GET request to
<http://localhost:8080/status>.

Our issue now is that we don't actually want the `/hello` endpoint to be
protected anymore. That's something we did for testing, but in reality we
actually want any user who interfaces with our API to be able to get our basic
greeting message.

To specify that we want the `/hello` endpoint to be public, or not
authenticated, and the `/status` endpoint to be protected, we can create a new
bean method to return a `SecurityFilterChain`. Add the following method to the
`SecurityConfiguration` class:

```java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.httpBasic().and().authorizeRequests().antMatchers("/hello").permitAll();
        httpSecurity.httpBasic().and().authorizeRequests().anyRequest().authenticated();
        return httpSecurity.build();
    }
```

So what is this code doing?

- We first specify that if the request coming through matches the endpoint
  `/hello` to always permit it without having to be authenticated. We accomplish
  this in the line:
  `httpSecurity.httpBasic().and().authorizeRequests().antMatchers("/hello").permitAll();`
- Next, we force all other requests coming through to be authenticated with the
  line:
  `httpSecurity.httpBasic().and().authorizeRequests().anyRequest().authenticated();`
- Notice the difference between these two lines of code. The `permitAll()`
  method will allow requests to pass through while the `authenticated()` method
  forces the request to be authenticated.
- The `anyRequest()` method is generic to say we want all requests whereas the
  `antMatchers()` method takes in a specific pattern that the request must
  match. The name `antMatchers()` may look like a typo, but really it has been
  borrowed from the name Apache Ant to look for Ant-style path patterns.
  `antMatchers()` can match:
  - A specific URL path.
  - A specific HTTP method (GET, POST, etc.).
  - A specific URL path and a specific HTTP method.
  - A path pattern in place of a specific path, wherever a specific path could
    be used in the previous points. The following are the rules for the
    patterns:
    - A "?" matches a single character.
    - A single "\*" matches 0 or more characters.
    - A double "\**", i.e. "\*\*", matches 0 or more paths.
- `antMatchers` can be chained and should be ordered from most specific to least
  specific, which is why in our example, the `/hello` URL is open while all
  other URLs are protected and require authentication. Even though the `/hello`
  URL does match the more general pattern, since there is another rule that is
  more specific and precedes the general rule, it is matched first.

Let's test this out! Restart the application and open up Postman.

We'll first test out the case when the user is _not_ authenticated. We should be
able to hit the `/hello` endpoint but not the `/status` endpoint. To remove the
authentication, navigate to the "Authorization" tab in Postman and next to
"Type" select "No Auth" from the dropdown menu.

In the request URL, let's send a GET request:
"http://localhost:8080/hello?name=mary". Click "Send" to send the request. We
should see that without any authentication, we can reach the endpoint and get
back a message saying "Hello mary".

![postmnan-no-auth](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/postman-no-authentication.png)

Now let's change the GET request URL to "http://localhost:8080/status" to see if
the `/status` endpoint is being blocked since we aren't authenticated:

![postman-401-unauthorized](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/postman-unauthorized.png)

It looks like it worked!

Let's add our basic authentication back in the "Authorization" tab and ensure
we can hit the status endpoint.

![postman-authenticated](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/postman-authenticated-status.png)

## Code Check

Check the project structure and code in each class to ensure your code matches
what was covered in this lesson.

### Project Structure

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springsecuritydemo
    │   │               ├── SpringSecurityDemoApplication.java
    │   │               ├── config
    │   │               │   └── SecurityConfiguration.java
    │   │               ├── controller
    │   │               │   └── DemoController.java
    │   │               ├── entity
    │   │               │   ├── User.java
    │   │               │   └── UserWrapper.java
    │   │               ├── repository
    │   │               │   └── UserRepository.java
    │   │               └── service
    │   │                   └── UserService.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springsecuritydemo
                        └── SpringSecurityDemoApplicationTests.java
```

### SecurityConfiguration.java

```java
package com.example.springsecuritydemo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.httpBasic().and().authorizeRequests().antMatchers("/hello").permitAll();
        httpSecurity.httpBasic().and().authorizeRequests().anyRequest().authenticated();
        return httpSecurity.build();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### DemoController.java

```java
package com.example.springsecuritydemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {

    @GetMapping("/hello")
    public String hello(@RequestParam(defaultValue = "person") String name) {
        return String.format("Hello %s", name);
    }

    @GetMapping("/status")
    public String status() {
        return "Congratulations - you must be an admin since you can see the application's status information";
    }
}
```

### Users.java

```java
package com.example.springsecuritydemo.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private int id;

    private String username;

    private String password;
}
```

### UserWrapper.java

```java
package com.example.springsecuritydemo.entity;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.util.Collection;
import java.util.List;

public class UserWrapper implements UserDetails {

    private final User user;

    public UserWrapper(User user) {
        this.user = user;
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public String getPassword() {
        // We'll need to encode the user's password before we return it
        return new BCryptPasswordEncoder().encode(user.getPassword());
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> "read");
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

### UserRepository.java

```java
package com.example.springsecuritydemo.repository;

// Make sure you import the correct User class
import com.example.springsecuritydemo.entity.User;
import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface UserRepository extends CrudRepository<User, Integer> {

    Optional<User> findUserByUsername(String username);
}
```

### UserService.java

```java
package com.example.springsecuritydemo.service;

import com.example.springsecuritydemo.entity.User;
import com.example.springsecuritydemo.entity.UserWrapper;
import com.example.springsecuritydemo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
public class UserService implements UserDetailsService {

    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<User> user = userRepository.findUserByUsername(username);

        return user.map(UserWrapper::new).orElseThrow(() -> new UsernameNotFoundException("Username not found"));
    }
}
```

## Conclusion

We have learned how to set up authentication in this lesson. In the next one, we
will go over authorization.

## References

- [Spring Security in Action](https://learning.oreilly.com/library/view/spring-security-in/9781617297731/OEBPS/Text/02.htm#heading_id_7)
- [UserDetailsService Documentation](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetailsService.html)
- [InMemoryUserDetailsManager Documentation](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/provisioning/InMemoryUserDetailsManager.html)
- [PasswordEncoder Documentation](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/password-encoder.html)
- [Spring Security Fundamentals - Lesson 2 - Managing Users](https://youtu.be/dFvbHZ8CuKM)
- [Spring Security: Authentication with a Database-backed UserDetailsService](https://www.baeldung.com/spring-security-authentication-with-a-database)
- [Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)
- [HttpSecurity Documentation](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)
- [Spring Security - security none, filters none, access permitAll](https://www.baeldung.com/security-none-filters-none-access-permitAll)
