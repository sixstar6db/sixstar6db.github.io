---
title: "[java] lamda 기본-동작파라미터화"
excerpt: "lamda"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/lamda-basic/

toc: true
toc_sticky: true

date: 2022-04-15
last_modified_at: 2022-04-15
---

## 🦥 람다란?

**동작 파라미터화**
  - 익명클래스처럼 이름이 없으면서, 인수로 전달 또는 변수로 저장 가능
  - 함수형 인터페이스(하나의 추상메서드 선언)만 람다 사용 가능
  - 아래는 가장 기본 형태로 사용하는 Predicate, Function, Consumer 등을 람다로 표현한 예제이다.
  
```java
public static <T> List<T> filter(List<T> list , Predicate<T> predicate, Predicate<T> predicate2){
		List<T> result = new ArrayList<>();
		for(T t : list) {
			if(predicate.or(predicate2).test(t)) result.add(t);
		}
		return result;
	}
	
	public static <T> void forEach(List<T> list, Consumer<T> c) {
		for(T t : list) {
			c.accept(t);
		}
	}
	
	public static <T, R> List<R> map(List<T> list, Function<T,R> f){
		List<R> results = new ArrayList<>();
		for(T t : list) {
			results.add(f.apply(t));
		}
		return results;
	}
	
	public static void main(String[] args) {
		
		List<Integer> samples = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
		List<String> samples2 = Arrays.asList("a", "ab", "abc");
		
		//1. predicate test
		System.out.println("----predicate test----");
		List<Integer> filteredNum = filter(samples, (i) -> i%2 != 0, (i) -> i > 5);
		System.out.println(filteredNum);
		
		//2. consumer test
		System.out.println("----consumer test----");
		forEach(samples, (i) -> System.out.println(i));
		
		//3. function test
		System.out.println("----consumer test----");
		List<Integer> lengths = map(samples2, s -> s.length());
		System.out.println(lengths);

		// 1. 불리언 표현
		Predicate<List<String>> p = list -> list.isEmpty();
		System.out.println(p.test(Collections.emptyList()));
		
		//2. 객체 생성
		Supplier<Apple> s = () -> new Apple(10);
		System.out.println(s.get());
		
		//3. 객체에서 소비
		Consumer<Apple> c = apple -> System.out.println(apple.getWeight());
		c.accept(new Apple(50));

	}
```

  - 기본형 함수인터페이스 외에 특화된 형태도 존재한다.
  - 아래 예제에서 Predicate 를 사용했다면, int -> Integer 로 박싱하는 과정이 필요하지만, IntPredicate 를 사용하면 박싱이 필요없다.
  - IntPredicate 외에, DoublePredicate, IntConsumer, LongBinaryOperator, IntFunction 처럼 다양한 형식을 다루는 인터페이스가 있다.

```java
	    //4, IntPredicate
		//Boxing 발생
		Predicate<Integer> evenNumber1 = (i) -> i % 2 == 0;
		System.out.println(evenNumber1.test(101)); //101의 int 가 Integer로 박싱됨. 
		
		//Boxing으로 인한 성능 이슈 없음
		IntPredicate evenNumber2 = (int i) -> i % 2 == 0;
		System.out.println(evenNumber2.test(101));
```   

  - 기본형 외에도 다른 특화된 형태의 함수형 인터페이스에 대한 간단한 예제를 만들어보았다.

```java
        //5. LongBinaryOperator
		// <Long, Long> -> Long
		LongBinaryOperator op1 = (i , j) -> i * j;
		System.out.println(op1.applyAsLong(10L, 100L));
		
		//6. BiFunction
		// <T, U> -> R
		BiFunction<String, Integer, Long> bi = (String operator, Integer i) -> {
			if(operator.equals("*")) {
				return (long) (i * i);
			}else {
				return (long) (i + i);
			}
		};
		System.out.println(bi.apply("*", 10));

		//7. BiPredicate
		// < T, U> -> Boolean
		BiPredicate<String, Integer> bp = (x, i) -> Integer.parseInt(x) == i;
		System.out.println(bp.test("2", 2));

		//8. IntFunction
		IntFunction<String> intFunc = i -> "result : " + i;
		System.out.println(intFunc.apply(100));

		//9. ToIntFunction ( IntFunction 과 반대로 동작 )
		ToIntFunction<String> toIntFunc = s -> Integer.parseInt(s);
		System.out.println(toIntFunc.applyAsInt("10"));

```

|**함수형 인터페이스** | 함수 디스크립터 | 기본형 특화|
|------|---|---|
|Predicate<T> | T -> boolean | IntPreciate, LongPredicate, DoublePrecdicate|
|Consumer<T> | T -> void | IntConsumer, LongConsumer, DoubleConsumer|
|Function<T, R> | T -> R | IntFunction<R>, IntToDoubleFunction, IntToLongFunction, 
ToIntFunction<T> 등등|
|Supplier<T>|() -> T|IntSupplier, DoubleSupplier등|
|UnaryOperator<T>|T -> T|IntUnaryOperator, LongUnaryOperator, DoubleUnaryOpearot|
|BinaryOperator<T>|<T, T> -> T|IntBinaryOperator, LongBinaryOperator, DoubleBinaryOpearot|
|BiPredicate<L, R>|<L,R> -> Boolean||
|BiConsumer<T, U>|<T,U> -> void|ObjIntConsumer<T>, ObjLongConsumer<T>|
|BiFunction<T,U,R>|(T,U) -> R|ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>|

## 람다 예외에 대해
 - 기존 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 
 - 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의 하거나 또는 람다를 try/catch 로 감싸야 한다. 

```java
	@FunctionalInterface
	interface BufferedReaderProcessor{
		String process(BufferedReader reader) throws IOException;
	}
	
	public static void main(String[] args) throws IOException{
		
		
		BufferedReader reader = new BufferedReader(new FileReader("d:/insys_alarm.html"));
		BufferedReaderProcessor p = r -> r.readLine();
		String result1 = p.process(reader);
		System.out.println(result1);
		
		//Function 으로 만든다면?
		Function<BufferedReader , String> f = r -> {
			try {
				return r.readLine();
			} catch (IOException e) {
				throw new RuntimeException(e);
			}
		};
		
		String result2 = f.apply(reader);
		System.out.println(result2);
		
	}
```
