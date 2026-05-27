---
draft: true
---

For an internal application in our company we had a special usecase concerning the user management. The authentication should be done using the OIDC from Microsoft, but the authorization should be handled in the application itself.  
In other words, everyone that is able to login using Microsoft and is assigned to our company should be able to login to the application. The role management, who is admin, who is user etc. should be handled in the application itself.

## Why
We knew it would have been possible to do the role management in the OIDC provider, but chose an setup like this, because it would be easier for the person responsible for the application to do the user and role management.  
In fact our solution also allows it to create roles in the OIDC provider, because it does not change the ones given. The "local" role management just comes on top.

## What
We were using [Spring](https://spring.io/) to develop the application and wanted to use as much as possible provided by the framework. Sticking to the defaults usually makes code better maintainable and readable for everyone involved.  
That is why for authorization we wanted to stick to the [expression based access control](https://docs.spring.io/spring-security/reference/6.0/servlet/authorization/expression-based.html) provided by Spring Security.  
This meant our solution should be as minimally intrusive as possible and ideally should use given functionality.

## Background
To connect to the OIDC provider we used the [Spring OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html) library.

To connect to the server we set the property `spring.security.oauth2.resourceserver.jwt.issuer-uri`.

TODO mention used libraries and properties to set
TODO add user and role entities and repositories

## Idea

Write about how to use this class:

``` java
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtGrantedAuthoritiesConverter;

public class CustomJwtGrantedAuthoritiesConverter
    implements Converter<Jwt, Collection<GrantedAuthority>> {
  protected static final String CLAIM_USER_ID = "oid";
  protected static final String CLAIM_USER_NAME = "name";
  protected static final String CLAIM_USER_EMAIL = "preferred_username";

  private UserRepository userRepository;
  private UserRoleRepository userRoleRepository;

...

  @Override
  public Collection<GrantedAuthority> convert(Jwt jwt) {
    JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();
    converter.setAuthorityPrefix("");
    Collection<GrantedAuthority> oldAuthorities = converter.convert(jwt);

    String userId = jwt.getClaimAsString(CLAIM_USER_ID);
    if (userId == null) {
      return oldAuthorities;
    }

    String userName = jwt.getClaimAsString(CLAIM_USER_NAME);
    String userEmail = jwt.getClaimAsString(CLAIM_USER_EMAIL);
    Optional<User> user = userRepository.findById(userId);
    if (user.isEmpty()) {
      createNewUser(userId, userName, userEmail);
      return oldAuthorities;
    }
    updateUserIfNecessary(user.get(), userName, userEmail);

    List<GrantedAuthority> grantedAuthorities = new ArrayList<>(oldAuthorities);
    grantedAuthorities.addAll(userRoleRepository.findByIdUser(user.get()));
    return grantedAuthorities;
  }

  private void createNewUser(String userId, @Nullable String userName, @Nullable String userEmail) {
    if (userName != null && userEmail != null) {
      User newUser = new User(userId, userName, userEmail, true);
      userRepository.save(newUser);
    }
  }

  private void updateUserIfNecessary(
      User user, @Nullable String userName, @Nullable String userEmail) {
    if ((userName != null && !userName.equals(user.getName()))
        || (userEmail != null && !userEmail.equals(user.getEmail()))) {
      user.setName(userName);
      user.setEmail(userEmail);
      userRepository.save(user);
    }
  }
}
```



And how it is used:

``` java
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;

...

  @Bean
  public JwtAuthenticationConverter jwtAuthenticationConverter(
      UserRepository userRepository, UserRoleRepository userRoleRepository) {
    JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
    converter.setJwtGrantedAuthoritiesConverter(
        new CustomJwtGrantedAuthoritiesConverter(userRepository, userRoleRepository));
    return converter;
  }

...
```


With this solution there is initially no administrator for this application. To achieve it you have to login once and add the administrator role to the created user by logging into the database and firing an SQL query. An alternative solution would be to set the authority in the OIDC provider.