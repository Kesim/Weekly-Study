# Spring Security

스프링 시큐리티는 Authentication, Authorization과 일반적인 공격을 방어하는 법을 제공하는 프레임 워크다. 스프링에서 제공하는 스프링 시큐리티 기술은 어플리케이션의 종류(Servlet, Reactive)에 따라 큰 차이가 있으나, 여기서는 자바의 Servlet 기술을 기준으로 설명한다.

# Authentication 인증

인증은 특정한 자원에 접근하는 사람의 신원을 확인하는 방법이다. 일반적인 방법으로는 아이디, 패스워드 방식을 사용한다. 인증을 한 후에 인가 작업을 수행할 수 있다.

모든 요청마다 패스워드를 확인하면 어플리케이션의 성능에 큰 저하가 발생한다. 패스워드 인증으로 얻은 장기 자격 증명을 세션 및 OAuth 토큰 등의 단기 자격 증명으로 교환하면 보안상의 손실 없이 빠르게 검증할 수 있다.

## Password Storage

사용자의 비밀번호를 저장할 때는 다시 복호화 하여 비밀번호를 알아낼 수 없도록 **단방향 알고리즘**으로 암호화 하여 저장한다. 스프링 시큐리티에서는 `PasswordEncoder` 인터페이스의 구현체를 사용하여 비밀번호를 암호화 하거나 서로 일치하는지 확인한다.

사용할 수 있는 구현체로는 `BCryptPasswordEncoder, Argon2PasswordEncoder, Pbkdf2PasswordEncoder, SCryptPasswordEncoder`등이 있으며, 직접 알고리즘을 구현하여 사용할 수 도 있다. 위 구현체들은 의도적으로 많은 자원을 사용하여 비밀번호를 확인할 때 **1초**가 걸리도록 설정하여 사용한다. 공격자가 암호를 유추하기 어렵도록 하기 위함이다.

서비스 기간에 따라 사용하는 알고리즘은 변할 수 있다. `DelegatingPasswordEncoder`을 사용하면 여러 알고리즘을 **맵의 형태로 관리**하여 여러 알고리즘을 사용하여 인증을 할 수 있다. 스프링 **시큐리티에서는 기본적으로** `DelegatingPasswordEncoder`를 사용한다.

사용자의 비밀번호는 자바 단에서 암호화를 하는 것이 아니라, 클라이언트 단에서 암호화된 데이터를 전달해야 더 안전하다. 자바의 디버깅 모드나 로그에서 평문으로된 비밀번호를 확인할 수 없기 때문이다.

### **RequestCache**

아직 인증이 되지 않은 사용자가 인증이 필요한 경로에 접속하면 로그인 페이지로 이동한다. 인증이 성공했을 때 단순히 메인 페이지로 넘어간다면 사용자가 직접 다시 경로에 접속해야하는 불편함이 있다. 이를 해결하기 위해 스프링 시큐리티는 사용자의 요청(`HttpServletRequest`)을 `RequestCache`에 보관한 후, 인증에 성공하면 캐시에 저장된 정보를 읽어 **다시 요청을 진행**한다. `HttpSessionRequestCache`를  `RequestCache`의 기본 구현체로 사용한다.

요청을 캐시에 저장하고 싶지 않다면 `NullRequestCache`를 사용한다.

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
	HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
	// 요청의 query parameter에 특정한 값이 매칭될 경우에만 수행하는 설정
	// requestCache.setMatchingRequestParameterName("continue");
	// 캐시에 저장하지 않을 경우
	// RequestCache nullRequestCache = new NullRequestCache();
	http
		// ...
		.requestCache((cache) -> cache
			.requestCache(requestCache)
		);
	return http.build();
}
```

## Architecture

스프링 시큐리티의 각 요소들의 역할

- `SecurityContextHolder` : 인증된 사람의 상세 정보를 스프링 시큐리티가 보관하는 곳이다.
- `SecurityContext` : `SecurityContextHolder`에서 얻을 수 있으며, 현재 인증된 사용자의 `Authentication`을 포함한다.
- `Authentication` : 인증을 위한 정보를 갖고 있거나, 현재 인증된 사용자를 나타낸다.
- `GrantedAuthority` : `Authentication` 의 식별자에게 부여된 권한. role, scope 등.
- `AuthenticationManager` : 스프링 시큐리티의 필터가 수행하는 방법을 정의한 API.
- `ProviderManager` : `AuthenticationManager`의 가장 일반적인 구현체.
- `AuthenticationProvider` : `ProviderManager`가 특정한 타입의 인증을 수행할 때 사용한다.
- Request Credentials with `AuthenticationEntryPoint` : 클라이언트에게 자격 증명을 요청할 때 사용한다. 로그인 페이지로 리다이렉트 시키거나, WWW-Authenticate 응답을 보낸다.
- `AbstractAuthenticationProcessingFilter` : 인증에 사용되는 기본 필터다. 높은 수준의 인증 흐름과  여러 요소가 협업하는데 좋은 아이디어를 준다.

### SecurityContextHolder

![https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/securitycontextholder.png](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/securitycontextholder.png)

스프링 시큐리티 **인증 모델의 가장 핵심**이다. 스프링 시큐리티는 `SecurityContextHolder`에 데이터가 어떻게 들어오든지 상관하지 않는다. 값을 가지고 있다면, **현재 인증된 사용자**로서 사용된다.

- `SecurityContextHolder`에 인증된 사용자를 저장하는 예시

```java
// 현재 쓰레드의 데이터가 아닌 새로운 데이터 생성
SecurityContext context = SecurityContextHolder.createEmptyContext();
// TestingAuthenticationToken는 간단한 Authentication 인스턴스를 생성
// 실무에서는 주로 UsernamePasswordAuthenticationToken(userDetails, password, authorites)를 사용한다.
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);
// SecurityContextHolder에 SecurityContext를 저장. 이 정보로 인가 작업을 한다.
SecurityContextHolder.setContext(context);
```

- `SecurityContextHolder`에서 인증된 식별자의 정보를 가져오는 예시

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```

기본적으로 `SecurityContextHolder`는 `ThreadLocal`을 이용하여 인증 정보를 보관한다. 따라서 같은 쓰레드 안에서는 각 메서드에서 `SecurityContext` 데이터를 항상 사용할 수 있다. 데이터의 저장 정책은 설정을 통해 변경할 수 있다.

### Authentication

사용자의 인증을 위한 정보는 `AuthenticationManager`의 입력값으로 사용된다. 또한 현재 인증된 사용자를 나타내는데, 현재 `Authentication`값은 `SecurityContext`에서 얻을 수 있다.

`Authentication`은 아래 3가지를 포함한다.

- principal : **사용자를 식별**하는 정보. 비밀번호 방식일 경우 `UserDetails`객체가 주로 사용된다.
- credentials : 사용자가 사용한 **자격 정보**로 비밀번호인 경우가 많다. 대부분의 경우, 인증된 후에는 정보가 유출되지 않도록 삭제된다.
- authorities : `GrantedAuthority`객체로 사용자에게 부여되는 높은 수준의 권한이다. role, scope가 있다.

### GrantedAuthority

`Authentication.getAuthorities()` 메서드를 사용하여 `GrantedAuthority` 객체의 `Collection`을 얻을 수 있다. 이 인가에는 role, scope가 사용될 수 있다. role은 이후에 웹 인가, 메소드 인가, 도메인 객체 인가를 위해 구성된다. 패스워드 기반의 인증에서 `GrantedAuthority` 객체는 주로 `UserDetailsService`에서 가져온다.

`GrantedAutority` 객체는 특정 도메인 객체의 한정된 것(예, 54번 직원 객체의 인가)이 아니라, 어플리케이션 전체의 권한이다. 너무 자세하게 표현하게 되면 메모리 부족이나, 사용자 인증에 오랜 시간이 걸릴 수 있다.

### **AuthenticationManager**

반환된 `Authentication` 은 `AuthenticationManager`을 호출한 컨트롤러(즉, 스프링 시큐리티 필터 인스턴스)에 의해 `SecurityContextHolder`에 설정된다. 시큐리티 필터 인스턴스를 구현하지 않는다면, `AuthenticationManager`을 사용하지 않고 직접 `SecurityContextHolder`에 설정할 수 있다.

`AuthenticationManager`의 가장 일반적인 구현체는 `ProviderManager`이다.

### ProviderManager

`ProviderManager`는 `AuthenticationProvider` 인스턴스 리스트에 위임한다. 각 `AuthenticationProvider`는 인증이 실패, 성공인지 확인할 기회가 있으며, 결정을 할 수 없는 경우엔 다음 `AuthenticationProvder`에 넘긴다. 인증할 수 있는 `AuthenticationProvider` 인스턴스가 하나도 없다면, `ProviderNotFoundException` 와 함께 인증이 실패한다. 이 예외는 `ProviderManager`가 입력받은 `Authentication` 타입을 지원할 수 있는 `ProviderManager`가 설정되지 않았다는 것을 알려준다.

실제로 `AuthenticationProvider`는 여러 타입의 인증을 수행할 수 있다. 오직 `AuthenticationManager` 빈 하나만 노출시킨 채, 유저/패스워드 방식, SAML 등 매우 다양한 인증 방식을 지원할 수 있다.

`Authentication` 객체가 인증 성공 요청에 의해 반환되면, 기본적으로 `ProviderManager`는 `Authentication`에서 **민감한 자격 정보를 지운다**. 비밀번호 등의 정보를 `HttpSession`에 불필요하게 오래 유지되는 것을 방지한다.

자격 정보를 지우는 것은 무상태 어플리케이션 등의 사용자 객체를 캐시하는 경우 이슈가 생길 수 있다. `Authentication`이 캐시에 있는 `UserDetails` 인스턴스 같은 객체의 참조를 포함하고 이것의 자격 정보가 지워졌다면, 더 이상 캐시된 값에 대해 인증할 수 없다. 확실한 해결책으로, 캐시 구현체 또는 반환된 `Authentication` 객체를 생성한 `AuthenticationProvider`에서 먼저 객체의 복사본을 만드는 것이다. 또는 `ProviderManager`에서 자격정보를 제거하지 않도록 설정을 바꿀 수 도 있다.

### AuthenticationProvider

**유저/패스워드** 인증의 `DaoAuthenticationProvider`, **JWT 토큰** 인증의 `JwtAuthenticationProvider` 등이 있다.

## Username/Password Authentication

스프링 시큐리티는 `HttpServletRequest` 에서 유저/패스워드 인증 방식의 정보를 가져오는 내장 방식을 지원한다.

- Form
- Basic
- Digest

### Form

스프링 시큐리티에서 제공하는 디폴트 로그인 페이지를 사용하거나, 별도의 로그인 페이지를 사용할 수 있다.

스프링 시큐리티는 인증되지 않은 사용자가 인증이 필요한 요청을 할 경우, 로그인 페이지로 **리다이렉트** 시켜준다.

```java
public SecurityFilterChain filterChain(HttpSecurity http) {
	// 스트링 시큐리티 디폴트 로그인 페이지 사용
	//http
	//	.formLogin(withDefaults());
	http
		.formLogin(form -> form
			.loginPage("/login")
			.permitAll()
		);
	// ...
}
```

로그인 페이지를 위와 같이 지정했을 경우, GET /login 요청을 처리할 컨트롤러가 필요하며 해당 페이지의 Form 태그에서는 username, password 파라미터를 POST /login 으로 보내야 한다.

### Basic

유저, 패스워드를 base64로 인코딩하여 전달하는 방식이다. Authorization 헤더에 ‘Basic base64({username}:{password})’의 값을 담아 전달한다. base64로 다시 디코딩할 수 있어 안전하지 않다.

인증이 필요할 경우 응답 헤더에 **WWW-Authenticate**를 전달한다.

기본적으로 스프링 부트는 HTTP Basic 인증을 지원하지만, 서블릿 기반 설정을 한 경우엔 명시적으로 HTTP Basic 사용 설정을 해야만 한다.

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
	http
		// ...
		.httpBasic(withDefaults());
	return http.build();
}
```

## Password Storage

### In Memory

`InMemoryUserDetailsManager`는 `UserDetailsService` 인터페이스를 구현하여 유저/패스워드 기반의 인증 정보를 메모리에 저장한다.  또,`UserDetailsManager` 인터페이스를 구현하여 `UserDetails`의 관리한다.

- 메모리 저장 방식의 빈 등록 예시

```java
@Bean
public UserDetailsService users() {
	// The builder will ensure the passwords are encoded before saving in memory
	UserBuilder users = User.withDefaultPasswordEncoder();
	UserDetails user = users
		.username("user")
		.password("password")
		.roles("USER")
		.build();
	UserDetails admin = users
		.username("admin")
		.password("password")
		.roles("USER", "ADMIN")
		.build();
	return new InMemoryUserDetailsManager(user, admin);
}
```

지정한 password를 인코딩하여 메모리에 저장한다. 그러나 비밀번호가 소스 코드에 그대로 드러나므로 운영에서는 위의 방식을 사용하면 안된다.

### JDBC

`JdbcDaoImpl`은 `UserDetailsService`를 구현하여 유저/패스워드 방식의 인증 정보를 JDBC를 이용하여 관리한다. `JdbcDaoImpl`를 확장한 `JdbcUserDetailsManager`는 `UserDetailsManager` 인터페이스를 구현하여 `UserDetails` 를 관리한다.

스프링 시큐리티는 JDBC 기반의 인증을 위해 **디폴트 스키마와 쿼리**를 제공한다. 각각 실제로 사용하는 스키마에 따라 커스터마이징 하여 사용해야 한다.

```java
@Bean
UserDetailsManager users(DataSource dataSource) {
	UserDetails user = User.builder()...
	UserDetails admin = User.builder()...
	JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
	users.createUser(user);
	users.createUser(admin);
	return users;
}
```

### UserDetails

`UserDetailsService`로 부터 반환되는 값이다. `DaoAuthenticationProvider`는 `UserDetails`를 검증하고, 설정된 `UserDetailsService`에서 반환된 `UserDetails`를 자격정보로 갖는 `Authentication`을 반환한다.

### UserDetailsService

username, password를 가져오거나 다른 인증과 관련된 속성값을 가져오기 위해 `DaoAuthentiationProvider`에 의해 사용된다.

커스텀한 `UserDetailsService`를 빈으로 사용하여 인증을 커스텀할 수 있다.

## Persistence

사용자가 한번 인증을 한 후 다음 요청에서는 **인증이 된 상태가 유지**된 채 요청을 할 수 있어야 한다. 인증 후의 요청에는 세션 고정 공격을 방지하기 위해 인증된 새로운 세션 ID를 연결한다.

```java
# 로그인
POST /login HTTP/1.1
Host: example.com
Cookie: SESSION=91470ce0-3f3c-455b-b7ad-079b02290f7b

username=user&password=password&_csrf=35942e65-a172-4cd4-a1d4-d16a51147b3e
# 이후 인증된 새로운 세션을 사용하여 요청
GET / HTTP/1.1
Host: example.com
Cookie: SESSION=4c66e474-3f5a-43ed-8e48-cc1d8cb1d1c8
```

### SecurityContextRepository

스프링 시큐리티에서는 `SecurityContextRepository`를 사용하여 사용자의 다음 요청을 연결 짓는다. 디폴트 구현체는 `DelegatingSecurityContextRepository`이며, `HttpSessionSecurityContextRepository`와 `RequestAttributeSecurityContextRepository`를 위임하며 스프링 6에서는 이 두 가지를 기본 설정으로 사용한다.

`HttpSessionSecurityContextRepository`는 `SecurityContext`를 `HttpSession`에 연결한다.

`RequestAttributeSecurityContextRepository`는 `SecurityContext`를 요청 attribute로 저장한다. 이를 통해 `SecurityContext`를 한 요청에서 어디서든 사용할 수 있게 된다.

`NullSecurityContextRepository`를 사용하면 `SecurityContext`를 따로 저장하지 않는다.

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	http
		// ...
		// HttpSession 과 Request attribute에 저장하도록 설정
		.securityContext((securityContext) -> securityContext
			.securityContextRepository(new DelegatingSecurityContextRepository(
				new RequestAttributeSecurityContextRepository(),
				new HttpSessionSecurityContextRepository()
			))
		);
	return http.build();
}
```

`SecurityContextPersistenceFilter`는 `SecurityConetextRepository`를 사용해 요청 사이에서 `SecurityContext`를 유지할 책임이 있다. 반면 `SecurityContextHolderFilter`는 `SecurityContextRepository`에서 `SecurityContext`를 가져오기만 할 뿐, 따로 저장하지는 않는다. `SecurityContext`를 유지하려면 명시적으로 저장하는 로직이 필요하다.

## Session Management

세션 관리는 `SecurityContextHolderFilter`, `SecurityContextPersistenceFilter`, `SessionManagementFilter`의 조합으로 구성된다.

### Authentication 저장 공간 변경

기본적으로 스프링 시큐리티는 보안 정보를 **HTTP 세션**에 저장한다. 그러나 수평적 확장을 위해 캐시 또는 DB에 저장하는 등의 이유로 저장 공간을 변경할 수 있다.

`SecurityContextRepository`의 구현체를 만들거나, `HttpSessionSecurityContextRepository` 처럼 이미 존재하는 구현체를 사용하여 `HttpSecurity`에 설정하면 된다.

- `SecurityContextRepository` 구현체 등록 예시

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
    SecurityContextRepository repo = new MyCustomSecurityContextRepository();
    http
        // ...
        .securityContext((context) -> context
            .securityContextRepository(repo)
        );
    return http.build();
}
```

또는 필터를 사용하지 않고 직접 `Authentication`을 저장할 수도 있다.

- 컨트롤러에서 `Authentication`을 `HttpSession`에 저장하는 예시

```java
private SecurityContextRepository securityContextRepository =
        new HttpSessionSecurityContextRepository(); 
private final SecurityContextHolderStrategy securityContextHolderStrategy = 
				SecurityContextHolder.getContextHolderStrategy();

@PostMapping("/login")
public void login(@RequestBody LoginRequest loginRequest, HttpServletRequest request, HttpServletResponse response) { 
		// 인증되지 않은 유저/패스워드 토큰 생성
    UsernamePasswordAuthenticationToken token = UsernamePasswordAuthenticationToken.unauthenticated(
        loginRequest.getUsername(), loginRequest.getPassword()); 
		// 토큰을 인증하여 Authentication 객체 생성
    Authentication authentication = authenticationManager.authenticate(token); 
		// 비어있는 SecurityContext 생성
    SecurityContext context = securityContextHolderStrategy.createEmptyContext();
		// SecurityContext에 인증된 Authentication 저장
    context.setAuthentication(authentication);
		// SecurityConetxtHolder에 SecurityConetxt 저장
		// static 메서드를 사용해 SecurityConetxt를 저장하는 경우, 멀티 Application Conetxt 환경에서
		// 경쟁 상태를 일으킬 수 있다. ex) SecurityContextHolder.setContext(context);
    securityContextHolderStrategy.setContext(context);
		// 여기서는 HttpSession을 사용하는 SecurityContextRepository에 SecurityContext 저장
    securityContextRepository.saveContext(context, request, response); 
}
```

### Stateless 인증 유지

HTTP Basic 같은 무상태 인증의 경우엔 매 요청마다 다시 인증을 진행한다. 이런 경우엔 `HttpSession`을 전혀 사용하지 않을 수 있다.

- 세션을 생성하지 않는 설정 예시

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
    http
        // ...
        .sessionManagement((session) -> session
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        );
    return http.build();
}
```

비슷한 설정값으로 `SessionCreationPolicy.NEVER` 가 있으나, 인증 성공 후 이전 요청을 다시 진행하기 위해 요청값을 세션에 저장하는 과정에서 세션이 생성된다.

## Anonymous

보안적으로 봤을 때 모든 URL을 기본적으로 거부 상태로 두고, 필요한 URL만 허용하게 설정하는 것이 좋다. 비슷한 맥락으로, 인증되지 않은 사용자에게 접근 가능한 URL을 설정하는 것도 몇 가지의 URL만 지정하고 나머지는 인증이 필요하다고 설정할 수도 있다. 인증이 불필요한 URL의 경우, 필터 체인에서 예외 처리를 하여 처리할 수도 있지만, `Authentication` 값이 null 이 되버리는 등 추가적인 확인이 필요하게 된다. 인증이 안된 상태로 두지 않고 인증되지 않은 `Authentication`값을 사용하도록 하면 동일한 로직을 사용하여 처리할 수 있다.

**익명 인증**을 구성하려면 아래 3개의 요소가 필요하다.

- 익명의 보안 정보를 저장할 Authentication의 구현체 `AnonymousAuthenticationToken`
- `AnonymousAuthticationToken`의 인증을 처리할 `AnonymousAuthenticationProvider`
- 일반적인 인증 절차 이후 `AnonymouseAuthenticationToken`을 `SecurityConexstHolder`에 저장해줄 `AnonymousAuthenticationFilter`

OAuth 2 프레임워크의 Bearer 인증 방식은 다음에..

# 참고

[https://docs.spring.io/spring-security/reference/index.html](https://docs.spring.io/spring-security/reference/index.html)

[https://docs.spring.io/spring-security/reference/servlet/index.html](https://docs.spring.io/spring-security/reference/servlet/index.html)

[https://ws-pace.tistory.com/category/프로젝트/JWT 방식 인증%26인가 시리즈](https://ws-pace.tistory.com/category/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/JWT%20%EB%B0%A9%EC%8B%9D%20%EC%9D%B8%EC%A6%9D%26%EC%9D%B8%EA%B0%80%20%EC%8B%9C%EB%A6%AC%EC%A6%88)