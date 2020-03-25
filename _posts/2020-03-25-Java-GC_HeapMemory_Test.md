---
layout: post
title: "[Java] GC가 Heap 메모리에서 해제하는 시점"
date: 2020-03-25
excerpt: "GC가 Heap 메모리에서 해제하는 시점은 참조형 변수를 호출한 메소드의 종료시점"
tags: [Java, Memory Leak, Heap, GC]
comments: true
---

## 메모리 Leak 테스트 중 알게된 점
- 메소드 안의 변수가 참조형 변수일 경우 Heap 메모리에 저장
- 참조형 변수를 참조하지 않는 시점은 해당 변수를 호출한 메소드가 종료된 시점
- 해당 변수를 호출하고 line을 지나간 시점 X, 메소드 블럭(`{}`)이 종료된 시점 O
- 따라서 **GC가 Heap 메모리에서 해제하는 시점은 참조형 변수를 호출한 메소드의 종료시점**

<br>

------------------------

<br>

## TEST 1.

{% highlight java %}
public class Outer {
	public String content = "This is Outer";

	private static class Inner {
		public String content = "This is Inner";
	}

	public Outer() {
		Inner[] inner = new Inner[1];
		for(int i=0; i<inner.length; i++) {
			// 인스턴스 생성은 하지만 현재 참조 X -> GC의 메모리 해제 대상
			inner[i] = new Inner();
		}
	}
}
{% endhighlight %}

{% highlight java %}
public class OuterTest {
	public static void main(String[] args) {
		// main 메소드가 종료되는 시점까지 메모리에 남아 참조되는 변수
		Outer[] outer = new Outer[10000000];

		for(int i=0; i<outer.length; i++) {
			outer[i] = new Outer();
		}

		System.out.println("GC Point"); // 여기서 GC를 수행
	}
}
{% endhighlight %}

![TEST1](https://user-images.githubusercontent.com/34757921/77542336-5cb4a780-6ee9-11ea-86a2-3e0ac7c4ebfa.png)

- Outer 인스턴스가 생성될 때 Heap 메모리에 생성된 Inner 인스턴스는 현재 참조하고 있지 않기 때문에 GC에 의해 메모리 해제
- **남아있는 Heap 메모리 : 현재 main 메소드에서 Outer 인스턴스 10,000,000개의 배열을 참조하고 있음 (용량이 큼)**

<br>

## TEST 2.
{% highlight java %}
public class Outer {
	public String content = "This is Outer";

	private static class Inner {
		public String content = "This is Inner";
	}

	public Outer() {
		Inner[] inner = new Inner[10000000];
		for(int i=0; i<inner.length; i++) {
			// 인스턴스 생성은 하지만 현재 참조 X -> GC의 메모리 해제 대상
			inner[i] = new Inner();
		}
	}
}
{% endhighlight %}

{% highlight java %}
public class OuterTest {
	public static void main(String[] args) {
		// main 메소드가 종료되는 시점까지 메모리에 남아 참조되는 변수
		Outer[] outer = new Outer[1];

		for(int i=0; i<outer.length; i++) {
			outer[i] = new Outer();
		}

		System.out.println("GC Point"); // 여기서 GC를 수행
	}
}
{% endhighlight %}

![TEST2](https://user-images.githubusercontent.com/34757921/77542432-840b7480-6ee9-11ea-86f9-576fca7d81cd.png)

- Outer 인스턴스가 생성될 때 Heap 메모리에 생성된 Inner 인스턴스는 현재 참조하고 있지 않기 때문에 GC에 의해 메모리 해제
- **남아있는 Heap 메모리 : 현재 main 메소드에서 Outer 인스턴스 1개의 배열을 참조하고 있음 (용량이 작음)**
