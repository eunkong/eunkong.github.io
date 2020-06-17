---
layout: post
title: "[Java] Java Security Provider"
date: 2020-06-17
excerpt: "JCA와 커스텀 Provider 적용하는 방법"
tags: [Java, JCA, JCE, Java Security, Provider]
comments: true
---

# JCA (Java Cryptography Architecture)

- Java에서 제공하는 **보안 관련 기능에서 중요한 부분**
- Provider 아키텍쳐, 디지털 서명, 메시지 다이제스트(hashes), 인증서, 인증서 유효성 검사, 암호화, 키 생성 및 관리, secure 난수 생성 등의 API를 제공
- 개발자는 해당 API들을 사용하여 보안 관련 기능들을 Applicaiton에 쉽게 적용 가능
- **JCA** 와 **JCE (Java Cryptography Extention)**
	+ JDK 1.4 이전: JCA, JCE로 별도의 구성
	+ JDK 1.4 이후: JCA, JCE가 JDK에 포함되어 제공됨에 따라 구분이 없어지고 있음
	+ JCE는 JCA와 같은 아키텍쳐를 사용하고 있기 때문에 **JCE는 JCA의 일부** 라고 볼 수 있다.

<br>

# Cryptographic Service Providers (CSP)

`java.security.Provider` 는 모든 Provider의 기본 Class이다.

<br>

![Provider_Structure](https://user-images.githubusercontent.com/34757921/84899914-497a3800-b0e4-11ea-8c8d-7d867184ba65.PNG)

CSP는 `java.security.Provider` 클래스의 구현체이며, 해당 **Provider의 이름** 과 **구현하는 보안 서비스, 알고리즘 목록을 가지고 있는 클래스의 인스턴스** 를 포함하고 있다.

<br>

![Provider_Flow](https://user-images.githubusercontent.com/34757921/84899939-50a14600-b0e4-11ea-8dea-b6759de64750.PNG)

각 CSP가 제공하는 보안 서비스, 알고리즘이 중복이 되는 경우가 있다. 이 경우에 특정 Provider를 지정하지 않고 여러 Provider에서 제공하는 동일한 알고리즘을 호출한 경우(위 그림의 *Figure 1 Provider Searching*)에는 `java.security`의 순서를 따라간다.
- `java.security` 위치: **JDK/jre/lib/security/java.security**
```
# Provider 목록
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=sun.security.mscapi.SunMSCAPI
```
	+ `security.provider.1` : 1순위

<br>

# 커스텀 Provider 생성 및 적용
**간단하게 Provider 적용 방법에 대해서만 작성**

- 커스텀 Provider Class 작성 방법
	+ `java.security.Provider` 상속
	+ `final class`
	+ `super(providerName, providerVersion, providerInfo)` 호출 <br>
		+ providerInfo: Provider에 대한 정보와 지원하는 알고리즘 설명
		```java
		// Example
		super("CryptoX", 1.0, "CryptoX provider v1.0, implementing " +
		"RSA encryption and key pair generation, and AES encryption.");
		```
- **Code Example**
```java
final class CustomProvider extends Provider {
		private static final long serialVersionUID = 7235487291138418986L;
		public CustomProvider() {
			super("CustomProvider", 1.0, "CustomProvider v1.0");
		}
}
```

<br>

## 소스에서 등록하는 방법

- 프로젝트 내부에 커스텀 Provider Class 존재해야 함. (라이브러리 가능)

1. 커스텀 Provider Class 생성
```java
final class CustomProvider extends Provider {
		private static final long serialVersionUID = 7235487291138418986L;
		public CustomProvider() {
			super("CustomProvider", 1.0, "This is CustomProvider.");
		}
}
```

2. Provider 등록
```java
public class ProviderTest {
		public static void main(String[] args) {
			Security.addProvider(new CustomProvider()); // Provider 등록

			Provider[] providers = Security.getProviders(); // Provider 목록 확인
			for(Provider p : providers) {
				System.out.println(p.getName());
			}
		}
}
```
3. **결과**
```
SUN
SunRsaSign
SunEC
SunJSSE
SunJCE
SunJGSS
SunSASL
XMLDSig
SunPCSC
SunMSCAPI
CustomProvider
```

<br>

## JDK 직접 등록하는 방법 (java.security 사용)

1. 커스텀 Provider Class 생성

2. 커스텀 Provider Class -> `jar` Export

3. 해당 `jar` 파일을 **JDK/jre/lib/ext** 경로로 이동

4. `java.security`에 Provider 등록
  - key: `security.provider.숫자` 의 숫자는 오름차순으로 증가해야 함
  - value: 커스텀 Provider Class의 패키지명을 포함하여 작성
```
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=sun.security.mscapi.SunMSCAPI
security.provider.11=com.user.provider.CustomProvider
# security.provider.11 추가됨
```

5. Provider 목록 확인
```java
public class ProviderTest {
		public static void main(String[] args) {
			Provider[] providers = Security.getProviders();
			for(Provider p : providers) {
				System.out.println(p.getName());
			}
		}
}
```

6. **결과**
```
SUN
SunRsaSign
SunEC
SunJSSE
SunJCE
SunJGSS
SunSASL
XMLDSig
SunPCSC
SunMSCAPI
CustomProvider
```

<br>

------------------------

+) 참조 :

<https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html>
<https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/HowToImplAProvider.html#Terminology>
<http://andang72.blogspot.com/2017/02/blog-post.html>
