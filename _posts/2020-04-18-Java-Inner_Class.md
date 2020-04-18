---
layout: post
title: "[Java] Inner Class (static Inner Class와 non-static Inner Class 비교)"
date: 2020-04-18
excerpt: "Inner Class는 static Inner Class 또는 non-static Inner Class로 구분된다. 인스턴스 생성 방법과 Heap 메모리 사용에 차이가 존재한다."
tags: [Java]
comments: true
---

# Inner Class 란?
Class 또는 Interface 내부에서 선언된 Class이다. 중첩 클래스라고도 한다. <br>
Inner Class는 **static Inner Class** 또는 **non-static Inner Class** 로 구분된다.

- static Inner Class와 non-static Inner Class는 모두 인스턴스 생성이 가능하다.
- 인스턴스 생성 방법에는 차이가 있다.
  + non-static Inner Class: **인스턴스 생성시 부모의 인스턴스 필요**
  + static Inner Class: **독립적인 인스턴스 생성 가능**

{% highlight java %}
public class OuterClass {
   public class NonStatic { }
   public static class Static { }

   public static void main(String[] args) {
      Static s = new Static(); // static Inner Class 인스턴스 생성

      // ERROR
//    NonStatic ns = new NonStatic();
      OuterClass outer = new OuterClass();
      NonStatic ns = outer.new NonStatic(); // non-static Inner Class 인스턴스 생성
   }
}
{% endhighlight %}

기본적인 Class (바깥의 Class, *OuterClass*)는 static class이다. `static` 이 생략된 채로 사용하기 때문에 `static` 을 붙이게 되면 에러가 발생한다.

- **static Class는 non-static Class를 상속할 수 없다.**
- non-static Class는 Inner Class에서만 가능하다.

{% highlight java %}
public class OuterClass {
   public class NonStatic { }
   public static class Static { }

   // ERROR
// public static class A extends NonStatic { }
   public class B extends NonStatic { }
   public static class C extends static { }
   public class D extends Static { }

   public static class E extends OuterClass { } // OuterClass는 static Class
}
{% endhighlight %}

<br>

static 메소드와 static 멤버는 동일하지만, **static Class의 인스턴스는 동일하지 않다.**

{% highlight java %}
public class OuterClass {
   public static class Static { }

   public static void main(String[] args) {
      Static s1 = new Static();
      Static s2 = new Static();
      System.out.println("s1 : " + s1 + " / " + s1.hashCode());
      System.out.println("s2 : " + s2 + " / " + s2.hashCode());
      System.out.println("s1 == s2 ? " + (s1 == s2));
   }
}
{% endhighlight %}

```
// 결과
s1 : kr.study.test.OuterClass$Static@15db9742 / 366712642
s2 : kr.study.test.OuterClass$Static@6d06d69c / 1829164700
s1 == s2 ? false
```
<br>

## Heap 메모리 비교
- **non-static Inner Class > static Inner Class** <br>
non-static Inner Class에는 클래스에 대한 참조 (내부적으로 outer Class에 대한 참조)가 있기 때문에 메모리 소비가 더 크다.

### Heap 메모리 비교 테스트 1 - non-static Inner Class 테스트

{% highlight java %}
public class Outer {
   public String content = "This is Outer";

   private class Inner { // non-static
      public String content = "This is Inner";
   }

   public Outer() {
      Inner[] inner = new Inner[10000000];
      for (int i=0; i<inner.length; i++) {
         inner[i] = new Inner();
      }
   }
}
{% endhighlight %}

{% highlight java %}
public class OuterTest {
   public static void main(String[] args) {
      Outer[] outer = new Outer[1];

      for (int i=0; i<outer.length; i++) {
         outer[i] = new Outer();
      }
   }
}
{% endhighlight %}

![TEST1_non-static](https://user-images.githubusercontent.com/34757921/79630181-1cfc7b00-818a-11ea-9523-59332d701d6e.png)


### Heap 메모리 비교 테스트 2 - static Inner Class 테스트
{% highlight java %}
private staic class Inner { // static 변경
{% endhighlight %}

![TEST2_static](https://user-images.githubusercontent.com/34757921/79630184-284fa680-818a-11ea-88cb-013876ad8d36.png)

<br>

------------------------

+) 출처 및 참조 :

<https://stackoverflow.com/questions/20380600/gc-performance-hit-for-inner-class-vs-static-nested-class> <br>
<https://m.blog.naver.com/iq_up/220013622883>
