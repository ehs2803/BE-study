# thymeleaf란?

타임리프는 View Template(뷰 템플릿)이라고 부른다. 
뷰 템플릿은 컨트롤러가 전달하는 데이터를 이용하여 동적으로 화면을 구성할 수 있다. 기존 JSP에서는 많은 기능을 제공하고 전체적인 화면을 디자인하는데 부족하였다. 

타임리프는 스프링 프레임워크외에 노드JS나 다른 웹 서버에서도 뷰 템플릿으로 이용가능하다.

​타임리프는 html태그를 기반으로하여 th:속성을 이용하여 동적인 View를 제공.

웹에서 가장 기본이되는 HTML로 진입장벽이 낮고 쉽게 배울 수 있다는 장점이 있음.

타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면
JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야한다.

이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿(natural templates)이라 한다.

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



23,24

th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"
th:onclick="|location.href='@{/basic/items}'|"

th:action
HTML form에서 action 에 값이 없으면 현재 URL에 데이터를 전송한다.

취소시 상품 목록으로 이동한다.
th:onclick="|location.href='@{/basic/items}'|"


