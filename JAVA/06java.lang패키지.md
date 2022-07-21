# java.lang 패키지

java.lang패키지는 자바프로그래밍에 가장 기본이 되는 클래스들이 포함되어 있다. 그렇기 때문에 java.lang패키지의 클래스들은 import문 없이 사용할 수 있다.

String 클래스나 System클래스를 import문 없이 사용할 수 있었던 이유가 바로 java.lang패키지에 속한 클래스이기 때문이다.

# Object 클래스

Object클래스는 모든 클래스의 최고 조상이기 때문에 Object 클래스의 멤버들은 모든 클래스에서 바로 사용이 가능하다.

![image](https://user-images.githubusercontent.com/65898555/180112731-5f132966-d91e-4476-9dfc-b1ef010fe848.png)

Object클래스는 멤버변수는 없고, 오직 11개의 메서드만 가지고 있다. 이 메서드들은 모든 인스턴스가 가져야할 기본적인 것들이다.

### equals(Object obj)

매개변수로 객체의 참조변수를 받아서 비교하여 그 결과를 boolean값으로 알려 주는 역할을 한다.

```java
public boolean equals(Object obj){
    return (this==obj);
}
```

