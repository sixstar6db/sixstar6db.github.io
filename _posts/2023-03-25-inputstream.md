---
title: "[java] Inputstream and InputStreamReader"
excerpt: "Inputstream"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/inputstream/

toc: true
toc_sticky: true

date: 2023-03-24
last_modified_at: 2023-03-24
---

## 개요 

 - 우연히 InputStreamReader 에 관해 검색하던중, 좋은 블로그 글을 읽게되어 간단히 정리하고 싶어져서 글을 올린다.

 - Java는 문자열을 처리할때, 내부 메모리 상에서는 UTF-16 인코딩을 사용하고, 송수신에서는 UTF-8, 문자열 입/출력시에는 사용자지정 인코딩 또는 운영체제의 기본 인코딩 값으로 문자열을 인코딩한다.

### UTF-8, UTF-16
 - UTF-8은 1byte~4byte 가변방식으로, 1byte 는 아스키코드와 동일하다. 영문은 2byte, 한글은 3byte를 사용한다. 
 - UTF-16은 거의 모두 2byte로 되어 있다.

### InputStream
 - java 의 기본 입력 스트림은 InputStream 이다. 
 - System 클래스의 in 변수는 콘솔, 명령줄 인수등을 통해 입력을 받을 수 있다. 

```java
public static void inputStream() throws IOException {
	InputStream in = System.in;
	int read = in.read();
	System.out.println(read);
	System.out.println((char)read);
}
```

- 콘솔창에서 문자열 하나를 입력하면, 해당문자에 해당하는 아스키코드 10진수 값이 출력된다. 그런데 한글은 이상한 값이 나온다. '가'를 입력하면 10진수 값은 234 가 나오고, 문자는 'ê' 가 나온다. 

- 파일인코딩이 UTF-8 이므로 입력값은 UTF-8 인코딩 되어, 3byte 로 inputstream 에 읽혀졌을 것이다.

- 유니코드(UTF-8) 표를 찾아보면 '가'는234, 176, 128 3개의 10진수값(3byte)으로 이루어져 있고, 유니코드로는 AC00 이다. 즉, InputStream의 read 메소드는 1byte 밖에 못 읽기 때문에, 첫번째 1byte 인 234 만 읽은 것이고,  234에 해당하는 unicode 문자가 출력된 것이다. 
- 234 는 16진수로 00EA이고, java 내부적으로 UTF-16으로 인코딩되므로, 유니코드(UTF-16 Encoding)에 라틴소문자 'ê'에 매핑되게 된다. 그래서 'ê' 출력이 된 것이다.


### InputStreamReader

 - InputStream은 1byte 씩 읽어올 수 있고, 읽혀진 byte 의 10진수 값은 UTF-16으로 인코딩되어 저장된다. 때문에 한글은 1byte 만 읽혀져서 이상한 문자가 출력되게 된다. 
 - InputStreamReader 는 InputStream 의 byte 단위로 읽혀진 것을 문자(Character) 단위 데이터로 변환시킨다.


```java
public static void inputStreamReader() throws IOException {
	InputStream in = System.in;
	InputStreamReader inReader = new InputStreamReader(in);
	int read = inReader.read();
	System.out.println(read);
	System.out.println((char)read);
}
``` 

 - 위 코드에서는 InputStreamReader 를 사용했다. 한글 '가' 를 입력하면 10진수값 44032가 나오는데 이는 16진수 'AC00' 이다. AC00은 유니코드(UTF16)표 상에서 '가'에 해당되고, 콘솔창에 정상적으로 '가' 라고 출력되는 것이다. 

 - InputStreamReader는 내부적으로 Charset 을 가지고 있다. Charset 은 bytes 와 16bit 유니코드 문자사이를 변환하는 역할을 한다. java api 에 보면, 다음과 같이 기술되어 있다. 
 `A named mapping between sequences of sixteen-bit Unicode`

 ```java
 System.out.println(
				Charset.defaultCharset().decode(ByteBuffer.wrap("가".getBytes()))
		);
 ```

- 다음과 같이 char[] 을 이용해, 문자열을 받을수 도 있다.

 ```java
 public static void inputStreamReader() throws IOException {
	InputStream in = System.in;
	InputStreamReader inReader = new InputStreamReader(in);
	char[] ch = new char[3];
	int read = inReader.read(ch);
	for(char c : ch) {
		System.out.println(c);
	}
}
 ```

 - 즉, InputStreamReader 는 바이트단위 데이터를 문자 단위 데이터로 처리할수 있도록 변환해주며, char 배열을 이용해 문자열도 받을수 있다.


### BufferedReader

 - 문자열 같은 경우 InputStreamReader 는 별도로 버퍼를 계속 만들어줘야 했었다.

```java
public static void bufferedReader() throws IOException {
	InputStream in = System.in;
	BufferedReader inReader = new BufferedReader(new InputStreamReader(in));
	String readLine = inReader.readLine();
	System.out.println(readLine);
}
```

 - InputStream 을 통해 byte 단위로 데이터를 받고, 그 데이터를 문자 형태로 변환하기 위해 InputStreamReader 로 감싼다. 거기에 Buffer 를 사용하여 문자열을 처리할 수 있도록 BufferedReader 로 감싼다. 기본버퍼크기는 8192개의 문자를 저장할 수 있다. 버퍼가 다 차거나 일정 조건이 되면 프로그램으로 데이터를 보내버려 버퍼를 비워버린다. 

### DataInputStream , DataOutputStream

 - DataInputStream/DataOutputStream 은 byte 단위 데이터를 데이터타입으로 받기 위해 Wrapping 한 클래스이다. 예를 들어, DataOutputStream 의 writeInt 메서드는,
 정수값을 4byte로 만들어 보낸다. wirteLong 메서드는 8byte로 보낸다. 받는 측은, 8byte를 받아, Long 타입 값으로 뽑아내면 된다. DataInputStream 의 readLong 메서드를 통해
 8byte 를 읽어 Long 타입 데이터로 변환할 수 있다. writeUTF 메서드는 처음 4byte는 문자열의 길이를 보내고, 이후 실제 문자열 값의 UTF8인코딩된 값을 보내게 된다. 
 받는 측에서는 readUTF를 통해 변환된 데이터로 받을 수 있다.

```java

ServerSocket serverSocket = new ServerSocket(8000);
Socket socket = serverSocket.accept();
InputStream in = socket.getInputStream();
int read;
while((read = in.read())!=-1) {
	System.out.println(read);
}

``` 

 - 예를 들어, 위와 같은 서버 소켓 프로그램이 있고, 아래와 같이 클라이언테에서 데이터를 보낸다면

```java
Socket socket = new Socket("localhost", 8000);
DataOutputStream outputStream = new DataOutputStream(socket.getOutputStream());
outputStream.writeUTF("이상국");
```

-->

0
9
236
157
180
236
131
129
234
181
173

 - 처음 2byte는 문자열 byte 길이(UTF-8에서 한글 1글자는 3byte)이고, 236, 157, 180 은 한글 '이'를 유니코드 3byte 십진수 값으로 나타낸 것이다.

 - 236, 157, 180 은 UTF16에서는 16진수 C774에 매핑된다. 4bit 가 16진수 한글자를 표현하므로, 4글자면 2바이트가 됨. 즉 한글이 UTF8에서는 3byte, UTF16에서는 2byte로 나타낸다.
