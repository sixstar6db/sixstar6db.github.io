---
title: "[java] Generic Wildcard의 super 와  extends"
excerpt: "generic"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/generic-wild/

toc: true
toc_sticky: true

date: 2023-04-20
last_modified_at: 2023-04-20
---

## 개요 

 - 제너릭 wildcard에 대해서 알아보고자 한다. generic 에 보면, <? super T> : lower bound 또는 <? extends T> : upper bound 형태의 제너릭 wildcard 를 사용하는 것을 자주 볼 수 있다.
 항상 super 와 extends 적용되는 부분에 혼란이 있어, 정리를 해볼까 한다. 

 - <? extends T> : upper bound(상위한정) 은 타입을 T로 한정한다. 
 - <? extends T> : upper bound(하위한정) 은 가장 하위 타입을 T로 한정한다. 

 - 아래 소스에 보면, 두 메서드에 arguments 에 인자값을 넣을때, 상위타입에 하위타입의 인자는 들어가지만, 하위타입에 상위타입의 인자는 컴파일 에러가 난다. 당연한 얘기인데, 이 원리를 가지고 와일드카드의 super 와 extends 를 살펴보고자 한다. 
 - 

```java
public static void main(String[] args) {
	Number num1 = 100;
	Integer num2 = 200;
	printNumber(num2);
	printInteger(num1);//compile error
}

public static void printNumber(Number num) {
	System.out.println(num);
}
public static void printInteger(Integer num) {
	System.out.println(num);
}
```

### Collections로 와일드 카드 이해하기

 - 자바 util 패키지에 Collections 클래스에 구현한 메서드를 보면 제너릭 와일드 카드가 사용되어진 곳이 많다. 
 - 예를 들어, 아래 코드에 Collections.max(...) 를 보면 첫번째 인자는 Iterator 를 통해 메서드 안에서 T 타입의 값을 꺼내어 사용하고 있다. 
 두 번째 Comparator 인자는 값을 꺼내어 쓰기 보다는 Comparator 의 메서드를 통해 외부 인자를 파라미터로 받아 메서드를 수행하고 있다. 
 아레 메서드에서는 next, candidate 2 개의 인자를 받는데 둘 다 T 타입이다. 즉, compare 메서드는 T의 하위타입을 인자를 받기 위해 최소한 T타입 또는 T타입보다 상위타입어야햐 한다. 

```java
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp) {

	Iterator<? extends T> i = coll.iterator();
	T candidate = i.next();

	while (i.hasNext()) {
		T next = i.next();
		if (comp.compare(next, candidate) > 0)
			candidate = next;
	}
	return candidate;
}
```

 - 아래 예를 보면, comparing 메서드는 function 을 인자로 받아, Comparator<T>를 반환한다. function은 T인자를 받아, U 인자로 변환시키는데, U인자는 compareTo 메서드를 갖는,, 즉
 Comparable 을 구현한 객체이어야 한다. Function 을 통해 비교 가능한 객체로 변환하여, Comparator를 람다형태로 반환하는 것이다.
 여기서 `U extends Comparable<? super U> ` 를 보면, Comparable은 U의 상위타입을 파라미터 타입으로 갖는다. U의 내부값을 꺼내서 사용하는게 아니라 U의 compareTo 메서드를 통해 외부인자를 받아 실행시키기 때문이다. compareTo 인자는 U보다 상위타입이면 안되기 때문에,,하위한정 U로 선언한것이다.
 - 두번째 라인에 `Function<? super T, ? extends U>`는 T를 하위 한정으로 선언했는데, 그 이유는 Comparator<T>로 반환하기 위해서는 `(c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2))`로 인자를 전달해야 한다. c1 과 c2는 최소한 T타입이상이어야 T의 하위타입들을 인자로 받을 수 있기 때문에 `Function<? super T, ? extends U>`로 선언한 것이다. 그리고 반환 타입이 `? extends U` 인 것은, `keyExtractor.apply(c1)`의 결과가 U의 하위타입이기만 하면, compareTo를 사용할 수 있기 때문이다. 

```java

public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
			Function<? super T, ? extends U> keyExtractor){
		return (Comparator<T> & Serializable)
			(c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
}

```

- 결론적으로, 제너릭으로 선언된 객체의 파라미터타입으로 선언된 값을 꺼내어 처리하는 건, extends 로하고, 
- 제너릭으로 선언된 객체의 메서드를 통해 외부의 값을 인자로 전달할때는 super 로 선안하면 된다.
