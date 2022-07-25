# java.util.Objects 클래스

Object클래스의 보조 클래스로 모든 메서드가 static이다. 객체의 비교나 널체크에 유용하다.


```java
import java.util.*;
import static java.util.Objects.*;  // ObjectsÅ¬·¡½ºÀÇ ¸Þ¼­µå¸¦ static import

class ObjectsTest {
	public static void main(String[] args) {
		String[][] str2D   = new String[][]{{"aaa","bbb"},{"AAA","BBB"}};
		String[][] str2D_2 = new String[][]{{"aaa","bbb"},{"AAA","BBB"}};

		System.out.print("str2D  ={");
		for(String[] tmp : str2D) 
			System.out.print(Arrays.toString(tmp));
		System.out.println("}");

		System.out.print("str2D_2={");
		for(String[] tmp : str2D_2) 
			System.out.print(Arrays.toString(tmp));
		System.out.println("}");

		System.out.println("equals(str2D, str2D_2)="+Objects.equals(str2D, str2D_2));
		System.out.println("deepEquals(str2D, str2D_2)="+Objects.deepEquals(str2D, str2D_2));

		System.out.println("isNull(null) ="+isNull(null));
		System.out.println("nonNull(null)="+nonNull(null));
		System.out.println("hashCode(null)="+Objects.hashCode(null));
		System.out.println("toString(null)="+Objects.toString(null));
		System.out.println("toString(null, \"\")="+Objects.toString(null, ""));
	
		Comparator c = String.CASE_INSENSITIVE_ORDER;
			   
         System.out.println("compare(\"aa\",\"bb\")="+compare("aa","bb",c));
	     System.out.println("compare(\"bb\",\"aa\")="+compare("bb","aa",c));
	     System.out.println("compare(\"ab\",\"AB\")="+compare("ab","AB",c));
	}
}
```
```
str2D ={[aaa,bbb] [AAA, BBB]}
str2D_2 = {[aaa, bbb] [AAA, BBB]}
equals(str2D, str2D_2) = false
deepEquals(str2D, str2D_2) = true
isNull(null)=true
nonNull(null)=false
hasCode(null)=0
toString(null)=null
toString(null,"")=
compare("aa","bb")=-1
compare("bb","aa")=1
compare("ab","AB")=0
```
isNull은 해당 객체가 널인지 확인해 null이면 true를 반환하고 아니면 false를 반환한다. nonNull은 isNull과 정반대 일을 한다.
requireNonNull은 해당 객체가 널이 아니어야 하는 경우에 사용한다. 만약 객체가 널이면 NullPointerException을 발생시킨다.

compare은 두 비교대상이 같으면 0, 크면 양수, 작으면 음수를 반환한다.

Objects클래스의 equals는 Object클래스의 equals와 다르게 null검사를 하지 않아도 된다.

2차원 문자열 배열을 비교할 때, 단순히 equals를 써서 비교할 수 없다. equals와 반복문을 써야하는데, deepEquals를 쓰면 간단히 끝난다.

toString도 equals처럼 내부적으로 널 검사를 한다는 것 빼고는 틀별한 것이 없다. toString의 두번째 매개변수에는 널일때 대신 사용할 값을 지정할 수 있다.

hasCode는 내부적으로 널 검사를 한 후에 Object클래스의 hasCode를 호출할 뿐이다. 만약 널이면 0을 반환한다.

static import문을 사용함에도 불구하고 Object클래스의 메서드와 이름이 같은 것들은 충돌이 난다. 즉 컴파일러가 구분을 못해 이럴 때는 클래스의 이름을 붙여줘야 한다.


# java.util.Random클래스

난수를 얻는 방법은 Math.random말고 Random클래스를 사용할 수 있다.

Math.random은 내부적으로 Random 클래스의 인스턴스를 생성해 사용하므로 둘 중에서 편한 것을 사용하면 된다.

```java
double randNum = Math.random();
double randNum = new Random().nextDouble(); // 위의 문장과 동일
```

![image](https://user-images.githubusercontent.com/65898555/180713482-cd33f90d-fe1e-45e1-a05f-a2def55c10a6.png)

![image](https://user-images.githubusercontent.com/65898555/180713526-cd16f67f-5fb7-40c0-91ac-b0cde85a67b9.png)

생성자 Random은 종자값 System.currentTimeMillis로 하기때문에 실행할 때마다 얻는 난수가 달라진다.

만약 생성할 때 같은 난수를 입력하면 각각의 인스턴스에서 같은 값이 나온다. 즉 같은 종자값을 갖는 Random인스턴스는 시스템이나 실행시간 등에 관계없이 항상 같은 값을 같은 순서로 반환할 것을 보장한다.


```java
import java.util.*;

class RandomEx3 {
	public static void main(String[] args) {
		for(int i=0; i < 10;i++)
			System.out.print(getRand(5, 10)+",");
		System.out.println();

		int[] result = fillRand(new int[10], new int[]{ 2, 3, 7, 5});

		System.out.println(Arrays.toString(result));
	}

	public static int[] fillRand(int[] arr, int from, int to) {
		for(int i=0; i < arr.length; i++) {
			arr[i] = getRand(from, to);
		}

		return arr;
	}

	public static int[] fillRand(int[] arr, int[] data) {
		for(int i=0; i < arr.length; i++) {
			arr[i] = data[getRand(0, data.length-1)];
		}

		return arr;
	}

	public static int getRand(int from, int to) {
		return (int)(Math.random() * (Math.abs(to-from)+1)) + Math.min(from, to);
	}
}
```
Math.random을 이용해서 실제 프로그래밍에 유용할만한 메서드를 만든것이다.



# java.util.regex 패키지 (정규식 - Regular Expression)

정규식이란 텍스트 데이터 중에서 원하는 조건(패턴)과 일치하는 문자열을 찾아내기 위해 사용하는 것으로 미리 정의된 기호와 문자를 이용해서 작성한 문자열을 말한다.

원래 Unix에서 사용하던 것으로 Perl의 강력한 기능이었는데 요즘은 JAVA를 비롯해 다양한 언어에서 지원하고 있다.

정규식을 사용하면 많은 양의 텍스트 파일 중에서 원하는 데이터를 손쉽게 뽑아낼 수 있다. 또 입력된 데이터가 형식에 맞는지 체크할 수도 있다.
예를 들어 html문서에서 전화번호나 이메일 주소만을 따로 추출한다던가, 입력한 비밀번호가 숫자와 영문자 조합으로 되어 있는지 확인할 수도 있다.

```java
import java.util.regex.*;	// Pattern과 Matcher가 속한 패키지

class RegularEx1 {
	public static void main(String[] args) 
	{
		String[] data = {"bat", "baby", "bonus",
				    "cA","ca", "co", "c.", "c0", "car","combat","count",
				    "date", "disc"};		
		Pattern p = Pattern.compile("c[a-z]*");	// c로 시작하는 소문자영단어

		for(int i=0; i < data.length; i++) {
			Matcher m = p.matcher(data[i]);
			if(m.matches())
				System.out.print(data[i] + ",");
		}
	}
}
```
```
ca,co,car,combat,count,
```

Pattern은 정규식을 정의하는데 사용되고 Matcher는 정규식 패턴을 데이터와 비교하는 역할을 한다.

![image](https://user-images.githubusercontent.com/65898555/180715787-1fc7e062-0e6b-498b-8ce9-acff60975a2b.png)



# java.util.Scanner 클래스

Sacnner는 화면, 파일, 문자열과 같은 입력소스로부터 문자데이터를 읽어오는데 도움을 줄 목적으로 jdk1.5부터 추가되었다.

```
Scanner(String soure)
Scanner(File source)
Scanner(InputStream source)
Scanner(Readable source)
Scanner(ReadablBytheChannel soure)
Scanner(Path source)
```
Scanner에는 다음과 같은 생성자를 지원해 다양한 입력소스로부터 데이터를 읽을 수 있다.

```
boolean nextBoolean()
byte    nextByte()
short   nextShort()
int     nextInt()
long    nextLong()
double  nextDouble()
float   nextFloat()
String  nextLine()
``
Scanner는 다양한 메서드를 제공함으로써 입력받은 문자를 다시 변환하는 수고를 덜어준다.

# java.util.StringTokenizer 클래스

긴 문자열을 지정된 구분자(delimiter)를 기준으로 토큰이라는 여러개의 문자열로 잘라내는 데 사용한다. 이 방법 말고 String에 split을 이용해도 된다. 혹은 Scanner의 useDelimiter를 사용할 수 
있지만 이 두가지 방법은 정규식 표현을 사용해야한다.

![image](https://user-images.githubusercontent.com/65898555/180716739-a6b8a4a3-b921-4821-b6e4-5ea2e1fe2f5c.png)

```java
import java.util.*; 

class StringTokenizerEx1 { 
	public static void main(String[] args){ 
		String source = "100,200,300,400";
		StringTokenizer st = new StringTokenizer(source, ",");

		while(st.hasMoreTokens()){
			System.out.println(st.nextToken());
		}
	} // mainÀÇ ³¡
} 
```
```
100
200
300
400
```

```java
import java.util.*; 

class StringTokenizerEx2 { 
	public static void main(String[] args) { 
		String expression = "x=100*(200+300)/2";
		StringTokenizer st = new StringTokenizer(expression, "+-*/=()", true);

		while(st.hasMoreTokens()){
			System.out.println(st.nextToken());
		}
	} // mainÀÇ ³¡
} 
```
```
x
=
100
*
(
200
+
300
)
/
2
```
StringTokenizer는 단 한문자의 구분자만 사용할 수 있어 ```"+-*/=()"``` 전체가 하나의 구분자가 아니라 각각의 문자가 모두 구분자이다.


```java
import java.util.*;

class StringTokenizerEx5 {
	public static void main(String[] args) {
		String data = "100,,,200,300";

		String[] result = data.split(",");
		StringTokenizer st = new StringTokenizer(data, ",");

		for(int i=0; i < result.length;i++)
			System.out.print(result[i]+"|");

		System.out.println("°³¼ö:"+result.length);

		int i=0;
		for(;st.hasMoreTokens();i++)
			System.out.print(st.nextToken()+"|");

		System.out.println("°³¼ö:"+i);
	} // main
}
```
```
100|||200|300|개수:5 <-split()
100|200|300|개수:3 <-StringTokenizer
```
split은 빈 문자열도 토큰으로 인식하는 반면 StringTokenizer는 빈 문자열을 토큰으로 인식하지 않기 때문에 인식하는 토큰의 개수가 다르다.


# java.util.BigInteger 클래스

정수형으로 표현할 수 있는 값의 한계가 있다. 가장 큰 정수형 타입인 long으로 표현할 수 있는 값은 10진수로 19자리 정도이다.

BigInteger는 내부적으로 int배열을 사용해 값을 다룬다. 성능은 long타입보다 떨어진다. 또 String처럼 불변이다.

# java.math.BigDecimal 클래스

double 타입으로 표현할 수 있는 값은 상당히 범위가 넓지만, 정밀도가 13자리밖에 되지 않고 실수형 특성상 오차를 피할 수 없다. BigDecimal는 실수형과 달리 정수를 이용해서 실수를 표현한다.







