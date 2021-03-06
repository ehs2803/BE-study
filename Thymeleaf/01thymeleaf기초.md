# thymeleaf란?

타임리프는 View Template(뷰 템플릿)이라고 부른다. 
뷰 템플릿은 컨트롤러가 전달하는 데이터를 이용하여 동적으로 화면을 구성할 수 있다. 기존 JSP에서는 많은 기능을 제공하고 전체적인 화면을 디자인하는데 부족하였다. 

타임리프는 스프링 프레임워크외에 노드JS나 다른 웹 서버에서도 뷰 템플릿으로 이용가능하다.

​타임리프는 html태그를 기반으로하여 th:속성을 이용하여 동적인 View를 제공.

웹에서 가장 기본이되는 HTML로 진입장벽이 낮고 쉽게 배울 수 있다는 장점이 있음.

타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면
JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야한다.

이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿(natural templates)이라 한다.

# 서버 사이드 HTML 렌더링 (SSR)

타임리프는 백엔드 서버에서 HTML을 동적으로 렌더링 하는 용도로 사용


# 타임리프 시작

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
maven

```
dependencies {
...
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```
gradle

```html
<html xmlns:th="http://www.thymeleaf.org">
```
타임리프를 사용하기 위해서 html에 사용 선언을 해준다.

# 타임리프 핵심

핵심은 th:xxx 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. th:xxx 이 없으면 기존 html의 xxx 속성이 그대로 사용된다.

HTML을 파일로 직접 열었을 때, th:xxx 가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시한다.

따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.

# 타임리프 베뉴얼

타임리프는 크게 2가지 메뉴얼을 제공한다.

기본 메뉴얼: https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html
스프링 통합 메뉴얼: https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html

타임리프는 스프링 없이도 동작하지만, 스프링과 통합을 위한 다양한 기능을 편리하게 제공한다.

# 기초

**URL 링크 표현식 - @{...}**
```
th:href="@{/css/bootstrap.min.css}"
```
@{...} : 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다. 이것을 URL 링크 표현식이라 한다.
URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.

<br><br>

**속성 변경 - th:onclick**
```
onclick="location.href='addForm.html'"

th:onclick="|location.href='@{/basic/items/add}'|"
```
여기에는 다음에 설명하는 리터럴 대체 문법이 사용되었다. 

리터럴 대체 - |...|

|...| :이렇게 사용한다.

타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.

<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
	
<span th:text="|Welcome to our application, ${user.name}!|">

결과를 다음과 같이 만들어야 하는데
location.href='/basic/items/add'
그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"
	
리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.
th:onclick="|location.href='@{/basic/items/add}'|"

<br><br>
	
**반복 출력 - th:each**
```
<tr th:each="item : ${items}">
```
반복은 th:each 를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩 포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
컬렉션의 수 만큼 <tr>..</tr> 이 하위 테그를 포함해서 생성된다.

<br><br>
	
**변수 표현식 - ${...}**
```	
<td th:text="${item.price}">10000</td>
```
모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다. 프로퍼티 접근법을 사용한다. ( item.getPrice() )
	
<br><br>
	
**내용 변경 - th:text**
```
<td th:text="${item.price}">10000</td>
```
내용의 값을 th:text 의 값으로 변경한다.
여기서는 10000을 ${item.price} 의 값으로 변경한다.

<br><br>
	
**URL 링크 표현식2 - @{...}**
```
th:href="@{/basic/items/{itemId}(itemId=${item.id})}"
```
상품 ID를 선택하는 링크를 확인. URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다. 경로 변수( {itemId} ) 뿐만 아니라 쿼리 파라미터도 생성한다.
	
- 예) th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"
- 생성 링크: http://localhost:8080/basic/items/1?query=test
	
URL 링크 간단히
```
th:href="@{|/basic/items/${item.id}|}"
```
	
상품 이름을 선택하는 링크를 확인해보자.
리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.
	
	

