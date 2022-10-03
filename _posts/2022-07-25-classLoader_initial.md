---
title: "[java] java 클래스 로딩 초기화"
excerpt: "java 클래스 로딩 초기화"

categories:
  - java-advanced
tags:
  - [java]

permalink: /java-advanced/classLoader_initial/

toc: true
toc_sticky: true

date: 2022-07-25
last_modified_at: 2022-07-25
---

## JVM 클래스 로딩 시점

 - 클래스 로딩이란, 클래스로더에 의해 JVM 메모리에 클래스를 올려놓는 것을 말한다. 

 - java -verbose:class 'class명' 을 사용하면 클래스 로딩을 디버그 할 수 있다.

```java
package com.sk.app;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Set;
import java.util.stream.Collectors;

public class ReflectionUtil {
	
	public static final int finalVar = 0;
	
	public static int nonFinalVar = 0;
	
	private ReflectionUtil() {}

	static {
		System.out.println("ReflectionUtil 초기화");
	}
	
	/*sigleton lazy initialization*/
	private static class ReflectionUtilHolder{
		public static ReflectionUtil INSTANCE = new ReflectionUtil();
	}
	
	public static ReflectionUtil getInstance() {
		return ReflectionUtilHolder.INSTANCE;
	}
	
	//private static final Logger logger = LoggerFactory.getLogger(ReflectionUtil.class);
	
	@SuppressWarnings("rawtypes")
	public Set<Class> findClassesByPackageName(String packageName){
		InputStream resourceAsStream = getClassLoader().getResourceAsStream(packageName.replaceAll("[.]", "/"));
		BufferedReader reader = new BufferedReader(new InputStreamReader(resourceAsStream));
		return reader.lines()
				.filter(line -> line.endsWith(".class"))
				.map(line -> getClass(line, packageName))
				.collect(Collectors.toSet());
	}
	
	@SuppressWarnings("rawtypes")
	private Class getClass(String className, String packageName) {
		try {
			return Class.forName(packageName + "." + className.substring(0, className.lastIndexOf(".")));
		}catch(ClassNotFoundException e) {
		//	logger.error(e.getMessage());
		}
		return null;
	}

	private ClassLoader getClassLoader() {
		ClassLoader cl = Thread.currentThread().getContextClassLoader();
		if(cl == null) {
			cl = this.getClass().getClassLoader();
			if(cl == null)
				cl = ClassLoader.getSystemClassLoader();
		}
		return cl;
	}
}
```

```java
package com.sk.app;

public class Main {
	public static void main(String[] args) {
		System.out.println(ReflectionUtil.finalVar);
	}
}

```

 - ReflectionUtil 의 static final 변수를 호출하면 ReflectionUtil이 로딩되지 않는다. 

 
```java
package com.sk.app;

public class Main {
	public static void main(String[] args) {
		System.out.println(ReflectionUtil.finalVar);
	}
}

```

- java -verbose:class com.sk.app.Main 수행

`[Loaded com.sk.app.ReflectionUtil from file:/D:/win10Dev/eclipse_for_work/channel-app-sts/workspace/9_nextstep/my-http-server-2/webapp/WEB-INF/classes/]`

 - ReflectionUtil 의 static 변수를 호출하면 ReflectionUtil이 로딩된다.

> 클래스 로딩되는 시점은

  1. 클래스 인스턴스 생성시
  2. 클래스 정적 변수 사용시(final선언된 것은 제외)
  3. 클래스 정적 메소드 호출시

> static 블락이 수행되는 시점은 클래스가 로딩 후 초기화를 위해 바로 진행됨.

 - 초기화 진행순서는 static 블록 > 정적 변수 > 생성자

> 클래스 로딩 및 초기화는 딱 한번만 수행되므로, 이것을 활용하여 싱글톤 객체를 생성할 수 있다. 

 - 이른 초기화 : 인스턴스가 생성을 위해 클래스 로딩
    
	`private static final ReflectionUtil instance = new ReflectionUtil();`

 - 지연 초기화

   위 예제처럼 ReflectionUtilHolder 활용, getInstance()인 정적 메소드 호출시 ReflectionUtil가 로딩된다.