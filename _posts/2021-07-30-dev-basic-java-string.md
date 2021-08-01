---  
layout: post  
title: "String StringBuilder StringBuffer"  
subtitle: "String StringBuilder StringBuffer"  
categories: dev  
tags: dev basic String
comments: true
---  

# String StringBuilder StringBuffer 차이

String 은 immutable 입니다. 
String에 문자열을 더하거나 자르거나 할 떄마다 new String이 생성됩니다. (새로운 객체 생성)
그렇기 때문에 기존 값이 변하지 않으므로 String이 동기화 기능이 없더라도 멀티스레드나 여러 객체에서 공유할 때
안전하게 공유 될 수 있습니다. 값 변경이 적고 같은 값 참조를 많이 할 경우 좋습니다.

+ 연산자나 concat , substring을 사용 할 때 마다 기존 String이 변하는 것이 아니라 new String이 생성 됩니다. 그리고 그 이전의 String은 가비지 컬렉션에서 회수합니다. 이러한 작업들이 많아지면 힙에 garbage를 많이 생성하게 되고 성능이 떨어집니다.
+ 연산자로 String을 더할 때 내부에서는 StringBuilder가 동작합니다. 

JDK 1.5 버전 이후에는 String 을 쓰더라도 StringBuilder로 컴파일이 됩니다. 하지만 반복 루프를 사용해서 문자열을 더할 떄는 객체를 계속 생성합니다. 그렇기 때문에 StringBuilder를 사용하는것이 더 좋습니다. (쓰지 않는 객체들이 메모리에 많이 쌓일 것입니다.)

```java
String myString = ""; 
for(int i = 0; i < 100; i++) {  
   myString += i; 
} 
System.out.println(myString);
```

bytecode view 를 하면  

```text
javap -c className.class

public static void main(java.lang.String...);

Code:
0: ldc     #2   // String
2: astore_1    
3: iconst_0     
4: istore_2     
5: iload_2     
6: ldc     #3   // int 100
8: if_icmpge  36    
11: new     #4  // class java/lang/StringBuilder    
14: dup
15: invokespecial #5  // Method java/lang/StringBuilder."<init>":()V    
18: aload_1    
19: invokevirtual #6  // Method java/lang/StringBuilder.append:      
22: iload_2
23: invokevirtual #7  // Method java/lang/StringBuilder.append:
26: invokevirtual #8  // Method java/lang/StringBuilder.toString:    
29: astore_1
30: iinc    2, 1    
33: goto    5    
36: getstatic  #9  // java/lang/System.out:Ljava/io/PrintStream;
39: aload_1
40: invokevirtual #10  // Method java/io/PrintStream.println:    
43: return


The lines 5 and 33 is the for loop of the example. You can see that in the body of the loop a new StringBuilder is created (line 11), every time in the loop's body with the current contents of myString (line 19) and then the current value of i (line 23) is appended to the builder too. Then the current value of the StringBuilder is converted toString and myString gets this value assigned (line 26).
```

```java
StringBuilder sb = new StringBuilder();
for(int i = 0; i < 100; i++) { 
    sb.append(i); 
}

System.out.println(sb.toString());
```

Here you create the StringBuilder outside of the for loop, append the value of ito the builder and at the end you print out the result.Here the extra object creation will not take place.

(String Literal)String을 생성하면 JVM String pool에 같은 값이 있는지를 찾고 같은 값이 존재한다면 해당 값의 reference를 반환합니다. 없다면 pool에 추가가 되고 reference를 반환합니다. (By String interning concept - storing only one copy of String in the pool)
이렇게 함으로 인해서 JVM은 여러 threads가 같은 String 값을 쓸 때에 불변이기 때문에 thread-safe 해서 동기화를 신경 안써도 되고 공간을 낭비하지 않을 수 있습니다.
(String Object) new() 를 사용하여 String 객체를 생성할 경우에는 언제나 새로운 객체를 heap memory에 생성합니다.
```java
String compare = "abc";
String compare2 = new String "abc";
System.out.println(compare == compare2) //false;
```

또한 지금은 한 번만 생성되었지만 수십번 String이 더해지는 경우에는 각 String의 주소값이 stack에 쌓이고 생성된 객체는 Garbage Collector가 호출되기 전까지 heap에 지속적으로 쌓이게 된다. 메모리 관리에 안좋다.

    Your example C is kind of different tho, because you're using the += operator which when compiled to bytecode it uses StringBuilder to concatenate the strings, so this creates a new instance of StringBuilder Object thus pointing to a different reference.
    String is a final class with all the fields as final except “private int hash”. This field contains the hashCode() function value and created only when hashCode() method is called and then cached in this field. Furthermore, hash is generated using final fields of String class with some calculations, so every time hashCode() method is called, it will result in same output. For caller, its like calculations are happening every time but internally it’s cached in hash field.

###StringBuilder와 StringBuffer는 mutable 입니다.

문자열 연산이 있을때 기존의 버퍼(???) 크기가 변경 됩니다. 

    The constructor with no parameters will create an instance of StringBuilder with the buffer size of 16. This is a default buffer size meaning it can contain 16 characters before it needs to be adjusted. When the buffer is full, StringBuilder will reallocate a new array which will have two times bigger capacity than the previous instance plus 2 additional buffer locations. 2n+2 
    The StringBuffer and StringBuilder are implemented like an ArrayList (except dealing with an array of characters instead of an array of Object instances). When you add new content, if there’s capacity, then the content is added at the end; if not, a new array is created with double-plus-two size, the content backing store is copied to a new array, and then the old array is thrown away. As a result this step can take between O(1) and O(n lg n) depending on whether the initial capacity is exceeded.


StringBuffer는 synchronized 합니다. 멀티스레드 환경에서도 동기화가 됩니다.

동기화 처리로 인해 StringBuffer가 StringBuilder보다 성능이 좋지 않습니다.

```java
performance test
public class ConcatTest{  
    public static void main(String[] args){  
        long startTime = System.currentTimeMillis();  
        StringBuffer sb = new StringBuffer("Java");  
        for (int i=0; i<10000; i++){  
            sb.append("Tpoint");  
        }  
        System.out.println("Time taken by StringBuffer: " + (System.currentTimeMillis() - startTime) + "ms");  
        startTime = System.currentTimeMillis();  
        StringBuilder sb2 = new StringBuilder("Java");  
        for (int i=0; i<10000; i++){  
            sb2.append("Tpoint");  
        }  
        System.out.println("Time taken by StringBuilder: " + (System.currentTimeMillis() - startTime) + "ms");  
    }  
}  
```

StringBuilder와 StringBuffer를 테스트 해보자. 
아래의 결과를 보면 다른 값이 나온 것을 볼 수 있다. 
StringBuilder의 값이 더 작은 것을 볼 수 있는데 이는 쓰레드들이 동시에 StringBuilder클래스에 접근할 수 있기 때문에 일어난 결과다. 
이와 달리 StringBuffer는 multi thread환경에서 다른 값을 변경하지 못하도록 하므로 web이나 소켓환경과 같이 비동기로 동작하는 경우가 많을 때는 StringBuffer를 사용하는 것이 안전할 것이다.

```java
StringBuffer stringBuffer = new StringBuffer();
StringBuilder stringBuilder = new StringBuilder();

new Thread(() -> {
    for(int i=0; i<10000; i++) {
        stringBuffer.append(i);
        stringBuilder.append(i);
    }
}).start();

new Thread(() -> {
    for(int i=0; i<10000; i++) {
        stringBuffer.append(i);
        stringBuilder.append(i);
    }
}).start();

new Thread(() -> {
    try {
        Thread.sleep(5000);

        System.out.println("StringBuffer.length: "+ stringBuffer.length());
        System.out.println("StringBuilder.length: "+ stringBuilder.length());
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();
```
결과: 
    StringBuffer.length: 77780
    StringBuilder.length: 76412


    Immutable Object는 불변객체로써, 값이 변하지 않는다. 변경하는 연산이 수행되면 변경하는 것처럼 보이더라도 실제 메모리에는 새로운 객체가 할당 되는 것이다.

자바에서 Wrapper class에 해당하는 Integer, Character, Byte, Boolean, Long, Double, Float, Short 클래스는 모두 Immutable 이다.
그래서 heap에 있는 같은 오브젝트를 레퍼런스 하고 있는 경우라도, 새로운 연산이 적용되는 순간 새로운 오브젝트가 heap에 새롭게 할당된다.

###이유는?
Integer클래스의 구현을 보면 final이라는 키워드가 붙어있다.
클래스의 붙어있는 final은 상속을 제한하는 목적으로 붙이는 제어자이다.
Integer클래스 내부에서 사용하는 실제 값인 value라는 변수가 있는데,
이 변수가 private final int value;로 선언되어있다.
즉, 생성자에 의해 생성되는 순간에만 초기화되고 변경불가능한 값이 된다.
이것 때문에 Wrapper class들도 Immutable한 오브젝트가 되는것이다.

Heap
-Heap영역에는 주로 긴 생명주기를 가지는 데이터들이 저장된다.
-애플리케이션의 모든 메모리 중 stack에 있는 데이터를 제외한 부분이라고 보면 된다.
-모든 Object 타입(Integer, String,ArrayList, …)은 heap 영역에 생성된다.
-단 하나의 Heap영역만 존재한다.
-Heap영역에 있는 오브젝트들을 가리키는 레퍼런스 변수가 stack에 올라가게 된다.

Stack
- Heap 영역에 생성된 Object 타입의 데이터의 참조값이 할당.
- 원시타입의 데이터가 값과 함께 할당된다.
- 지역변수들은 scope에 따른 visibility를 가진다.
- 각 Thread는 자신만의 stack을 가진다.(공유는 되지않음)

변수 scope (가시성, Visibility)
-프로그램 내 변수에 접근할 수 있는 유효 범위/영역
:정의된 변수가 보이는 유효 범위
Stack에는 heap영역에 생성된 Object 타입의 데이터들에 대한 참조를 위한 값들이 할당.
원시타입의 데이터들이 할당, 이때 원시타입의 데이터들에 대해서는 참조값을 저장하는것이 아니라 실제 값을 stack에 직접 저장하게 된다.

Stack메모리는 Thread 하나당 하나씩 할당된다. 즉, 새로운 스레드가 생성되는 순간 해당 스레드를 위한 stack도 함께 생성된다.
각 스레드에서 다른 스레드의 stack영역에는 접근할 수 없다.

String one = "someString"; 
JVM 스택에 생성된 변수 one은 힙 메모리에 생성된 객체를 참조하며, 객체는 다시 메소드 메모리에 있는 상수풀(String Liberal Pool)을 참조한다. syntax에는 객체 생성 연산자인 new를 쓰지 않지만, 내부적으로는 객체를 생성한다는 뜻이다.

그림에는 문자열 리터럴(“someString”)이 객체에 생성된 것처럼 보이나 실제로는 상수풀에 생성되며, 객체는 상수풀을 참조하는 주소값만 가진 것으로 볼 수 있다. 따라서 (1)은 변수(JVM 스택), 객체(힙 메모리), 상수풀(메소드)이 서로 참조를 통해 긴밀히 연결된 일종의 네트워크다.

String two = new String("someString");
string object on the heap, and having a reference to it from the string pool

반면에 == 연산의 경우 객체의 주소값을 비교합니다. 그렇기때문에 new 연산자를 통해 Heap 영역에 생성된 String과 리터럴을 이용해 String Constant Pool 영역에 위치한 String과의 주소값은 같을 수가 없지요.

Spring Constant Pool
상수풀(Spring Constant Pool)의 위치는 Java 7 부터 Perm영역에서 Heap 영역으로 옮겨졌습니다. Perm영역은 실행시간(Runtime)에 가변적으로 변경할 수 없는 고정된 사이즈이기 때문에 intern 메서드의 호출은 저장할 공간이 부족하게 만들 수 있었습니다. 즉 OOM(Out Of Memory) 문제가 발생할 수 있었지요. Heap 영역으로 변경된 이후에는 상수풀에 들어간 문자열도 Garbage Collection 대상이 됩니다.

- 관련 링크) JDK-6962931 : move interned strings out of the perm gen(Oracle Java Bug Database)
  [https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6962931](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6962931)

번외로 Java 7버전에서 상수풀의 위치가 Perm영역(정확히 말하면 Permanent Generation)에서 Heap으로 옮겨지고 Java 8 버전에서는 Perm영역은 완전히 사라지고 이를 MetaSpace라는 영역이 대신하고 있습니다.
