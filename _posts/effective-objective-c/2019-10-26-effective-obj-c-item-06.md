---
title: "[Effective Objective-C 정리][아이템 06] 프로퍼티를 이해하라"
category:
  - objective-c
tags: 
  - objective-c
---

# 아이템 6: 프로퍼티를 이해하라

## 요약

- `@property` 문법은 객체의 데이터 캡슐화를 정의하는 방법을 제공한다.
- 저장되는 데이터를 위한 정확한 방법(semantics)을 제공하기 위해 속성을 사용하라.
- 인스턴스 변수에 대한 모든 프로퍼티에는 따라야하는 시맨틱을 반드시 정의하라.
- iOS에서는 **nonatomic**을 사용하라. **atomic**을 사용하면 여러가지 성능 문제를 일으킨다.

## 프로퍼티 속성

컴파일러가 자동으로 생성하는 접근자 메서드(getter/setter)를 제어할 때 사용할 수 있는 속성들이다. 아래와 같이 사용할 수 있다.

```objc
@property (nonatomic, readwrite, copy) NSString *name;
```

### 원자성 (atomic)

- atomic
- nonatomic

자동 생성된 접근자 메서드가 원자적으로 동작하게 만드는 락(lock)을 포함할지에 대한 속성이다. 스스로 접근자 메서드를 정의하는 경우 메서드가 원자적으로 동작하도록 구현해야 한다. 원자성 속성을 설정하지 않으면 자동적으로 atomic으로 적용된다.

**atomic**으로 설정하면 접근자 메서드가 동작할 때 락이 사용된다.

**nonatomic**으로 설정하면 락이 사용되지 않는다.

### 읽기/쓰기

- readwrite
- readonly

어떤 접근자 메서드를 사용할지에 대한 속성이다. 읽기/쓰기 속성을 설정하지 않으면 자동적으로 readwrite 속성으로 적용된다.

**readwrite**는 getter, setter 모두 사용한다는 의미이다. 해당 속성일 경우 컴파일러는 두 메서드를 모두 생성한다.

**readonly**는 getter만 사용한다는 의미이다. 해당 속성일 경우 컴파일러는 getter 메서드만 생성한다. 프로퍼티를 외부에 읽기 전용으로 공개하고 싶을 때 사용할 수 있지만 클래스 확장 카테고리에서 내부적으로 읽기/쓰기로 재선언된다. (아이템 27 참고)

### 메모리 관리 시맨틱

- assign
- strong
- weak
- unsafe_unretained
- copy

프로퍼티는 데이터를 캡슐화 하고 그 데이터는 구체적인 소유권 시맨틱(symantic)이 있어야 한다.

**assign** setter는 CGFloat, NSInteger 같은 스칼라 타입에 사용하는 간단한 대입 연산이다.

**strong** 프로퍼티가 값을 소유한다는 것을 나타낸다. 새로운 값이 설정되면 먼저 그 값을 리테인(retain)하고 원래 값은 릴리즈(release)한다. 그런다음 리테인한 새 값을 설정한다.

**weak** 프로퍼티가 값을 소유하지 않는 것을 나타낸다. 새로운 값이 설정되면 이 값을 리테인하지 않을 뿐만 아니라 이전 값을 릴리즈하지도 않는다. 이것은 assign과 비슷하지만 프로퍼티가 가리키던 객체는 언제든 파괴될 수 있다. 파괴되면 프로퍼티는 nil로 설정된다.

**unsafe_unretained** 이것은 assign과 같지만 타입이 소유하지 않는 관계(리테인하지 않는)일 때 사용된다. 객체가 파괴되었을 때 weak와 다르게 값이 nil로 설정되지 않는다(unsafe).

**copy** strong과 비슷하게 값을 소유한다는 것을 나타낸다. 차이점은 값을 리테인하는 대신 복사하는 것이다. 이 속성은 세터로 전달된 값이 NSMutableString과 같은 가변 객체일 수 있을 경우에 사용된다. 가변 객체일 경우 프로퍼티에 설정된 후에 내용이 변경될 수 있기 때문에 가변일 수 있는 객체는 반드시 copy 속성을 사용해야 한다.

### 메서드 이름

- getter=\<name\>
- setter=\<name\>

**getter=\<name\>** 자동 생성될 getter의 이름을 정의한다.

```objc
@property (getter=isOn) BOOL on;
```

**setter=\<name\>** 자동 생성될 setter의 이름을 정의한다. 이 메서드는 보통 사용되지 않는다.

```objc
@property (setter=setName:) NSString *name; // 예시를 위한 것이고 보통 이렇게 사용하지 않는다.
```