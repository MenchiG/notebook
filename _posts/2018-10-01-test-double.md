---
layout: post
title: Test Double
key: 10013
tags: blog
category: java test
---

Original: [Test Doubles — Fakes, Mocks and Stubs](https://blog.pragmatists.com/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da) written by Michal Lipski
# Test Doubles

## Dummy

objects are passed around but never actually used. Usually they are just used to fill parameter lists.

用于传递给调用者但是永远不会被真实使用的对象，通常它们只是用来填满参数列表。


## Fake

Fakes are objects that have working implementations, but not same as production one. Usually they take some shortcut and have simplified version of production code.

Fake对象常常与类的实现一起起作用，但是只是为了让其他程序能够正常运行，譬如内存数据库就是一个很好的例子。

![fake](https://cdn-images-1.medium.com/max/800/0*snrzYwepyaPu3uC9.png)

```java

@Profile("transient")
public class FakeAccountRepository implements AccountRepository {
       
       Map<User, Account> accounts = new HashMap<>();
       
       public FakeAccountRepository() {
              this.accounts.put(new User("john@bmail.com"), new UserAccount());
              this.accounts.put(new User("boby@bmail.com"), new AdminAccount());
       }
       
       String getPasswordHash(User user) {
              return accounts.get(user).getPasswordHash();
       }
}
```

Apart from testing, fake implementation can come handy for prototyping and spikes. We can quickly implement and run our system with in-memory store, deferring decisions about database design. Another example can be also a fake payment system, that will always return successful payments.

## Stub & Spies

Stub is an object that holds predefined data and uses it to answer calls during tests. It is used when we cannot or don’t want to involve objects that would answer with real data or have undesirable side effects.

Stubs 通常用于在测试中提供封装好的响应，譬如有时候编程设定的并不会对所有的调用都进行响应。Stubs也会记录下调用的记录，譬如一个email gateway就是一个很好的例子，它可以用来记录所有发送的信息或者它发送的信息的数目。简而言之，Stubs一般是对一个真实对象的封装。

---

Spies are stubs that also record some information based on how they were called. One form of this might be an email service that records how many messages it was sent.

---

An example can be an object that needs to grab some data from the database to respond to a method call. Instead of the real object, we introduced a stub and defined what data should be returned.


![](https://cdn-images-1.medium.com/max/800/0*KdpZaEVy6GNnrUpB.png)

```java

public class GradesService {
    private final Gradebook gradebook;
    
    public GradesService(Gradebook gradebook) {
        this.gradebook = gradebook;
    }
    
    Double averageGrades(Student student) {
        return average(gradebook.gradesFor(student));
    }
}

```

Instead of calling database from Gradebook store to get real students grades, we preconfigure stub with grades that will be returned. We define just enough data to test average calculation algorithm.

```java
public class GradesServiceTest {
    private Student student;
    private Gradebook gradebook;

    @Before
    public void setUp() throws Exception {
        gradebook = mock(Gradebook.class);
        student = new Student();
    }

    @Test
    public void calculates_grades_average_for_student() {
    	 //Stub
        when(gradebook.gradesFor(student)).thenReturn(grades(8, 6, 10)); //stubbing gradebook
        double averageGrades = new GradesService(gradebook).averageGrades(student);
        assertThat(averageGrades).isEqualTo(8.0);
    }
}

```

## Mock

Mock are pre-programmed with expectations which form a specification of the calls they are expected to receive. They can throw an exception if they receive a call they don't expect and are checked during verification to ensure they got all the calls they were expecting.

针对设定好的调用方法与需要响应的参数封装出合适的对象。

![](https://cdn-images-1.medium.com/max/800/0*k7mwTF60slyMxRlm.png)

```java

public class SecurityCentral {
    private final Window window;
    private final Door door;

    public SecurityCentral(Window window, Door door) {
        this.window = window;
        this.door = door;
    }

    void securityOn() {
        window.close();
        door.close();
    }
    
```

We don’t want to close real doors to test that security method is working, right? Instead, we place door and window mocks objects in the test code.

```java

public class SecurityCentralTest {
    Window windowMock = mock(Window.class);
    Door doorMock = mock(Door.class);

    @Test
    public void enabling_security_locks_windows_and_doors() {
        SecurityCentral securityCentral = new SecurityCentral(windowMock, doorMock);
        securityCentral.securityOn();
        verify(doorMock).close();
        verify(windowMock).close();
    }
}

```

After execution of securityOn method, window and door mocks recorded all interactions. This lets us verify that window and door objects were instructed to close themselves. That's all we need to test from SecurityCental perspective.

You may ask how can we tell if door and window will be closed for real if we use mock? The answer is that we can’t. But we don’t care about it. This is not responsibility of SecurityCentral. This is responsibility of Door and Window alone to close itself when they get proper signal. We can test it independently in different unit test.





