---
layout: single 
title: "String vs StringBuilder vs StringBuffer"
date: 2025-01-10 15:00:00 +0900 
last_modified_at : 2025-01-10 15:00:00 +0900 
categories: [Java]
tags: [Java, CS, 자료구조]
toc: true
toc_sticky: true
permalink: /:categories/:title

comments: true
---
# 결론
시간이 없는 사람들을 위해 결론을 먼저 말하자면, 경우에 따라서 다릅니다. 하지만, 보통의 경우 **크기가 커지면 커질수록 StringBuilder가 더 많은 메모리를 차지**합니다. 

# 서론
지난번 [포스팅](/algorithm/백준-24337-가희와-탑#stringbuilder와-string) 에서 시간 단축을 위해 출력을 String에서 StringBuilder로 바꿨습니다. 이렇게 바꾸니 시간 단축만 된것이 아니라 메모리도 단축되었습니다. 그래서 **'String보다 StringBuilder가 더 적은 메모리를 가지나?'**라는 한 가지 의문을 가졌고, 이 물음의 정답을 찾아보겠습니다.  

해당 물음에 정답을 알아보기에 앞서, String, StringBuilder, StringBuffer에 관해 아주 간략하게 설명하겠습니다.   

# String, StringBuilder, StringBuffer
다른 블로그에서 아주 잘 설명되어 있는 내용이라서 아주 간략하게 설명하겠습니다. 

## String
String은 불변으로, 한번 선언하면 바꿀 수 없는 자료형입니다. 그래서 문자열의 변경이나 추가가 안 됩니다. 되는 것처럼 보이지만, 내부에서는 새로운 문자열을 생성하여 바꾸는 것이고, 이전의 문자열을 제거 대신이 된 채 메모리에 남게 됩니다.   

## StringBuilder
StringBuilder는 가변으로, 계속해서 바꿀 수 있습니다. 또한 StringBuffer와 다르게 스레드 환경에서 안전하지는 않습니다. 그렇기 때문에 싱글 스레드에서 더 빠른 성능을 가집니다.   

## StringBuffer
StringBuffer는 가변으로, 계속해서 바꿀 수 있습니다. 또한 스레드 환경에서 안전합니다. 스레드 환경에서 안전하게 만들기 위해 StringBuilder와 다르게 함수 앞에 `synchronized`이 붙어 있습니다.   

## 시간복잡도, 공간복잡도 비교
제가 직접 5회, 100회, 10,000회, 10,000,000회를 실행하며 시간복잡도와 공간복잡도를 구했습니다.    
   
### 시간복잡도

|걸린시간(ms)|5회|100회|10,000회|10,000,000회|
|---|---:|---:|---:|---:|
|String|18 ms|18 ms|132 ms|41,200 ms|
|StringBuilder|0 ms|0 ms|18 ms|10,125 ms|
|StringBuffer|0 ms|0 ms|22 ms|10,039 ms|   
   
모두가 예상했고, 저도 예상했다시피 **StringBuilder와 StringBuffer가 String보다 더 빠릅니다.** 

### 공간 복잡도

|메모리(byte)|5회|100회|10,000회|10,000,000회|
|---|---:|---:|---:|---:|
|String|2,371,856 byte|2,371,856 byte|3,420,456 byte|10,232,320 byte|
|StringBuilder|2,097,152 byte|2,097,152 byte|2,880,008 byte|664,254,976 byte|
|StringBuffer|2,097,152 byte|2,097,152 byte|2,880,008 byte|664,043,112 byte|   

저의 예상과 다르게 **10,000,000회에서는 StringBuilder와 StringBuffer가 더 많은 메모리를 차지**했습니다. 그래서 이렇게된 이유를 찾아보았습니다. 

#### Buffer 사이즈
**StringBuilder와 StringBuffer의 초기 버퍼 사이즈는 16**입니다. 그리고 **이 사이즈는 초과할 때마다 2배씩 늘어납니다**. 즉, 버퍼 사이즈가 1024에서 그다음으로 늘어날 때는 2048이 됩니다. 그럼 1024에서 문자를 하나만 넣어도, 2048로 사이즈가 만들어지고, 남은 1023의 공간은 낭비가 됩니다. 그래서 **사이즈가 커지면 커질수록 메모리를 많이 잡아먹는 것입니다.**   

![버퍼 사이즈](/assets/images/2025-01-01-String-vs-StringBuilder-vs-StringBuffer/버퍼사이즈.png)   

저는 무조건 StringBuilder가 String보다 메모리가 적다고 생각했습니다. 하지만, 이는 틀렸습니다. 그래서 제가 기존에 알고있던 StringBuilder가 String보다 빠르다는 것도 다시 알아봐야겠다고 생각했습니다.   

### String과 StringBuilder 속도
이전에 비교했던 속도는 사실 조금 이상했습니다. String은 계속해서 I/O를 하고 있었고, StringBuilder는 한 번 만 I/O가 진행되었습니다. 그래서 위에서 했던 것은 사실 String과 StringBuilder의 속도 차이가 아니라 I/O에서 오는 속도 차이일 수도 있습니다. 그렇기때문에, 이번에는 I/O없이 속도를 비교해보겠습니다.   

#### 속도 비교
이전과 같이 제가 직접 5회, 100회, 10,000회, 10,000,000회를 실행하며 속도를 구했습니다.   

|걸린시간(ms)|5회|100회|10,000회|10,000,000회|
|---|---:|---:|---:|---:|
|String|0 ms|0 ms|2 ms|311 ms|
|StringBuilder|0 ms|0 ms|2 ms|367 ms|   
   
위와 같은 결과가 나왔습니다. 즉, **String은 StringBuilder보다 느리지 않습니다.** 여기서 알 수 있는 것은 I/O가 없다면 속도는 거의 차이가 없다는 것입니다. 하지만, 많은 알고리즘 문제에서 I/O을 줄이기 위해 문자열을 한 번에 이어붙여서 출력합니다. 그리고 **이어붙이는 상황에서는 당연히 String이 StringBuilder보다 느립니다.** 

#### 버퍼 사이즈 복사
위에서 나오는 조금의 차이는 버퍼 사이즈를 늘리는 과정에서 일어나는 복사가 아닐지 생각해 봤습니다. 그래서 이 부분은 코드로 직접 확인해봤습니다.   

먼저 StringBuilder의 append메소드를 확인해봤습니다.   
![childAppend](/assets/images/2025-01-01-String-vs-StringBuilder-vs-StringBuffer/childAppend.png)   
이를 보면 부모의 append를 그대로 가지고 오고 있습니다.   

그래서 부모의 append를 확인했습니다. 
![append](/assets/images/2025-01-01-String-vs-StringBuilder-vs-StringBuffer/append.png)   
제가 박스친 ensureCapacityInternal 메소드가 보입니다. 해당 메소드를 확인해보면 아래와 같습니다.   

![ensureCapacityInternal](/assets/images/2025-01-01-String-vs-StringBuilder-vs-StringBuffer/ensureCapacityInternal.png)   
그리고 해당 메소드를 보면, 원래 길이보다 입력으로 들어온 길이가 짧으면, value라는 배열에 복사를 합니다. 그리고 이 value가 buffer라고 이해하면 됩니다.   

여기서 알 수 있는 것은 **buffer 사이즈를 초기에 쓸 만큼 설정하는 것이 시간 측면에서 좀 더 효율적**이라는 것입니다. 그렇지 않으면 buffer 사이즈가 늘어날 때마다 계속해서 배열 복사가 일어나고, 그만큼 시간이 소모됩니다.   

#### 코드
제가 실험에 사용했던 코드는 토클로 넣어놓겠습니다. 자바는 17버전을 사용했습니다. 

<details>
  <summary>코드</summary>
  <div markdown="1">
  ```
  public class Main {
    static int[] loopNumArr = {5, 100, 1_0000, 1000_0000};
    static int loopNum;

    public static void main(String[] args) {
        loopNum = loopNumArr[3];

//        methodString();
//        methodStringBuilder();
//        methodStringBuffer();
//        methodPrint();
//        timeCompare();
//        copyOrNot();
    }

    static void methodString() {
        /*
                첫 번째 방법
            - String으로 loopNum번 출력하고 시간 및 메모리 보기
         */
        long beforeTime = System.currentTimeMillis();

        for (int i = 0; i < loopNum; i++) {
            System.out.println(i + "번 출력");
        }
        long afterTime = System.currentTimeMillis();

        System.out.println(Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory() + " byte");
        System.out.println((afterTime - beforeTime) + "ms");
        System.out.println("loop " + loopNum + "번 일 때 String 방법");
    }

    static void methodStringBuilder() {
        /*
                두 번째 방법
            - StringBuilder로 loopNum번 출력하고 시간 및 메모리 보기
         */
        long beforeTime = System.currentTimeMillis();

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < loopNum; i++) {
            sb.append(i).append("번 출력").append("\n");
        }
        System.out.println(sb);

        long afterTime = System.currentTimeMillis();

        System.out.println(Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory() + " byte");
        System.out.println((afterTime - beforeTime) + "ms");
        System.out.println("loop " + loopNum + "번 일 때 Builder 방법");
    }

    static void methodStringBuffer() {
        /*
                세 번째 방법
            - StringBuffer로 loopNum번 출력하고 시간 및 메모리 보기
         */
        long beforeTime = System.currentTimeMillis();

        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < loopNum; i++) {
            sb.append(i).append("번 출력").append("\n");
        }
        System.out.println(sb);

        long afterTime = System.currentTimeMillis();

        System.out.println(Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory() + " byte");
        System.out.println((afterTime - beforeTime) + "ms");
        System.out.println("loop " + loopNum + "번 일 때 Buffer 방법");

    }

    static void methodPrint() {
        /*
                네 번째 방법
            - 그냥 print로 loopNum번 출력하고 시간 및 메모리 보기
         */
        long beforeTime = System.currentTimeMillis();


        for (int i = 0; i < loopNum; i++) {
            System.out.println();
        }
        System.out.println("The end");

        long afterTime = System.currentTimeMillis();

        System.out.println(Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory() + " byte");
        System.out.println((afterTime - beforeTime) + "ms");
        System.out.println("loop " + loopNum + "번 일 때 print 방법");
    }

    static void timeCompare() {
        StringTime();
        StringBuilderTime();
    }

    static void StringTime() {
        /*
                목적은 String을 변화 안시키면 속도의 차이가 있는지 확인하기
                String을 이어 붙이는 것이 아닌, 그냥 가지고 있기
         */
        long beforeTime = System.currentTimeMillis();

        String tmp;
        for (int i = 0; i < loopNum; i++) {
            tmp = String.valueOf(i);
        }

        long afterTime = System.currentTimeMillis();

        System.out.println("String");
        System.out.println((afterTime - beforeTime) + "ms");
    }

    static void StringBuilderTime() {
        /*
                StringBuilder에 값 추가
         */
        long beforeTime = System.currentTimeMillis();

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < loopNum; i++) {
            sb.append(i);
        }

        long afterTime = System.currentTimeMillis();

        System.out.println("StringBuilder");
        System.out.println((afterTime - beforeTime) + "ms");
    }

    static void copyOrNot() {
        int loopNum = 2 << 22;
        long beforeTime, afterTime;

        beforeTime = System.currentTimeMillis();
        goCopy(loopNum);
        afterTime = System.currentTimeMillis();

        System.out.println("copy");
        System.out.println((afterTime - beforeTime) + "ms");


        beforeTime = System.currentTimeMillis();
        notCopy(loopNum);
        afterTime = System.currentTimeMillis();

        System.out.println("no copy");
        System.out.println((afterTime - beforeTime) + "ms");
    }

    static void goCopy(int loopNum) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < loopNum; i++) {
            sb.append(i);
        }
    }

    static void notCopy(int sbSize) {
        StringBuilder sb = new StringBuilder(sbSize);
        for (int i = 0; i < sbSize; i++) {
            sb.append(i);
        }
    }
}
  ```
  </div>

</details>

## 이상한 점
String, StringBuilder, StringBuffer를 10,000,000회 돌릴 때, 이상한 점이 하나 있었습니다. 그것은 String의 메모리 사용량이 일정하지 않았습니다. 가끔씩 기존에 적혀 있던 것의 약 50배 정도의 메모리를 사용했습니다. 그래서 제가 했던 가정은 Garbage Collector가 메모리를 안비워서 생긴 문제라는 가정을 새웠습니다. 이러한 과정이 맞는지 틀리는지는 Garbage Collector에 관해 공부해서 추후에 포스팅해 보겠습니다. 