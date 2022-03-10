---
layout: post
title: '[JAVA] Enum(열거형) 활용'
subtitle: 'Enum에 대한 정리'
date: 2022-02-17
author: Sankook Lee
tags: [java]
comments: true
sitemap :
  changefreq : daily
  priority : 1.0
---

## Enum 활용1

**스프링MVC에서의 enum 활용**
  - 스프링 mvc 에서 요청 파라미터를 String이 아닌 enum 을 활용
  - enum 을 활용하여 요청값에 타입 안정성을 부여
  - 간단한 스프링부트 어플리케이션을 통해 샘플코드를 작성함.
  - 본 블로그는 타 유명개발자의 블로그를 참조하였으며, 하단에 참조블로그 표시

  - pom.xml
 ```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!--thymeleaf-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>nz.net.ultraq.thymeleaf</groupId>
            <artifactId>thymeleaf-layout-dialect</artifactId>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.github.gavlyukovskiy</groupId>
            <artifactId>p6spy-spring-boot-starter</artifactId>
            <version>1.5.6</version>
        </dependency>
 </dependencies>		
 ``` 
 
  - 간단한 직원(Employee) 관리 어플리케이션이다. 먼저 데이터 도메인을 정의한다. 
```java
@Data
public class Employee {
	
	private long id;
	private String name;
	private DutyStep dutyStep;
	
	public Employee(String name, DutyStep dutyStep) {
		this.name = name;
		this.dutyStep = dutyStep;
	}
	
	//EnumModel을 통해 클라이언트에서 enum을 key/value 형태로 사용할 수 있게 한다.
	public enum DutyStep implements EnumModel{
		
		EL("사원"), IL("대리"), VL("과장");
		
		private String dutyName;
		
		DutyStep(String dutyName){
			this.dutyName = dutyName;
		}
		
		public String getDutyName() {
			return dutyName;
		}
		
		@Override public String toString() {
			return super.toString() +"[["+ getDutyName()+"]]";
		}
		
		@Override
		public String getKey() {
			return name();
		}
		
		@Override
		public String getValue() {
			return dutyName;
		}
	}
}
```
- Employee 를 저장하는 repository 를 만든다. 간단하게 메모리(Map)에 저장하는 방식으로 한다.
```java
@Repository
public class EmployeeRepository {
	
	private static final Map<Long, Employee> map = new HashMap<>();
	private static long sequence = 0L;
	
	public Employee save(Employee emp) {
		emp.setId(++sequence);
		map.put(emp.getId(), emp);
		return emp;
	}
	
	public List<Employee> findAll(){
		return new ArrayList<>(map.values());
	}
}
```
- 클라이언트에서 Select box 는 key 와 value 로 이루어져 있다. 
- 서버에서 enum 의 key 와 value 를 제공해야 하며, 해당 기능을 먼저 인터페이스로 선언한다. 
```java
public interface EnumModel {
	
	String getKey();
	String getValue();

}

```
- 클라이언트에 넘길 enum Data 객체를 정의한다. 
```java
public class EnumValue {
	
	private String key;
	private String value;
	
	public EnumValue(EnumModel model) {
		this.key = model.getKey();
		this.value = model.getValue();
	}
	public String getKey() {
		return key;
	}
	public String getValue() {
		return value;
	}
}
```

- 보통 공통코드는 어플리케이션 기동 초기에 로딩한다. 매번 컨트롤러에서 enum Data 를 생성하지 않기 위해 enum 제공빈을 생성한다. 
```java
public class EnumMapper {
	
	// key : enum type , value : enum 상수별 인스턴스(EnumValue)
	private Map<Class<? extends EnumModel>, List<EnumValue>> map = new HashMap<>();
	
	private List<EnumValue> toEnumValue(Class<? extends EnumModel> clazz){
		return Arrays.stream(clazz.getEnumConstants())
				.map(EnumValue::new)
				.collect(Collectors.toList());
	}
	
	public void put(Class<? extends EnumModel> clazz) {
		map.put(clazz, toEnumValue(clazz));
	}
	
	public Map<Class<? extends EnumModel>, List<EnumValue>> getAll(){
		return map;
	}
	
	public List<EnumValue> get(Class<? extends EnumModel> clazz){
		return map.get(clazz);
	}
}
```
- @Configuration을 통해 EnumMapper를 빈으로 등록한다. 
```java
@Configuration
public class AppConfig {
	
	@Bean
	public EnumMapper enumMapper() {
		EnumMapper enumMapper = new EnumMapper();
		//공통 코드 enum 을 추가하려면 아래 enumMapper 에 put 을 한다. 
		enumMapper.put(Employee.DutyStep.class);
		return enumMapper;
	}
}
```

- 컨트롤러를 만든다.
```java
@Slf4j
@Controller
@RequestMapping("/employee")
@RequiredArgsConstructor
public class EmployeeController {
	
	private final EmployeeRepository repository;
	private final EnumMapper enumMapper;
	
	@GetMapping("/all")
	@ResponseBody
	public List<Employee> getEmployeeList() {
		return repository.findAll();
	}
	
	@GetMapping("/registerForm")
	public String getDutyStep(Model model){
		//select box 에서 key  와 value 로 매핑한다. 
		model.addAttribute("dutyStep", enumMapper.get(Employee.DutyStep.class));
		
		return "registerForm";
	}
	
	@PostMapping("/register")
	public String post(@RequestParam(value="dutyStepName") DutyStep type, @RequestParam(value="name") String name) {
		log.info("type is {}, {}", type, name);
		repository.save(new Employee(name, type));
		return "redirect:/employee/all";
	}
}
```

- 클라이언트 html 코드를 만든다. 뷰 템플릿엔진은 Thymeleaf 를 사용한다. 
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
	<div class="container">
		<form role="form" action="/employee/register" method="post">
			<div class="form-group">
				<label for="count">이름</label> <input type="text" name="name"
					class="form-control" id="name" placeholder="이름을 입력하세요">
			</div>
			<div class="form-group">
				<label for="employee">직급</label> <select name="dutyStepName"
					id="dutyStepId" class="form-control">
					<option value="">직급</option>
					<option th:each="step : ${dutyStep}" th:value="${step.key}"
						th:text="${step.value}" />
				</select>
			</div>
			<button type="submit" class="btn btn-primary">Submit</button>
		</form>
		<br />
	</div>
</body>
</html>

```
- 스프링 부트 기동 클래스를 만든다. 
- 초기화 클래스에는 Employee 객체를 몇개 생성하여둔다. 
```java
@SpringBootApplication
@ComponentScan(basePackageClasses = {Page2BasePackage.class})
public class App {
	
	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
	@Component
	@RequiredArgsConstructor
	class AppInit implements ApplicationRunner{
		
		private final EmployeeRepository repository;

		@Override
		public void run(ApplicationArguments args) throws Exception {
			repository.save(new Employee("김윤아", Employee.DutyStep.EL));
			repository.save(new Employee("김개똥", Employee.DutyStep.IL));
			repository.save(new Employee("홍길동", Employee.DutyStep.VL));
		}
	}
}
```


# Reference
> - [블로그 참고](https://jojoldu.tistory.com/122?category=635881
