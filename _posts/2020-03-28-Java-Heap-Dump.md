---
layout: post
title: "[Java] Heap Dump 생성하기"
date: 2020-03-28
excerpt: "OOME가 발생한다면 Heap Dump를 생성하여 분석할 필요가 있다."
tags: [Java, Heap, Heap Dump, Memory Leak, OOME, jcmd, jmap]
comments: true
---

# Heap
Java의 메모리 영역 중 하나이다. 간단하게는 **인스턴스 같은 참조형 데이터가 저장되는 영역** 이라고 생각하면 된다. Heap에는 Object정보, Class 메타정보 및 각 Object들에 대한 참조 정보들이 저장된다. Heap에 존재하는 모든 Object는 Root 노드부터 Parent/Children 형식의 tree 구조를 형성한다.

Heap 정보를 분석하는 것은 Java의 메모리 문제(Out Of Memory Error, OOME)를 해결하기 위한 중요한 작업이다.
(Java는 Memory Leak가 없는 구조이지만, 실제로 Memory Leak가 발생할 수 있다.)

Java에서는 메모리 할당 및 해제에 대한 기능은 자체적인 GC(Garbage Collection)가 처리한다. 다른 언어보다 개발의 편의성을 제공하지만 이에 따른 **Memory 문제점을 내포** 하고 있다.

﻿```OutOfMemoryError: GC overhead limit exceeded```

**OOME가 발생한다면 Heap Dump를 생성하여 분석** 할 필요가 있다.

Java의 벤더 별로 Heap Dump를 생성하는 방법이 조금씩 다르다.

<br>

# Heap Dump 생성하기

RHEL 환경에서 테스트

## JDK 1.8

1. JVM이 실행되고 있는 서버로 접속

2. Java pid 확인 <br> ```# jcmd```

3. help 명령어 테스트 <br> ```# jcmd $JAVA_PID help```
   - 만약 실행되지 않는다면, 해당 프로세스의 소유자가 다른 경우일 수도 있다.
   - 그렇다면 해당 프로세스의 소유자로 시도 <br> ```# sudo -u $PROCESS_OWNER jcmd $JAVA_PID help``` <br>
   참조 : <https://gist.github.com/noahlz/865cc30e0fd93ad48369>

4. Heap Dump 생성 <br> ```﻿# jcmd $JAVA_PID GC.heap_dump $FILENAME.hprof```
   - 다른 소유자일 때 <br> ```# sudo -u $PROCESS_OWNER jcmd $JAVA_PID GC.heap_dump $FILENAME.hprof```
      - 만약 ```Permission denied``` 가 발생한다면 파일 생성 디렉토리의 권한을 확인하여 변경한다.
      - 권한 변경: 777 <br> ```# chmod 777 $DIRECTORY```

<br>

## JDK 1.6 ~ JDK 1.7
JDK 1.8부터 ```jcmd``` 사용할 수 있으므로 이전 버전에서는 사용할 수 없다. 따라서 JDK 1.6 ~ JDK 1.7에서는 ```jmap``` 을 사용할 수 있다.

현재 openjdk 1.7.0.121을 사용중인데 해당 JDK안에는 ```jmap``` 이 없기 때문에 직접 설치하여 사용했다. 현재 yum을 사용할 수 없으므로 rpm을 직접 다운받아서 적용했다.

+) 참조 : <https://stackoverflow.com/questions/25715067/jmap-command-not-found>

1. openjdk-devel.rpm 다운로드 <br>
   <https://buildlogs.centos.org/c7.1611.u/java-1.7.0-openjdk/20161121184241/1.7.0.121-2.6.8.0.el7_3.x86_64/>

2. rpm 설치 <br> ```# rpm -ivh java-1.7.0-openjdk-devel-1.7.0.121-2.6.8.0.el7_3.x86_64.rpm```

3. Java pid 확인 <br> ```# jps -v```

4. Heap dump 생성 <br> ```# jmap -dump:format=b,file=heapdump.hprof $JAVA_PID```


**+) Heap Dump 이용하여 분석하는 방법은 공부 필요 !**

<br>

------------------------

+) 출처 및 참조 :
<https://m.blog.naver.com/bumsukoh/110123438564>
<https://ktdsoss.tistory.com/439>
