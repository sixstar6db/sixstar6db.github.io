---
title: "[java] DataTable 사용법(1)"
excerpt: "DataTable 사용법(1)"

categories:
  - java-advanced
tags:
  - [java, datatable]

permalink: /java-advanced/IO_datatable/

toc: true
toc_sticky: true

date: 2022-07-08
last_modified_at: 2022-07-08
---

## 개요

- datatables 는 html 에서의 테이블을 편리하게 생성하고, 유용한 기능들을 제공해주는 라이브러리이다. UI가 그렇게 훌륭한 편은 아니라, 관리자 시스템등의 화면 개발시에 사용하면 도움이 될 것으로 보인다. 

## 스프링 부트 기본 프로그램 작성

 - 다른 부분은 생략하고, controller의 요청 처리 부분이다. Records를 List 로 만들어, 응답으로 보낸다. 

```java
	List<Records> findAll = recordsService.findAll();
	
	DataTable<Records> dataTable = new DataTable<>();
	dataTable.setData(findAll);
	
	return ResponseEntity.ok(dataTable);
```

 - DataTable 은 다음과 같이 만들었다. Http Body에 실릴 데이터 구조를 정의한 것이다. 

```java
@Data
public class DataTable<T> {
	
	private int draw;
    private int start;
    private long recordsTotal;
    private long recordsFiltered;
    private List<T> data;
}
``` 

- 아래는 실제 json 형태의 응답 body 메시지이다. 

```json
{"draw":0,
"start":0,
"recordsTotal":0,
"recordsFiltered":0,
"data":[{"createDate":"2022-07-07T18:55:26.489","modifiedDate":"2022-07-07T18:55:26.489","id":1,"title":"first0","content":"content0","category":"notice"},{"createDate":"2022-07-07T18:55:26.512","modifiedDate":"2022-07-07T18:55:26.512","id":2,"title":"first1","content":"content1","category":"notice"},
```

- 아래는 html 코드이다. 

```html
<table id="mainTable" width="100%">
	<thead  id="sql_result_head">
	<tr>
		<th scope="col">id</th> 
		<th>category</th>
		<th>title</th>
		<th>content</th>
		<th>createDate</th>
	</tr>
	</thead>
	<tfoot id="sql_result_foot">
	<tr>
		<th>id</th> 
		<th>category</th>
		<th>title</th>
		<th>content</th>
		<th>createDate</th>
	</tr>
	</tfoot>
</table>
```

- 아래는 javascript 소스코드이다. 페이지가 로딩이 다 된후 실행된다. 

```js
$(document).ready(function() {
	
	/*1. datatable 생성 파라미터 객체 정의*/
	var dtObj = {

        "ajax": {
            "url": "/api/records",
            "method":"POST", //기본은 GET
            "dataSrc": function (response) { // 응답 메시지 처리
                var data = response.data; // your data list
                var all = [];
                for (var i = 0; i < data.length; i++) {

                    var row = {
                        rows: response.start + i + 1,
                        id: data[i].id, // name ... ,
                        category: data[i].category,
                        title: data[i].title,
                        content: data[i].content,
                        createDate: data[i].createDate
                    };
                    all.push(row);
                }
                return all;
            }
        },
        "columns": [
            { "data": "id"},
            { "data": "category"},
            { "data": "title"},
            { "data": "content"},
            { "data": "createDate"}
        ]
    };
	/*2. 페이지 로딩 후, datatable 생성*/
    $('#mainTable').DataTable(dtObj);

});
```

 - 지금까지는 전체 데이터를 전부 가져와서 화면에서 페이징 처리를 하는 방식이다. 다음은 서버에서 페이징하여 데이터를 조회하는 방식에 대해 설명하겠다.

 - 화면에 페이지 번호를 클릭하면, start 와 legnth 가 파라미터로 넘어간다. start 는 조회할 데이터 순번이고, length 는 한 페이지에 보여줄 데이터 수이다. 100번째 데이터를 10개씩 보여준다고 하면, page 는 10페이지가 된다. 

 - Spring Data Jpa 에서는 페이징 처리를 위한 api 를 제공한다. 

```java
@PostMapping("/api/records2")
	public ResponseEntity<DataTable<Records>> records2(Model model,
            @RequestParam("draw") int draw, 
            @RequestParam("start") int start,
            @RequestParam("length") int length){
		
		System.out.println("draw:" + draw + ", start:" + start + ", length: "+length);
		
		int page = start / length; //Calculate page number
		
		Pageable pageable = PageRequest.of(
                page,
                length,
                Sort.by("id").descending()
        ) ;

		Page<Records> responseData = recordsRepository.findAll(pageable);
		
		DataTable<Records> dataTable = new DataTable<>();
		dataTable.setData(responseData.getContent());
		dataTable.setRecordsTotal(responseData.getTotalElements());
		dataTable.setRecordsFiltered(responseData.getTotalElements());
		
		dataTable.setDraw(draw);
		dataTable.setStart(start);
		
		return ResponseEntity.ok(dataTable);
	}
```

 - 자바스크립트에서는 일부 속성들을 추가로 선언해야 한다. 공식 홈페이지에서는 데이터가 10만개 이상이면 서버사이드로 처리하라고 한다. 

```js
$(document).ready(function() {
	
	/*1. datatable 생성 파라미터 객체 정의*/
	var dtObj = {
	        "processing": true,
	        "serverSide": true, //서버사이드 페이징 사용
	        "pageLength": 10, // 한 화면에 보여줄 데이터 수
	        "searching": false,
	        "info" : true,

	        "ajax": {
	            "url": "/api/records2",
	            "method":"POST",
	            "data":function(d){
	            	console.log(d);
	            },
	            "dataSrc": function (response) {
	            	console.log(response);
	                var data = response.data; // your data list
	                var all = [];
	                for (var i = 0; i < data.length; i++) {

	                    var row = {
	                        rows: response.start + i + 1,
	                        id: data[i].id, // name ... ,
	                        category: data[i].category,
	                        title: '<b>' + data[i].title + '</b>',
	                        content: data[i].content,
	                        createDate: data[i].createDate
	                    };
	                    all.push(row);
	                }
	                return all;
	            }
	        },
	        "columns": [
	            { "data": "id"},
	            { "data": "category"},
	            { "data": "title"},
	            { "data": "content"},
	            { "data": "createDate"}
	        ]
	    };
	/*2. 페이지 로딩 후, datatable 생성*/
    $('#mainTable').DataTable(dtObj);

});
```

# Reference

 - 
