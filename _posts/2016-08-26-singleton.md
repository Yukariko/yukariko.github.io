---
layout: post
title: 디자인 패턴 - 단일체(Singleton)
categories: [DesignPattern]
tags: [DesignPattern, Singleton, UML]
fullview: false
comments: true
---

----

### 정의

----

어떤 클래스의 인스턴스는 오직 하나임을 보장하며, 이 인스턴스에 접근할 수 있는 전역적인 접촉점을 제공하는 패턴

----

### 상황

----

문득 C#에서 자체 제공하는 Garbage Collection(GC)이 실제로 잘 이루어지고 있는지 테스트하고 싶어졌다.

시간별로 메모리 사용량을 체크하면 정확하겠지만, 좀 더 간단하게 확인해보기로 했다.

Garbage Collection은 참조되고 있지 않은 객체들을 모아뒀다가 어느 시점에 해제하는 식으로 동작한다.

따라서 현재 생성된 객체의 수를 체크한다면, 객체의 수가 줄어드는 순간이 Garbage Collection이 작동하는 순간이라는 것을 알 수 있다.

그렇다면 객체의 수는 어떻게 알 수 있을까?

한가지 쉬운 방법은 클래스에 카운팅 하는 변수를 static으로 선언하는 것이다.

생성자가 호출되는 순간 변수를 증가시키고, 소멸자가 호출되는 순간 변수를 감소시키면 된다.

하지만, 이것은 한 클래스에만 국한되어 있기 때문에 여러 클래스 객체들의 개수를 세는것은 무리가 있다.

즉, 다양한 클래스의 객체를 세기 위해서는 다음의 조건들을 만족해야 한다.

1. 객체를 세는 책임을 가지고 있는 클래스가 있어야 한다.
2. 이 클래스는 단 한개의 인스턴스만을 가져야 한다.

(~~간단한 실험을 너무 장황하게 표현하긴 했지만 넘어가주자~~)

위 조건을 만족해주는 패턴이 바로 단일체 패턴이다.

단일체 패턴을 코드를 통해 이해해보도록 하자.

```csharp

class ObjectCounter
{
    private static ObjectCounter instance = null;
    public int count { get; set; } = 0;

    protected ObjectCounter() { count++; }

    public static ObjectCounter getInstance()
    {
        if(instance == null)
            instance = new ObjectCounter();
        return instance;
    }
}

```

ObjectCounter는 객체의 개수를 세기 위한 클래스로, count 변수에 객체의 수를 저장한다.

단일체 패턴의 핵심은 생성자를 protected로 선언한 점이다.

생성자를 protected로 선언하게 되면 외부에서 이 객체를 생성할 수 없게된다.

즉, `new ObjectCounter();` 같은 코드는 컴파일에러를 출력하게된다.

따라서 이 클래스의 객체를 생성하는 역할은 오직 클래스 내부에만 존재하게 되고, getInstance 메소드가 이를 담당한다.

ObjectCounter는 처음부터 인스턴스를 생성하지 않고 getInstance 메소드가 처음으로 호출되는 순간 생성하게 되는데,
이를 `lazy loading` 기법이라고 부른다.

외부 클래스에서 ObjectCounter의 인스턴스를 얻기 위해선 오직 getInstance 메소드를 통해야 한다.

그렇다면 이제 ObjectCounter를 사용하는 예제코드를 보도록 하자.

```csharp

class Item
{
    public Item() { ObjectCounter.getInstance().count++; }

    ~Item() { ObjectCounter.getInstance().count--; }
}

class Item2
{
    public Item2() { ObjectCounter.getInstance().count++; }

    ~Item2() { ObjectCounter.getInstance().count--; }
}

class Test
{
    public static void Main()
    {
        ObjectCounter oc = ObjectCounter.getInstance();

        for(int i=0; i < 10000; i++)
        {
            new Item();
            new Item2();
            Console.WriteLine("Object Count : {0}", oc.count);
        }
    }
} 

```

Item, Item2 클래스는 객체가 생성 또는 소멸되는 순간 카운트를 증가/감소 시킨다.

메인에서는 이 두 클래스 객체를 각각 1만번 생성하면서 주기적으로 카운트값을 출력한다.

GC가 작동하지 않는다면, 객체의 수는 꾸준히 증가하여 최종적으로는 대략 2만개의 객체가 생성되어있을것이다.

실험결과, 마지막으로 출력된 객체의 수는 8094개로, 절반이 넘는 객체가 GC에 의해 소멸되었다.

1만개 까진 선형적으로 증가하다가 그 이후부터 개수가 점점 감소하는 현상을 보였다.

객체를 한 반복에 하나씩만 생성한 경우는 6000개 정도에서 GC가 동작한것으로 보아 메모리 사용량 뿐만 아니라, 일정시간마다 GC가 동작하는것으로 보여진다.

실험은 여기까지. 이제 단일체 패턴의 장단점을 알아보도록 하자.

----

### 장단점

----

#### 장점

1. 유일하게 존재하는 인스턴스로의 접근을 통제할 수 있다.
2. 전역 변수를 사용하는것의 문제점을 없앤다.
3. 상속을 통해 연산을 재정의 할 수 있다.
4. 인스턴스의 개수를 조절할 수 있다.

#### 단점

???

----

### 클래스 다이어그램

----

![UML](http://imgur.com/uUfyy9d.jpg)

단일체 패턴의 이름이 말해주듯이 클래스 다이어그램 또한 매우 단순해진다.

----

### 결론

----

단일체 패턴은 오직 한 개의 인스턴스만을 갖도록 보장하는 패턴이다.