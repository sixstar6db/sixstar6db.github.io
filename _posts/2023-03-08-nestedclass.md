---
title: "[java] 중첩 클래스(nested class)"
excerpt: "중첩 클래스(nested class)"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/nestedclass/

toc: true
toc_sticky: true

date: 2023-03-09
last_modified_at: 2023-03-09
---

#### 중첩클래스는 클래스 안에 다른 클래스가 선언된 모든 경우를 말한다. 

> inner class 와 static nested class

 - 중첩클래스는 크게 비정적 클래스와 정적 클래스라고도 하는데, 비정적 클래스는 inner class 라고 하고, 정적 클래스는 static nested class 라고 부른다.

> 예제 소스

```java
public class MyOuterClass {
	
	private int outerNum;
	
	public MyOuterClass(int outerNum) {
		this.outerNum = outerNum;
	}
	
	public MyInnerClass getInnerObj() {
		return new MyInnerClass(1);
	}

	/*inner 중첩 클래스*/
	class MyInnerClass{
		private int innerNum;
		
		public MyInnerClass(int innerNum) {
			this.innerNum = innerNum;
		}

		public void call() {
			System.out.println(outerNum + innerNum);
		}
	}
	
	/*정적 중첩 클래스*/
	static class MyStaticNestedClass{
		private int staticNestedNum;
		
		public MyStaticNestedClass(int staticNestedNum) {
			this.staticNestedNum = staticNestedNum;
		}

		public void call(MyOuterClass outer) {
			//System.out.println(outerNum);//정적 중첩클래스에서는 outer 클래스에 멤버변수에 접근할 수 없다.
		}
	}
	
	public static void main(String[] args) {
		MyOuterClass outerObj = new MyOuterClass(5);
		MyInnerClass myInnerClass = outerObj.new MyInnerClass(1);
		//Inner Class 객체 생성 방식
		//1.
		myInnerClass.call();
		//2.
		outerObj.getInnerObj().call();
		
	}
}
```

 - inner class에서는 outer class의 멤버변수에 자유롭게 접근이 가능하다. 
 - inner class가 객체가 되기 위해서는 outer class가 먼저 객체가 된 상태에서만 가능하다. 
 - inner class 인스턴스는 outer class 인스턴스에 대한 숨은 참조를 갖는데, 이것은 garbage 컬렉션에서 수거되지 못하여 메모리 누수가 생길수 있다.
 때문에 되도록이면 정적 중첩 클래스로 선언하는 것이 좋다.
 - 정적중첩클래스(static nested class)는 사실상 독립적인 클래스와 동일하다고 봐도 되지만, 중첩으로 선언하는 주 목적은 가독성을 위해서다. 논리적으로 연관있는 class를 
 한곳에 모아둠으로써 코드의 문맥을 이해하기 쉬워진다.