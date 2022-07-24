# Math 클래스

### Math 클래스의 메서드

![image](https://user-images.githubusercontent.com/65898555/180626417-dd2a57f9-5f97-4262-933a-a56f68541c4c.png)

![image](https://user-images.githubusercontent.com/65898555/180626421-d45c33ee-c2c7-4c3f-a99b-3c5e81daaf9c.png)

### Math 클래스 예제

Math 클래스는 기본적인 수학계산에 유용한 메서드로 구성되어 있다.

Math 클래스의 생성자는 접근 제어자가 private이므로 다른 클래스에서 Math인스턴스를 생성할 수 없더록 되어있다. 그 이유는 클래스 내에 인스턴스변수가 하나도 없이서 인스턴스를 생성할 필요가 없기 때문이다.

```java
public static final double E = 2.7182818284590452354; // 자연로그의 밑
public static final double PI = 3.14159265358979323846; // 원주율
```
또 Maht 클래스의 메서드는 모두 static이며 상수는 2개가 있다.


```java
import static java.lang.Math.*;
import static java.lang.System.*;

class MathEx1 {
	public static void main(String args[]) {
		double val = 90.7552;
		out.println("round("+ val +")=" + round(val));  // 반올림

		val *= 100;
		out.println("round("+ val +")=" + round(val));  // 반올림

		out.println("round("+ val +")/100  =" + round(val)/100);  // 반올림
		out.println("round("+ val +")/100.0=" + round(val)/100.0);  // 반올림
		out.println();
		out.printf("ceil(%3.1f)=%3.1f%n",  1.1, ceil(1.1));   // 올림
		out.printf("floor(%3.1f)=%3.1f%n", 1.5, floor(1.5));  // 버림	
		out.printf("round(%3.1f)=%d%n",    1.1, round(1.1));  // 반올림
		out.printf("round(%3.1f)=%d%n",    1.5, round(1.5));  // 반올림
		out.printf("rint(%3.1f)=%f%n",     1.5, rint(1.5));   // 반올림
		out.printf("round(%3.1f)=%d%n",   -1.5, round(-1.5)); // 반올림
		out.printf("rint(%3.1f)=%f%n",    -1.5, rint(-1.5));  // 반올림
		out.printf("ceil(%3.1f)=%f%n",    -1.5, ceil(-1.5));  // 올림
		out.printf("floor(%3.1f)=%f%n",   -1.5, floor(-1.5)); // 버림
	}
}
```
```
round(90.7552) = 91
round(9075.52) = 9076
round(9075.52) / 100 = 90
round(9075.52) / 100.0 = 90.76

ceil(1.1) = 2.0
floor(1.5) 1.0
round(1.1) = 1
round(1.5) = 2
rint(1.5) = 2.000000
round(-1.5) = -1
rint(-1.5) = -2.000000
ceil(-1.5) = -1.000000
floor(-1.5) = -2.000000
```
올림, 버림, 반올림 함수이다.



```java
import static java.lang.Math.*;
import static java.lang.System.*;

class MathEx2 {
	public static void main(String args[]) {
		int i = Integer.MIN_VALUE;

		out.println("i ="+i);
		out.println("-i="+(-i));

		try {
			out.printf("negateExact(%d)= %d%n",  10, negateExact(10));
			out.printf("negateExact(%d)= %d%n", -10, negateExact(-10));

			out.printf("negateExact(%d)= %d%n", i, negateExact(i)); // 예외발생
		} catch(ArithmeticException e) {
			// i를 long타입으로 형변환 다음에 negateExact(long a)를 호출
		     out.printf("negateExact(%d)= %d%n",(long)i,negateExact((long)i));
		}
	} // main의 끝
}
```
jdk1.8부터 메서드 이름에 Exact가 포함된 메서드들이 추가되었다. 이들은 정수형간의 연산에서 발생할 수 있는 오버플로우를 감지하기 위한 것이다.

연산자는 단지 결과를 반환할 뿐, 오버플로우의 발생여부에 대해 알려주지 않는다. 그러나 위의 메서드들을 오버플로우가 발새하면, 예외(ArithmeticException)를 발생시킨다.



```java
import static java.lang.Math.*;
import static java.lang.System.*;

class MathEx3 {
	public static void main(String args[]) {
		int x1=1, y1=1;  // (1, 1)
		int x2=2, y2=2;  // (2, 2)

		double c = sqrt(pow(x2-x1, 2) + pow(y2-y1, 2));
		double a = c * sin(PI/4);  // PI/4 rad = 45 degree
		double b = c * cos(PI/4);
//		double b = c * cos(toRadians(45));

		out.printf("a=%f%n", a);   
		out.printf("b=%f%n", b);  
		out.printf("c=%f%n", c);  
		out.printf("angle=%f rad%n", atan2(a,b));	
		out.printf("angle=%f degree%n%n", atan2(a,b) * 180 / PI);	
//		out.printf("angle=%f degree%n%n", toDegrees(atan2(a,b)));	

		out.printf("24 * log10(2)=%f%n",   24 * log10(2));  // 7.224720
		out.printf("53 * log10(2)=%f%n%n", 53 * log10(2));  // 15.954590
	}
}
```
삼각함수와 지수, 로그 함수이다.

### StrictMath 클래스

Math 클래스는 최대한의 성능을 얻기 위해 JVM이 설치된 OS의 메서드를 호출해서 사용한다. 즉, OS에 의존적인 계산을 하고 있는 것이다.

예를 들어 부동소수점 계산의 경우, 반올림의 처리방법 설정이 OS마다 다를 수 있기 때문에 자바로 작성된 프로그램임에도 불구하고 컴퓨터마다 결과가 다를 수 있다

이러한 차이를 없애기 위해 성능을 다소 포기하는 대신, 어떤 OS에서 실행되어도 항상 같은 결과를 얻도록 Math 클래스를 새로 작성한 것이 StrictMath클래스이다.



# 래퍼(wrapper) 클래스

자바에서는 8개의 기본형을 객체로 다루지 않는데 기본형 변수도 어쩔 수 없이 객체로 다뤄야 하는 경우가 있다.

이때 사용하는 것이 래퍼클래스이다. 8개의 기본형을 대표하는 8개의 래퍼클래스가 있는데, 이 클래스들을 이용하면 기본형 값을 객체로 다룰 수 있다.

![image](https://user-images.githubusercontent.com/65898555/180626432-e345c0d9-a284-4d0c-8ebe-9193f8ad505a.png)

래퍼 클래스의 생성자는 매개변수로 문자열이나 각 자료형의 값들을 인자로 받는다. 이때 주의할점은, 생성자의 매개변수로 문자열을 제공할 때, 각 자료형에 알맞은 문자열을 사용해야 한다.
예를 들어 new Integer("1.0");을 하면 NumberFormatException이 발생한다.

래퍼 클래스들은 모두 equals가 오버라이딩되어 있어서 주소값이 아닌 객체가 가지고 있는 값을 비교한다. 오토박싱이 된다고 해도 Integer객체에 비교연산자를 사용할 수 없다. 대신 compareTo를 제공한다.

또 toString을 오버라이딩되어 있어서 객체가 가지고 있는 값을 문자열로 반환이 가능하다. 그 외에 래퍼 클래스들은 MAX_VALUE, MIN_VALUE, SIZE, BYTES, TYPE 등의 static 상수를 공통적으로 가지고 있다.

### Number클래스

Number 클래스는 내부적으로 숫자를 멤버변수로 갖는 래퍼 클래스들의 조상이다. 기본형 중에서 숫자와 관련된 래퍼 클래스들은 모두 Number 클래스의 자손이다.

![image](https://user-images.githubusercontent.com/65898555/180626471-8157ec26-922d-490f-b8a6-08066bb49f26.png)

래퍼클래스(회색)의 상속계층도이다.

BigInteger와 BigDecimal 등은 각각 long으로도 다룰 수 없는 큰 범위의 정수를, double로 다룰 수 없는 큰 범위의 부동 소수점수를 처리하기 위한 것으로 연산자의 역할을 대신하는 다양한 메서드를 제공한다.


### 문자열을 숫자로 변환

![image](https://user-images.githubusercontent.com/65898555/180626526-93ddf70b-1853-4eee-84c7-ef5e77778484.png)

jdk1.5부터 도입된 오토박싱(autoboxing) 기능 때문에 반환값이 기본형일 때와 래퍼클래스일 때의 차이가 없다. 그래서 그냥 valueOf를 사용해도 되지만, 성능이 조금 더 느리다.


### 오토박싱 & 언박싱(autoboxing & unboxing)

jdk1.5이전에는 기본형과 참조형 간의 연산이 불가능했지만, 컴파일러가 자동으로 변환하는 코드를 넣어주어 가능해졌다.

그 외에 내부적으로 객체 배열을 가지고 있는 Vector 클래스나 ArrayList클래스에 기본형 값을 저장해야할 때나 형변환이 필요할 때도 컴파일러가 자동적으로 코드를 추가한다.

```java
ArraList<Integer> list = new ArrayList<Inteber>();
list.add(10); // autoboxing. 10 -> new Integer(10)
int value = list.get(0_; // unboxing. new Integer(10) -> 10
```
기본형 값을 래퍼 클래스의 객체로 자동변환해주는 것을 오토박싱이라하고, 그 반대로 변환하는 것을 언박싱이라고 한다.



