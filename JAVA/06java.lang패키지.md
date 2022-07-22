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
equals는 위 코드로 구현되어있다. 두 객체의 같고 다름을 참조변수의 값으로 판단한다. 따라서 서로 다른 두 객체를 equals메서드로 비교하면 항상 false를 결과로 얻게 된다.


```java
class EqualsEx1 {
	public static void main(String[] args) {
		Value v1 = new Value(10);
		Value v2 = new Value(10);		

		if (v1.equals(v2)) {
			System.out.println("v1과 v2는 같습니다.");
		} else {
			System.out.println("v1과 v2는 다릅니다.");		
		}

		v2 = v1;

		if (v1.equals(v2)) {
			System.out.println("v1과 v2는 같습니다.");
		} else {
			System.out.println("v1과 v2는 다릅니다.");		
		}
	} // main
} 

class Value {
	int value;

	Value(int value) {
		this.value = value;
	}
}
```
```
v1과 v2는 다릅니다.
v1과 v2는 같습니다.
```
위 예제는 멤버변수 value의 값이 같을지라도 서로 다른 인스턴스이므로 false를 반환하게 된다. 하지만 v2=v1을 실행 후 참조변수 v2는 v1을 참조하고 있는 인스턴스의 주소값이 저장되므로 v2와 v1이 같다고 판단된다.

Object클래스로부터 상속받은 equals메서드는 결국 두 개의 참조변수가 같은 객체를 참조하고 있는지, 즉 두 참조변수가 저장된 값(주소값)이 같은지를 판단하는 기능밖에 없다.

```java
class Person {
	long id;

	public boolean equals(Object obj) {
		if(obj!=null && obj instanceof Person) {
			return id ==((Person)obj).id;
		} else {
			return false;
		}
	}

	Person(long id) {
		this.id = id;
	}
}

class EqualsEx2 {
	public static void main(String[] args) {
		Person p1 = new Person(8011081111222L);
		Person p2 = new Person(8011081111222L);

		if(p1==p2)
			System.out.println("p1과 p2는 같은 사람입니다.");
		else
			System.out.println("p1과 p2는 다른 사람입니다.");

		if(p1.equals(p2))
			System.out.println("p1과 p2는 같은 사람입니다.");
		else
			System.out.println("p1과 p2는 다른 사람입니다.");
	
	}
}
```
```
p1과 p2는 다른 사람입니다.
p1과 p2는 같은 사람입니다.
```
Value클래스에서 equals 메서드를 오버라이딩하여 주소가 아닌 객체에 저장된 내용을 비교하도록 변경하면 된다.

String 클래스 역시 Object클래스의 equals메서드를 그대로 사용하는 것이 아니라 오버라이딩을 통해 String 인스턴스가 갖는 문자열 값을 비교하도록 되어있다.


### hashCode()

이 메서드는 해싱기법에 사용되는 해시함수를 구현한 것이다. 해시함수는 찾고자하는 값을 입력하면, 그 값이 저장된 위치를 알려주는 해시코드를 반환한다.

일반적으로 해시코드가 같은 두 객체가 존재하는 것은 가능하지만, Object클래스에 정의된 hashCode메서드는 객체의 주소값으로 해시코드를 만들어 변환하기 때문에 32bit JVM에서는 서로 다른 객체는 결코 같은 해시코드를 가질 수 없었지만, 64bit JVM에서 8byte 주소값으로 해시코드(4byte)를 만들기 때문에 해시코드가 중복될 수 있다.

클래스의 인스턴스벼수 값으로 객체의 같고 다름을 판단해야하는 경우 equals 메서드 뿐 아니라 hashCode메서드도 적절히 오버라이딩해야 한다. 같은 객체라면 hashCode메서드를 호출했을 때의 결과값인 해시코드가 같아야 하기 때문이다.

```java
class HashCodeEx1 {
	public static void main(String[] args) {
		String str1 = new String("abc");
		String str2 = new String("abc");

		System.out.println(str1.equals(str2));
		System.out.println(str1.hashCode());
		System.out.println(str2.hashCode());
		System.out.println(System.identityHashCode(str1));
		System.out.println(System.identityHashCode(str2));
	}
}
```
```
true
96354
96354
27134973
1284693
```
String 클래스는 문자열의 내용이 같으면, 동일한 해시코드를 반환하도록 hashCode를 오버라이딩되어 있기 때문에, 문자열의 내용이 같으면 항상 동일한 해시코드값을 얻는다.

반면에 System.identityHashCode(Object x)는 Object클래스의 hashCode메서드처럼 객체의 주소값으로 해시코드를 생성하기 때문에 모든 객체에 대해 항상 다른 해시코드값을 반환할 것을 보장한다. 

### toString()

```java
public String toString(){
    return getClass().getName()+"@"+Integer.toHexString(hashCode());
}
인스턴스에 대한 정보를 문자열로 제공할 목적으로 정의한 것이다. 인스턴스의 정보를 제공한다는 것은 대부분의 경우 인스턴스의 변수에 저장된 값들을 문자로열로 표현한다는 것이다.


```java
class Card {
	String kind;
	int number;

	Card() {
		this("SPADE", 1);
	}

	Card(String kind, int number) {
		this.kind = kind;
		this.number = number;
	}
}

class CardToString {
	public static void main(String[] args) {
		Card c1 = new Card();
		Card c2 = new Card();

		System.out.println(c1.toString());
		System.out.println(c2.toString());
	}
}
```
```
Card@19e0bfd
Card@139a55
```
위 예제처럼 클래스 작성시 toString을 오버라이딩하지 않으면, 클래스이름에 16진수의 해시코드를 얻게 될 것이다.

```java
class ToStringTest {
	public static void main(String args[]) {
		String str = new String("KOREA");
		java.util.Date today = new java.util.Date();

		System.out.println(str);
		System.out.println(str.toString());
		System.out.println(today);
		System.out.println(today.toString());
	}
}
```
```
KOREA
KOREA
Fri Oct 23 21:48:29 KST 2015
Fri Oct 23 21:48:29 KST 2015
```
String 클래스와 Date클래스의 tostring은 오버라이딩해 다른 값이 나온다.

이처럼 toString은 일반적으로 인스턴스나 클래스에 대한 정보 또는 인스턴스 변수들의 값을 문자열로 변환하여 반환하도록 오버라이딩되는 것이 보통이다.


```java
class Card {
	String kind;
	int number;

	Card() {
		this("SPADE", 1);  // Card(String kind, int number)를 호출
	}

	Card(String kind, int number) {
		this.kind = kind;
		this.number = number;
	}

	public String toString() {
		// Card인스턴스의 kind와 number를 문자열로 반환한다.
		return "kind : " + kind + ", number : " + number;
	}
}

class CardToString2 {
	public static void main(String[] args) {
		Card c1 = new Card();
		Card c2 = new Card("HEART", 10);
		System.out.println(c1.toString());
		System.out.println(c2.toString());
	}
}
```
```
kind : SPADE, number : 1
kind : HEART, number : 10
```
위 예제는 toString을 오버라이딩했다. 

오버라이딩할 때, Object클래스에 정의된 toString의 접근 제어자가 public이므로 Card클래스의 toString의 접근제어자도 public으로 해야한다.
조상에 정의된 메서드를 자손에서 오버라이딩할 때는 조상에 정의된 접근범위보다 같거나 더 넓어야해서 무조건 public으로 할 수 밖에 없다.

### clone()

자신을 복제해 새로운 인스턴스를 생성하는 일을 한다. Object클래스에 정의된 clone은 단순히 인스턴스변수의 값만 복사하기 때문에 참조타입의 인스턴스변수가 있는 클래스는 완전한 인스턴스 복제가 이루어지지 않는다. 예를들어, 배열의 경우 복제된 인스턴스도 같은 배열의 주소를 갖기 때문에 복제된 인스턴스의 작업이 원래의 인스턴스에 영향을 미치게 된다. 이런 경우 clone메서드를 오버라이딩해서 새로운 배열을 생성하고 배열의 내용을 복사해야 한다.

```java
class Point implements Cloneable {
	int x;
	int y;

	Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	public String toString() {
		return "x="+x +", y="+y;
	}

	public Object clone() {
		Object obj = null;
		try {
			obj = super.clone();  // clone()은 반드시 예외처리를 해주어야 한다.
		} catch(CloneNotSupportedException e) {}
		return obj;
	}
}

class CloneEx1 {
	public static void main(String[] args){
		Point original = new Point(3, 5);
		Point copy = (Point)original.clone(); // 복제(clone)해서 새로운 객체를 생성
		System.out.println(original);
		System.out.println(copy);
	}
}
```
clone을 사용하려면 먼저 복제할 클래스가 Cloneable인터페이스를 구현해야하고, clone을 오버라이딩하면서 접근 제어자를 protected에서 public으로 변경한다. 그래야만 상속관계가 없는 다른 클래스에서 clone을 호출할 수 있다.

Cloneable인터페이스를 구현한 클래스의 인스턴스만 clone을 통한 복제가 가능하다. 그 이유는 인스턴스의 데이터를 보호하기 위해서이다. Cloneable인터페이스가 구현되어 있다는 것은 클래스 작성자가 복제를 허용한다는 의미이다.

### 공변 반환타입

jdk1.5부터 공변 반환타입(covariant return type)이라는 것이 추가되었는데, 이 기능은 오버라이딩할 때 조상 메서드의 반환타입을 자손 클래스의 타입으로 변경을 허용하는 것이다.

```java

```
위 예제에서 clone의 반환타입을 Object에서 Point로 변경한 것이다. 즉, 조상의 타입에서 자손의 타입으로 변경한 것이다. 그리고 return문에 Point타입으로 형변환도 추가했다. 예전에는 오버라이딩할 때 조상에 선언된 메서드의 반환타입을 그대로 사용해야 했다.

```java
Point copy = (Point)original.clone();

Point copy = original.clone();
```
위 코드처럼 공변 반환타입을 사용하면 조상의 타입이 아닌, 실제로 반환되는 자손객체의 타입으로 반환할 수 있어서 번거로운 형변환을 줄어든다는 장점이 있다.

```java
import java.util.*;

class CloneEx2 {
	public static void main(String[] args){
		int[] arr = {1,2,3,4,5};

        // 배열 arr을 복제해서 같은 내용의 새로운 배열을 만든다.
		int[] arrClone = arr.clone(); 
		arrClone[0]= 6;

		System.out.println(Arrays.toString(arr));
		System.out.println(Arrays.toString(arrClone));
	}
}
```
배열도 객체이기 때문에 Object클래스를 상속받으며, 동시에 Cloneable인터페이스와 Serializable인터페이스가 구현되어 있다. 그래서 Object클래스의 멤버들을 모두 상속받는다.

Object클래스에는 protected로 정의되어있는 clone을 배열에서 public으로 오버라이딩해서 직접 호출이 가능하다. 원본과 같은 타입을 반환하므로 형변환이 필요없다.

배열뿐 아니라 java.util패키지의 Vector, ArrayList, LinkedList, HashSet, TreeSet, HashMap, TreeMap, Calendar, Date와 같은 클래스들이 이와 같은 방식으로 복제가 가능하다.

### 얕은 복사와 깊은 복사

clone은 단순히 객체에 저장된 값을 그대로 복제할 뿐, 객체가 참조하고 있는 객체까지 복제하지 않는다.

바로 위 예제에서 기본형 배열의 경우에는 아무런 문제가 없지만, 객체배열을 clone으로 복제하는 경우에는 원본과 복제복이 같은 객체를 공유하므로 완전한 복제라고 보기 어렵다.

이러한 복제를 얕은 복사(shallow copy)라고 한다. 얕은 복사에서는 원본을 변경하면 복사본도 영향을 받는다.

반면에 원본이 참조하고 있는 객체까지 복제하는 것을 깊은 복사(deep copy)라고 한다. 깊은 복사에서는 원본과 복사본이 서로 다른 객체를 참조하기 때문에 원본의 변경이 복사본에 영향을 미치지 않는다.


```java
import java.util.*;

class Circle implements Cloneable {
	Point p;  // 원점
	double r; // 반지름

	Circle(Point p, double r) {
		this.p = p;
		this.r = r;
	}

	public Circle shallowCopy() { // 얕은 복사
		Object obj = null;

		try {
			obj = super.clone();
		} catch (CloneNotSupportedException e) {}

		return (Circle)obj;
	}

	public Circle deepCopy() { // 깊은 복사
		Object obj = null;

		try {
			obj = super.clone();
		} catch (CloneNotSupportedException e) {}

		Circle c = (Circle)obj; 
		c.p = new Point(this.p.x, this.p.y); 

		return c;
	}

	public String toString() {
		return "[p=" + p + ", r="+ r +"]";
	}
}

class Point {
	int x;
	int y;

	Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	public String toString() {
		return "("+x +", "+y+")";
	}
}

class ShallowCopy {
	public static void main(String[] args) {
		Circle c1 = new Circle(new Point(1, 1), 2.0);
		Circle c2 = c1.shallowCopy();
		Circle c3 = c1.deepCopy();
	
		System.out.println("c1="+c1);
		System.out.println("c2="+c2);
		System.out.println("c3="+c3);
		c1.p.x = 9;
		c1.p.y = 9;
		System.out.println("= c1의 변경 후 =");
		System.out.println("c1="+c1);
		System.out.println("c2="+c2);
		System.out.println("c3="+c3);
	}
}
```
```
c1=[p=(1, 1), r=2.0]
c2=[p=(1, 1), r=2.0]
c3=[p=(1, 1), r=2.0]
= c1의 변경 후 =
c1=[p=(9, 9), r=2.0]
c2=[p=(9, 9), r=2.0]
c3=[p=(1, 1), r=2.0]
```

### getClass()

자신이 속한 클래스의 class 객체를 반환하는 메서드이다. Class객체는 클래스의 모든 정보를 담고 있으며, 클래스 당 1개만 존재한다. 그리고 클래스 파일이 클래스 로더에 의해서 메모리에 올라갈 때, 자동으로 생성된다. 클래스로더는 실행 시에 필요한 클래스를 동적으로 메모리에 로드하는 역할을 한다.

먼저 기존에 생성된 클래스 객체가 메모리에 존재하는지 확인하고, 있으면 객체의 참조를 반환하고 없으면 클래스 패스에 지정된 경로를 따라서 클래스 파일을 찾는다.
못 찾으면 ClassNotFoundException이 발생하고, 찾으면 해당 클래스 파일을 읽어서 Class객체로 변환한다.

파일형태로 저장되어 있는 클래스를 읽어 Class클래스에 정의된 형식으로 변환하는 것이다. 즉, 클래스 파일을 읽어서 사용하기 편한 형태로 저장해 놓은 것이 클래스 객체이다.

```java
Class obj = new Card().getClass(); // 생성된 객체로 부터 얻는 방법
Class obj = Card.class; // 클래스 리터럴로부터 얻는 방법
Class obj = Class.forName("Card"); // 클래스 이름으로 부터 얻는 방법
```
forName은 특정 클래스 파일. 예를 들어 데이터베이스 드라이버를 메모리에 올릴때 주로 사용한다.

```java
final class Card {
	String kind;
	int num;

	Card() {
		this("SPADE", 1);
	}

	Card(String kind, int num) {
		this.kind = kind;
		this.num  = num;
	}

	public String toString() {
		return kind + ":" + num;
	}
}

class ClassEx1 {
	public static void main(String[] args) throws Exception {
		Card c  = new Card("HEART", 3);       // new연산자로 객체 생성
		Card c2 = Card.class.newInstance();   // Class객체를 통해서 객체 생성

		Class cObj = c.getClass();

		System.out.println(c);
		System.out.println(c2);
		System.out.println(cObj.getName());
		System.out.println(cObj.toGenericString());
		System.out.println(cObj.toString());		
	}
}
```
Class 객체를 이용하면 클래스에 정의된 멤버의 이름이나 개수 등 클래스에 대한 모든 정보를 얻을 수 있기 때문에 Class객체를 통해서 객체를 생성하고 메서드를 호출하는 등 보다 동적인 코드를 작성할 수 있다.


