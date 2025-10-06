---
layout: post
category: basic
---

Do it Java에서 16-2장 Thread 클래스와 여러가지 메서드에서 스레드 중지에 대한 설명이 잠깐 나온다. 책에서는 스레드가 특정 메서드 호출(sleep, wait, join)로 Not Runnable 상태일 때 interrupt()메서드를 호출하면 InterrupedException이 발생하여 Not Runnable 상태를 빠져 나온다고 하는데 설명이 부족하여 찾아보았다.


<u>인터럽트(interrupt)는 스레드에게 현재 수행중인 작업을 중단하도록 요청하는 신호이며 이 신호에 대한 처리는 개발자의 몫</u>이다. 스레드의 interrupt()를 호출하면 해당 스레드의 인터럽트가 플래그가 설정되는데, 이 신호를 보고 InterruptedException을 던지는 메서드가 sleep(), wait(), join()등이다. 이렇게 예외를 던지는 메서드가 아닌 경우 interrupted() 또는 Thread.<em>interrupted()</em>를 사용해 주기적으로 인터럽트 상태를 확인해 대응해야한다.
| 메서드        | 설명                 |
| ------------------------------------------------------------- | ---------------------------------------------- |
| interrupt()      | 인터럽트 플래그 설정(true)                              |
| isInterrupted()   | 인터럽트 플래그가 설정되어 있는지 확인                          |
| Thread.*interrupted()*    | 현재 실행 중인 스레드가 인터럽트되었는지 확인하고 플래그를 초기화(false)한다. 인터럽트 요청이 있는 상태에서 연속으로 두 번 호출하면 첫 번째는 true, 두 번째는 false를 반환한다.  |

메서드의 동작은 조금 다르나 인터럽트 상태를 확인하는 목적은 동일한데 isInterrupted()는 인스턴스 메서드, Thread.<em>interrupted()</em>는 정적 메서드로 선언되어 있는 이유가 궁금해서 찾아보았다.<br>
Thread.<em>interrupted()</em>는 정적 메서드로 인스턴스 생성없이 호출할 수 있는 메서드이다. 해당 메서드는 내부적으로 Thread.<em>curreuntThread()</em>를 호출해 현재 실행중인 스레드의 인터럽트 상태를 확인 후 초기화 한다.<br>

```kotlin
    //Thread.java
    ..
    public static boolean interrupted() {
        return currentThread().getAndClearInterrupt();
    }
    ..
```

isInterrupted()는 인스턴스 메서드로 특정 스레드의 인터럽트 상태를 외부에서 확인시 사용하는 메서드이다. 읽기 전용으로 인터럽트 상태 값을 초기화 하지 않는다. Thread.<em>interrupted()</em>는 현재 실행중인 스레드의 인터럽트 상태를, isInterrupted()는 다른 스레드에서 해당 스레드의 인터럽트 상태 확인을 위한 메서드이다. 

즉, Thread.<em>interrupted()</em>는 현재 실행중인 스레드에 대한 참조가 명확하기 때문에 static 메서드로 선언되었고 isInterrupted()는 다른 스레드의 인터럽트 상태를 확인하기 위해 명확한 참조가 필요하기에 인스턴스 메서드로 선언되었음을 이해할 수 있었다. 

그렇다면 현재 실행중인 스레드의 인터럽트를 확인하는 Thread.<em>interrupted()</em> 메서드는 왜 인터럽트 값을 무조건 초기화 하는지도 궁금했다. 

<em>Thread.interrupted()</em>의 주석에서 알 수 있듯이 자바의 인터럽트 설계는 한번 확인하고 처리하면 끝나는 신호(read-and-forget)로 설계 했음을 추측 할수있다. 


>Tests whether the current thread has been interrupted.
>현재 실행 중인 스레드가 인터럽트 되었는지 검사합니다.
>The interrupted status of the thread is cleared by this method.
>이 메서드는 스레드의 인터럽트 상태(interrupted status)를 지웁니다.

이러한 인터럽트 설계의 일관성을 유지하기 위해 InterruptedException가 발생하면 인터럽트 상태를 초기화(false)한다.두 개의 메서드에 대해 다시 정리하면 아래와 같다. 얼핏 보면 비슷하나 사용 목적이 다름을 알 수 있었다.

 - Thread.*interrupted()*
   현재 실행 중인 스레드의 인터럽트 요청 상태를 확인  
   true면 요청을 처리한 것으로 간주하고 상태값을 false로 초기화

 - isInterrupted()
   다른 스레드의 인터럽트 상태를 확인

인터럽트 처리에 대한 일관성을 유지하기 위해 상황에 맞춰 인터럽트 상태를 초기화 해줄뿐 java DOC에서 설명했듯 인터럽트 신호에 대한 처리는 오롯이 개발자의 몫임을 잊지말자. 인터럽트는 스레드 중지에 대한 요청이지 강제 중단이 아니다.


<br>
<strong>참고</strong>
<br>
https://docs.oracle.com/javase/tutorial/essential/concurrency/interrupt.html
https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#interrupted
https://stackoverflow.com/a/2523785