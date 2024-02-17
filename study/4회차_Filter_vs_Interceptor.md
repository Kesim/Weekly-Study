# Filter, Interceptor

Filter와 Interceptor는 요청을 공통적인 기능으로 처리하기 위해 사용한다. 요청을 처리하기 전/후에 필요한 기능을 수행할 수 있다는 점은 서로 유사하지만, 실행 위치와 세부적인 사항 등 차이가 있다. 공통적인 기능을 처리한 다는 점에서는 AOP 와도 유사하지만, 필터와 인터셉터는 그 대상이 컨트롤러에 제한되어 있고 `HttpServletRequest`, `HttpServletResponse` 값을 기본적으로 사용할 수 있어 요청을 처리하는데 더 편리하다.

## Filter

Client ↔ Filter ↔ Dispatcher Servlet ↔ Interceptor ↔ Controller

디스패쳐 서블릿은 스프링 컨텍스트의 최전방에 위치하여 가장 먼저 요청을 받는 프론트 컨트롤러다. 들어온 요청을 확인하여 요청을 처리할 컨트롤러와 매핑해주는 역할을 한다.

디스패쳐 서블릿의 바깥에 있는 필터는 스프링 컨텍스트가 아닌 서블릿 컨텍스트에 속하며 J2EE 표준 스펙의 기능으로 톰캣과 같은 서블릿 컨테이너가 관리한다. 따라서 기본적으로 빈으로 등록하거나 빈을 주입 받을 수 없다. 디스패쳐 서블릿에 요청이 전달되기 전/후에 URL 패턴을 확인하여 공통 로직을 수행한다. HTTP 이 아닌 요청도 처리할 수 있다.

필터는 스프링이 관리하지 않는다고 했지만, 스프링이 제공하는 `DelegatingFilterProxy` 덕분에 빈으로 등록할 수도 있다. 이 경우, 빈으로 등록된 필터를 프록시 형태로 필터 체인에 등록한다.

### Filter의 구현

```java
package javax.servlet;

public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {}

    void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;

    default void destroy() {}
}
```

필터는 `Filter` 인터페이스를 구현한다. `init`, `destory` 메서드는 디폴트 메서드가 정의되어 있어 `doFilter`만 필수적으로 구현하면 된다.

- init - 서블릿 컨테이너가 실행될 때 호출하여 필터 객체를 초기화한다. 초기화 후에는 들어오는 요청을 `doFilter`를 호출하여 처리한다.
- destroy - 서블릿 컨테이너가 종료될 때 호출하여 필터 객체를 제거한다. 이 후 요청이 들어오더라도 `doFilter`로 처리하지 않는다.
- doFilter - 필터에서 실제로 로직을 수행하는 부분이다. 파라미터로 받는 `FilterChain` 객체의 `doFilter(ServletRequest, ServletResponse)`를 호출하여 현재 요청을 다음으로 넘기며, 해당 메서드 호출 전/후로 필터의 로직을 구현한다. 필터가 여러개 등록되어있을 경우 다음 필터가 처리하며, 없을 경우 디스패쳐 서블릿으로 요청을 넘긴다. `FilterChain.doFilter()`를 호출하지 않을 경우 요청을 더 이상 처리하지 않는다.
    - `FilterChain.doFilter()` 는 `ServletRequest`, `ServletResponse` 객체를 전달 받을 수 있어 현재 필터에서 객체의 값을 조작하거나 아예 다른 객체를 전달 할 수 있다.

주의할 점으로, 필터는 서블릿 컨텍스트에 위치하므로 스프링의 예외 처리를 사용할 수 없다. 필터에서 예외가 발생하면 서블릿으로 500 응답으로 넘어가기 때문에 다른 응답을 전달하려면 reponse에 다른 값을 지정해야한다.

### Filter의 용도

- 공통된 보안 및 인증/인가 관련 작업(XSS 방어)
- 모든 요청에 대한 로깅 또는 감사
- 이미지/데이터 압축 및 문자열 인코딩
- Spring과 분리되어야 하는 기능

### Filter의 적용

이전에는 web.xml 에 필터 설정을 하는 방법도 있었으나 생략한다.

1. FilterRegistrationBean 빈으로 등록
    
    ```java
    @Configuration
    public class FilterConfig {
        @Bean
        public FilterRegistrationBean<Filter> logFilter() {
            FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<>();
    
            bean.setFilter(new LogFilter());
            bean.setOrder(1);
            bean.addUrlPatterns("/*");
    
            return bean;
        }
    }
    ```
    
    필터를 빈으로 등록하지 않는다. `setOrder()`로 순서를 지정하며, `addUrlPatterns()`으로 매핑 대상을 지정한다. 필터 설정을 한 설정 클래스 파일 안에서 관리할 수 있다는 장점이 있다.
    
2. @WebFilter, @Component, @order
    
    ```java
    @WebFilter
    @Component
    @Order(1)
    public class LogFilter implements Filter { ... }
    ```
    
    필터를 빈으로 등록한다. `@Order`로 순서를 지정하며, `@WebFilter`의 value 또는 urlPatterns 값에 매핑 대상을 지정한다.
    

## Interceptor

인터셉터는 디스패쳐 서블릿 다음에 위치하여 스프링 컨텍스트가 관리한다. 필터와 달리 빈으로 등록하거나 빈을 주입받을 수 있다. 디스패쳐 서블릿은 `HandlerMapping` 을 통해 요청을 처리할 컨트롤러를 찾아 실행 체인을 반환한다. 실행 체인에 1개 이상의 인터셉터가 있다면 순서대로 인터셉터가 처리한 후 컨트롤러에 요청을 넘긴다.

### Interceptor의 구현

```java
package org.springframework.web.servlet;

public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
```

인터셉터는 `HandlerInterceptor` 인터페이스를 구현한다. 모든 메서드에 디폴트 메서드가 있어 필요한 부분만 직접 구현하여 사용한다.

- preHandle - 컨트롤러에서 요청을 처리 하기 전에 호출된다. 반환 타입이 boolean인데, true를 반환해야 요청이 다음 흐름으로 넘어간다.(다음 인터셉터 또는 컨트롤러). false일 경우 요청을 더 이상 처리하지 않는다. 세 번째 파라미터의 handler은 `HandlerMethod` 타입의 객체로 @RequestMapping과 그 하위 어노테이션(`@GetMapping`, `@PostMapping` 등)이 붙은 메소드의 정보를 추상화한 객체다.
- postHandle - 컨트롤러에서 요청이 정상적으로 처리된 후 호출된다. 파라미터로 `ModelAndView`를 받아 값을 조작할 수 있다.
- afterCompletion - 컨트롤러에서 요청이 정상/에러 반환 후 호출된다.

필터의 doFilter와 달리 `HttpServletRequest`, `HttpServletResponse` 객체 자체를 바꿀 수는 없으나 값 조작은 가능하다.

참고로 Java 8 이전에는 인터페이스에 디폴트 메서드를 정의할 수 없어 `HandlerInterceptorAdapter` 를 상속받아 인터셉터를 구현하였다. 현재는 `HandlerInterceptor` 인터페이스에 모두 정의되어있으므로 바로 해당 인터페이스를 바로 구현하면 되며 `HandlerInterceptorAdapter` 은 deprecated 되었다.

### Interceptor의 용도

- 세부적인 보안 및 인증/인가 공통 작업
- API 호출에 대한 로깅 또는 감사
- Controller로 넘겨주는 정보(데이터)의 가공

### Interceptor의 적용

이전에는 servlet-context.xml에 인터셉터 설정을 하는 방법이 있었으나 생략한다.

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {
    private final LogInterceptor logInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(logInterceptor)
                .addPathPatterns("/*");
		}
}
```

`WebMvcConfigurer` 인터페이스 중 `addInterceptors()` 를 오버라이드 하여 구현한다. 여러 인터셉터를 추가할 경우 순서대로 실행된다. `addPathPatterns()` 또는 `excludePathPatterns()` 등으로 패턴을 지정할 수 있다.

## Spring Security Filter

스프링에서 필터는 일반적인 의미로 사용되는 서블릿 컨텍스트에서 관리하는 서블릿 필터 외에도, 스프링 컨텍스트에서 관리하는 스프링 시큐리티의 시큐리티 필터가 있다.

시큐리티 필터는 FilterProxyChain 에 등록되는데, 이 체인은 빈으로 등록되므로 DelegatingFilterProxy 에 의해 관리되어 필터 체인에 등록된다.

![Untitled](image\filterChain_structure.png)

서블릿 필터와 시큐리티 필터의 순서는 기본적으로 시큐리티 필터 체인이 먼저 수행되고, 그 외의 필터가 수행된다. 이 순서를 바꾸려면, application.yml 파일에 `spring.security.filter.order={숫자}` 로 순서를 지정해야 한다. 

### Security Filter 적용

```java
@Configuration
public class SecurityConfigration {
    @Autowired
    private SecurityUserService service;
	
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		...
		http
        .addFilterBefore(new SimpleSecurityFilter(), AnotherSecurityFilter.class)
		...
    return http.build();
}
```

`HttpSecurity` 파라미터를 주입받아 시큐리티 필터를 등록하여 `SecurityFilterChain` 타입의 빈을 반환한다.

기존에는 시큐리티 필터를 등록하려면 `WebSecurityConfigurerAdapter` 클래스를 상속 받아  `void configure(HttpSecurity)`메서드에 설정했으나, `WebSecurityConfigurerAdapter` 클래스가 Spring Security 5.7 부터 deprecated 되었다.

주의할 점으로, 필터를 빈으로 등록하여 시큐리티 필터 체인에 등록하면 컴포넌트 스캔에 의해 서블릿 필터로 등록되어 필터가 2번 등록되는 현상이 발생할 수 있다. 빈으로 등록할 경우 서블릿 필터에 등록하지 않도록 설정이 필요하다.

```java
@Bean
public FilterRegistrationBean<SimpleSecurityFilter> simpleSecurityFilter(TenantFilter filter) {
    FilterRegistrationBean<SimpleSecurityFilter> registration = new FilterRegistrationBean<>(filter);
    registration.setEnabled(false); // 필터 등록 X
    return registration;
}
```

# 참고

[https://mangkyu.tistory.com/173](https://mangkyu.tistory.com/173)

[https://www.inflearn.com/chats/786858/filter를-등록하는-4가지-방법](https://www.inflearn.com/chats/786858/filter%EB%A5%BC-%EB%93%B1%EB%A1%9D%ED%95%98%EB%8A%94-4%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95)

[https://docs.spring.io/spring-security/reference/servlet/architecture.html](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

[https://12teamtoday.tistory.com/141](https://12teamtoday.tistory.com/141)