Steps to setup OAuth2 server:

Add the below dependencies to the project

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>

Setup the application name in bootstrap.yaml file

spring:
  application:
    name: oauthserver

Configure the application.yaml file to register with Eureka server and set the server port to 8081

eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      default-zone: http://localhost:8761/eureka

server:
  port: 8081

Add the below annotations in the Root Configuration file

@EnableResourceServer   // this is a protected resource
@EnableAuthorizationServer  //acts as a OAuth2 service


Create a OAuath2Config class and extend AuthorizationServerConfigurerAdapter


Override the method to setup the clientDetails and AuthorizationServrEndpoint


@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients.inMemory()
            .withClient("my-awesome-app")
            .secret("{noop}welcome")
            .authorizedGrantTypes("refresh_token", "password", "client_credentials")
            .scopes("webclient", "mobile");
}

 @Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    endpoints
            .authenticationManager(authenticationManager)
            .userDetailsService(this.userDetailsService);
}


Create configuration class and extends WebSecurityConfigurerAdapter


Override the configure method and setup the AuthenticationManager


Configure to return both authenticationManager and userDetailsService bean


Once the token is generated, to validate the token, create a Controller class



Add a GetMapping with /user

Accept OAuth2Authentication parameter
Extract the principal and authorities
Object principal = user.getUserAuthentication().getPrincipal();
Collection<? extends GrantedAuthority> grantedAuthorities = user.getUserAuthentication().getAuthorities();
     Set<String> authorities = AuthorityUtils.authorityListToSet(grantedAuthorities);


Return the user details

Map<String, Object> userDetails = new HashMap<>();
       userDetails.put("username", principal);
       userDetails.put("authorities", authorities);
       return userDetails;

Check the user information by making a GET request

GET http://localhost:8081/user -- headers Authorization Bearer <access_token>
