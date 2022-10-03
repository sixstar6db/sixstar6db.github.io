---
title: "[java] Http Server 직접 구현(1)"
excerpt: "Http Server 직접 구현(1)"

categories:
  - service-develop
tags:
  - [java]

permalink: /http_server_implementation/

toc: true
toc_sticky: true

date: 2022-09-11
last_modified_at: 2022-09-11
---

## 개요

> Http Web Server 를 직접 구현해보기

 - 최소한의 라이브러리만을 사용하여 Http Web Server 를 구현해본다. 
 - 박재성님의 자바 웹 프로그래밍 Next Step 을 참고 하지만, 그대로 배끼지 않고 다르게 구현해 볼것이다.

> 기본적인 Maven Repository 프로젝트를 생성한다.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.sk.http</groupId>
	<artifactId>server</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	</properties>

	<dependencies>
		<!-- unit testing -->
		<dependency>
			<groupId>org.junit.jupiter</groupId>
			<artifactId>junit-jupiter-engine</artifactId>
			<version>5.5.2</version>
			<scope>test</scope>
		</dependency>
		
		<dependency>
			<groupId>org.assertj</groupId>
			<artifactId>assertj-core</artifactId>
			<version>3.13.2</version>
			<scope>test</scope>
		</dependency>

		<!-- logger -->
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.1.2</version>
		</dependency>
	</dependencies>

	<build>
		<finalName>my-http-server</finalName>
		<sourceDirectory>src/main/java</sourceDirectory>
		<testSourceDirectory>src/test/java</testSourceDirectory>
		<testOutputDirectory>target/test-classes</testOutputDirectory>

		<resources>
			<resource>
				<directory>src/main/resources</directory>
			</resource>
		</resources>

		<plugins>
			<plugin>
				<artifactId>maven-eclipse-plugin</artifactId>
				<version>2.9</version>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>utf-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<version>2.4</version>
				<executions>
					<execution>
						<id>copy-dependencies</id>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

- 'copy-dependencies' 는 pom.xml 에 정의된 dependeicies 를 지정된 위치로 복사해둔다.

## Socket을 이용한 기본적인 Server 프로그램 작성

```java
package com.sk.http;

import java.io.BufferedInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	public void run() {
		try(ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);) {

        	while(!Thread.currentThread().isInterrupted()) {
        		log.info("server start wating...");
        		
        		Socket accept = serverSocket.accept();
        		
        		InputStream inputStream = new BufferedInputStream(accept.getInputStream());
        		
        		int readCount = 0;
        		byte[] buffer = new byte[1024];
        		
        		//inputstream -> buffer(byte[] size 만큼 읽어들임)
        		while((readCount = inputStream.read(buffer))!=-1) {
        			//byte -> string
        			String result = new String(buffer);
        			CharSequence subSequence = result.subSequence(0, readCount);//읽은 byte 수 만큼 추출
        			log.info("req data : " + subSequence.toString());
        		}
        		inputStream.close();
        	}
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
}
```

- 위의 예제는 client 에서 데이터를 보내고, Server 에서 데이터를 읽었다.  다음은 input과 output 을 주고 받는 것을 해보았다. 

```java
//서버
public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	public void run() {
		try(ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);) {

    		log.info("server start wating...");
    		
    		Socket accept = serverSocket.accept();
    		
    		InputStream inputStream = new BufferedInputStream(accept.getInputStream());
    		
    		int readCount = 0;
    		byte[] buffer = new byte[1024];
    		
    		//inputstream -> buffer(byte[] size 만큼 읽어들임)
    		while((readCount = inputStream.read(buffer))!=-1) { //다음버퍼에 데이터가 읽혀지기를 기다린다.
    			//byte -> string
    			String input = new String(buffer);
    			
    			CharSequence subSequence = input.subSequence(0, readCount);//읽은 byte 수 만큼 추출
    			String result = subSequence.toString();
    			if(result.equals("0")) break; // Client 에서 0을 전송하면, 결과 수신을 멈추고 종료 처리.
    			log.info("req data : " + result);
    		}

    		OutputStream outputStream = accept.getOutputStream(); //응답을 위한 outputstream
    		
    		outputStream.write("success".getBytes());
    		
    		outputStream.flush();
    		outputStream.close();
    		inputStream.close();
    		
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
}
//클라이언트
public class ClientTest {
	
	private static final Logger log = LoggerFactory.getLogger(ClientTest.class);
	Scanner sc = new Scanner(System.in);
	
	@BeforeAll // test 전에 딱 1번 실행되는 어노테이션 선언
	public static void init() {
		//Runnable 인터페이스를 구현하면, Thread 를 생성할 수 있다.메인쓰레드와 다른 쓰레드에서 실행된다.
		Thread t = new Thread(new Server());
		t.start();
		log.info("server start!");
	}

	@Test
	public void test() {
		try {
			Socket socket = new Socket("localhost", 8080);
			OutputStream outputStream = socket.getOutputStream();
			
			while(true) {//순회하며 입력값을 계속해서 서버로 보낸다. 
				
				System.out.println("입력-->");
				String input = sc.nextLine();
				outputStream.write(input.getBytes());
				outputStream.flush();
				
				if(input.equals("0")) break; //0을 입력하면, 그만 보낸다. 
				
			}
			
			InputStream inputStream = socket.getInputStream();
			
			int readData = 0;
			//1byte 씩 읽어들이기
			while((readData = inputStream.read()) > 0) {
				log.info("response is " + (char)readData);
			}
			
			inputStream.close();
			
			socket.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	@AfterAll
	public static void end() {
		log.info("server end!");
	}

}
```

- 다음은 byte 타입으로 받지 않고, char 타입으로 주고받겠다. BufferedReader 를 사용한다. 

```java
//서버
public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	public void run() {
		try(ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);) {

    		log.info("server start wating...");
    		
    		Socket accept = serverSocket.accept();
    		
    		BufferedReader reader = new BufferedReader(new InputStreamReader(accept.getInputStream(), StandardCharsets.UTF_8));
    		
    		String readLine = ""; 
    		// readLine()은 line 을 \n 또는 \r 으로 끝나는 부분을 하나의 line 으로 본다.  client 에서 라인 단위로 보내야 한다. 
    		
    		while((readLine = reader.readLine()) != null) {
    			log.info("req data : " + readLine);
    			if(readLine.equals("")) break;
    		}
    		
    		PrintWriter writer = new PrintWriter(accept.getOutputStream());
    		
    		writer.println("success\r\n");
    		writer.println("\r\n");
    		
    		writer.close();
    		reader.close();
    		
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
}
//클라이언트(테스트코드)
@Test
public void test() {
	try {
		Socket socket = new Socket("localhost", 8080);
		PrintWriter writer = new PrintWriter(socket.getOutputStream());
		writer.print("first\r\n");
		writer.print("second\r\n");
		writer.print("\r\n");
		writer.flush();
		
		BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
		
		String line = "";
		while((line = reader.readLine())!=null) {
			if(line.equals("")) break;
			log.info("response is " + line);
		}
		
		writer.close();
		reader.close();
		
		socket.close();
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

- readLine()은 라인단위로 읽어들이며, \r 또는 \n 가 있는 곳이 라인이 끝나는 지점으로 본다. 만약 \r 또는 \n 이 넘어오지 않으면, 계속해서 들어올때까지 기다릴것이다. 
- 이스케이프 문자 \r 과 \n 의 의미에 대해 잠시 알아보자, \r 은 캐리지리턴(CR)으로 커서를 행의 앞으로 이동한다는 의미이고, \n은 LineFeed(LF) 라고 해서 다음 행으로 바꾸는 것을 의미한다. 보통 윈도우에서는 CR + LF 를 합쳐서 줄바꾸기 문자로 사용하며, 리눅스 계열에서는 LF 만으로 가능한듯 하다. 

## Socket을 이용한 기본적인 HTTP 서버

 - HTTP 프로토콜 메시지를 주고 받는 서버-클라이언트(테스트) 코드를 작성하였다. 

```java
public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	public void run() {
		try(ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);) {

    		log.info("server start wating...");
    		
    		Socket accept = serverSocket.accept();
    		
    		BufferedReader reader = new BufferedReader(new InputStreamReader(accept.getInputStream(), StandardCharsets.UTF_8));
    		
    		String readLine = ""; 
    		// readLine()은 line 을 \n 또는 \r 으로 끝나는 부분을 하나의 line 으로 본다.  client 에서 라인 단위로 보내야 한다. 
    		
    		while((readLine = reader.readLine()) != null) {
    			log.info("req data : " + readLine);
    			if(readLine.equals("")) break;
    		}
    		
    		String result = "lee sang kook";
    		byte[] body = result.getBytes();
    		int lengthContent = body.length;
    		
    		DataOutputStream outStream = new DataOutputStream(accept.getOutputStream());
    		
    		outStream.writeBytes("HTTP/1.1 200 OK\r\n");
    		outStream.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
    		outStream.writeBytes("Content-Length: "+lengthContent + "\r\n");
    		outStream.writeBytes("\r\n"); // header 와 body의 구분은 한줄의 공백이 있어야 한다. 
    		
    		outStream.write(body, 0, lengthContent);
    		outStream.writeBytes("\r\n");
    		
    		outStream.flush();
    		
    		outStream.close();
    		reader.close();
    		
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
	
	public static void main(String[] args) {
		new Server().run();
	}
}
```

```java
package com.sk.http;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class ClientTest {
	
	private static final Logger log = LoggerFactory.getLogger(ClientTest.class);
	
	@BeforeAll // test 전에 딱 1번 실행되는 어노테이션 선언
	public static void init() {
		//Runnable 인터페이스를 구현하면, Thread 를 생성할 수 있다.메인쓰레드와 다른 쓰레드에서 실행된다.
		Thread t = new Thread(new Server());
		t.start();
		log.info("server start!");
	}

	@Test
	public void test() {
		try {
			Socket socket = new Socket("localhost", 8080);
			PrintWriter writer = new PrintWriter(socket.getOutputStream());
			writer.print("GET / HTTP/1.1\r\n");
			writer.print("Host: localhost:8080\r\n");
			writer.print("Connection: keep-alive\r\n");
			writer.print("\r\n");
			writer.flush();
			
			BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			
			StringBuilder lines = new StringBuilder();
			
			while(true) {
				String line = reader.readLine();
				if(line == null)break;
				lines.append(line + "\r\n");
			}
			
			String content = lines.toString();
			log.info("\r\n" + content);
			
			writer.close();
			reader.close();
			socket.close();
			
			Assertions.assertThat(lines).contains("HTTP/1.1 200 OK");
			
			
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	@AfterAll
	public static void end() {
		log.info("server end!");
	}
}
```

## Thread 를 사용하여, 요청/응답 처리 분리

 - 보통 서버는 클라이언트의 요청 처리를 하나의 쓰레드가 아닌 멀티 쓰레드로 처리한다. 요청/응답 처리 부분을 쓰레드로 분리하여 구현하여 보자.
 - 쓰레드로 분리하였기 때문에, junit 에서 테스트 해보면, 'request wating...' 로그가 2번 찍히게 된다. 

```java
public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	public void run() {
		
		try (ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);){
			
    		//connection 생성시까지 block
    		while(!Thread.currentThread().isInterrupted()) {
    			log.info("request wating...");
    			Socket socket = serverSocket.accept();
				//생성된 socket을 인자로 받아 요청데이터를 처리하는 쓰레드
    			Thread thread = new Thread(new Worker(socket));
    			thread.start();
    		}
    		
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
	
	/*요청,응답 처리 객체*/
	private class Worker implements Runnable{
		
		private Socket socket;
		
		public Worker(Socket socket) {
			this.socket = socket;
		}

		@Override
		public void run() {
			
			try(BufferedReader reader 
					= new BufferedReader(new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8));
				DataOutputStream outStream = new DataOutputStream(socket.getOutputStream());) {
	    		
	    		String readLine = ""; 
	    		// readLine()은 line 을 \n 또는 \r 으로 끝나는 부분을 하나의 line 으로 본다.  client 에서 라인 단위로 보내야 한다. 
	    		
	    		while((readLine = reader.readLine()) != null) {
	    			log.info("req data : " + readLine);
	    			if(readLine.equals("")) break;
	    		}
	    		
	    		String result = "lee sang kook";
	    		byte[] body = result.getBytes();
	    		int lengthContent = body.length;
	    		
	    		outStream.writeBytes("HTTP/1.1 200 OK\r\n");
	    		outStream.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
	    		outStream.writeBytes("Content-Length: "+lengthContent + "\r\n");
	    		outStream.writeBytes("\r\n");
	    		
	    		outStream.write(body, 0, lengthContent);
	    		outStream.writeBytes("\r\n");
	    		
	    		outStream.flush();
	    		
			}catch(IOException e) {
				log.error(e.getMessage(), e);
			}
		}
	}
	
	public static void main(String[] args) {
		new Server().run();
	}
}
```

> main 메서드와 junit 차이

- 여기서,, main 메서드에서 Server 를 동작할때와 junit 으로 테스트 할때, 쓰레드 종료되는 부분이 차이가 있다. main 메서드로 실행할 경우에는 서버가 죽지 않고, 
계속해서 요청 대기를 하지만, junit 은 테스트가 종료되면서 끝나버린다. 일반적으로 main 메서드가 종료되어도, 자식 쓰레드가 종료될때까지 프로세스가 종료되지 않지만, 
junit 에서는 테스트가 종료되면 jvm 을 종료하여 프로세스가 끝나버린다. 


 - request/response 를 처리할 Worker 클래스를 생성하였다. ServerHandler를 통해 request를 읽고, response 를 쓰는 인터페이스를 정의하였다.
 - Server 에서는 최대한 상세 구현에 의지 하지 않도록 만들어갈 예정이다. 
 - ServerHandlerFactory : ServerHandler 생성 팩토리, HttpServerHandler 는 HttpServerHandler 를 생성할 수 있는 팩토리 객체를 만들어낸다.

```java
public class Server implements Runnable{
	
	private static final Logger log = LoggerFactory.getLogger(Server.class);
	
	private static final int DEFAULT_PORT = 8080;
	
	private ServerHandlerFactory serverHandlerFactory;
	
	public Server(ServerHandlerFactory serverHandlerFactory) {
		this.serverHandlerFactory = serverHandlerFactory;
	}
	
	public void run() {
		try (ServerSocket serverSocket = new ServerSocket(DEFAULT_PORT);){
    		//connection 생성시까지 block
    		//생성된 socket을 인자로 받아 요청데이터를 처리하는 쓰레드 생성
    		while(!Thread.currentThread().isInterrupted()) {
    			log.info("request wating...");
    			Socket socket = serverSocket.accept();
    			Thread thread = new Thread(new Worker(serverHandlerFactory.newServerHandlerInstance(socket)));
    			thread.start();
    		}
		} catch (IOException e) {
			log.error(e.getMessage(), e);
		}
	}
	
	private class Worker implements Runnable{
		
		private ServerHandler serverHandler;
		
		public Worker(ServerHandler serverHandler) {
			this.serverHandler = serverHandler;
		}

		@Override
		public void run() {
			try {
				serverHandler.readRequst();
				serverHandler.writeResponse(null);
				serverHandler.close();
			} catch (IOException e) {
				// TODO 오류처리
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		new Server(HttpServerHandler.newServerHandlerFactory()).run();
	}
}
```

## 소스코드 리팩토링하기

 - HTTP외에 다른 프로토콜의 TCP 소켓 통신도 처리할 수 있도록 인터페이스를 통한 주요 기능 추상화
 - 상세구현에 의존하지 않고, 인터페이스에 의존하도록 변경
 - 주요 인터페이스는 아래와 같다. 
   1. ProtocolProcessor : request 를 받아 비즈니스 로직을 수행 후, response 생성
   2. ServerHandler : 소켓을 통해 request 를 읽고, response 쓰기 작업을 수행
   3. Request : request data 저장
   4. Response : response data 저장


-  ExecutorService 를 이용하여, 멀티쓰레드를 적용한다.
-  크롬에서 창을 여러개 띄워놓고 동일 url로 접속하면, 멀티쓰레드 적용이 되지 않았다. 처음에는 서버 프로그램 문제인줄 알았으나, 알고 보니 크롬 브라우저상에서 동일 url 에 대해 멀티쓰레드가 적용되지 않았다. 결국 클라이언트단에서 동시 접속 처리를 하지 않았던 것이다. 

```java

```

# Reference

 - 자바 웹 프로그래밍 Next STep (박재성)