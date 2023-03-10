---
title: "[java] 자동 형변환(AutoBoxing, UnBoxing)"
excerpt: "자동 형변환"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/autoboxing/

toc: true
toc_sticky: true

date: 2023-03-10
last_modified_at: 2023-03-10
---

#### 자바는 기본형 타입에 대해 Wrapper class를 만들어 두었다.

>  Primitive Type - Wrapper class

 - boolean - Boolean / byte - Byte / char - Character / float - Float / double - Double / long - Long / shor - Short / int - Integer


> int는 primitive type 이고, Integr 는 Object 인테, 아래 코드는 어떻게 컴파일 오류가 발생하지 않을까?

```java
		List<Integer> li = new ArrayList<>();
		for(int i=0; i < 50; i += 2) {
			li.add(i);
		}
		
		int sum = 0;
		
		for(Integer i : li)
			if( i % 2 ==0)
				sum  += i;
```

> 런타임시 아래와 같이 컴파일러가 코드를 자동으로 형변환 시켜주기 때문이다. 실제 디컴파일러를 이용하여 확인해보면, 위 소스가 아래 소스로 변환된것을 볼 수 있다. 

```java
		List<Integer> li = new ArrayList<>();
		for(int i=0; i < 50; i += 2) {
			li.add(Integer.valueOf(i));
		}
		
		int sum = 0;
		
		for(Integer i : li)
			if( i.intValue() % 2 ==0)
				sum  += i.intValue();
```

 - 보통의 코드에서는 자동형변환에 대해 신경쓰지 않아도 되지만, 성능에 민감한 계산로직시에는 박싱/언박싱에 대한 비용이 발생하므로 주의할 필요가 있다.

 ```java
 public static List<Integer> asList(final int[] a){
		return new AbstractList<Integer>() {

			@Override
			public Integer get(int index) {
				return a[index]; //boxing
			}

			@Override
			public Integer set(int index, Integer element) {
				Integer oldVal = a[index]; //boxing
				a[index] = element; //unboxing
				return oldVal;
			}

			@Override
			public int size() {
				return 0;
			}
		};
	}
 ```

 - 예를 들어 위 코드의 성능은 좋지 않은편이다. 일반적인 상황에서는 문제없겠지만, 성능이 중요한 반복문등에서 저 방식을 쓰는것은 피해야 한다.
 - 그럼 autoboxing, unboxing 은 언제 사용할까? Map 이나 Set 과 같은 collection 에서는 primitive type 을 사용할 수 없으니, 그 때 사용하면 좋다.
 - 그리고 꼭 필요한게 아니라면 Primitive Type 을 사용하도록 하자.