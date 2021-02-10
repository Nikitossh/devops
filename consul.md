#Consul. 

### Why we need it?
1. Centralized place for configs as K/V store
2. Refresh config values in spring apps without restart app.

### Potential cases for future:
1. Service discovery to join k8s cluster with apps in other places.

### Goals for today:
1. How to enable it as central confuration tool for spring applications.
2. How to work with config refresh.

### Install Consul as HA cluster.
// todo: add link to ansible role

### Spring.
build.gradle
<pre>
dependencies {
  ...
  // Consul
  compile 'org.springframework.cloud:spring-cloud-starter-consul-all:1.3.0.RELEASE'
  ...
}
</pre>
bootstrap.yml
<pre>
spring:
  application:
    name: sblintegration
  cloud:
    consul:
      enabled: true
      discovery:
        enabled: false
        register: false
        health-check-path: /actuator/health
      config:
        prefix: config
        enabled: true
        watch:
          enabled: false
        format: yaml
      host: consul.lorus-labs.com
      port: 8500
</pre>

#### Enable `/actuator/refresh` endpoint. Add to application.yml in Consul
<pre>
management:
  endpoints:
    web:
      exposure:
        include: refresh
</pre>
Useful endpoints and features of actuator here: https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints

#### Refresh scope
```The RefreshScope is a bean in the context and it has a public method refreshAll() to refresh all beans in the scope by clearing the target cache. There is also a refresh(String) method to refresh an individual bean by name. This functionality is exposed in the /refresh endpoint (over HTTP or JMX).```

Add `@RefreshScope` annotation to `@Configuration` class  
https://cloud.spring.io/spring-cloud-static/spring-cloud.html#_refresh_scope

To use refresh for config variables you must send POST request to /refresh endpoint:
<pre>
$ curl -d '{}' -H 'Content-Type: application/json' http://localhost:8080/actuator/refresh
</pre>


