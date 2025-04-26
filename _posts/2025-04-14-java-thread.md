---
title: "[자바8] 프로세스 內 실행 작업단위 스레드와 자바의 스케줄링 및 스레드 상태전이"
date: 2025-04-14 16:25 +0900
lastmod: 2025-04-26 16:25 +0900
categories: 자바8
tags:
  [Thread, Runnable, Stack Frame, Thread.State, Scheduling, Context Switching]
---

## 🪵 멀티태스킹과 멀티프로세싱

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🪵 OS는 스케줄링이라는 기법을 이용해 동시에 여러 작업을 수행한다
</div>

하나의 CPU 코어로 여러개의 프로그램을 동시에 실행하는 것처럼 보이게 하는 기술을 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">' 멀티태스킹 '</span>이라 한다.

즉, CPU가 매우 빠르게 여러 개의 프로그램 코드를 번갈아 연산하는 것이다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 프로그램 실행 = CPU 연산</span>  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">프로그램의 실행시간을 분할</span>해서, 마치 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">동시에 실행되는 것처럼 보이게 하는 기법</span>을  
<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">시분할(Time Sharing) 기법</span>이라 한다.

OS가 각 프로그램의 실행시간을 결정하는데, 이를 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">스케줄링</span>이라 한다.  
단순히 시간으로만 작업을 분할하지는 않고, CPU를 최대한 활용할 수 있는 <span style='color:rgb(196,58,26);'>우선순위</span>와 <span style='color:rgb(196,58,26);'>최적화 기법</span>을 사용한다.

멀티프로세싱은 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">프로세서(CPU 코어)</span>가 여러개여서, <span style='color:rgb(196,58,26);'>물리적으로 동시에 여러 개의 프로그램을 처리</span>할 수 있는 것을 말한다.

즉, <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">멀티태스킹</span>은 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">운영체제 소프트웨어 관점</span>에서의,  
<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">멀티프로세싱</span>은 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">하드웨어 관점</span>에서의 동시에 처리하는 기술을 의미한다.

## 👹 프로세스와 스레드

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    👹 실행중인 프로그램의 인스턴스를 프로세스라 한다
</div>

각 프로세스는 독립적인 메모리 공간을 가지고 있다.

- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">코드 섹션</span>: 실행할 프로그램의 코드가 저장되는 영역
- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">데이터 섹션</span>: 전역변수 및 정적변수가 저장되는 영역
- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">힙</span>: 동적으로 할당되는 메모리 영역
- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">스택</span>: 메서드 호출 시, 생성되는 지역변수와 반환주소가 저장되는 영역

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">※ os에서 독립된 실행단위로 취급되어 관리된다.</span>

스레드는 프로세스 내에서 실행되는 작업의 단위이다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 실제로 CPU에 의해 실행되는 단위이다.</span>  
하나의 프로세스 내부의 여러 스레드는 프로세스가 제공하는 동일한 메모리 공간 및 자원을 공유하지만,  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">각 스레드는 개별 스택을 가지고 있다.</span>

스레드는 프로세스 내 작업의 단위이자, 코드를 실행하는 흐름을 말한다.

```
⏱️ 단일코어 스케줄링
os는 내부에 스케줄링 큐를 가지고 있고, 각 스레드는 스케줄링 큐에서 대기한다.
os는 스레드를 큐에서 꺼내 CPU를 통해 실행한다.
최적화 기법에 의해 잠시 스레드를 멈추고, 큐에 다시 넣은 뒤,
다음 스레드를 큐에서 꺼내 반복 수행한다.

※ 멀티코어 스케줄링
스케줄링 큐에 대기하는 각각의 스레드를 각 CPU코어에서 병렬로 실행한다.
```

## 🍝 컨텍스트 스위칭

<div style="margin-bottom:15px;font-size:20px;background-color:#9B111E;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 멀티태스킹이 반드시 효율적이지는 않다
</div>

다음 스레드를 실행하기 위해, 현재 스레드를 멈추는 시점에서 <span style='color:rgb(196,58,26);'>CPU가 사용하던 값들을 메모리에 저장</span>해야 한다.  
이후에 해당 스레드를 다시 실행할 때, <span style='color:rgb(196,58,26);'>이 값들을 CPU에 다시 불러와야 한다.</span>  
이러한 과정을 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">' 컨텍스트(context, 현재 작업하던 코드문맥) 스위칭 '</span>이라 한다.  
즉, <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">컨텍스트 스위칭</span> 과정에는 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">약간의 전환비용이 발생한다.</span>  
때문에, 스레드가 많아지면 컨텍스트 스위칭 비용이 늘어난다.

---

스레드가 하는 작업은 크게 2가지로 구분할 수 있다.

- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">CPU 바운드 작업</span>  
  : CPU의 연산능력을 요구하는 작업 (CPU의 처리속도가 작업의 완료시간을 결정하는 경우)
- <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">i/o 바운드 작업</span>  
  : 디스크, 네트워크, 파일 시스템과 같은 입출력을 요구하는 작업  
  <span style='color:rgb(196,58,26);'>※ i/o 작업이 완료될 때까지 대기시간이 많이 발생한다.</span>  
  이 때 스레드가 CPU를 사용하지 않아, CPU가 대기상태에 있는 경우가 많다.

즉, 스레드의 작업 유형과 성능테스트 결과에 따라, 스레드 수를 적절히 조절해야 한다.

## ⚔️ 자바 메모리 구조

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 자바 메모리 구조
</div>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🔅 메서드 영역 ( <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Method Area</span> )</span>  
: 프로그램에 필요한 공통데이터를 관리하며, 해당 영역은 프로그램의 모든 영역에서 공유한다.

- 클래스 정보: 클래스의 실행코드(바이트코드) [필드, 메서드, 생성자 등]
- static 영역: static 변수를 보관
- 런타임 상수 풀: 공통 리터럴 상수 보관

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🔅 스택 영역 ( <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Stack Area</span> )</span>  
: 스레드 별로 실행 스택이 생성된다.  
메서드를 호출할 때마다, 스택프레임이 쌓이고, 메서드가 종료되면 제거된다.  
각 스택 프레임은 지역변수, 중간 연산결과, 메서드 호출정보등을 포함한다.

```
🧶 main 메서드를 실행하는 메인스레드
프로세스가 작동하려면 코드를 실행시킬 스레드가 최소 하나는 있어야 하는데
자바는 실행시점에 main이라는 이름의 스레드를 만들고, 프로그램의 진입점인 main 메서드를 실행한다.
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🔅 힙 영역 ( <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Heap Area</span> )</span>
: 객체(인스턴스)와 배열이 생성되는 영역이다.  
가비지 컬렉션(GC)이 이루어지는 주요 영역이며, 더이상 참조되지 않는 객체는 GC에 의해 제거된다.

## 🗿 스레드 생성 2가지 방법

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🗿 스레드 객체 생성 3가지 방법
</div>

```
🧶 스레드 객체 생성1 - Thread 클래스 상속
자바는 스레드도 객체로 다룬다.
즉, 스레드가 필요하면 Thread 클래스를 상속받아 스레드 객체를 생성해서 사용하면 된다.
단, 자바는 단일상속만을 허용하므로, 이미 다른 클래스를 상속받고 있는 경우 Thread 클래스를 상속받을 수 없다.
```

```java
public class CustomThread extends Thread {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run");
    }
}
```

```java
public class TestMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main 메서드 진입");

        CustomThread thread = new CustomThread();

        System.out.println(Thread.currentThread().getName() + ": thread.run() 호출 전");
        thread.run(); // main: run()
        System.out.println(Thread.currentThread().getName() + ": thread.run() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": thread.start() 호출 전");
        thread.start(); // Thread-0: run()
        System.out.println(Thread.currentThread().getName() + ": thread.start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main 메서드 종료");
    }
}
```

```
🪄 Thread.currentThread()
해당 코드를 실행하는 스레드 객체를 조회한다.

🪄 start와 run
start()는 스레드를 실행하는 메서드이다. 해당 메서드를 호출하면 해당 스레드가 run() 메서드를 실행한다.
반드시 start()를 호출해야 별도의 스레드에서 run()이 실행된다.

run()을 직접 호출하면, 해당 스레드가 run()을 실행하는 것이 아닌,
main 스레드가 run()을 직접 실행하게 된다. 즉, main 스레드가 사용하는 스택 위에 run() 스택프레임이 올라간다.

start()를 호출해야 자바는 스레드를 위한 별도의 스택공간을 할당한다. (동시에 스레드를 시작한다.)
※ 시스템 쿼리를 통해 스레드가 os에 의해 할당된다.

start를 호출한 해당 스레드는 run 메서드 스택프레임을 스택에 올리면서 run 메서드를 시작한다.
main 메서드는 start를 호출한 스레드에게 코드의 실행을 지시만 하고, start 메서드를 바로 빠져나온다.
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">📕 데몬(daemon) 스레드</span><br>
사용자(non-daemon, user) 스레드가 프로그램의 주요 작업을 수행한다면, 데몬스레드는 백그라운드에서 보조적인 작업을 수행한다.<br>
모든 사용자 스레드가 종료되면, JVM도 종료되고 데몬스레드의 실행완료를 기다리지 않는다. (데몬스레드도 종료)<br><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">CustomThread thread = new CustomThread();</span><br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">thread.setDaemon(true);</span> // 데몬스레드 여부는 start() 실행전에 결정해야 한다.<br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;font-family:var(--bs-font-monospace);font-size:.85rem;">thread.start();</span>
</div>

---

```
🧶 스레드 객체 생성2 - Runnable 인터페이스 구현
스레드가 실행할 작업을 스레드로부터 분리할 수 있다.
```

```java
public class ToDoRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}
```

```java
public class TestMain {
    public static void main(String[] args) {
        ToDoRunnable runnable = new ToDoRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
    }
}

public class TestMain2 {
    public static void main(String[] args) {
        // 익명 클래스
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + ": run()");
            }
        }

        Thread thread = new Thread(runnable);
        thread.start();
    }
}

public class TestMain3 {
    public static void main(String[] args) {
        // 더 간단히, 익명클래스
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + ": run()");
            }
        });
        thread.start();
    }
}

public class TestMain4 {
    public static void main(String[] args) {
        // 람다 (람다를 사용하면 메서드(코드블록)을 전달할 수 있다)
        Thread thread = new Thread(() -> System.out.println(Thread.currentThread().getName() + ": run()"));
        thread.start();
    }
}

```

## ⚔️ 스레드 기본정보 및 상태

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 스레드 기본정보 및 상태
</div>

| 🚀 내장 메서드   |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| threadId()       | 스레드 고유식별자(id)                                                                                                                                                                                                 |
| getName()        | 스레드 이름                                                                                                                                                                                                           |
| getPriority()    | 높을수록 스케줄링에서 조금 더 많이 실행된다. (default: 5)                                                                                                                                                             |
| getThreadGroup() | 스레드가 속한 그룹, default: 모든 스레드는 부모스레드와 동일한 그룹에 속하게 된다.<br>그룹단위로 특정작업(일괄 종료, 우선순위 설정)을 수행할 수 있다.<br> main스레드는 기본적으로 제공되는 main 스레드 그룹에 속한다. |
| getState()       | 스레드 현재 상태                                                                                                                                                                                                      |

| 🧶 Thread.State |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NEW             | 스레드가 생성, 시작되지 않은 상태 (start 메서드 호출 전)                                                                                                                                                       |
| RUNNABLE        | 스레드가 실행중 or 실행될 준비가 된 상태 (start 호출 시점)<br>CPU에서 실행되고 있는 스레드는 모두 RUNNABLE 상태이다. (os 스케줄러의 실행대기열 포함)                                                           |
| BLOCKED         | 차단상태<br>스레드가 동기화 락을 기다리는 상태 (ex. synchronized 블록에 진입하기 위해 락을 얻어야 하는 경우)                                                                                                   |
| WAITING         | 대기상태<br>스레드가 무기한으로 다른 스레드의 작업이 완료되기를 기다리는 상태<br>(wait(), join() 메서드가 호출될 때, 다른 스레드가 notify() or notifyAll()을 호출하거나 join()이 완료될 때까지 기다려야 한다.) |
| TIMED_WAITING   | 시간제한 대기상태<br>일정시간 동안 다른 스레드의 작업완료를 기다리는 상태<br>(sleep(long millis), wait(long millis), join(long millis) 메서드가 호출될 때)                                                     |
| TERMINATED      | 종료상태<br>※ 스레드는 한번 종료되면 다시 시작할 수 없다.                                                                                                                                                      |

| ♻️ 상태전이 |
| From | To |
| --------- | ------------- |
| RUNNABLE | BLOCKED / WAITING / TIMED WAITING |
| 스레드가 락을 얻지 못하거나, wait() / sleep() 호출할 때 |
| BLOCKED / WAITING / TIMED WAITING | RUNNABLE |
| 스레드가 락을 얻거나, 기다림이 완료될 때 |
| RUNNABLE | TERMINATED |
| 스레드의 run() 메서드가 완료될 때 |

## 🍝 체크 예외 재정의

<div style="margin-bottom:15px;font-size:20px;background-color:#9B111E;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 자바에서 메서드를 재정의할 때, 재정의 메서드가 지켜야할 예외와 관련된 규칙이 있다
</div>

- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 부모메서드가 체크예외를 던지지 않는 경우, 재정의된 자식메서드도 체크예외를 던질 수 없다.</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 자식메서드는 부모메서드가 던질 수 있는 체크예외의 하위타입만 던질 수 있다.</span>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 Unchecked(런타임) 예외는 얼마든지 던질 수 있다.</span>

```java
@FunctionalInterface
public interface Runnable {

    /**
     * Runs this operation.
     */
    void run();
}
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:#F7F6F3;color:#624a3d;font-weight:normal;">🍪 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">run</span> 메서드는 해당 메서드 바깥으로 예외를 던질 수 없다.</span>  
: <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">무조건 안에서 잡아서 해결해야 한다.</span>  
`throws Exception`을 선언하지 않은 이유는 바깥으로 던지는 것을 허용할 시,  
<span style='color:rgb(196,58,26);'>예외가 적절히 처리되지 않는 수 있는 상황을 막아</span> 프로그램이 비정상 종료되는 상황을 막기 위해서이다.  
즉, 내부에서 예외처리를 강제함으로써, 스레드의 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🏆 안정성</span>과 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🏆 일관성</span>을 유지할 수 있다.  
※ 단, 최근에는 체크예외를 사용하기보다는 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Unchecked</span> (런타임) 예외</span>를 선호한다.

## 🔌 join(대기)와 interrupt(중단)

<div style="margin-bottom:15px;font-size:20px;background-color:#336666;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🔌 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">join</span> (대기) 과 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">interrupt</span> (중단)
</div>

`🍍 join()`  
: 호출한 스레드가 무기한 대기상태(`WAITING`)에 빠진다. (인자로 ms를 넘기면 특정 시간만큼만 대기, `⏱️ TIMED_WAITING`)  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">코드 라인이 다음으로 넘어가지 않는다.</span>  
특정 스레드가 끝날 때까지 기다려야 할 때 (대상 스레드가 `TERMINATED` 상태가 될 때까지)

`🍍 interrupt()`  
: 인터럽트 상태를 true로 만든다. 해당 상태에서 `InterruptedException`을 던지는 메서드(ex. `Thread.sleep()`)를 만날 때,  
`InterruptedException` 예외를 던지고, 해당 스레드는 `RUNNABLE` 상태가 된다.  
즉, `WAITING`, `TIMED_WAITING` 같은 대기상태의 스레드를 깨워서, `RUNNABLE` 상태로 만든다.  
⚠️ <span style="padding:3px 6px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">flag</span> 변수</span>를 사용해서 스레드 작업을 중단시킬 수 있지만, 스레드가 즉각 반응할 수 없다.  
&nbsp;  
인터럽트를 걸어놨다가, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">인터럽트 예외를 던지는 메서드를 호출하면 해당 인터럽트 상태는 풀린다.</span>  
해당 인터럽트 상태가 자동으로 풀리는 이유는,  
그렇지 않으면 이후에 인터럽트 예외를 던지는 메서드를 호출할 때 인터럽트가 발생하게 되기 때문이다.  
즉, 초기에 한번 인터럽트 목적을 달성하면 자동으로 인터럽트 상태가 풀린다.

`🍞 Thread.currentThread().isInterrupted()` <span style="font-size:.98rem;font-weight:normal;padding:3px 5px">: 스레드 인터럽트 상태 확인 (상태변경x)</span>

<span style="font-size:0.85rem;font-family:var(--bs-font-monospace);padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:var(--highlighter-rouge-color);font-weight:normal;">🍞 Thread.interrupted()</span> <span style="font-size:.98rem;font-weight:normal;padding:3px 5px">(상태변경 ㅇ)</span>
: 스레드가 인터럽트 상태이면 true를 반환하고, <span style='color:rgb(196,58,26);'>인터럽트 상태를 false로 변경한다.</span>  
스레드가 인터럽트 상태가 아니면 false를 반환한다.

## 🔌 sleep(휴식)과 yield(양보)

<div style="margin-bottom:15px;font-size:20px;background-color:#336666;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🔌 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">sleep</span> (휴식) 과 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">yield</span> (양보)
</div>

`🍍 Thread.sleep(ms)`  
: 해당 스레드가 `⏱️ TIME_WAITING` 상태가 되면서 실행 스케줄링에서 제외된다.  
때문에 다른 스레드를 실행할 수 있게되는데, 실행할 다른 스레드가 없다면 그냥 CPU는 쉰다.

`🍍 Thread.yield()`  
: 다른 스레드에게 CPU 실행을 양보한다.  
해당 스레드의 상태가 바뀌진 않고 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">(<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Runnable</span> 유지)</span>, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">스케줄링 큐에 다시 들어갈 뿐이다.</span>  
`Thread.yield()`를 호출하는 것은 운영체제에게 스케줄링 상태를 `(RUNNING -> READY)`로 만들도록 <span style='color:rgb(196,58,26);'>힌트</span>를 주는 것이다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">⚠️ 단, 강제로 실행순서를 지정하지 않는다. 양보할 스레드가 없다면 해당 스레드가 계속 실행된다.</span>

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;color:#34343c;">
<span style="font-weight:bold;">⏱️ 운영체제 관점의 스케줄링</span><br>
자바의 스레드가 RUNNABLE 상태일 때,<br>
운영체제의 스케줄링은 2가지 상태를 가질 수 있다.<br>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">실행상태 (<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Running</span>)</span> : CPU에서 실행<br>
- <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;color:#34343c;">실행대기상태 (<span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Ready</span>)</span> : 스케줄링 큐에서 대기<br><br>
※ 자바는 위 두 상태를 구분할 수 없다.
</div>

```java
public class ThreadTestMain {
    public static void main(String[] args) {
        CustomThread custom = new CustomThread();
        Thread thread = new Thread(custom, "custom");
        thread.start();

        Scanner userInput = new Scanner(System.in);
        while(true) {
            System.out.println("작업할 문서를 입력하세요. 종료 (q): ");
            String input = userInput.nextLine();
            if(input.equals("q")) {
                thread.interrupt();
                break;
            }
            custom.addJob(input);
        }
    }

    static class CustomThread implements Runnable {
        Queue<String> jobQueue = new ConcurrentLinkedQueue<>();

        @Override
        public void run() {
            while(!Thread.interrupted()) {
                if(jobQueue.isEmpty()) {
                    Thread.yield();
                    continue;
                }

                try {
                    String job = jobQueue.poll();
                    System.out.println("작업:" + job + ", 대기작업: " + jobQueue);
                    Thread.sleep(3000);
                    System.out.println("작업 완료!");
                } catch (InterruptedException e) {
                    System.out.println("인터럽트!");
                    break;
                }
            }
            System.out.println("작업 종료");
        }

        public void addJob(String input) {
            jobQueue.offer(input);
        }
    }
}
```

```
🪄 ConcurrentLinkedQueue
여러 스레드가 동시에 접근하는 경우, 동시성 컬렉션을 사용해야 한다.
```
