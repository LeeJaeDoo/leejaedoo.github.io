---
date: 2020-09-15 20:40:40
layout: post
title: 자바 8 람다의 힘 Functional Programming in Java 8
subtitle: 5. 리소스를 사용한 작업
description: 5. 리소스를 사용한 작업
image: https://leejaedoo.github.io/assets/img/lambda.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/lambda.jpeg
category: java
tags:
  - java
  - lambda
  - functional_programming
  - 책 정리
paginate: true
comments: true
---
JVM에서 모든 GC를 자동으로 처리한다고 알고 있었지만 이는 내부 리소스만 사용할 때지 외부 리소스를 사용한다면 리소스에 대한 GC는 개발자가 책임져야 하는 부분이다.<br>
즉, JVM에서 외부 리소스는 자동으로 가바지 컬렉션을 하지 않는다.(ex. 데이터베이스 연결, 파일이나 소켓 오픈 등)

이번 장에서는 자바에서 리소스를 클린 업하는 여러 가지 방법 중 람다 표현식을 활용하는 방법을 알아본다.

# 리소스 클린업

* 파일 생성과 close

```java
import java.io.FileWriter;
import java.io.IOException;

public class FileWriterARM implements AutoCloseable {
    private final FileWriter writer;

    public FileWriterARM(final String fileName) throws IOException {
        this.writer = new FileWriter(fileName);
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    @Override
    public void close() throws Exception {
        System.out.println("close auto");
        writer.close();
    }
}

public static void main(String[] args) throws IOException {
    try (final FileWriterARM writerARM = new FileWriterARM("hello2.txt")) {
        writerARM.writeStuff("hello!");

        System.out.println("done");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

* 결과

```text
done
close auto
```

try 블록을 빠져나오자 마자 close() 메서드가 자동으로 호출된다.

ARM을 활용하여 try-with-resources 형태로 FileWriterARM 클래스의 인스턴스를 생성하고 그 블록 안에서 writerStuff()를 호출한다.<br>
컴파일러는 AutoCloseable 인터페이스의 구현에 대한 관리 리소스 클래스가 필요하고, 이 인터페이스는 close() 메서드만을 갖는다.

일일히 writeStuff() 메서드를 호출해야 하고 AutoCloseable을 구현해야 한다는 것과 try-with-resource 구조를 사용하고 있다는 사실을 알고 있어야 한다.
# 리소스 클린업에 활용되는 람다 표현식
위에서 알아본 ARM은 올바른 방법이긴 하지만 효과적이진 않다. 이번엔 람다 표현식을 활용하는 고차 함수를 적용해본다.

```java
import java.io.FileWriter;
import java.io.IOException;

public class FileWriterEAM {
    private final FileWriter writer;

    public FileWriterEAM(final String fileName) throws IOException {
        this.writer = new FileWriter(fileName);
    }

    private void close() throws IOException {
        System.out.println("close called auto...");
        writer.close();
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    public static void use(final String fileName,
                           final UseInstance<FileWriterEAM, IOException> block) throws IOException {
        final FileWriterEAM writerEAM = new FileWriterEAM(fileName);
        try {
            block.accept(writerEAM);
        } finally {
            writerEAM.close();
        }
    }
}

@FunctionalInterface
public interface UseInstance<T, X extends Throwable> {
    void accept(T instance) throws X;
}
```

**FileWriterEAM.use() 메서드**는 execute around pattern(실행 어라운드 패턴)의 구조로 구현됐다. 

중요한 액션은 **accept() 메서드**에 있는 인스턴스를 사용한다는 점이다. 그러나 생성과 클린업 오퍼레이션이 이 accept() 메서드의 호출을 둘러싸고 있다.

**UseInstance 인터페이스**는 함수형 인터페이스로 자바 컴파일러가 자동으로 람다 표현식이나 메러드 레퍼런스를 합성할 수 있도록 해주는 기능을 제공한다. UseInstance 인터페이스를 생성하면 accept() 메서드는 generic 타입의 인스턴스를 받을 수 있다. 이 예제에서는 이것을 FileWriterEAM의 인스턴스에 묶었다.

이로써 아래와 같이 사용될 수 있다.

```java
public static void main(String[] args) throws IOException {
    // 1.
    FileWriterEAM.use("eam.txt", writerEAM -> writerEAM.writeStuff("sweet"));

    // 2.
    FileWriterEAM.use("eam1.txt", writerEAM -> {
        writerEAM.writeStuff("sweet");
        writerEAM.writeStuff("sweet1");
    });
}
```

먼저, 개발자는 인스턴스를 직접 생성하지 못한다. 직접 생성하지 못하게 됨으로써 인스턴스가 만료되는 시점 이후로 리소스의 클린업을 지연시키는 코드의 생성을 막는다. 컴파일러가 생성자나 close() 메서드가 호출되는 것을 막을 수 있기 때문에 개발자들은 use() 메서드만을 사용하는 것이 이득이 됢을 알 수 있다.<br>
use() 메서드를 사용하면 생성자나 close() 메서드의 호출을 기다릴 필요 없이 사용하려는 인스턴스를 얻어오게 된다.

**2번**과 같이 긴 람다 표현식을 사용하는 것은 코드의 간결성, 가독성, 유지보수의 간편함이라는 장점을 잃게 된다. 지양하는 것이 좋다.

# 잠금(Lock) 관리
잠금(Lock)은 병렬로 실행되는 자바 애플리케이션에서 중요한 역할을 한다. 여기서는 람다 표현식을 활용하여 세부적인 부분에 대한 잠금(Lock)처리를 설정해본다.

기존에 사용되어 온 **synchronized** 키워드는 사용에 제한적이었다.<br>
첫째로, 메서드가 호출되는 시간 제어가 불가능했고 이는 deadlock과 livelock 발생 가능성을 증가시키게 된다.<br> 
둘째로, 실제 synchronized 실제 적용이 어렵다.

이를 해결하기 위해 자바 5에서는 **Lock 인터페이스**가 추가됐으나 양방향 synchronized가 아닌 개발자가 직접 locking과 unlocking 설정을 일일히 해줘야 한다는 문제가 있었다.

* Lock 사용한 예제 코드

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static com.company.Locker.runLocked;

public class Locking {
    Lock lock = new ReentrantLock();

    public void setLock(final Lock mock) {
        this.lock = mock;
    }
    
    // 1.
    public void doOp1() {
        lock.lock();
        try {
            /* ...critical code ...*/
        } finally {
            lock.unlock();
        }
    }
}
```
**1번**과 같이 개발하게 되면 장황하고 오류 발생하기 쉬운 코드가 된다.

* 람다 표현식 적용

```java
import java.util.concurrent.locks.Lock;

public class Locker {

    public static void runLocked(Lock lock, Runnable block) {
        lock.lock();

        try {
            block.run();
        } finally {
            lock.unlock();
        }
    }
}

// 2.
public void doOp2() {
    runLocked(lock, () -> {/*..critical code ...*/});
}

public void doOp3() {
    runLocked(lock, () -> {/*..critical code ...*/});
}

public void doOp4() {
    runLocked(lock, () -> {/*..critical code ...*/});
}

```
Locker 클래스에서 Lock 인터페이스에서 동작할 번거로운 작업들을 모두 캡슐화 한 후 **2번**과 같이 runLocked() 메서드를 사용하여 execute aroutn pattern이 적용된 구조로 크리티컬 섹션을 래핑할 수 있다. 이로써 메서드는 상당히 간결해졌다.  

# 정리
우리는 자동 GC에 완전히 의존할 수는 없다.(특히 외부 리소스 활용할 경우)<br>
execute around method 패턴은 실행 flow에 대한 세밀한 제어와 외부 리소스를 해제하는데 도움을 준다. `람다 표현식은 이 execute around pattern을 구현하는데 적합하다.`<br>
 