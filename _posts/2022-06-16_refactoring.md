---
title:  "[java] 리팩토링 맛보기"
excerpt: "리팩토링 맛보기"

categories: Java
tags:
  - [java]

toc: true
toc_sticky: true
 
date: 2022-06-16
last_modified_at: 2022-06-16
author_profile: false  
---

## 리팩토링 전

 -  다음 예제는 마틴파울러의 리팩토링 2판의 자바스크립트 소스 코드를 자바 버전으로 바꾸어 연습해본 것이다. 
 - 1장에 공연 청구서 출력 기능을 리팩토링 하는 과정이 나온다. 
 - 먼저 데이터 도메인 객체를 정의한다. 

```java
/*
 * 청구서
 */
@Data
public class Invoice {
	
    private String customer;
    private List<Performance> performances;
    
}
```

```java

/*
 * 공연 청구 정보
 */
@Data
@AllArgsConstructor
public class Performance {
	
    private String playId;
    private int audience;
    
}
```

```java
/**
 * 연극 정보 
 */
@Data
@AllArgsConstructor
public class Play {
    private String name;
    private String type;
    
}
```

 - 다음은 청구서를 출력하는 statement 메서드이다. 

```java
public class InvoiceStatement {
    public String statement(Invoice invoice, Map<String, Play> plays){
        int totalAmount = 0;
        int volumeCredits = 0;
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            Play play = plays.get(performance.getPlayId());
            
            int thisAmount = 0;

            switch(play.getType()){
                case "tragedy" :
                    thisAmount = 40000;
                    if(performance.getAudience() > 30)
                        thisAmount += 1000 * (performance.getAudience() - 30);
                    break;
                case "comedy" :
                    thisAmount = 30000;
                    if(performance.getAudience() > 20)
                        thisAmount += 10000 + 500*(performance.getAudience() - 20);
                    thisAmount += 300 * performance.getAudience();
                    break;
                default:
                    throw new IllegalStateException("알수없는 장르 " + play.getType());
            }
            
            // 포인트를 적립한다.
            volumeCredits += Math.max(performance.getAudience() - 30, 0);

            //희극 관객 5명 마다 추가 포인트를 제공한다. 
            if("comedy".equals(play.getType()))
                volumeCredits += Math.floor((double) performance.getAudience() / 5);

            //청구 내역을 출력한다. 
            result.append(play.getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
            totalAmount += thisAmount;
        }

        result.append("총액 :").append(String.format("%,.2f", (double) (totalAmount / 100)));
        result.append("\n적립포인트 :").append(volumeCredits).append("점");

        return result.toString();
    }
}
```

 - 다음은 테스트 메서드이다. 

```java
@Test
    void statement(){

        Play play1 = new Play("Hamlet","tragedy");
        Play play2 = new Play("As you Like It", "comedy");
        Play play3 = new Play("Othello", "tragedy");

        Map<String, Play> map = new HashMap<>();
        map.put("hamlet", play1);
        map.put("as-like", play2);
        map.put("othello", play3);

        Invoice invoice = new Invoice();
        invoice.setCustomer("BigCo");
        List<Performance> list = new ArrayList<>();
        list.add(new Performance("hamlet",55));
        list.add(new Performance("as-like",35));
        list.add(new Performance("othello",40));

        invoice.setPerformances(list);

        InvoiceStatement statement = new InvoiceStatement();
        String result = statement.statement(invoice, map);

        System.out.println(result);

        Assertions.assertThat(result).contains("Hamlet:650(55석)");
        Assertions.assertThat(result).contains("As you Like It:580(35석)");
        Assertions.assertThat(result).contains("Othello:500(40석)");
        Assertions.assertThat(result).contains("총액 :1,730.00");
        Assertions.assertThat(result).contains("적립포인트 :47점");
    }
``` 

 - 올바른 테스트인지는 모르겠으나, 일단 잘 통과함
 - 먼저 statement 메서드에 switch 문 부분을 메서드 추출한다. 
 - 값이 변하지 않는 performance, play 는 파라미터로 전달하고, thisAmount 변수는
 값을 변경하고 있기 때문에, 메서드 안에서 지역변수로 선언후 결과로 리턴해준다.
 - thisAmount 변수명은 result 로 변경한다. 그러면 변수의 역할을 쉽게 알 수 있다.


```java
private int amountFor(Performance performance, Play play) {
		int thisAmount = 0;
		switch(play.getType()){
		    case "tragedy" :
		        thisAmount = 40000;
		        if(performance.getAudience() > 30)
		            thisAmount += 1000 * (performance.getAudience() - 30);
		        break;
		    case "comedy" :
		        thisAmount = 30000;
		        if(performance.getAudience() > 20)
		            thisAmount += 10000 + 500*(performance.getAudience() - 20);
		        thisAmount += 300 * performance.getAudience();
		        break;
		    default:
		        throw new IllegalStateException("알수없는 장르 " + play.getType());
		}
		return thisAmount;
	}
```

 - 다음엔 play 변수를 제거한다. 
 - performance는 loop를 돌면서 값이 변하는데, play 는 performance 에서 매번
 가져오는 값이므로, 굳이 매개변수로 전달할 필요가 없다. 
 - play 같은 변수가 많아지면, 로컬범위에 변수들이 많아져 복잡해진다. 
 - 이런 리팩터링 기법은 '임의변수를 질의함수로 바꾸기' 라고 한다. 

 - plays 와 invoice 는 다른 메서드에서 같이 사용하기 때문에 멤버변수로 선언하였다.
 - playFor란 메서드를 만들어 performance 에서 Play 객체를 가져올수 있도록 하였다.


```java
public class InvoiceStatement {
	
	private final Map<String, Play> plays;
	private final Invoice invoice;
	
	public InvoiceStatement(Invoice invoice, Map<String, Play> plays) {
		this.invoice = invoice;
		this.plays = plays;
	}
	
    public String statement(){
        int totalAmount = 0;
        int volumeCredits = 0;
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            //Play play = plays.get(performance.getPlayId());
            
            int thisAmount = amountFor(performance);
            
            // 포인트를 적립한다.
            volumeCredits += Math.max(performance.getAudience() - 30, 0);

            //희극 관객 5명 마다 추가 포인트를 제공한다. 
            if("comedy".equals(playFor(performance).getType()))
                volumeCredits += Math.floor((double) performance.getAudience() / 5);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
            totalAmount += thisAmount;
        }

        result.append("총액 :").append(String.format("%,.2f", (double) (totalAmount / 100)));
        result.append("\n적립포인트 :").append(volumeCredits).append("점");

        return result.toString();
    }
    
    private Play playFor(Performance performance) {
    	return plays.get(performance.getPlayId());
    }

	private int amountFor(Performance performance) {
		int result = 0;
		switch(playFor(performance).getType()){
		    case "tragedy" :
		        result = 40000;
		        if(performance.getAudience() > 30)
		            result += 1000 * (performance.getAudience() - 30);
		        break;
		    case "comedy" :
		        result = 30000;
		        if(performance.getAudience() > 20)
		            result += 10000 + 500*(performance.getAudience() - 20);
		        result += 300 * performance.getAudience();
		        break;
		    default:
		        throw new IllegalStateException("알수없는 장르 " + playFor(performance).getType());
		}
		return result;
	}
}
```

 - 다음은 포인트 적립 부분을 메서드 추출한다. 

```java
volumeCredits += volumeCreditsFor(performance);
```

```java

private int volumeCreditsFor(Performance performance) {
		
		int result = 0;
		// 포인트를 적립한다.
		result += Math.max(performance.getAudience() - 30, 0);

		//희극 관객 5명 마다 추가 포인트를 제공한다. 
		if("comedy".equals(playFor(performance).getType()))
			result += Math.floor((double) performance.getAudience() / 5);
		return result;
}

``` 

 - 다음은 totalAmount 포맷처리 하는 부분을 메서드로 추출한다. 
 - 기능을 메서드로 추출하고, usd 이라는 이름을 써서 무슨일을 하는 메서드인지를 잘 나타낸다.
 - 이름이 좋으면 메서드 내부를 보지 않고도 무슨일을 하는지 알 수 있다.

 ```java
 private String usd(int totalAmount) {
    	return String.format("%,.2f", (double) (totalAmount / 100));
    }
 ```

 - 다음으로 살펴볼 변수는 volumeCredits다. 이 변수는 반복문을 돌 때마다 값을 누적하기 때문에 까다롭다. 
 - 반복문쪼개기 기법을 이용하여 volumeCredits 값이 누적되는 부분을 따로 빼낸다. 

 ```java
 public String statement(){
        int totalAmount = 0;
        int volumeCredits = 0;
        
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
            totalAmount += thisAmount;
        }
		//volumeCredits 구하는 for문을 분리한다. 
        for (Performance performance : invoice.getPerformances()) {
            volumeCredits += volumeCreditsFor(performance);
        }

        result.append("총액 :").append(usd(totalAmount));
        result.append("\n적립포인트 :").append(volumeCredits).append("점");

        return result.toString();
    }
 ```

  - 이어서 문장슬라이드하기 를 적용하여 volumeCredits 변수를 반복문 앞으로 옮긴다. 

```java
public String statement(){
        int totalAmount = 0;
        
        
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
            totalAmount += thisAmount;
        }
        
        int volumeCredits = 0;
        for (Performance performance : invoice.getPerformances()) {
            volumeCredits += volumeCreditsFor(performance);
        }

        result.append("총액 :").append(usd(totalAmount));
        result.append("\n적립포인트 :").append(volumeCredits).append("점");

        return result.toString();
    }
```

 - volumeCredits이 한데 모여있어, '임시변수를 잘의함수로 바꾸기' 가 수월해진다. 
 - volumeCredits값 계산코드를 메서드추출한다. 

```java
 int volumeCredits = totalVolumeCredits();
```

```java
private int totalVolumeCredits() {
		int result = 0;
        for (Performance performance : invoice.getPerformances()) {
        	result += volumeCreditsFor(performance);
        }
		return result;
	}
``` 

 - volumeCredits는 임시변수이므로 인라인한다. 

 - 다음은 같은 방식으로 totalAmount 를 메서드 추출한다. 
 - 여기서 계속 for문이 많아지게 되는데, 이정도가지고 성능에 미치는 영향은 미미하다.

 ```java
 public String statement(){
        int totalAmount = 0;
        
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
        }
        
        totalAmount = appleSource(); // <-- 함수 추출 & 임시 이름 부여
        
        result.append("총액 :").append(usd(totalAmount));
        result.append("\n적립포인트 :").append(totalVolumeCredits()).append("점");

        return result.toString();
    }

	private int appleSource() {
		int result = 0;
		for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);
            result += thisAmount;
        }
		return result;
	}

 ```

 - totalAmount를 인라인 하면서 이름을 바꿔준다. 기존에 있던 지역변수 totalAmount는 삭제 한다. 

 ```java
 public String statement(){
        
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
        }
        
        result.append("총액 :").append(usd(totalAmount()));
        result.append("\n적립포인트 :").append(totalVolumeCredits()).append("점");

        return result.toString();
    }

	private int totalAmount() {
		int result = 0;
		for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);
            result += thisAmount;
        }
		return result;
	}
 ```

  - 지금까지 리팩토링 한 결과를 보자. statement 메서드가 한결 단순해졌다. 

 ```java
 public class InvoiceStatement {
	
	private final Map<String, Play> plays;
	private final Invoice invoice;
	
	public InvoiceStatement(Invoice invoice, Map<String, Play> plays) {
		this.invoice = invoice;
		this.plays = plays;
	}
	
    public String statement(){
        
        StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
        }
        
        result.append("총액 :").append(usd(totalAmount()));
        result.append("\n적립포인트 :").append(totalVolumeCredits()).append("점");

        return result.toString();
    }

	private int totalAmount() {
		int result = 0;
		for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);
            result += thisAmount;
        }
		return result;
	}

	private int totalVolumeCredits() {
		int result = 0;
        for (Performance performance : invoice.getPerformances()) {
        	result += volumeCreditsFor(performance);
        }
		return result;
	}
    
    private String usd(int totalAmount) {
    	return String.format("%,.2f", (double) (totalAmount / 100));
    }

	private int volumeCreditsFor(Performance performance) {
		
		int result = 0;
		// 포인트를 적립한다.
		result += Math.max(performance.getAudience() - 30, 0);

		//희극 관객 5명 마다 추가 포인트를 제공한다. 
		if("comedy".equals(playFor(performance).getType()))
			result += Math.floor((double) performance.getAudience() / 5);
		return result;
	}
    
    private Play playFor(Performance performance) {
    	return plays.get(performance.getPlayId());
    }

	private int amountFor(Performance performance) {
		int result = 0;
		switch(playFor(performance).getType()){
		    case "tragedy" :
		        result = 40000;
		        if(performance.getAudience() > 30)
		            result += 1000 * (performance.getAudience() - 30);
		        break;
		    case "comedy" :
		        result = 30000;
		        if(performance.getAudience() > 20)
		            result += 10000 + 500*(performance.getAudience() - 20);
		        result += 300 * performance.getAudience();
		        break;
		    default:
		        throw new IllegalStateException("알수없는 장르 " + playFor(performance).getType());
		}
		return result;
	}
}
 ```

  - 단계 쪼개기 : 첫 단계에서는 statement()에 필요한 데이터를 처리하고, 
  - 다음단계에서는 출력을 하도록 한다. 첫번째단계에서는 두 번째 단계로 전달할 중간 데이터
  구조를 생성하는 것이다. 
  - 단계를 쪼개려면 두 번째 단계가 될 코드들을 메서드 추출한다. 

```java
public String statement(){
        
        return renderPlainText(); //<-- 본문 전체를 메서드로 추출
    }

	private String renderPlainText() {
		StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");

        for (Performance performance : invoice.getPerformances()) {
            int thisAmount = amountFor(performance);

            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(thisAmount / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
        }
        
        result.append("총액 :").append(usd(totalAmount()));
        result.append("\n적립포인트 :").append(totalVolumeCredits()).append("점");

        return result.toString();
	}
```

- totalAmount 와 totalVolumeCredits 메서드의 for 문을 stream 으로 해서 간결하게 작성

```java
	private int totalAmount() {
		return invoice.getPerformances().stream().map(p -> amountFor(p)).reduce(0, Integer::sum);
	}

	private int totalVolumeCredits() {
		return invoice.getPerformances().stream().map(p -> volumeCreditsFor(p)).reduce(0, Integer::sum);
	}
```

- 공연료와 적립포인트를 계산하는 별도의 class 를 만들어 분리

```java
public abstract class PerformanceCalculator {
	
	public static PerformanceCalculator createCalculator(String type) {
		switch(type){
		    case "tragedy" :
		       return new TragedyCalculator();
		    case "comedy" :
		        return new ComedyCalculator();
		    default:
		        throw new IllegalStateException("알수없는 장르 " + type);
		}
	}
	
	public abstract int amountFor(Performance perf);
	public abstract int volumeCreditsFor(Performance perf);
	
}

```

 - tragedy 와 comedy 를 계산하는 class를 구현한다. (템플릿 패턴)

```java
public class ComedyCalculator extends PerformanceCalculator {

	public int amountFor(Performance perf) {
		int result = 30000;
		if (perf.getAudience() > 20) {
			result += 10000 + 500 * (perf.getAudience() - 20);
			result += 300 * perf.getAudience();
		}
		return result;
	}

	@Override
	public int volumeCreditsFor(Performance perf) {
		// 포인트를 적립한다.
		int result = Math.max(perf.getAudience() - 30, 0);
		//희극 관객 5명 마다 추가 포인트를 제공한다. 
		result += Math.floor((double) perf.getAudience() / 5);
		return result;
	}
}
``` 

```java
public class TragedyCalculator extends PerformanceCalculator{

	public int amountFor(Performance perf) {
		int result = 40000;
		if(perf.getAudience() > 30)
            result += 1000 * (perf.getAudience() - 30);
		return result;
	}

	@Override
	public int volumeCreditsFor(Performance perf) {
		
		return Math.max(perf.getAudience() - 30, 0);
	}
}

```

 - InvoiceStatement 최종

```java
public class InvoiceStatement {
	
	private final Map<String, Play> plays;
	private final Invoice invoice;
	
	public InvoiceStatement(Invoice invoice, Map<String, Play> plays) {
		this.invoice = invoice;
		this.plays = plays;
	}
	
    public String statement(){
    	
        return renderPlainText(); //<-- 본문 전체를 메서드로 추출
    }

	private String renderPlainText() {
		StringBuilder result = new StringBuilder("청구내역(고객명:" + invoice.getCustomer() + ")\n");
        for (Performance performance : invoice.getPerformances()) {
            //청구 내역을 출력한다. 
            result.append(playFor(performance).getName())
            .append(":")
            .append(amountFor(performance) / 100)
            .append("(")
            .append(performance.getAudience())
            .append("석)\n");
        }
        result.append("총액 :").append(usd(totalAmount()));
        result.append("\n적립포인트 :").append(totalVolumeCredits()).append("점");

        return result.toString();
	}

	//전체 공연료
	private int totalAmount() {
		return invoice.getPerformances().stream().map(p -> amountFor(p)).reduce(0, Integer::sum);
	}

	//전체 적립포인트
	private int totalVolumeCredits() {
		return invoice.getPerformances().stream().map(p -> volumeCreditsFor(p)).reduce(0, Integer::sum);
	}
    
    private String usd(int totalAmount) {
    	return String.format("%,.2f", (double) (totalAmount / 100));
    }

	private int volumeCreditsFor(Performance performance) {
		return PerformanceCalculator.createCalculator(playFor(performance).getType())
				.volumeCreditsFor(performance);
	}
    
    private Play playFor(Performance performance) {
    	return plays.get(performance.getPlayId());
    }
    
	private int amountFor(Performance performance) {
		return PerformanceCalculator.createCalculator(playFor(performance).getType())
				.amountFor(performance);
	}
}
```

# Reference
> - 리팩터링 2판(한빛 미디어)