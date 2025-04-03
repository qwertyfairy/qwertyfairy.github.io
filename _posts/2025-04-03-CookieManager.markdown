---
layout: post
category: basic
---

## CookieManager

2025년 03월 27일   
</br>

로그인 API 응답으로 내려 받은 사용자 정보를 앱 쿠키에 설정해 놓고 앱내 웹뷰의 로그인 상태를 유지하고 있었다.
이 과정에 CookieManager를 제대로 사용하고있는가 의문이 들었고 CookieManager의 작동과 사용법을 학습하고자 한다.  

</br>

Android의 웹뷰에 사용되는 쿠키 정보는 CookieSyncManger와 CookieManager가 관리한다. (CookieSyncManager는 API 21에서 deprecated) 
기본적으로 CookieManager는 웹뷰에 들어오는 쿠키 정보(Set-Cookie)를 자동으로 관리한다. 하지만 아래 메서드를 통해 개발자가 직접 쿠키를 등록할 수 있다.   

```kotlin
CookieManager.getInstance().setCookie(url, cookie)
```

메서드의 파라미터에 대한 설명은 다음과 같다.  

- url : URL에 쿠키를 지정한다.
- cookie : HTTP Set-Cookie 포맷을 따른다.
 

url은 해당 쿠키 값을 지니는 url이고, 단순히 생각하면 domain 검증이 들어갈 url을 입력하면 된다. 보통 검증이 수행되는 url의 도메인만 넣어준다.  

- domain : 쿠키에 접근 가능한 도메인을 지정한다.

예를 들어 다음과 같은 쿠키가 있다고 하자.

```
username=qwerty; domain=qwerty.com; path=/ 
```

위 쿠키는 domain이 qwerty.com에서만 접근할 수 있는 쿠키이다.  
해당 쿠키를 CookieManger에 설정하려면 다음과 같이 할 수 있다. 참고로 이름(name *쿠키의 key를 의미), 도메인(domain), 경로(path)로 쿠키를 구분하기 때문에 세 가지가 같은 값을 가진 쿠키가 들어온다면 값은 덮어 쓰이게 된다.  

```kotlin
/*쿠키는 url인코딩하여 등록해야 한다.*/
CookieManager.getInstance().setCookie("qwerty.com","username=qwerty; domain=qwerty.com; path=/")
```

이제 웹뷰에서 읽어드린 페이지의 도메인이 qwerty.com인 경우, 위에서 설정해둔 쿠키 값을 읽을 수 있다.  
<br/>

## 쿠키의 속성
쿠키에 다양한 속성이 존재하나, 이해하고 있으면 좋을 두 가지만 설명하고자 한다.
   
<br/>
  
**domain**  
쿠키에 접근 가능한 domain(도메인)을 지정한다. 해당 옵션에 아무 값도 넣지 않았다면 쿠키를 설정한 도메인에서만 쿠키에 접근할 수 있다. qwerty.com에서 설정한 쿠키는 other.com에서 얻을 수 없다. 

또한 서브 도메인(sub domain) two.qwerty.com에서도 쿠키 정보를 얻을 수 없는데, 이 경우 루트 도메인인 domain=qwerty.com를 명시적으로 설정해주면 된다.

오래된 표기법이지만 하위 호환성 유지를 위해 앞에 .을 붙여도 동일하게 작동한다.   
``` 
eg. domain=.qwerty.com   
```
   
<br/>
  
**secure**  
secure 옵션이 있는 경우 url앞에 https://가 붙어야 한다. 쿠키는 기본적으로 도메인만 확인하지 프로토콜을 확인하지 않기 때문인데 secure 속성이 true가 되면 프로토콜도 확인하게 된다. 

```kotlin
val cookie = userid=qwerty12; domain=naver.com; path=/; secure // secure 속성을 지닌 쿠키

CookieManager.getInstance().setCookie("https://qwerty.com", cookie) //등록 가능
CookieManager.getInstance().setCookie("qwerty.com", cookie) //등록 불가, 안됨

CookieManager.getInstance().getCookie("https://qwerty.com") // 조회가능
CookieManager.getInstance().getCookie("qwerty.com") // 조회 불가, null 반환
```
<br/>

## CookieManager에 등록되는 쿠키

**세션 쿠키(Session Cookie)**  
브라우저가 닫힐 때 같이 삭제되는 쿠키로 만료일(expires)이 지정되지 않은 쿠키이다.  

CookieManager는 쿠키의 타입과 상관없이 영구 저장소인 데이터베이스에 등록해 관리하기 때문에 앱에서는 세션 쿠키를 제거하는 처리를 반드시 해줘야 한다.

웹에서 브라우저는 앱에서 앱 그 자체일수도 있고, 웹뷰 액티비티 일 수도 있기에 개발자의 적절한 판단이 필요하다. 아래는 세션 쿠키를 제거하는 메서드이다. 

```kotlin
CookieManager.getInstance().removeSessionCookies(null) 
//수행 결과를 받는 리스너를 등록할수있다. 필요없기에 null을 넣음
```

CookieManager를 반드시 사용하지 않아도 된다면  앱 내 메모리에 세션 쿠키를 저장해 처리 하는 방법도 고려할 수 있다.   
   
<br/>

**지속 쿠키(Persistent Cookie)**  
만료일이 지정된 쿠키로 GMT시간 기준으로 관리된다. 웹뷰는 만료된 쿠키를 자동으로 제거한다. 

웹뷰에서 사용되는 쿠키는 CookieManager에 자동으로 동기화 되는데 갱신이나 삭제는 동일한 도메인과 경로에서만 해야한다. 즉 쿠키를 설정할 때 지정했던 도메인이나 경로를 사용해야 한다. 
  
<br/>

## CookieManager의 Database 구조 확인
테스트를 통해 CookieManger를 통해 저장되는 DB 정보를 확인해보자.

```kotlin
TimeZone.setDefault(TimeZone.getTimeZone("GMT"))
val calendar = Calendar.getInstance(TimeZone.getTimeZone("GMT"))
calendar.set(MINUTE, calendar.get(MINUTE) + 2) /** 2분후 만료*/

val persistentCookie = Cookie.Builder()
                .domain("qwerty.com")
                .name("userid")
                .value("qwerty1004")
                .expiresAt(calendar.timeInMillis)
                .build()
CookieManager.getInstance().setCookie("qwerty.com", persistentCookie.toString())

val sessionCookie = Cookie.Builder()
                .domain("qwerty.com")
                .name("username")
                .value("kimqwerty")
                .build()
CookieManager.getInstance().setCookie("qwerty.com", sessionCookie.toString())
```

Android Studio의 Device Explorer에서 Cookies DB를 추출할 수 있다.

![image](/public/img/cookie01.png)   
</br>
추출된 파일은 SQLite Viewer를 통해 확인한다.   

![image](/public/img/cookie02.png)

테스트를 위해 등록한 쿠키가 정상적으로 등록되어있다. 
필드 expries_utc(만료일), is_persistent(지속 쿠키 여부)를 통해 각 쿠키 타입을 구분할 수 있다.

지속 쿠키로 등록된 userid는 만료일이 있고 지속 쿠키 여부가 1(true) 이다. 해당 쿠키는 2분이 지난 상태로 웹뷰에 의해 자동으로 제거된다.  

세션 쿠키로 등록된 username은 개발자가 직접 제거해주지 않는 이상 계속 지니고 있게 된다. 
같은 이름, 도메인, 경로가 입력된다면 갱신 될 가능성은 있으나 적절히 제거하는 처리가 필요하다.
   
<br/>
  
## 결론

- secure 속성이 있으면 프로토콜도 검증하기 때문에 url에 https://가 반드시 붙어야 한다.
https:// 없이 쿠키를 설정하면 에러 문구가 출력 되고 해당 쿠키는 등록되지 않는다. 
Please either use the 'https:' scheme for this URL or omit the 'Secure' directive in the cookie value.
- 세션 쿠키는 브라우저가 닫히는 시점에 제거 되어야 한다, 즉 개발자가 적절히 판단하여 제거하는 작업을 해줘야 한다.
- 지속 쿠키는 웹뷰에 의해 자동으로 관리된다. 단, 갱신이나 삭제는 동일한 도메인과 경로에서만 해야한다.
- 쿠키는 설정된 domain으로만 접근할 수 있다.

<br/>

**참고 자료**  
https://i-ten.tistory.com/339  
https://ko.javascript.info/cookie#ref-61  
https://web.dev/articles/understanding-cookies?hl=ko   
https://sqliteviewer.app/#/Cookiesafter1.db/table/cookies (SQLite 뷰어)  
<br/>

**CookieManager 구현클래스**  
https://android.googlesource.com/platform/frameworks/webview/+/f989a7846d5fe4116de1a0635a8aade09d2be7b2/chromium/java/com/android/webview/chromium/CookieManagerAdapter.java
https://chromium.googlesource.com/chromium/src.git/+/master/android_webview/java/src/org/chromium/android_webview/AwCookieManager.java
