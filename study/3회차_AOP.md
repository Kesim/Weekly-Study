# AOP

담당자: 강성민
진행 상황: 진행중
유형: 백엔드
마감일: 2024년 2월 11일
게시일: 2024년 2월 11일
회차: 3

# Aspect Oriented Programming

## AOP 란

AOP는 뜻을 해석하면 **관점 지향 프로그래밍**으로, 여러 기능에 펴져있는 **Aspect**라고 부르는 **관점, 관심사**를 기능에서 분리하는 것을 목적으로 하는 프로그래밍 패러다임이다. Aspect 서비스의 핵심 기능과 구분되는 **부가 기능**을 말한다. 이런 부가 기능은 다양한 곳에서 사용될 수 있어, 불필요한 반복과 유지보수의 어려움, OOP에 위배될 수 있다. AOP는 핵심 기능의 원본 코드를 수정하지 않고 부가 기능을 적용하는 것을 목적으로 한다.

## AOP의 예

대표적인 예로 로깅이 있다. 몇 개의 클래스 또는 메서드에 로그를 추가하는 것은 어렵지 않지만, 그 개수가 점점 많아지면 불필요한 코드의 반복이 생기고 유지보수가 어려워진다. AOP를 이용하면 로그를 남기는 코드를 하나 작성 후, 이 기능이 필요한 대상을 선정하여 일괄적으로 적용할 수 있다.

핵심 기능 A, B, C, ..

부가 기능 a, b, c, ..

A - a, b

B - a

C - a, c

…

## 자바 AOP 적용 시점

자바에서 AOP를 적용하는 방법은 시점에 따라 3가지가 있다. 이 중 컴파일, 클래스 로딩 시점에서 AOP를 적용하는 방법은 AspectJ 라는 외부 프로그램의 도움이 필요하다.

- 컴파일 시점
    - .java 파일을 컴파일하여 .class 로 변환하는 컴파일 시점에 파일을 조작하여 부가 기능 코드를 직접 넣어준다. 결과적으로 핵심 코드에 부가 코드가 추가된 형태로 AOP가 적용 된다.
    - AspectJ에서 제공하는 컴파일러를 사용해야 한다.
- 클래스 로딩 시점
    - 자바는 실행 후 컴파일된 .class 파일을 JVM의 클래스 내부에 보관한다. 이 보관 시점에 파일을 조작하여 부가 기능을 적용할 수 있다.
    - AspectJ가 필요하며, 자바 실행 시 `java -javaagent` 옵션을 사용해야 한다.
- **런타임 시점**
    - **프록시**를 이용하여 부가 기능을 적용한다. 위의 두 방법과 달리 자바의 본래 기능만으로도 구현이 가능하다.
    - 프록시를 적용하기 위해선 적용 대상의 **인터페이스**를 구현하거나 대상을 **상속**받아 코드를 작성해야 한다. 프록시 코드를 직접 작성할 수도 있지만,AspectJ 에서 제공하는 **어노테이션**을 사용하여 AOP를 간단하게 적용할 수 있다.
    - 상속 방식은 대상의 생성자 접근자에 따라 구현이 불가능 하지만, **CGLIB** 라이브러리를 이용하면 모든 클래스를 상속할 수 있게 한다. CGLIB를 스프링 내부에 포함시켜 스프링만으로도 AOP를 적용할 수 있다.
    - 다른 방법에 비해 AOP 기능에 제약이 있지만, 일반적으로 실무에서 사용하는데 충분하다.

## 스프링 AOP 용어

- Advice : Aspect의 기능 즉, 부가 기능을 의미한다.
- Pointcut : Advice를 ****적용할 대상을 지정한다.
- Advisor : Advice + Pointcut 의 조합으로, 어느 곳에 부가 기능을 적용할 지를 나타낸다.
- Aspect : 스프링에서 Advisor의 의미로 사용한다.
- Join point : Advice를 적용할 수 있는 시점을 의미한다. 자바에서는 **프록시**를 사용하므로 메서드 호출 시점만 가능하다. 호출 객체, 메서드, 파라미터와 실제로 호출되는 대상의 정보를 갖는다.

## 스프링 AOP 구현

@Asepct를 명시한 클래스 안에 Pointcut 관련 어노테이션을 사용한 메서드를 구현한다. 메서드의 내용이 Advice가 된다.

### 스프링 AOP 어노테이션

아래의 어노테이션은 Advice를 구현할 메서드에 사용한다. 어노테이션의 value 값으로 Pointcut을 지정할 수 있다. value의 Pointcut은 @Pointcut를 빈 메서드에 사용하여 따로 분리할 수 있다.

- **@Around** : 메서드 호출 전후에 수행
- @Before : 메서드 호출 이전에 실행
- @After : 메서드 정상 호출 또는 예외 발생 후 실행
- @AfterReturning : 메서드가 예외 없이 반환 후 실행
- @AfterThrowing : 메서드 호출 중 예외 발생 후 실행

위의 어노테이션이 같은 대상에 적용 될 경우 순서는 아래와 같다.

- 클라이언트 - Around - Before - 메서드 호출 - (예외 발생 시 AfterThrowing) - (정상 반환 시 AfterReturning) - After - Around - 클라이언트 반환

```java
// Pointcut 분리 예시
@Pointcut("execution(* hello.aop.order..*(..))")
private void allOrder(){}

@Around("allOrder()")
...
```

한 메서드에 여러 프록시를 적용하여 여러 Aspect가 사용될 때 기본적으로 **순서를 보장하지 않는다**. 원하는 순서로 적용하려면 @Aspect과 **@Order** 를 같이 사용하여 Order의 값으로 순서를 정해야 한다. @Aspect는 클래스에 적용되므로 적용 순서는 클래스 단위로만 정할 수 있다.

```java
@Aspect
@Order(1)
..

@Aspect
@Order(2)
```

### @Around과 ProceedingJoinPoint vs JoinPoint

Advice를 위한 어노테이션의 메소드는 첫 번째 파라미터로 ProceedingJoinPoint 또는 Joinpoint의 파라미터를 사용할 수 있다. 이 중 ProceedingJoinPoint는 **@Around**에서만 사용할 수 있으며 **proceed**() 메서드로 타겟 객체의 메서드를 직접 실행 할 수 있다. proceed() 로 메서드를 실행하지 않을 경우 타겟 메서드 실행이 안되므로 주의해야한다.

이러한 실수를 방지하기 위해 Around 외의 어노테이션을 사용할 수 있으며, 더 의도를 명확하게 표현할 수 있다.

Around를 잘 활용하면 아래 처럼 다른 어노테이션의 기능을 한 번에 모두 사용할 수 있다. 

```java
@Around("hello.aop.order.aop.Pointcuts.orderAndService()")
public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
		try {
				//@Before
				log.info("[around][트랜잭션 시작] {}", joinPoint.getSignature());
				Object result = joinPoint.proceed();
				//@AfterReturning
				log.info("[around][트랜잭션 커밋] {}", joinPoint.getSignature());
				return result;
				} catch (Exception e) {
				//@AfterThrowing
				log.info("[around][트랜잭션 롤백] {}", joinPoint.getSignature());
				throw e;
				} finally {
				//@After
				log.info("[around][리소스 릴리즈] {}", joinPoint.getSignature());
		}
}
```

### 포인트컷 지시자의 종류

- **execution** : 선언 타입, 메소드 명, 파라미터 등을 이용하여 매칭
- within : 특정 타입 매칭
- args : 파라미터 타입 매칭
- this : 스프링 빈 **객체**(스프링 AOP 프록시) 대상
- target : 타겟 **객체**(스프링 AOP 프록시가 가르키는 실제 대상) 대상
- @target : 인스턴스의 모든 **메서드** 대상 ⇒ 부모 클래스 까지 적용
- @within : 해당 타입 내의 **메서드**만 대상 ⇒ 타겟 클래스에 정의된 메서드만. 부모 클래스에만 정의된 메서드는 해당 안됨
- @annotation : 메서드에 특정 애노테이션을 적용한 대상
- @args : 파라미터에 특정 애노테이션을 적용한 대상
- bean : 빈의 이름 대상. 스프링 전용 포인트컷 지시자

이 중 this, target, args,@target, @within, @annotation, @args을 사용하면 Advice 메서드에 직접 파라미터를 받을 수 있다. 지시자에 사용한 파라미터와 Advice 메서의 파라미터는 순서와 이름이 같아야한다.

```bash
@Around("args(arg)")
public void logArgs3(String arg) {
		// ...
}
```

포인트컷 지시자는 &&(AND), ||(OR), !(NOT) 로 한번에 여러개 사용이 가능하다.

### execution 지시자 문법

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?namepattern(param-pattern) throws-pattern?)
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
```

- ? : 생략 가능
- * : 아무 값이 들어와도 됨
- 패키지
    - . : 정확히 해당 위치
    - .. : 해당 패키지와 하위 패키지 모두 포함
- 파라미터 .. : 타입, 개수 상관없음

## AOP 사용 시 주의할 점

실무에서 AOP 사용 시 Pointcut 작성을 주의해야 한다. 잘못 작성된 Pointcut은 프록시가 필요없는 대상까지 적용시킬 수 있어 에러 또는 불필요한 부하를 일으킬 수 있다. 꼭 필요한 대상만 적용되도록 적용 대상을 최소화 해야 한다.

스프링의 AOP는 프록시를 사용하기 때문에 한 클래스 안에서 내부 메소드를 직접 호출하는 경우 적용이 되지 않는다. 프록시 대상 클래스는 내부 메소드를 직접 호출하지 않게 구조를 변경하거나, 스프링의 지연 로딩을 이용하여 자신의 객체를 주입받는 방법이 있다. `ApplicationContext` 또는 `ObjectProvider<Class>`를 사용하여 자신의 객체를 지연 로딩하여 주입 받아 사용한다.

# 참고

- 인프런 - 스프링 핵심 원리 고급편
    - [https://www.inflearn.com/course/스프링-핵심-원리-고급편/dashboard](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)