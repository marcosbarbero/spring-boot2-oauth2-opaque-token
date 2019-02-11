Resource Server Opaque Token - RemoteTokenServices
---

The resource server hosts the [HTTP resources](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web) 
in which can be a document a photo or something else, in our case it will be a REST API protected by OAuth2.

## Dependencies

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
           
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth.boot</groupId>
            <artifactId>spring-security-oauth2-autoconfigure</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>                
   </dependencies>
```

## Defining our protected API

The code bellow defines the endpoint `/me` and returns the `Principal` object and it requires the authenticated 
user to have the `ROLE_USER` to access. 

```java
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.security.Principal;

@RestController
@RequestMapping("/me")
public class UserController {

    @GetMapping
    @PreAuthorize("hasRole('ROLE_USER')")
    public ResponseEntity<Principal> get(final Principal principal) {
        return ResponseEntity.ok(principal);
    }

}
```

The `@PreAuthorize` annotation validates whether the user has the given role prior to execute the code, to make it work
it's necessary to enable the `prePost` annotations, to do so add the following class:

```java
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;

@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfiguration {

}
```

The important part here is the `@EnableGlobalMethodSecurity(prePostEnabled = true)` annotation, the `prePostEnabled` flag
is set to `false` by default, turning it to `true` makes the `@PreAuthorize` annotation to work. 

## Resource Server Configuration

Now let's add the Spring's configuration for the resource server.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;

@Configuration
@EnableResourceServer
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {

}
```

The `@EnableResourceServer` annotation, from the javadoc:

>Convenient annotation for OAuth2 Resource Servers, enabling a Spring Security filter that authenticates requests via
>an incoming OAuth2 token. Users should add this annotation and provide a <code>@Bean</code> of type
>{@link ResourceServerConfigurer} (e.g. via {@link ResourceServerConfigurerAdapter}) that specifies the details of the
>resource (URL paths and resource id). In order to use this filter you must {@link EnableWebSecurity}
>somewhere in your application, either in the same place as you use this annotation, or somewhere else.

Now that we have all the necessary code in place we need to configure a [RemoteTokenServices](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/token/RemoteTokenServices.html),
lucky for us Spring provides a configuration property where we can set the url where the tokens can be translated to
an `Authentication` object. 

```yaml
security:
  oauth2:
    resource:
      user-info-uri: http://localhost:9001/profile/me
```