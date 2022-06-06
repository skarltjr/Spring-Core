### AOP란?
```
관점 지향 프로그래밍
어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누고, 그 관점을 기준으로 각각 공통된 로직이나 기능을 하나의 단위로 묶는다.
예를들어 아래에서 핵심 로직은 개별logic()부분이며 부가적인 관점으론 all or nothing인 트랜잭션이 수행되어야한다는것이다.
여기서 부가적인 관점인 트랜잭션이 어느곳에서든 동작할 수 있도록(재활용성) 이를 하나의 단위로 묶는것이 AOP

@Transaction
public void hello()
{
  ..개별 logic
}

장점으론 공통되게 사용되는 기능적인 역할을 하는 코드를 재활용하여 사용할 수 있도록 만들어주는 방법
```
- 개념 및 사용법 : https://github.com/skarltjr/Memory_Write_Record/issues/69

### 동작 원리 1
```
스프링의 AOP는 Proxy기반으로 동작하며 Dynamic proxy & CGlib 방식이 존재한다

⭐️ 전반적인 동작 흐름은 동적으로 생성된 Proxy Bean은 타깃의 메소드가
호출되는 시점에 부가기능을 추가할 메소드를 자체적으로 판단하고 가로채어 부가기능을 주입

프록시란 가짜 객체이다. 즉 Target호출시 Target을 부가기능으로 감싼 프록시가 요청을 대신받고 부가기능 + 호출 매서드 동작 수행한다.
아래에서 hello()매서드가 불릴 때 먼저! 동적으로 생성된 프록시빈이 설정한 기능을 추가로 주입한 후 매서드가 동작

----- 
ex)
@Transaction
public void hello()
{
  ...
}
```

### 동작 원리 2 (Dynamic proxy)
- <img width="1175" alt="스크린샷 2022-06-06 오후 2 39 47" src="https://user-images.githubusercontent.com/62214428/172101665-54cefe2f-5a42-4a1a-a55d-704c96c88c35.png">
```
그럼 AOP proxy는 어떻게 생성되는가?
우선 타겟이란? 우리가 aop를 통해 어떤 기능을 적용하고자 하는, 대상이되는 매서드 혹은 클래스 등

만약 타겟이 하나 이상의 인터페이스를 구현하고 있다면 Dynamic Proxy 방식으로
없다면 CGLIB 방식으로 생성된다

⭐️1. Dynamic Proxy
Dynamic Proxy는 자바 리플렉션 패키지의 Proxy클래스를 통해 생성된 객체이다.
Dynamic Proxy의 핵심은 타겟의 인터페이스를 기준으로 Proxy를 생성해주는것
!! 그래서 인터페이스를 상속받아야하고 의존관계 자동주입(Autowired)시 반드시 인터페이스 타입으로 지정해줘야한다

그렇기에
장점 : 개발자가 직접 프록시 객체를 만들필요가 없다.
단점 : 인터페이스가 강제되며 리플렉션을 활용하기에 성능이 떨어진다.

ex)
@Autowired
private xxService xxService;
```

### 동작 원리 3 (CGLIB)
```
스프링부트는 이를 택한 CGLib은 Code Generator Library의 약자로, 
클래스의 바이트코드를 조작하여 Proxy 객체를 생성해주는 라이브러리이다
따라서 인터페이스가 아닌 타겟의 클래스에 대해서도 Proxy를 생성해준다

1. CGLIB은 타겟의 클래스를 상속받는다
2. 타겟 클래스에 포함된 모든 매서드를 재정의하여 Proxy를 생성해준다
3. 다만 Final이 붙은 매서드 or 클래스에 대해서는 재정의할 수 없으므로 Proxy 생성이 불가능하지만 리플렉션보단 성능이 좋다
4. private 매서드도 상속이 불가능
⭐️그리고 CGLIB을 위해선 타겟이 기본생성자가 필요하다.
이게 지금까지 왜 스프링부트 & jpa사용할 때 lazy loading등에서 프록시를 위해선 기본 생성자가 필요하다의 이유였던것같다.

장점 : 인터페이스 강제 x 단순 클래스만으로 프록시 객체를 동적으로 생성가능
단점 : final은 재정의 불가능 혹은 private한 매서드등은 상속이 불가능
```
### aop와 추상화
```
내가 생각했을 때 aop는 oop를 더욱 세련되게 구현할 수 있는 방법이라고 생각한다
변화가 적고 공통된 기능을 뽑아내는 추상화에도 불구하고 분명히 중복되는 코드는 존재할 수 있다.
aop는 이러한 중복코드를 더욱 줄이고 재사용성을 극대화시킨다고 생각한다.
ex) 스프링의 어노테이션 @Tractional
```