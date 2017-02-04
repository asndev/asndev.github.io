---
layout: post
title: "JWT Authentication with Spring Boot"
description: "A super simple rest api using JWT for authentication"
tags: [dev, spring, java, jwt]
---

## JWT with Spring?

As I'm using node for nearly all of my current projects I wanted to find out how other technology stacks are doing jwt auth within a rest api. This time I tried to find out how easy it is to do it with spring and spring-boot.

Why spring? It's a lightweight framework for the EE platform, it makes heavy use of IoC which makes it (often) real fun and with Spring Boot I'm able to create a single jar file which makes the deployment process a breeze.

After doing some research I did find the [jjwt.jsonwebtoken](https://github.com/jwtk/jjwt) package, wich is very similar to the [jwt-simple](https://www.npmjs.com/package/jwt-simple) npm package (awesome!).

It provides an easy to use fluent builder to create and parse webtokens.

Exmaple:
{% highlight java %}
String jwtToken = Jwts.builder()
  .setSubject("admin")
  .setIssuedAt(new Date())
  .signWith(SignatureAlgorithm.HS512, "supersecret")
  .compact();

// later
JWT.parser()
  .setSigningKey("supersecret")
  .parseClaimsJws(jwtToken)
  .getBody()
  .getSubject(); // == admin
{% endhighlight %}

## How to get started?

### Install dependencies

I've done it with maven but feel free to do it with gradle or with your favorite package manager.

- create a pom file and add the default spring boot dependencies
  - see [spring boot quick start](https://projects.spring.io/spring-boot/#quick-start)
- also add the jwt dependencie
  - see [installation](https://github.com/jwtk/jjwt#installation)
- install the dependencies with 'mvn install'

You can find my final pom file here: [pom.xml](https://github.com/asndev/springboot-jwt-rest/blob/master/pom.xml)

### Authentication with JWT

In our REST api we want to intercept all requests which try to access the api route, eg '/api/*', and check whether a valid token is present. (similar to a middleware in Express)

For this we'll create a `JwtFilterBean` which will intercept those requests, and a `JwtFilter` which will do the actual parsing and validating.

#### Filter Bean
{% highlight java %}
@Component
public class JwtFilterBean {
    // This bean will intercept all requests matching
    // the "/api/*" url pattern and apply the JwtFilter to it.
    @Bean
    public FilterRegistrationBean jwtFilter() {
        final FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        // Set the filter on this bean
        registrationBean.setFilter(new JwtFilter("supersecret"));
        // Set the relevant URL pattern
        registrationBean.addUrlPatterns("/api/*");
        return registrationBean;
    }
}
{% endhighlight %}

#### Filter
{% highlight java %}
public class JwtFilter extends GenericFilterBean {

    public static final String AUTHORIZATION = "Authorization";
    private final String secret;

    public JwtFilter(String secret) {
        this.secret = secret;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) servletRequest;

        // We get the Authorization Header of the incoming request
        String authHeader = req.getHeader(AUTHORIZATION);

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            throw new ServletException("Not a valid authentication authHeader");
        }

        // and retrieve the token
        String compactJws = authHeader.substring(7);

        try {
            Claims token = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(compactJws)
                    .getBody();

            // Here we can extract different properties of the request,
            // such that the user or the issuedAt and make sure that the token
            // is still valid and that the user exists in our user db

            servletRequest.setAttribute("token", token);
        } catch (SignatureException ex) {
            throw new ServletException("Invalid Token");
        } catch (MalformedJwtException ex) {
            throw new ServletException("JWT is malformed");
        }

        filterChain.doFilter(servletRequest, servletResponse);
    }
}
{% endhighlight %}


### That's it

That's it! Now every request which maps to a particular route ("/api/*") will be handled via our filter to make sure that only valid tokens get delegated to the api controllers.

Now we could enrich the servletRequest by extracting different properties from the token.

- extract the subject, find the appropriate user in our database and add it to the request
  - now each api controller has a valid user in its scope
- make sure that the issuedAt is within a predefined range
  - token could be only valid for 24 hours


### Final code

You can find my example code at my [github repository](https://github.com/asndev/springboot-jwt-rest) where I just took the spring-boot quickstart and enriched it with JWT authenticaitons.
