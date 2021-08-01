---  
layout: post  
title: "JVM Structure"  
subtitle: "JVM & GC"  
categories: dev  
tags: dev basic gc
comments: true
---  

# GC Basic

## 소개
C언어의 가비지 수집은 코드에서 데이터를 수동으로 그리고 명시적으로 메모리를 할당하고 해제한다.
후에 사용되지 않은 메모리가 남용되면 결국 메모리 누수로 전락된다.
하지만 메모리를 비운다는 것은 사실상 쉽다. 코드상에서 충분히 제어가 가능하기때문이다. 
그런데 복잡한 프로그램상에 일일히 관리하기엔 신경써야할 코드가 너무 많기 때문에 훨씬 더 나은 접근 방법으로 
사용되지 않는 메모리를 효율적으로 관리하는 방법은 그것을 자동화하여 사람의 코드상에 실수의 가능성을 없애는 것! 
바로 이것이 "자동화 된 가비지 수집" 이다. (GC)
자바는 객체가 생성되면 자동으로 메모리를 할당하고 특정 객체가 더 이상 사용되지 않을 때 메모리를 자동으로 회수한다. 
메모리관리 작업을 자동으로 수행해 주기 때문에 C언어와 비교해 프로그램 상에서 메모리관리에는 조금 둔해지는것 같다. 
자바의 메모리관리는 시스템에 안정성에 있어 분명한 옵션으로 보인다. 
서비스의 성격에 따라 옵션의 수치들은 해당 시스템에 영향을 주는건 불가피하지만, 문자 그대로 그냥 옵션..
서비스를 운용하기 앞서 GC에 대한 인식을 정확히 짚고 넘어 갈 필요가 있다.
jvm과 GC의 대략적인 개념과 동작원리를 정리하고, 자바 구현과의 상관관계를 정리하였다.

## 배경
  > 수많은 자바라이브어리들과 서비스들이 효율적인 GC영역은 어떤구현이 되어있으며 어떤 옵션들로 메모리영역들을 관리 및 구현이 되어있는지 상당한 부분에 기본적인 개념이 많이부족하다느꼈다. 그래서 이해를 돕기위한 선행학습으로 구체적인 내부구조를 간략하게 정리해보았다.

## JVM(Java Virtual Machine) 기본구조 및 동작원리
메모리 영역을 체계적이고 자체 상호 명령으로 관리 되다보니 가상머신이란 이름이 붙여졌다. 자바로 작성된 소스 코드를 가상머신이 이해할 수 있는 중간 코드로 컴파일한 바이트코드(Bytecode)를 하드웨어 아키텍처에 맞는 기계어로 다시 컴파일하여 실행하는 장치로써 모든 OS에서 구분없이 실행가능하다. 크게 Class Loader, Execution Engine, Garbage Collector, Runtime Data Area 로 구성되며 Class Loader는 말그대로 클래스파일 "적재"용이다. "$JAVA_HOME\jre\" 이하에서 실행되는데 그 중 부트스트랩 클래스로더, 확장 클래스로더, 시스템 클래스로더의 각 로드의 역할로 java.ext.dirs의 시스템 속성에 지정된 디렉터리에 코드를 로드(적재)하여 Runtime Data Area에서 필요한 데이터를 가져다 쓰는 형태이다. 이 후 Execution Engine 이 자바 바이트코드를 실행하고 Garbage Collector로 메모리를 관리한다.

	# .class <-> Class Loader <-> .jar(.class)
	# Runtime Data Area
	- method-area : 메소드바이트코드, 클래스/인터페이스/메소드/필드 정보, 접근제어자, 리턴타입, 타입정보, 
	상수정보, Static/final class 변수 등이 저장됨.
	- stack-area : Thread 제어를 위해 사용되는 메모리 영역으로 메소드의 매개변수, 지역변수, 반환값, 임시변수, 로컬 변수 및 
	부분 결과를 보유하고 해당 메소드 관련 정보와 호출을 한 주소를 저장한다.
	- heap-area : 모든 클래스 인스턴스/객체 및 배열에 대한 메모리
	- pc-register : JVM의 PC-Register는 CPU에서 직접 명령을 수행하지 않는 Stack Base로 Stack에서 피연산자를 뽑아내어 PC-Register라는 
	별도의 메모리 공간에 저장한후 CPU에 그명령을 전달한다.
	- native-method-stacks : C, C++같은 언어로 구현된 메소드, 주로 하드웨어에 대한 접근용 메소드
	# Execution Engine(JIT, GC) <-> # native-method-interface <-> # naticve-method-library
	# 실행엔진 (Execution Engine) : 바이트코드를 명령어단위로 하나씩 인터프리팅된 결과를 실행하거나, JIT(Just In Time)컴파일러를 통해
	바이트코드 전체를 컴파일하여 네이티브 코드로 변경 후 캐시에 저장하여 실행한다.


## Heap area 구조 및 GC 과정
  > Runtime Data Area 중에서 Heap area은 GC의 주요 대상이며, Young, Old, Permanent Generation 로 구성되었다. 
    Young Generation은 젊은객체 즉, 생명주기가짧고 사용빈도가 일정 횟수 이하인 객체들로 위치하는 공간으로
    Eden과 두개의 Survivor 공간(From-Survivor, To-Survivor)으로 이루어졌다.

  > Eden은 new로 생성된 Java오브젝트의 인스턴스가 저장되며 해당 공간이 가득차면 From-Survivor에 복사되어 저장된다.
    그리고 From-Survivor가 가득차면 To-Survivor로 이동되는데 여기까지의 동작이 Minor GC 다.
    Young Generation 공간이 모두 차게 되면 Garbage Collector는 가용 메모리를 확보하기 위해 Major GC를 수행하는데 To-Survivor에서
    Old Generation로 이동된다. 여기서 이동 될 객체의 기준은 참조된 객체의 상태인데 판별을 위해 Mark and Sweep 알고리즘이 사용된다. 
    GC가 시작되면 먼저 모든 루트를 열거 한 다음 재귀적으로 참조되는 객체를 방문하기 시작한다. 메모리 그래프에서 노드를 이동해가며 객체에 도달하면 참조여부를 판단해 
    Live객체면 가비지가 아니라는 것을 표시한다. 이단계가 "Mark"이며 비트플래그로 표시되는데 이 단계에 reachability 이란 개념으로 unreachable를 구분하여 GC 처리한다.
    과정중에 힙 내의 다른 객체에 의한 참조, Java 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조, JNI(Java Native Interface)에 의해 생성된 
    객체에 대한 참조, 메서드 영역의 정적 변수에 의한 참조로 힙내의 다른객체에 의한 참조 중에 '힙 내의 다른 객체에 의한 참조'를 제외한 모든 참조를 
    "root set(최초참조)" 으로 기준삼아 시작한 참조 사슬에 속한 객체들과 그렇지 않은 객체로 reachable 객체와 unreachable 객체로 나눈다. 
    이 최초참조 사슬에 속한 객체들 중에서도 해당 객체의 유형을 강약으로 포지셔닝했는데 Strong reference > Soft reference > Weak reference >
    Phantom reference 순으로 객체를 정의하여 GC에 우선순위로 결정되는데 "root set"의 기준으로 참조된 객체는 Strong reference으로 
    GC대상에서 제외되고 그 외 Soft, Weak, Phantom 은 unreachable 객체로 GC 대상이된다.
    Soft, Weak, Phantom reference는 java.lang.ref 패키지로 일반적인 주소참조가아닌 특별한 참조 구성을 위한 Reference Object API는 객체를 가리키는 
    특별한 참조를 통하여 프로그램상에서 GC를 연계할수있다. 메모리상에 많은 객체를 갖거나 객체가 반환전에 정리작업을 위해 사용되는 객체의 reachability를 조절하며 
    동작을 다르게 지정하여 GC대상 여부를 판별하는 부분에 사용자 코드로써 직접적인 개입이 가능하다.  이 후 마크 단계가 끝나면 "Sweep" 단계가 된다. 
    이 단계까지 "Mark"되지 않은 메모리의 객체는 모두 가비지이며 "Sweep"은 전체 메모리를 반복하여 "Mark"되지 않은 메모리 블록을 해제한다. 
    그리고 매번 GC수행시에 JVM은 객체를 이동시키고 참조를 업데이트해야한다. 그 과정에 업데이트된 참조가 객체가 이동하지 않은 상태에서 이동을 시도하면 문제가 되어 
    GC쓰레드 이 외 모든 쓰레드를 잠시 동결시킨다. 이것을 STW(Stop The World) 라고 불리는데, STW가 결국 Serial, Parallel, Parallel, 
    Concurrent Mark & Sweep, G1 GC방식 들을 이용한 옵션들로 GC성능을 일부제어한다.

## Other GC Collections
 - http://openjdk.java.net
 - https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6668279
 - http://www.kdgregory.com/index.php?page=java.refobj
 - https://d2.naver.com/helloworld/329631
 - https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html#wp1086917
 - https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html
