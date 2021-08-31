# `AOP(Aspect Oriented Programming)란 무엇일까?`

![aop](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIQHz7%2FbtqGxmhDtVP%2FNGOxeaadAnr4LpvSr4IkX1%2Fimg.png)

스프링의 삼각형 중에 하나가 `AOP(Aspect Oriented Programming)`입니다. 이번 글에서는 어렵고 중요한 AOP에 대해서 정리해보겠습니다.

`AOP`는 `Aspect Oriented Programming`의 약자로 `관점 지향 프로그래밍`이라고 불립니다. 그리고 흩어진 Aspect를 `모듈화`할 수 있는 프로그래밍 기법입니다. 정의하면 이런데.. 무슨 말인지 잘 와닿지 않는 거 같습니다,, 

![aop](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbJOHOE%2FbtqF0QCJlCc%2F2sTwCFrVK71VUGTCK6Hkl0%2Fimg.png)

위의 A, B, C 클래스에서 동일한 색깔의 선들의 의미는 클래스들에 나타나는 `비슷한(중복되는) 메소드, 필드, 코드들이 나타난다는 것`입니다. 이러한 경우 만약 클래스 A에 주황색 부분을 수정해야 한다면 B, C 클래스들에 주황색 부분에 해당하는 곳을 찾아가 전부 코드를 수정해야 합니다. (SOLID 원칙도 위배하며 유지보수도 쉽지 않을 것입니다.) 이런식으로 반복되는 코드를 `흩어진 관심사 (Crosscutting Concerns)`라 부릅니다.

이렇게 `흩어진 관심사`를 `AOP는 Aspect를 이용해서 해결`합니다. 위의 사진의 아래쪽을 보면 흩어져 있는 부분들을 Aspect를 이용해서 모듈화 시킨 것을 볼 수 있습니다. (모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말합니다.) 그리고 개발자가 모듈화 시킨 Aspect를 사진에서 위에 클래스에 어느 곳에 사용해야 하는지만 정의해주면 됩니다.

> 결론적으로 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 AOP의 취지다.

<br>

대략 AOP가 어떤 것이고 언제, 왜 사용하는지에 대한 감은 오셨을 것입니다. 그런데 아직도 용어들이 쉽지는 않다 보니 많이 와닿지는 않을텐데요. 이제 좀 더 자세히 하나씩 알아보겠습니다. 

<br> <br>

## `스프링 AOP 특징`

스프링 AOP를 공부하다 보면 항상 나오는 예제가 있습니다. 바로 `실행 시간`을 출력하는 것인데요. 실행 시간을 출력하는 예제를 보면서 AOP 특징에 대해서 알아보겠습니다.

```java
public interface EventService {

    void createEvent();
    void publishEvent();
    void deleteEvent();
}
```

```java

@Service
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Published an event");
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {

    }
}
```

위와 같이 `EventService 인터페이스`를 구현한 `SimpleEventService 클래스`가 존재하는 상황입니다. `createEvent()`, `publishEvent()` 메소드의 실행 시간을 측정하려고 합니다.

하지만 위의 코드의 예시는 단순히 메소드가 2개만 존재할 뿐이기에 별거 아닐 수 있지만 실제 프로젝트로 치면 엄청나게 많은 중복 코드가 필요할 것입니다. 만약에 메소드가 30개만 됐다고 해도 엄청난.. 중복이 일어날텐데요. 이러한 상황에서 사용할 수 있는 것이 `AOP`라고 지금까지 얘기했는데요. 

`AOP는 어떤 방법을 사용했기에 기존 코드는 수정하지 않고 메소드들의 성능 측정을 할 수 있는 것일까요?` 바로 `프록시 패턴`을 사용하기 때문입니다. 

<br> <br>

## `프록시 패턴`

![proxy](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEECrr%2FbtqFWZhqAhT%2Fl8kDltgwVpC7mAEC1uwKG1%2Fimg.png)

Client는 Subject 인터페이스 타입으로 프록시 객체를 사용하게 되고, 프록시는 Real Subject를 감싸서 클라이언트의 요청을 처리하게 됩니다. `프록시 패턴의 목적은 기존 코드 변경 없이 접근 제어 또는 부가 기능을 추가하기 위해서입니다.`

여기서 위의 `EventService 인터페이스`를 구현하는 `프록시 클래스`를 만들어보겠습니다.

```java
@Primary
@Service
public class ProxySimpleEventService implements EventService {

    @Autowired
    private SimpleEventService simpleEventService;

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }
    
    @Override
    public void deleteEvent() {
        
    }
}
```

먼저 `@Primary` 어노테이션이 보이는데요. [저번 글](https://github.com/AUSG-Spring-Beginner/Spring_hoecholi/blob/master/week2/%EC%A0%95%EA%B7%A0/4.%20%EC%9D%98%EC%A1%B4%20%EC%9E%90%EB%8F%99%20%EC%A3%BC%EC%9E%85_%EC%A0%95%EA%B7%A0.md) 에서 같은 타입의 빈이 여러 개가 존재할 때 우선순위를 높게 줄 때 사용하는 어노테이션이라고 공부하였습니다. 

그리고 위의 `Proxy 클래스`의 코드를 보면 필드로 실제 핵심 코드를 담당하는 `SimpleEventService`를 가지고 있는 것을 볼 수 있습니다. 즉, 이러한 구조를 가진 패턴을 `프록시 패턴`이라고 하는데, 기존의 `SimpleEventService 클래스`의 코드에는 성능 측정하는 코드를 작성하지 않아도 된다는 장점이 생긴 것입니다. 

하지만 이러한 방식도 메소드가 많다면 코드의 중복이 많이 일어날 것이고 매번 `Proxy 클래스`도 직접 만들어야 한다는 큰 단점이 존재합니다. 그래서 이러한 단점을 해결하기 위해 나온 것이 바로 `Spring AOP` 입니다.

<br> <br>

## `Spring AOP란?`

![spring-aop](https://user-images.githubusercontent.com/45676906/122227178-a1af6a00-cef1-11eb-8c22-23cbcb43bc03.png)

스프링도 위에서 본 `프록시를 이용해서 AOP를 구현`하고 있습니다. AOP의 핵심 기능은 `코드를 수정하지 않으면서 공통 기능의 구현을 추가하는 것`이라고 강조하고 있습니다. 핵심 기능에 공통 기능을 추가하는 방법에는 아래와 같이 3가지 방법이 존재합니다. 

- 컴파일 : 자바 파일을 클래스 파일로 만들 때 바이트코드를 조작하여 적용된 바이트코드를 생성
- 로드 타임 : 컴파일은 원래 클래스 그대로 하고, 클래스를 로딩하는 시점에 끼워서 넣는다.
- 런타임 : A라는 클래스를 빈으로 만들 때 A라는 타입의 프록시 빈을 감싸서 만든 후에, 프록시 빈이 클래스 중간에 코드를 추가해서 넣는다.

<br>

스프링에서 많이 사용하는 방식은 `프록시를 이용한 세 번째 방법`입니다. 스프링 AOP는 프록시 객체를 자동으로 만들어줍니다. 따라서 위에서 본 `ProxySimpleEventService` 클래스 처럼 상위 타입의 인터페이스를 상속 받은 클래스를 직접 구현할 필요가 없습니다. 단지 공통 기능을 구현한 클래스만 잘 구현하면 됩니다.

<br> <br>

## `AOP 주요 개념`

- Aspect : 위의 사진에서 처럼 Aspect 안에 모듈화 시킨 것을 의미한다.
- Advice : 실질적으로 어떤 일을 해야하는지를 담고 있다.
- Pointcut : 어디에 적용해야 하는지에 대한 정보를 담고 있다.
- Target : Aspect에 적용이 되는 대상
- Join point : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능(여러가지 합류 지점임)

<br>

용어를 전부 외울 필요까지는 없을 거 같고 많이 보다 보면 자연스럽게 익혀질 것 같습니다.  

<br> <br>

## `Advice 종류`

종류는 여러가지 있지만, 대표적으로 `Around Advice`가 존재합니다. 

- `Around Advice`: 대상 객체의 메소드 실행 전, 후 또는 익셉션 발생 시전에 공통 기능을 실행하는데 사용됩니다. 

<br> <br>

## `Spring AOP 구현하기`

### maven

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### gradle

```
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

먼저 Spring에서 AOP를 사용하기 위해서는 `spring-starter-aop` 의존성을 추가해주어야 합니다. 

<br>

```java
@Component
@Aspect
public class PerfAspect {

    @Around("execution(* com.example..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object reVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return reVal;
    }
}
```

위에서 실행 시간을 측정하는 공통 코드를 `Aspect 모듈`로 분리시켜서 작성할 수 있습니다. 그리고 `@Around()`에서 `execution`을 사용하여 `Advice를 적용할 범위`를 지정할 때 사용할 수 있습니다. 

즉, 위의 코드로 해석을 해보면 `com.example 패키지 밑에 있는 모든 클래스에 적용을 하고, EventService 밑에 있는 모든 메소드에 적용해라` 라는 뜻입니다.

그런데 만약 어떤 곳에는 적용하고 싶고 어떤 곳에는 적용하고 싶지 않다면 어떻게 할 수 있을까요? `execution`으로도 사용할 수 있겠지만 좀 까다롭다는 단점이 있는데요. 그래서 이럴 때는 `어노테이션`을 사용해서 하는 방식도 존재합니다. 
실제로 제가 현재 진행하고 있는 프로젝트에서도 `Spring AOP, 어노테이션` 방식을 사용하고 있습니다.

![스크린샷 2021-09-01 오전 6 51 41](https://user-images.githubusercontent.com/45676906/131580506-41a7c6d0-6b2d-479e-a9b8-5fbed2d37c46.png)

Retention의 기본 값을 Class 입니다. Class로 사용하면 어노테이션의 정보가 바이트코드 까지 남아있게 됩니다. Class로만 두어도 충분하지만 저는 일단 Runtime 으로 사용하고 있습니다.

<br>

![스크린샷 2021-09-01 오전 6 48 23](https://user-images.githubusercontent.com/45676906/131580297-874ad784-8aa5-4611-8f2b-2d5850537220.png)

위의 코드에서 `@Around()`를 보면 이번에는 `execution`이 아니라 `annotation`을 사용하고 있는 것을 볼 수 있습니다. 즉, 어노테이션을 사용할 때 AOP가 실행된다는 뜻입니다. 

그리고 또 하나 볼 점이 매개변수에 `ProceedingJoinPoint`와 마지막에 `pjp.proceed()` 인데요. 이것은 `proceed()` 메소드를 사용해서 `실제 대상 객체`의 메소드를 호출하는 것입니다. 즉, 대상 객체를 실행시켜서 위의 공통 코드를 메소드 이전과 이후에 코드를 위치시키면 됩니다. 

<br>

![스크린샷 2021-09-01 오전 6 59 54](https://user-images.githubusercontent.com/45676906/131581379-abb60c5f-b924-4be4-aadb-de9b7efdc3e7.png)

위에서 만든 어노테이션을 추가해주기만 하면 위의 Controller가 실행될 때마다 정의한 Aspect의 Around가 실행됩니다.