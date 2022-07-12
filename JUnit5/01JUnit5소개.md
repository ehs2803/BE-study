# Junit

![image](https://user-images.githubusercontent.com/65898555/178390607-4f841970-67aa-412f-ab0a-15a09013ad96.png)


junit은 자바 개발자가 가장 많이 사용하는 테스팅 기반 프레임워크다.

JUnit(제이유닛)은 자바 프로그래밍 언어용 유닛 테스트 프레임워크이다. 
JUnit은 테스트 주도 개발 면에서 중요하며 SUnit과 함께 시작된 XUnit이라는 이름의 유닛 테스트 프레임워크 계열의 하나이다.

# JUnit5 소개 

junit5 버전은 자바8 이상부터 사용 가능하다.

junit4가 단일 jar이었던거에 비해, junit5는 크게 3가지 모듈로 구성된다.

![image](https://user-images.githubusercontent.com/65898555/178389035-99a7d8e8-43ad-430c-80c1-8e7a54e763a4.png)


JUnit5은 JUnit Platform + JUnit Jupiter + JUnit Vintage 세개가 합쳐진 것이다.

JUnit Platform : JVM에서 테스트 프레임워크를 실행하는데 기초를 제공한다. 또한 TestEngine API를 제공해 테스트 프레임워크를 개발할 수 있다.

JUnit Jupiter : JUnit 5에서 테스트를 작성하고 확장을 하기 위한 새로운 프로그래밍 모델과 확장 모델의 조합. API 구현체로 Junit5에서 제공함.

JUnit Vintage : 하위 호환성을 위해 JUnit3과 JUnt4를 기반으로 돌아가는 플랫폼에 테스트 엔진을 제공해준다. TestEngine API 구현체이다.

junit5는 테스트 작성자를 위한 API 모듈과 테스트 실행을 위한 API가 분리되어 있다.
Jnit Jupiter는 테스트 코드 작성에 필요한 junit-jupiter-api 모듈과 테스트 실행을 위한 junit-jupiter-engine 모듈로 분리되어 있다.
