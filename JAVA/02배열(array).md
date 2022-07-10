# 배열(array)

배열은 같은 타입의 여러 변수를 하나의 묶음으로 다루는 것이다.

# 배열의 선언과 생성

### 배열의 선언 방법

```java
타입[] 변수이름;
타입 변수이름[];
```
```java
int[] score;
String[] name;
```

원하는 타입의 변수를 선언하고 변수 또는 타입에 배열임을 의미하는 대괄호를 붙이면 된다.
대괄호는 타입 뒤에 붙여도 되고, 변수이름 뒤에 붙여도 된다.

### 배열의 생성

배열은 선언한 다음에 배열을 생성해야 한다.
배열 선언은 단지 생성된 배열을 다루기 위한 참조변수를 위한 공간이 만들어지는 것이다.
배열을 생성해야만 비로소 값을 저장할 수 있는 공간이 만들어지는 것이다.

```java
타입[] 변수이름; // 배열을 선언. 배열을 다루기 위한 참조변수 선언
변수이름 = new 타입[길이]; // 배열 생성. 실제 저장공간을 생성
```

# 배열 인덱스

생성된 배열의 각 저장공간을 배열의 요소라고 한다.
요소에 접근할 때 '배열이름[인덱스]' 형식으로 접근한다.
인덱스는 배열의 요소마다 붙여진 일련번호이다.

**인덱스의 범위는 0부터 배열길이-1 이다**

유효한 범위를 벗어난 값을 index로 사용하는 것은 컴파일러가 걸러주지 못한다.
왜냐하면 배열의 index로 변수를 많이 사용는데, 변수의 값은 실행 시에 대입되므로 컴파일러는 이 값의 범위를 확인할 수 없다.
유효하지 않은 index로 사용하면, 컴파일 후 실행 시에 에러(ArrayIndexOutOfBoundException)가 발생한다.

# 배열의 길이

자바에서는 JVM이 모든 배열의 길이를 별도로 관리한다.

배열이름.length를 통해 배열의 길이에 대한 정보를 얻을 수 있다.

배열은 한번 생성하면 길이를 변경할 수 없기 때문에, 이미 생성된 배열의 길이는 변하지 않는다. 따라서 배열이름.length는 상수다. 

# 길이가 0인 배열

길이가 0인 배열도 생성이 가능하다.

```java
int[] arr = new int[0];
```
위와 같이 길이가 0인 배열을 만들 수 있다.


# 배열 길이 변경

배열 생성 후 배열에 저장할 공간이 부족한 경우는 더 큰 길이의 새로운 배열을 생성하고 기존 배열 값들을 복사하면 된다.

1. 더 큰 배열 새로 생성
2. 기존 배열의 내용을 새로운 배열에 복사

# 배열 초기화

배열은 생성과 동시에 자동적으로 자신의 타입에 해당하는 기본값으로 초기화된다.

혹은 for문을 통해 배열을 초기화할 수 있다.

```java
int[] score = new int[]{50,60,70,80,90}; // 배열의 생성과 초기화를 동시에
int[] score = {50,60,70,80,90}; // 배열의 생성과 초기화를 동시에
```
for문을 이용한 초기화는 저장하려는 값에 일정한 규칙이 있을 때 가능하다. 따라서 위와 같은 방법으로 초기화할 수 있다.

```java
int[] score;
int score = new int[]{50,60,70,80,90}; // ok
score = {50,60,70,80,90}; // error
```
```java
int result = add(new int[]{1,2,3,4,5}); // ok
int result = add({1,2,3,4,5}); // error
```
```java
int[] score = new int[0]; // 길이 0인 배열
int[] score = new int[]{}; // 길이 0인 배열
int[] score = {}; // 길이 0인 배열
```

# 배열 출력


```java
int[] arr = {1,2,3,4,5};
for(int i=0;i<arr.length;i++){
  System.out.println(arr[i]+" ");
}
```
기본적으로 for문을 사용해 배열을 출력한다.

```java
import java.util.*; // 추가해야됨.

int[] arr = {1,2,3,4,5};

System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5] 출력
```
더 간단한 방법은 Arrays.toString(array name)메서드를 사용하면 된다.
이 메서드는 배열의 모든 요소를 [첫번째 요소, 두번째 요소, ...] 형태로 출력한다.


```java
char[] chArr = {'a','b','c','d','e'};

System.out.println(Arrays.toString(chArr)); // abcd 출력
```
예외적으로 char 배열은 println 메소드로 출력하면 각 요소가 구분자없이 그대로 출력된다.

# 배열 복사

배열 복사 시 for문을 이용하는 방법이 있다.

System.arraycopy()를 이용한 배열 복사 방법이 있다. 보다 간단하고 빠르게 배열을 복사할 수 있다.
for문은 배열의 요소 하나하나에 접근해 복사하지만, arraycopy 메서드는 지정된 범위의 값들을 한번에 통째로 복사한다.
각 요소들이 연속적으로 저장되어 있다는 배열의 특성 때문에 가능하다.

```java
System.arraycopy(num, 0, newNum, 0, num.lenth);
```
위 코드는 num[0]에서 newNum[0]으로 num.length개의 데이터를 복사한다.

# 배열 활용

### 섞기(shuffle)
```java
class ArrayEx7 {
	public static void main(String[] args) {
		int[] numArr = new int[10];

		for (int i=0; i < numArr.length ; i++ ) {
             numArr[i] = i;  // 배열을 0~9의 숫자로 초기화한다.
			System.out.print(numArr[i]);  
		}
		System.out.println();

		for (int i=0; i < 100; i++ ) {
			int n = (int)(Math.random() * 10);	// 0~9중의 한 값을 임의로 얻는다.

			int tmp = numArr[0];
			numArr[0] = numArr[n];
			numArr[n] = tmp;
		}

		for (int i=0; i < numArr.length ; i++ )
			System.out.print(numArr[i]);		
	} // main의 끝
}
```

### sort
```java
class ArrayEx10 {
	public static void main(String[] args) {
		int[] numArr = new int[10];

		for (int i=0; i < numArr.length ; i++ ) {
			System.out.print(numArr[i] = (int)(Math.random() * 10));
		}
		System.out.println();

		for (int i=0; i < numArr.length-1 ; i++ ) {
			boolean changed = false;	// 자리바꿈이 발생했는지를 체크한다.

			for (int j=0; j < numArr.length-1-i ;j++) {
				if(numArr[j] > numArr[j+1]) { // 옆의 값이 작으면 서로 바꾼다.
					int temp = numArr[j];
					numArr[j] = numArr[j+1];
					numArr[j+1] = temp;
					changed = true;	// 자리바꿈이 발생했으니 changed를 true로.
				}
			} // end for j

			if (!changed) break;	// 자리바꿈이 없으면 반복문을 벗어난다.

			for(int k=0; k<numArr.length;k++)
				System.out.print(numArr[k]); // 정렬된 결과를 출력한다.
			System.out.println();
		} // end for i
	} // main의 끝
}
```

# String 배열



```java
String[] name = new String[3];
```
3개의 String 타입의 참조변수를 저장하기 위한 저장 공간이 마련되고 참조형 변수의 기본값은 null이므로 각 요소의 값은 null로 초기화 된다.

|자료형|기본값|
|---|---|
|boolean|false|
|char|'\u0000'|
|byte,short,int|0|
|long|0L|
|float|0.0f|
|double|0.0d or 0.0|
|참조형변수|null|

변수의 타입에 따른 기본값이다.

```java
String[] name = new String[]{"kim", "park", "yi"};
String[] name = {"kim", "park", "yi"};

String[] name = new String[2];
name[0] = new String("kim");
name[0] = "kim";
```
String 배열 초기화 방법이다.

배열에 실제 객체가 아닌 객체의 주소가 저장되어 있다. 기본형 배열이 아닌 참조형 배열의 경우 배열에 저장되는 것은 객체의 주소이다.
참조형 배열을 객체배열이라고 한다.


```java
class ArrayEx12 {
	public static void main(String[] args) {
		String[] names = {"Kim", "Park", "Yi"};

		for(int i=0; i < names.length;i++) {
			System.out.println("names["+i+"]:"+names[i]);
		}

		String tmp = names[2]; // 배열 names의 세 번째요소를 tmp에 저장
		System.out.println("tmp:"+tmp);

		names[0] = "Yu"; // 배열 names의 첫 번째 요소를 "Yu"로 변경

		for(String str : names)   // 향상된 for문
			System.out.println(str);
	} // main
}
```

# char 배열과 String 클래스

문자열을 저장하는 String 타입의 변수는 사실 문자를 연이어 늘어놓은 문자배열인 char 배열과 같다.

자바에서 char배열이 아닌 String 클래스를 이용하는 방법은 String 클래스가 char배열에 여러가지 기능을 추가해 확장한 것이기 때문이다.

char배열과 String 클래스의 한 가지 중요한 차이점은, String 객체(문자열)는 읽을 수만 있을 뿐 내용을 변경할 수 없다.

|메서드|설명|
|---|-----|
|char charAt(int index)|문자열에서 해당 위치에 있는 문자 반환|
|int length()|문자열 길이 반환|
|String substring(int from, int to)|문자열에 해당 범위(from~to-1)에 있는 문자열을 반환한다.|
|boolean equals(Object obj)|문자열의 내용이 obj와 같은지 확인. 같은면 true, 틀리면 false|
|char[] toCharArray()|문자열을 문자배열로 변환해 반환|


```java
char[] chArr = {'a', 'b', 'c'};
String str = new String(chArr);  // char배열 -> String
char[] tmp = str.toCharArray();  // String -> char 배열
```

char 배열과 String 간 변환 방법이다.

# 길이가 0인 배열

```java
class ArrayEx16 {
	public static void main(String[] args) {
		System.out.println("¸Å°³º¯¼öÀÇ °³¼ö:"+args.length);

		for(int i=0;i< args.length;i++) {
			System.out.println("args[" + i + "] = \""+ args[i] + "\"");
		}
	}
}
```
Scanner 클래스 외에도 화면을 통해 사용자로부터 입력을 받을 수 있다. 커맨드라인을 이용하는 방법이다.
실행할 때 클래스 이름 뒤에 공백문자로 구분해 여러개의 문자열을 프로그램에 전달 할 수 있다.

main에서 args 배열에 담기게 된다.

커맨드 라인에 매개변수를 입력하지 않으면 크기가 0인 배열이 생성되어 args의 length는 0이 된다. 만일 입력된 매개변수가 없다고 해서 배열을 생성하지 않으면 참조변수 args의 값은 null이 된다. 그러나 JVM은 입력된 매개변수가 없을 때, null 대신 크기가 0인 배열을 생성해 args에 전달하도록 구현되어 있다.



