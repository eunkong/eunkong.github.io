---
layout: post
title: "[Java] Thread Dump And Thread Kill"
date: 2020-05-10
excerpt: "Thread Dump 생성 및 분석 그리고 Thread Kill"
tags: [Java, Thread, Thread Dump]
comments: true
---

특정 URI를 호출했을 때 무한정 로딩이 일어나는 상황이 발생했다. (Timeout도 발생하지 않음) <br>
따라서 Thread Dump를 통해 문제를 해결해 보았다.


## Thread Dump 생성 (Linux 환경)

1. JAVA_PID 확인 <br>
   `# ps -ef | grep java`

2. `jstack`을 사용하여 Thread Dump를 thread_dump.txt파일로 생성 <br>
   `# sudo -u $JAVA_PROCESS_OWNER jstack $JAVA_PID > thread_dump.txt`

<br>


## Thread Dump 분석

#### thread_dump.txt 분석 결과

1. 다수의 Thread들의 상태

   ```java
   "http--0.0.0.0-8443-5" daemon prio=10 tid=0x00007f234004c000 nid=0x386c waiting for monitor entry [0x00007f2345f5a000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at ...
	- waiting to lock <0x00000007a0d5b8a0> (a java.lang.Object)
	at ...
	...
   ```

   + `java.lang.Thread.State: BLOCKED`
   + `- wating to lock <0x00000007a0d5b8a0>`

   **"http--0.0.0.0-8443-5"** Thread가 ***<0x00000007a0d5b8a0>*** `Lock`을 소유하기 위해 대기중이다. (`BLOCKED`)


2. 이상 Thread의 상태

   ```java
   "http--0.0.0.0-8443-3" daemon prio=10 tid=0x00007f2340005800 nid=0x3811 waiting on condition [0x00007f22d0f90000]
      java.lang.Thread.State: WAITING (parking)
   	at sun.misc.Unsafe.park(Native Method)
   	- parking to wait for  <0x00000007a4ee4c70> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
   	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
   	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
   	...
   	- locked <0x00000007a0d5b8a0> (a java.lang.Object)
   	at ...
   	...
   ```

   + `java.lang.Thread.State: WAITING (parking)`
   + `- parking to wait for <0x00000007a4ee4c70>`
   + `- locked <0x00000007a0d5b8a0>`

   **"http--0.0.0.0-8443-3"** Thread가 ***<0x00000007a0d5b8a0>*** `Lock`을 소유하고 있고, park() 메소드에 의해 ***<0x00000007a4ee4c70>*** 을 대기중이다. (`WAITING`)

<br>


## 결론

**"http--0.0.0.0-8443-3"** Thread의 작업이 완료되지 않아 ***<0x00000007a0d5b8a0>*** `Lock`을 계속 소유중이므로, 해당 `Lock`을 얻기 위해 다른 Thread(**"http--0.0.0.0-8443-5"**, ...)들이 `BLOCKED` 상태로 쌓이게 된다.

Thread Dump에서 확인한 stack trace 기준으로 확인해보니, `BLOCKED` 상태의 Thread(**"http--0.0.0.0-8443-5"**, ...)가 멈춰있는 메소드의 `synchronized` 블록에서 특정 자원을 점유하지 못하여 대기중인 상태였다. (해당 자원은 **"http--0.0.0.0-8443-3"** Thread가 점유하고 있는 상태)

<br>


## 해결

1. WAS 재시작
2. Process Kill
3. (Process Kill 없이) Thread Kill
   + Thread를 Kill하면 Process도 함께 종료되기 때문에, 오픈소스인 **jkillthread** 를 사용하여 Process Kill 없이 Thread를 Kill 해보았다.
   + **jkillthread**
      + 다운로드 : <https://github.com/jglick/jkillthread>
      + 사용방법 (Linux 환경) <br>
      `# sudo -u $JAVA_PROCESS_OWNER java -jar jkillthread-1.0.jar $JAVA_PID $THREAD_NAME` <br>
      `# sudo -u user java -jar jkillthread-1.0.jar 14192 "http--0.0.0.0-8443-3"`
   + 테스트 결과
      + 이상 Thread (**"http--0.0.0.0-8443-3"**(`WATING`)) Kill 완료 후, 타 Thread (**"http--0.0.0.0-8443-5"**(`BLOCKED`), ...) 정상 동작 및 처리되는 것을 확인

------------------------

+) 참조 :

<https://d2.naver.com/helloworld/10963>
