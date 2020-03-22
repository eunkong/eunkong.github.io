---
layout: post
title: "[Java] String Pool"
date: 2020-03-22
excerpt: "Java 7부터는 String Pool의 위치가 Heap 영역으로 이동됐다. Heap 영역으로 이동된 String Pool의 문자열들은 GC의 대상이 된다."
tags: [Java, String Pool, String, Heap, PermGen]
comments: true
---

## String 생성 방법
 Java에서는 String을 생성하는 방법에는 2가지가 있다.

 1. new 연산자를 이용한 방식
  - Heap 영역에 저장
  - 같은 String의 값(Value)을 가진 String 객체를 생성하더라도 새로 생성됨
  - **동일한 주소의 String 재사용 X**

 2. 리터럴을 이용한 방식
  - String Pool 영역에 저장
  - 내부적으로 String의 intern() 메소드 호출
        + intern() 메소드에서는 주어진 문자열이 String Pool에 먼저 등록되어있는지 확인
           + 등록되어 있지 않다면 intern() 메소드가 자동 호출되면서 등록
           + 등록되어 있다면 그대로 사용
  - **동일한 String의 값(Value)인 경우 동일한 주소의 String을 재사용**


{% highlight java %}
String a = "Hello"; // String Pool
String b = "Hello"; // String Pool
String c = new String("Hello"); // Heap
String d = new String("Hello"); // Heap

System.out.println(a==b); // true
System.out.println(a==c); // false
System.out.println(c==d); // false

System.out.println("a : " + System.identityHashCode(a)); // a : 935563448
System.out.println("b : " + System.identityHashCode(b)); // b : 935563448
System.out.println("c : " + System.identityHashCode(c)); // c : 139607202
System.out.println("d : " + System.identityHashCode(d)); // d : 1326101490
{% endhighlight %}
+) == 연산자는 객체의 주소 값을 비교하기 때문에 위와 같은 결과가 나온다.



## String Pool
Java 6까지는 String Pool의 위치가 PermGen(Permanent Generation)에 있었다. **PermGen 영역은 고정된 크기** 이기 때문에 Runtime 중에 사이즈가 확장되지 않는다. PermGen 영역의 크기를 늘릴 수는 있으나, Runtime 중에 변경할 수는 없다. 따라서 Runtime 중에 String Pool에 저장될는 String의 크기가 거대하거나 방대할 경우 `OutOfMemory` 가 발생한다.

![String_Heap_Non-Heap](https://user-images.githubusercontent.com/34757921/77240937-598e9280-6c2f-11ea-9408-61492e2ae690.PNG)

이때문에 Java 7부터는 String Pool의 위치가 Heap 영역으로 이동됐다. **Heap 영역으로 이동된 String Pool의 문자열들은 GC의 대상** 이 된다. 따라서 Runtime 중에도 메모리에서 해제가 될 수 있다.

Java 8부터는 PermGen 영역이 사라지고 **Metaspace**로 변경되었다.

+) Metaspace 관련 내용은 공부 필요

------------------------

+) 출처 및 참조 :

<https://hoit89.tistory.com/entry/String-Stringintern-String-poolequals>
<https://doohong.github.io/2018/03/04/java-string-pool/>
