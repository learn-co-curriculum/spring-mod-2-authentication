# Authentication

## Learning Goals

- Explain DispatcherServlet.
- Set up authentication.

## Introduction

Before we dive into the specifics of how to configure security for a Spring Boot
application, we need to discuss servlets and how they are the underpinning of
the web functionality that the Spring Framework provides.

Afterwards, we will set up an authentication system for the project.

## DispatcherServlet, servlet filters and servlet chains

As you have learned, the Spring Framework is a Dependency Injection (DI) and
Aspect Oriented Programming (AOP) framework, and has many components built
around it that take advantage of its DI and AOP capabilities.

One such component is Spring MVC, which takes advantage of the
`DispatcherServlet` to redirect `HTTP` requests to the components that you have
marked with the `@Controller` or `@RestController` annotations. A "servlet" in
Java is a special class that can listen to incoming web connections, understand
the corresponding `HTTP` protocol, read the request parameters and knows how to
write a response back to the original caller.

Just as the Spring Framework's AOP support allows the Spring Framework to proxy
your service, Java supports "filters" for servlets. Servlet filters are the
ability to implement an interceptor for incoming `HTTP` requests that will see
every single request to your servlet before it actually reaches your servlet.
This interceptor will have the ability to examine your request, perform logic
and then decide whether to forward the original request to your servlet.

Not only that, but Java can actually implement servlet filter "chains", which
means that instead of handing the incoming request to your servlet after it's
done with it, a filter can actually hand the incoming request to another filter.
This is a key aspect of filters because it provides the foundation for being
able to have dedicated filters for each type of "request filtering" we might
want to implement. For example, we might want to:

1. Examine an incoming request to validate authentication information (are you
   who you claim to be)
2. Examine an incoming request to validate authorization informatin (are you
   allowed to perform the action you're trying to perform)

As we will see in this section and the next, these two concerns are quite
different and should not be handled by the same code.

### Spring `DefaultSecurityFilterChain`

Now that you have enabled Spring Security in your application, your startup log
should have a line with information about the `DefaultSecurityFilterChain`,
which will look something like this (the exact output may look slightly
different based on the exact version of the Spring Framework you're running):

```java
2022-07-05 02:18:20.592  INFO 96327 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.session.DisableEncodeUrlFilter@4b6e1c0, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@561b61ed, org.springframework.security.web.context.SecurityContextPersistenceFilter@1907874b, org.springframework.security.web.header.HeaderWriterFilter@1292071f, org.springframework.security.web.csrf.CsrfFilter@1aabf50d, org.springframework.security.web.authentication.logout.LogoutFilter@1b1f5012, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@5b3a7ef5, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@5d21202d, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@654c7d2d, org.springframework.security.web.session.SessionManagementFilter@5b9396d3, org.springframework.security.web.access.ExceptionTranslationFilter@1a4d1ab7, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5bc28f40]
```

This is one of those lines of text that's not very easy to read in a single
line, so let's break it up into multiple lines:

```java
2022-07-05 02:18:20.592  INFO 96327 --- [           main] o.s.s.web.DefaultSecurityFilterChain     :
        Will secure any request with
        [org.springframework.security.web.session.DisableEncodeUrlFilter@4b6e1c0,
        org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@561b61ed,
        org.springframework.security.web.context.SecurityContextPersistenceFilter@1907874b,
        org.springframework.security.web.header.HeaderWriterFilter@1292071f,
        org.springframework.security.web.csrf.CsrfFilter@1aabf50d,
        org.springframework.security.web.authentication.logout.LogoutFilter@1b1f5012,
        org.springframework.security.web.savedrequest.RequestCacheAwareFilter@5b3a7ef5,
        org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@5d21202d,
        org.springframework.security.web.authentication.AnonymousAuthenticationFilter@654c7d2d,
        org.springframework.security.web.session.SessionManagementFilter@5b9396d3,
        org.springframework.security.web.access.ExceptionTranslationFilter@1a4d1ab7,
        org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5bc28f40]
```

What the Spring Framework is telling you is that because you have enabled Spring
Security, it has now configured these default servlet filters and that they are
all chained together, which means you can choose to take advantage of any
combination of them.

Let's look at some of the most commonly used filters in this chain:

- BasicAuthenticationFilter: tells Spring Security to look for a
  `Basic Auth HTTP` header on the request and extract the username and password
  to authenticate the user.
- UsernamePasswordAuthenticationFilter: tells Spring Security to look for the
  username and password in the request parameter or `POST` body and use them to
  authenticate the user.
- DefaultLoginPageGeneratingFilter: tells Spring Security to generate a default
  login page.
- DefaultLogoutPageGeneratingFilter: tells Spring Security to generate a default
  logout page.
- FilterSecurityInterceptor: tells Spring Security to implement a default
  Authorization mechanism

Let's look at how to configure these filters.

## Spring Security Authentication

Let's go ahead and add our `configure()` method to the `SecurityConfiguration`
class again and give it a very basic implementation:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http.authorizeRequests()
              .anyRequest().permitAll();
    }
}
```

In this basic configuration, we are using the `HttpSecurity` object to call its
`authorizeRequests()` method and tell it to `permitAll()` for any request that
comes to our application.

This current setup, of course, is not very useful, since we're simply allowing
any incoming request to go through. We are going to come up with some
incrementally more customized security options in the next few steps.

First, let's explicitly force all requests be authenticated:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin();
    }
}
```

Let's put the 2 versions of the call to `authorizeRequests()` side by side:

- http.authorizeRequests().anyRequest().permitAll();
- http.authorizeRequests().anyRequest().authenticated().and().formLogin();

Let's examine the differences:

- As you can see, we are applying our new rule to the same types of requests as
  before by calling `anyRequest()`. Later in this section, we will look at how
  to apply our rules to a subset of requests.
- But now instead of asking any request to have the `permitAll()` method apply
  to it, we are saying that any request actually needs to be `authenticated()`
  in order to proceed
- The `and()` after the call to `authenticated()` allows us to chain together
  all the calls we need in order to configure our Spring Security, rather than
  having to assign the returns from each method to a variable and then calling
  additional methods on those variables
- The next instruction we give to Spring Security is to enable `formLogin()` as
  the authentication method.

These changes get us back to where we are now challenged with Spring's default
login page when we try to access our `hello` endpoint:

![Spring Framework default simple authentication](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-simple-auth.png)

When Spring is configured with default security, the default user it sets up is
`user` and the password is actually displayed on the console of your application
when it starts up. You should see an ouput similar to this:

```java
2022-07-02 11:37:40.962  WARN 13313 --- [           main] .s.s.UserDetailsServiceAutoConfiguration :

Using generated security password: b5de1159-3225-4224-9375-51098c6929a9

This generated password is for development use only. Your security configuration must be updated before running your application in production.
```

Use those credentials on the login screen you see, and you should now be able to
access your `hello` endpoint.

Now that we've configured some default security and that we've been able to
manually test that it works as expected, let's make sure that our automated
tests are running properly. Let's run our entire test suite:

![Failed Integration Tests](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-security-integration-tests-fail.png)

There are a couple of interesting results that highlight the value of separating
unit tests from integration and acceptance tests:

1. All our unit tests continue to pass: this is expected as those tests are not
   integrated with the Spring Framework and instead focus on the narrow
   functionality of specific methods. These are correctly not impacted by the
   general authentication concerns of the overall application.
2. Our integration and acceptance tests fail: this is also expected, as we have
   now changed how requests are allowed to get to our endpoint. Since both our
   integration and acceptance tests do submit a request to our endpoint, they
   now must also provide authentication credentials in order to be allowed to
   proceed.

Thankfully, the Spring Framework provides ways to handle authentication in our
tests.

First, we have to add a new dependency to our project to get access to some
useful annotations for security testing:

- If using Gradle, add the following dependency:

```json
  testImplementation 'org.springframework.security:spring-security-test'
```

- If using Maven, add the following dependency:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

The easiest way to let our `mockMvc` object know that credentials should be
provided with each request is to use the `@MockUser` annotation. This tells
Spring to create a mock user object and pass its credentials information along
with each request that is performed on the `mockMvc` object:

```java
package com.flatiron.spring.FlatironSpring;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class HelloControllerAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    @WithMockUser
    @Test
    void shouldGreetDefault() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello Stephanie")));
    }

    @WithMockUser
    @Test
    void shouldGreetByName() throws Exception {
        String greetingName = "Jamie";
        mockMvc.perform(get("/hello")
                        .param("targetName", greetingName))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello " + greetingName)));
    }
}
```

You should now be able to run the acceptance test and have it pass. Make the
same changes to your integration test and it will also pass:

```java
package com.flatiron.spring.FlatironSpring;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(HelloController.class)
class HelloControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private JokeService jokeService;

    @WithMockUser
    @Test
    void shouldGreetDefault() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello Stephanie")));
    }

    @WithMockUser
    @Test
    void shouldGreetByName() throws Exception {
        String greetingName = "Jamie";
        mockMvc.perform(get("/hello")
                .param("targetName", greetingName))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello " + greetingName)));
    }
}
```

It's important to understand that the `@WithMockUser` annotation creates an
authenticated user and adds it to the security context of `mockMvc`. In other
words, the "mock user" doesn't actually go through the security chain to have
its credentials validated - instead Spring injects an "authenticated" version of
that user in the security context _as if_ the user had gotten authenticated.

To drive that distinction home, we can actually give the `MockUser` a username
of a user that doesn't exist and see that our integration test will still pass:

```java
    // ...

    @WithMockUser(username = "fakeuser")
    @Test
    void shouldGreetDefault() throws Exception {

    // ...
```

Now that we have all our tests running and passing, let's look at some ways to
customize our default security configuration.

## Adding our own users

So far, we have only used the built-in default `user` user generated for us by
the Spring Framework.

We can, however, get users from any source we want by creating a custom
`UserDetailsService` that can either return users it creates itself, or use an
external data source to fetch the users from a database or another service.

Let's add a new method to our `SecurityConfiguration` class:

```java
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager userDetailService = new InMemoryUserDetailsManager();

        UserDetails user1 = User.withUsername("mary")
                .password(passwordEncoder().encode("test"))
                .authorities("read")
                .build();
        userDetailService.createUser(user1);

        return userDetailService;
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

```

Let's break it down:

1. We create a user details service that can manage all the users we will add to
   it "in memory" - this means these users will not be saved to file or in a
   database anywhere, and will cease to exist when the application is stopped.
   This would not work for a production-level application, but it is a good way
   to demonstrate the mechanism of `UserDetailsService` without having to worry
   about persistence or service integration.
2. We create a test user using the `User` class
3. We give that user a username and a password. Note that in Spring 5, we are
   required to encode the password, so we use a standard encoder's `encode()`
   method.
4. Even though we have not covered authorization yet, we do have to give our
   user a default "authority" - we will explain what that means in a later
   section.
5. We add our new user to the user details service and return the user details
   service.

When you restart your application, you will see that you now need to
authenticate using the username/password combination that we defined in our
custom `UserDetailsService`.

## Finer grain access control

Let's consider that we may not want all our endpoints to require a user to be
authenticated. With our current configuration any endpoint that we create will
require the user to be authenticated. Let's validate this by adding a new
endpoint in our `HelloController` class:

```java
    @GetMapping("/status")
    public String status() {
        return "Congratulations - you must be an admin since you can see the application's status information";
    }
```

As you can imagine, the status of our application shouldn't be something that
all users can access. Sure enough, based on our current configuration, this
endpoint will be protected just like our `/hello` endpoint. Go ahead and restart
your application to validate this.

Our issue now is that we don't actually want the `/hello` endpoint to be
protected anymore. That's something we did for testing, but in reality we
actually want any user who interfaces with our API to be able to get our basic
greeting message.

We can do this be modifying our `configure()` method in our
`SecurityConfiguration` class:

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/hello")
                .permitAll();

        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin();

    }
```

We have now added a new entry for a specific URL (`/hello`), using the
`antMatchers()` method, which works this way:

1. `antMatchers()` can match:
   1. A specific URL path
   2. A specific `HTTP` method (`GET`, `POST`, etc...)
   3. A specific URL path and a specific `HTTP` method
   4. A path pattern in place of a specific path, wherever a specific path could
      be used in the previous points. The following are the rules for the
      patterns:
      1. A "?" matches a single character
      2. A single "\*" matches 0 or more characters
      3. A double "\*", i.e. "\*\*", matches 0 or more paths

`antMatchers` can be chained and should be ordered from most specific to least
specific, which is why in our example the `/hello` URL is open while all other
URLs are behind the authentication form. Even though the `/hello` URL does match
the more general pattern, since there is another rule that is more specific and
precedes the more general rule, it is matched first and the `/hello` URL is
open.

## Logout

In the same way that Spring Framework can provide us with a standard login form,
it can also provide us with a standard logout form that logs the user out of the
system. This is handy for us to have as we continue to test different scenarios
and add more than one user.

We need to request a logout form in much the same way we requested a login form,
which we can do by adding to the `formLogin()` chain we already have:

```java
        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .and() // add to the chain
                .logout(); // request a logout form
```

You can now go to your logout form by going to the `/logout` URL on your
`localhost`.

## Conclusion

We have learned how to set up authentication in this lesson. In the next one, we
will go over authorization.
