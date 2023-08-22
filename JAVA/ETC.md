
- [자바 직렬화](#자바-직렬화)
  - [직렬화의 종류](#직렬화의-종류)
  - [자바 직렬화](#자바-직렬화-1)
  - [자바 직렬화를 사용하는 이유](#자바-직렬화를-사용하는-이유)
  - [자바 직렬화 조건](#자바-직렬화-조건)
  - [자바 역직렬화 조건](#자바-역직렬화-조건)
  - [자바 직렬화은 언제 어디서 사용될까?](#자바-직렬화은-언제-어디서-사용될까)
  - [자바 직렬화를 사용할 때, 주의해야 할 점](#자바-직렬화를-사용할-때-주의해야-할-점)
- [제너릭](#제너릭)
  - [Type parameter와 Type argument](#type-parameter와-type-argument)
  - [Raw Type](#raw-type)
  - [제너릭 메서드](#제너릭-메서드)
  - [Bounded Type parameter](#bounded-type-parameter)
  - [제너릭 vs 와일드 카드](#제너릭-vs-와일드-카드)
  - [제너릭의 상속에서 주의할 점](#제너릭의-상속에서-주의할-점)
  - [capture (공부 필요)](#capture-공부-필요)
- [String Constant Poll vs Constant Pool](#string-constant-poll-vs-constant-pool)
  - [String Constant Pool은 GC의 대상이 될까?](#string-constant-pool은-gc의-대상이-될까)
- [Metaspace](#metaspace)
  - [Native Memory](#native-memory)
  - [Java Heap Shrinkage (공부 필요)](#java-heap-shrinkage-공부-필요)
- [클래스 패스](#클래스-패스)

# 자바 직렬화

객체 직렬화란 자바가 객체를 바이트 스트림으로 인코딩하고(직렬화) 그 바이트 스트림으로부터 다시 객체를 재구성하는(역직렬화) 메커니즘이다.

시스템적으로 이야기하자면 JVM의 메모리에 상주(힙 또는 스택)되어 있는 개체 데이터를 바이트 형태로 변환하는 기술과 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 같이 이야기 한다.

> 간단하게 말해서, 객체에 대한 정보를 하나의 바이트 배열로 변환하거나, 그 반대의 변환을 하는 기술이다.

## 직렬화의 종류

- 문자열 형태의 직렬화 방법 : 직접 데이터를 문자열 형태로 확인 가능한 직렬화 방법입니다. 범용적인 API나 데이터를 변환하여 추출할 때 많이 사용된다.
  - Ex) CSV, JSON
- 이진 직렬화 방법 : 데이터 변환 및 전송 속도에 최적화하여 별도의 직렬화 방법을 제시하는 구조이다.
  - Ex) Protocol Buffer

## 자바 직렬화

CSV, JSON, Protocol Buffer 등은 시스템의 고유 특성과 상관없는 대부분의 시스템에서의 데이터 교환 시 많이 사용된다.

하지만 자바 직렬화 형태의 데이터 교환은 자바 시스템간의 데이터 교환을 위해서 존재한다.

## 자바 직렬화를 사용하는 이유

CSV, JSON등과 같은 직렬화를 사용하여, 자바 시스템간의 데이터 교환이 가능하다.

그럼에도 불구하고 자바 직렬화를 사용하는 이유는 자바 시스템에서 개발에 최적화 되어 있기 때문이다.
- 직렬화 기본 조건만 지키면, 바로 직렬화가 가능하다.
- 역직렬화가 되면 기존 객체처럼 바로 사용가능하다.
  - 데이터 타입이 자동으로 맞춰지기 때문

## 자바 직렬화 조건

자바 기본(primitive) 타입과 `java.io.Serializable` 인터페이스를 상속받은 객체는 직렬화 할 수 있는 기본 조건을 가진다.

## 자바 역직렬화 조건

- 직렬화 대상이 된 객체의 클래스가 캘래스 패스에 존재해야 하며, `import` 되어 있어야 한다.
  - `import`되어 있지 않다고 해서 에러가 발생하지는 않는다.
  - 다만 다운 캐스팅을 하지 못해서, 저장된 원본 객체의 메서드를 사용하지 못 할 뿐이다.
- 자바 직렬화 대상 객체는 동일한 `serialVersionUID`를 가지고 있어야 한다.
  - `serialVersionUID`의 기본 값은 클래스의 `해시값` 이다.

> 직렬화와 역직렬화를 진행하는 시스템이 서로 다를 수 있다는 것을 고려해야한다.

## 자바 직렬화은 언제 어디서 사용될까?

JVM의 메모리에서만 상주되어 있는 객체 데이터르 그대로 영속화가 필요할 때 사용된다. 영속화된 데이터이기 때문에 네트워크로 전송도 가능하다.

- 서블릿 세션 : 서블릿 기반의 WAS들은 대부분 세션의 자바 직렬화를 지원하고 있다.
- 캐시 : 자바 시스템에서 퍼포먼스를 위해 캐시(Ehcache, Redis, Memcached, ...) 라이브러리를 시스템을 많이 이용하게 된다.

## 자바 직렬화를 사용할 때, 주의해야 할 점

  1. 역직렬화시 클래스 구조 변경 : 자바 직렬화는 타입에 엄격하다. 또한 여러가지 의미로, 예외가 발생할 수 있음을 주의해야 한다.
  2. 용량 : 자바 직렬화시 타입에 대한 정보 등 클래스의 메타 정보도 가지고 있기 때문에, 상대적으로 다른 포멧에 비해서 용량이 크다는 단점이 존재한다.
  3. 호환성 : 자바 직렬화를 이용하여 외부 데이터를 저장히게 되면, 자바에서만 읽을 수 있다. 다른 언어를 통해 데이터를 읽기가 상당히 어렵다.

> 우아한 개발자가 말하는 직렬화를 사용 시점
>
> 1. 외부 저장소로 저장되는 데이터는 짧은 만료시간의 데이터를 제외하고 자바 직렬화를 사용을 지양합니다.
> 2. 역직렬화시 반드시 예외가 생긴다는 것을 생각하고 개발합니다.
> 3. 자주 변경되는 비즈니스적인 데이터를 자바 직렬화을 사용하지 않습니다.
> 4. 긴 만료 시간을 가지는 데이터는 JSON 등 다른 포맷을 사용하여 저장합니다.

> 출처
> 이펙티브자바
> [우아한 기술 블로그](https://techblog.woowahan.com/2550/)
> [오라클 공식 문서](https://docs.oracle.com/javase/6/docs/platform/serialization/spec/class.html#4100)

---

# 제너릭

- 컴파일 시점에 컴파일러가 타입을 체크를 하는 것
  - 컴파일러가 문법 오류를 잡아주기 때문에, 좀 더 타입 안전한 프로그래밍이 가능하다.

## Type parameter와 Type argument

```java
public class Generics {
  class Hello<T> { // T -> type parameter
  }

  public static main(String[] args) {
    new Hello<String>(); // String -> type argument
  }
}
```

## Raw Type

type argument를 사용하지 않은 타입을 말한다.

```java
public void someMethod() {
  List<Integer> ints = Arrays.asList(1, 2, 3); // type argument를 사용한 List

  List rawType = ints; // type argument를 사용하지 않는 List
}
```

Raw Type은 `컴파일이 타입 체크`를 해줄 수 없기 때문에, 컴파일 에러가 아닌 런타임 에러가 발생할 수 있다.

따라서, Raw Type의 사용은 지양해야 한다.

> 자바에서 Raw Type을 지원하는 이유
> 
> Java5 이전과의 호환성 때문이다. 제너릭은 Java5 부터 주가된 기능으로, 그 이전의 코드는 제너릭이라는 것이 없었다. 최신 버전의 Java에서 예전 자바의 코드 또한 실행되어야 하기 때문에, Raw Type을 지원한다.

## 제너릭 메서드

메서드 레벨에서 Type parameter를 적는 것

제너릭 메서드가 필요한 이유는 다음과 같다.
- 제너릭 클래스에 있는 type parameter가 아닌 메서드 에서 사용하고 싶은 type parameter가 있는 경우
- static 메서드에서 제너릭을 사용하기 위해
  - static 메서드는 인스턴스를 만들기 전에 사용이 가능하기 때문에, static 메서드 사용 시점에 type을 결정해야 한다.

## Bounded Type parameter

Type parameter는 어떠한 타입이 들어갈 수 있다.  
이러한 Type parameter에 제한된 타입만 들어갈 수 있도록 하는 것이 Bounded Type parameter이다.

이렇게 Type parameter에 제한을 두는 이유는 다음과 같다.
- 특정 인터페이스나 클래스를 구현 또는 상속한 클래스만 받고 싶을 경우
  - Ex) 비교 연산을 위해, Comparalbe 인터페이스를 구현한 클래스만 인자로 받고 싶을 경우

## 제너릭 vs 와일드 카드

제너릭 : type parameter에서 제공하는 기능이 필요할 때 사용
와일드 카드 : type parameter에서 제공하는 기능이 필요하지 않을 때 사용

```java
List<?> list; 
// 1. 원소를 꺼내 와서는 Object에 정의되어 있는 기능만 사용하겠다. equals(), toString(), hashCode()…
// 2. List에 타입이 뭐가 오든 상관 없다. 나는 List 인터페이스에 정의되어 있는 기능만 사용하겠다. size(), clear().. 단, 타입 파라미터와 결부된 기능은 사용하지 않겠다! add(), addAll()

List<T> list;
// 1. 원소를 꺼내 와서는 Object에 정의되어 있는 기능만 사용하겠다. equals(), toString(), hashCode()…
// 2. List에 타입이 뭐가 오든 상관 없다. 나는 List 인터페이스에 정의되어 있는 기능만 사용을 하고, 타입 파라미터와 결부된 기능도 사용하겠다.
```

## 제너릭의 상속에서 주의할 점

`Number`은 `Integer`의 super 클래스가 맞다.
하지만 `List<Number>`은 `List<Integer>`의 super 클래스가 아니다.

제너릭은 클래스간 상속에 영향을 주지 않는다.

## capture (공부 필요)


> 출처
> [토비의 봄 (제너릭)](https://www.youtube.com/watch?v=ipT2XG1SHtQ&list=PLv-xDnFD-nnmof-yoZQN8Fs2kVljIuFyC&index=12&ab_channel=%ED%86%A0%EB%B9%84%EC%9D%98%EC%8A%A4%ED%94%84%EB%A7%81)
> https://vvshinevv.tistory.com/54

---

# String Constant Poll vs Constant Pool

String Constant Poll과 Constant Poll은 이름이 비슷하다.
하지만 위의 두 용어는 완전히 다른 용어이다.

저장되는 위치, 저장하는 데이터의 종류, 관리 주체까지 모든 것이 다른 저장공간이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeaJffC%2FbtsgwMqYWOP%2FlWjjsBoXdDoUMWgrSr5ZCK%2Fimg.png)


|| String Constant Pool | Constant Pool inside a Java class file |	Runtime Constant pool |
|:-:|:-:|:-:|:-:|
|역할|	String 객체의 상수 값을 저장(캐싱)|클래스 파일의 상수 값을 저장|Class Constant pool에서 읽어온 상수 값, 클래스 메타데이터 저장|
|저장되는 값 | String 상수 값 <br> ex) "Hello", "World" | 클래스 파일에 포함된 상수 값 <br> ex) 123, 3.14, true | 클래스 파일에 포함된 상수 값. Class Constant pool에 저장되어있던 값이 런타임시 이 영역으로 저장된다. |
| 저장 위치 | Java 7 이전 - Perm <br> Java 8 이상 - Metaspace| Class file | 		Java 7 이전 - Perm <br> Java 8 이상 - Metaspace |
|저장 트리거| String.intern(), String을 리터럴로 생성 | 컴파일시 생성됨 | 클래스파일에 코드 레벨로 선언된 상수풀이 런타임시 로더의 판단에 의해 올라옴. |
|불변 여부|불변 여부	불변 (Immutable)|Class 파일 자체는 불변.|Runtime시 동적 로드에 의해 변경될 수 있음	불변이 아님.클래스파일이 동적으로 로딩되고 초기화되기 때문에, 로드될 때 마다 변경됨|

## String Constant Pool은 GC의 대상이 될까?

GC의 대상은 Heap 영역이라고 배웠다. 그리고 String Constant Pool이 저장되는 곳은 Metaspace(Java8 이상) 이라고 한다.

그렇다면 String Constant Pool에 저장되 있는 데이터는 GC의 대상이 되지 않는 걸까?

정답은 그렇다. String Constant Pool에 저장되어 있는 String literals은 GC의 대상이 되지 않는다.

다만 여기서 확실히 알아야 할 점은, 모든 String 객체가 String Constant Pool에 저장되는 것이 아니라는 점이다.
-> 동적으로 생성된 String 객체는 Heap 영역에 저장된다.


> 출처
> https://deveric.tistory.com/123
> https://stackoverflow.com/questions/18406703/when-will-a-string-be-garbage-collected-in-java
	 
---

# Metaspace

Java7에서 Java로 넘어올 때, Perm 영역이 Metaspace 영역으로 이관되었다.
이관된 주요 이유는 Class, Metadata 로딩 과정에서 메모리 릭이 발생하였고, perm 영역의 크기를 고정적으로 설정해야 했기 때문에 메모리 부족으로 OOM이 발생하였다.  

위의 문제를 Perm 영역을 JVM의 `Native Memory`를 사용한 Metaspace영역으로 이관하여 해결하였다.

![](https://miro.medium.com/v2/resize:fit:513/0*rKZvTnuUkEc5LoXW.jpg)

## Native Memory

Native Memory는 Heap 영역의 바깥인 Off-Heap 공간을 의미하는 것으로 쉽게 시스템의 기본 메모리라고 생각하면 된다.

> 출처
> [삼성SDS 기술 블로그](https://m.post.naver.com/viewer/postView.nhn?volumeNo=23726161&memberNo=36733075)
> https://deveric.tistory.com/123


## Java Heap Shrinkage (공부 필요)



# 클래스 패스

클래스 패스는 JMV이 프로그램을 실행할 때, 클래스를 찾기 위한 기준이 되는 경로이다.

> 출처
> https://vvshinevv.tistory.com/70

