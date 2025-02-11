---
layout: single 
title: "Garbage Collection 알아보기"
date: 2025-01-20 00:00:00 +0900 
last_modified_at : 2025-01-20 00:00:00 +0900  
categories: [Java]
tags: [CS]
toc: true
toc_sticky: true
permalink: /:categories/:title

comments: true
---
# 서론
저번에 String, StringBuilder, StringBuffer를 비교하면서 String의 메모리 사용량이 들쑥날쑥 했다.  
이유는 정확히 모르지만, Garbage Collection이라고 생각하려고 했다. 하지만, 그렇기에는 근거가 없다.  
특히, 내가 Garbage Collection에 대해서 잘 모른다. 그래서 이번에는 Garbage Collection에 대해서 알아보는 시간을 가지려고 한다.   

# Garbage Collection(GC)

## GC란?
**메모리 관리 기법 중의 하나로, 프로그램이 동적으로 할당했던 메모리 영역 중에서 필요없게된 영역을 해제하는 기능이다.** 라고 위키 백과에서 소개하고 있다.   
  
나도 메모리를 자동으로 관리해 준다고만 알고 있다. 하지만, 여기서 집중해서 봐야 하는 말은 **필요 없게 된 영역** 이다. 어떤 것이 **필요 없는 게 된 영역** 일까??  

### GC가 지우는 대상
GC가 지우는 대상이라는 것은 위에 적혀있는 **필요 없게 된 영역**을 의미한다.   
  
GC는 2가지 기준을 통해서 객체가 필요없는 객체인지 판단한다. 그 2가지는 **도달성**과 **도달능력**이다. 해당 기준으로 **Reachable**과 **Unreachable**으로 나눠지게 된다.  
- **Reachable** : **객체가 참조되고 있는 상태를 의미**
- **Unreachable** : **객체가 참조되고 있지 않은 상태**로, 필요 없게 된 영역 => GC가 지우는 대상
  
#### Unreachable가 되는 경우
크게 2가지 경우가 있다. 
- **더 이상 참조되지 않는 객체**
- **고립된 상태**
  
여기서 고립된 상태라는 것은 참조하고 있는 객체가 더 이상 참조되지 않는 객체이다. 
  
## GC가 객체를 지우는 방식
**Mark**, **Sweep**, **Compact** 방식을 이용해서 객체를 지운다.
- Mark : Unreachable객체 찾기
- Sweep : Unreachable 객체 Heap영역에서 제거
- Compact : 분산된 객체들을 압축하는 과정으로 단편화를 해결하기 위해 사용
  
후술하겠지만, 해당 방식때문에 **Stop The World**라는 현상이 발생한다. 
  
## GC 동작 과정
먼저 Heap 영역의 구조를 확인해봐야한다.  

![heap](/assets/images/2025-01-21-Garbage-Collection/heap.png)   

해당 영역을 본다면 **Young영역**과 **Old영역**이 있다. 이 2가지 영역이 GC가 동작하면서 필요한 영역이다. 

### Young영역
**생성된 객체가 할당되는 영역**   

대부분의 객체는 오랬동안 남아있지 않다. 즉, Young영역에서 사라진다. 그래서 Young영역은 Old영역에 비해 크기가 작다. 그리고 영역의 크기가 작기 때문에, GC에 소요되는 시간이 적다.  

**Young영역**에서 일어나는 GC는 **Minor GC**라고 부른다.  

이러한 Young영역을 더 효율적으로 사용하기 위해 **Eden**, **survivor0**, **survivor1**으로 영역을 나눴다.  

![young](/assets/images/2025-01-21-Garbage-Collection/young.png)
  
#### Eden
**new를 통해 새로 생성된 객체가 존재하는 영역**으로, GC에서 제거되지 않은 객체들은 survivor영역으로 보낸다. 

#### survivor0 / survivor1
최소 1번의 **GC 과정에서 살아남은 객체가 존재하는 영역**이다.  

##### survivor가 2개인 이유
당연하게 성능때문에 2개이다.   
  
먼저 하나의 survivor은 to, 다른 하나는 from이다. 그리고 from에서 to로 이동한 후, 둘의 역할은 바뀐다. 즉, 객체가 있는 곳이 항상 from이다. 여기서 만약 survivor가 1개라면 어떨까? survivor에 있는 객체가 없어지면, **단편화**가 발생한다. 이 문제를 해결하는 것보다, 그냥 **survivor를 하나 더 만들어서 살아남은 객체를 복사하는 것이 더 효율적**이다.  

이렇기 때문에, 2개의 survivor에 모두 객체가 있다면 문제가 발생했다는 것을 알 수 있다.  

#### Minor GC
Young영역에서 일어나는 GC  
   
**과정**
1. **객체가 생성**되면 Young영역의 **Eden영역에 들어간다**.
2. **Eden영역이 가득**차면 **GC가 실행**된다.
3. **Reachable한 객체를 식별**한다. - Mark동작
4. Eden영역에서 **살아남은 객체**는 **survivor영역으로 이동**시킨다. 
5. 살아남지 못한 객체는 제거된다. - Sweep동작
6. **살아남은 객체**의 **age를 1씩 증가**시킨다. 
7. 다시 **Eden영역이 가득**차면, **GC가 발생**하고 Mark동작을 실시한다. 
8. **Eden영역와 survivor영역**에 있는 객체 중 **Reachable한 객체는 비어있는 survivor로 이동**한 후, age를 1씩 증가시킨다.
9. 위 과정을 반복한다.
  

### Old영역
**Young영역에서 살아남은 객체가 복사되는 영역**이다.  

Young영역보다 크게 할당되고, 그래서 GC는 많이 일어나지 않는다. 하지만, 한 번 할 때, 시간이 오래걸린다.  

**Old영역**에서 일어나는 GC를 **Major GC**라고 부른다.  

#### Major GC
Old영역에서 일어나는 GC  

**과정**
1. **Young영역**에 있는 객체의 **age값이 임계값에 도달**하면 **Old영역으로 이동**된다. 이 과정을 Promotion이라고 한다.
2. 위 과정이 반복되면서 Old영역의 공간이 부족해지면 GC가 실행된다.
3. **Old영역에 있는 모든 객체를 검사**하고, **Unreachable한 객체를 한꺼번에 제거**한다. 
   - Old영역은 Young영역보다 크기 때문에, **이 과정이 오래 걸린다**. 시간은 Young영역에 10배라고 한다. 
   - 그리고 **Stop The World**(GC가 실행중 실행중인 모든 프로세스가 멈추는 현상)가 발생한다. 
  
<br>

Major GC가 일어나면 쓰레드가 멈추고 Mark, Sweep작업을 한다. 이 과정에서 Stop The World(STW)가 발생한다. **STW가 발생하는 이유**는 힙에서 객체를 재배치할 때, **JVM은 해당 객체에 대한 모든 참조를 수정**해야한다. 그래서 모든 동작을 멈춘 후, 객체의 주소를 다시 참조해준다.  
<br>
이러한 문제를 해결하기 위해 GC알고리즘이 발전했다. 

## GC 알고리즘
STW를 해결하기 위해 다양한 GC알고리즘이 발전했다.  

### Serial GC
가장 단순한 형태의 GC이다.  
Minor GC에서는 Mark, Sweep을 사용하고 Major GC에서는 Mark, Sweep, Compact를 사용한다. 

**실행 명령어**
```
java -XX:+UseSerialGC -jar Application.java
```

### Parellel GC
Serial GC와 같지만, Minor GC를 멀티 쓰레드로 수행하는 알고리즘이다. 

**실행 명령어**
```
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
java -XX:+UseParallelGC -jar Application.java 
```

### Parallel Old GC
Minor GC만 멀티 쓰레드를 쓰는 Parellel GC에서 Major GC도 멀티 쓰레드를 쓰는 알고리즘이다.  
<br>

Mark Sweep Compact 방식이 아닌 Mark Summary Compact 방식을 사용한다.
- Summary방식은 Sweep방식과 다르게 Old영역을 나눠 여러 쓰레드가 병렬적으로 살아있는 객체를 찾는 방식이다. 

**실행 명령어**
```
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
java -XX:+UseParallelOldGC -jar Application.java
```

### CMS GC
어플리케이션 쓰레드와 GC 쓰레드가 동시에 실행되어 STW을 최대한 줄이는 GC 알고리즘이다.  
<br>

매우 복잡하며, CPU 사용량이 다른 GC알고리즘에 비해 높다. 그리고 메모리 파편화 문제도 있어서, Java 14에서는 사용이 중지되었다. 
  
**실행 명령어**
```
java -XX:+UseConcMarkSweepGC -jar Application.java
```

### G1 GC
여러 알고리즘을 보면서 G1 GC가 굉장히 중요한 것 같다는 생각이 들었다. 그래서 좀 더 찾아보았다.   

Parellel GC와 CMS GC의 대안으로 나온 GC이다. 그리고 **일시 중시 시간 측면에서 더 나은 예측 가능성**을 제공하면서 **더 좋은 처리량을 달성**하도록 설계되었다.   

구조는 위와 같게 **Young Generation**과 **Old Generation** 영역으로 나누어져 있다. Young Generation역시 Eden과 2개의 Survivor로 나누어져 있다.   

조금 다른 점은 아래 그림에서 확인할 수 있다.   
![g1](/assets/images/2025-01-21-Garbage-Collection/g1.png){: width="50%"}   

각각의 블럭은 1MB ~ 32MB다. 블럭의 수는 대략 2048개이며 이 수를 초과할 수 없다.   

#### 과정
**1번 초기 마크**   
**Old Generation에서 Reachable한 객체를 식별하고 마크**한다. 이 단계에서는 어플리케이션의 스레드를 사용하여 어플리케이션의 성능에 최소한의 영향을 준다.
<br>

**2번 동시 마크**    
1번 과정 후, **힙 영역에 있는 모든 영역의 객체가 살아있는지 식별**한다. 이 과정 역시 스레드를 이용하며, 어플리케이션은 계속해서 동작중이다. 즉, STW가 없다.
<br>

**3번 remark**   
2번 과정을 한 번 완료 후, **수정된 객체를 식별하여 remark를 수행**한다. 즉, 위 과정을 하며 Unreachable 혹은 Reachable한 상태가된 객체를 식별하고 마크한다.

이 과정은 살아남은 객체를 결정하는데 있어 정확성을 보증해준다. 그리고 **이 과정에서 어플리케이션은 잠깐 멈춘다**. 즉, STW가 발생한다.
<br>

**4번 동시 정리**   
3번 과정이 끝난 후, 더 이상 사용되지 않는 영역에서 메모리를 회수한다. 이 과정도 어플리케이션 스레드와 동시에 진행된다.
<br>

**5번 피난**   
**살아있는 객체를 다른 영역으로 이동**시키는 과정이다.   
메모리가 가득찬 영역을 선정하고 대피 우선순위를 정한다. 즉, 메모리의 공간이 충분히자 않은 영역을 먼저 피난 시킨다.   

이 단계의 목표는 객체를 비워 파편화를 최소로 하고, 전체적인 힙 활용도를 향상시키는데 있다.
<br>

**6번 반복**   
위 과정을 반복한다.   

#### 특징
- **혼합 GC**
  - Young Generation과 Old Generation에 GC를 수행하기 위해 사용한다. 이를 통해 힙 활용도를 높이고 STW를 피할 수 있다.
- **수집 셋**
  - GC가 실행되는 영역 후보를 관리한다. 이 후보 영역들은 GC 사이클 동안 수집된다.
- **일시 정지 시간 목표**
  - G1 GC는 사용자가 정한 정지 시간을 달성하는 것을 목표로 한다. 그래서 수집 셋의 크기와 기다 매개변수를 제어하여 이 목표를 달성하기 위해 동작을 동적으로 조정한다.

#### 거대한 객체 (Humongous Object)
G1 GC에 **몇몇 영역 사이즈의 절반보다 큰 객체**를 **"거대한 객체"**라고 한다.
<br>

이 객체는 너무 커서 일반 객체와는 다르게 처리된다.   
- 작은 조각으로 나누지 않고, 특별한 메모리 공간에서 보관한다. 
- 이러한 이유는 파편화를 방지하기 위해서이다. 
- 해당 공간에서 이동하지 않으며, Full GC를 통해 제자리 압축이 된다. 


**실행 명령어**
```
java -XX:+UseG1GC -jar Application.java
```

### Shenandoah GC
CMS의 단편화, G1의 pause문제를 해결한 GC이다.   

**실행 명령어**
```
java -XX:+UseShenandoahGC -jar Application.java
```

### Z GC
대용량 메모리를 적은 지연으로 처리하기 위해 기획된 GC이다.   
해당 GC에 대해서 잘 설명된 글이다. 궁금하면 참고해도 될것같다.   

[Z GC 잘 설명된 글](https://d2.naver.com/helloworld/0128759)   

**실행 명령어**
```
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -jar Application.java
```

---
# 참고자료
[https://inpa.tistory.com/entry/JAVA-☕-가비지-컬렉션GC-동작-원리-알고리즘-💯-총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)

[https://velog.io/@yarogono/Java가비지-컬렉터Garbage-Collector란](https://velog.io/@yarogono/Java%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%ED%84%B0Garbage-Collector%EB%9E%80)

[https://mangkyu.tistory.com/118](https://mangkyu.tistory.com/118)

[https://azelhhh.tistory.com/106](https://azelhhh.tistory.com/106)

[https://stackoverflow.com/questions/10695298/java-gc-why-two-survivor-regions](https://stackoverflow.com/questions/10695298/java-gc-why-two-survivor-regions)

[https://stackoverflow.com/questions/21476348/java-gc-why-two-survivor-spaces](https://stackoverflow.com/questions/21476348/java-gc-why-two-survivor-spaces)

[https://stackoverflow.com/questions/16695874/why-does-the-jvm-full-gc-need-to-stop-the-world?rq=3](https://stackoverflow.com/questions/16695874/why-does-the-jvm-full-gc-need-to-stop-the-world?rq=3)

[https://mozzi-devlog.tistory.com/15](https://mozzi-devlog.tistory.com/15)

[https://medium.com/@perspectivementor/how-g1-garbage-collector-work-in-java-e468a94ebed6](https://medium.com/@perspectivementor/how-g1-garbage-collector-work-in-java-e468a94ebed6)

[https://www.oracle.com/technical-resources/articles/java/g1gc.html](https://www.oracle.com/technical-resources/articles/java/g1gc.html)