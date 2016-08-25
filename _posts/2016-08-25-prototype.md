---
layout: post
title: 디자인 패턴 - 원형(Prototype)
categories: [DesignPattern]
tags: [DesignPattern, Prototype, UML]
fullview: false
comments: true
---

----

### 정의

----

생성할 객체의 종류를 대표하는 원형 객체를 전달하고, 그 객체를 복사하는것으로 새로운 객체를 생성하는 패턴

----

### 상황

----

원형 패턴은 Cycle을 의미하는 원형하고 헷갈리지만 완전 다른 개념이다.

원형 패턴은 사실 [추상 팩토리](https://yukariko.github.io/designpattern/2016/08/19/abstract-factory.html)에서 이미 언급한 바가 있다.

Star객체를 생성자에서 인자로 받아 그 객체를 복사해서 사용하는 패턴이다.

이해하기 쉬운 패턴이므로 빠르게 코드로 넘어가도록 해보자.

우선 Item부분의 코드를 보자.

```csharp

class Item
{
    public virtual void print() { }
    public virtual Item clone() { return new Item(); }
}
class Star : Item
{
    public override void print() { Console.Write("*"); }
    public override Item clone() { return new Star(); }
}
class Blank : Item
{
    public override void print() { Console.Write(" "); }
    public override Item clone() { return new Blank(); }

}

class MoneyStar : Star
{
    public override void print() { Console.Write("$"); }
    public override Item clone() { return new MoneyStar(); }
}

```

새로운 메소드인 clone이 생겼다. 이 예제에선 맴버변수가 없기 때문에 단순하게 new 연산으로만 정의하였다.

사실 원형 패턴의 진수는 각종 맴버변수들의 값을 조정하여 넘겨주는것에 있다고 생각한다.

그것들을 모두 서브 클래스로 작성하는것은 번거롭기 때문이다.

이제 PrintStar의 코드를 보도록 하자.

```csharp

class PrintStar
{
    const int MapSize = 10;
    Item[,] map = new Item[MapSize, MapSize];
    public PrintStar(Star star, Blank blank)
    {
        map[3, 3] = star.clone();
        map[4, 2] = star.clone();
        map[4, 4] = star.clone();
        map[5, 3] = star.clone();
        for (int i = 0; i < MapSize; i++)
            for (int j = 0; j < MapSize; j++)
                if (map[i, j] == null)
                    map[i, j] = blank.clone();
    }

    public void print()
    {
        for (int i = 0; i < MapSize; i++)
        {
            for (int j = 0; j < MapSize; j++)
                map[i, j].print();
            Console.WriteLine();
        }
    }
}

```

star와 blank 객체를 매개변수로 받아서 clone을 사용해 생성하고 있다.

마지막으로 main 부분의 코드이다.

```csharp

class Test
{
    static void Main(string[] args)
    {
        PrintStar printStar = new PrintStar(new MoneyStar(), new Blank());
        printStar.print();
    }
}

```

가장 눈에띄는 특징은 객체 생성을 위한 클래스 수가 확실히 줄어들었다는 점이다. 

그렇다고 한 객체가 비대해진것도 아니다.

원형 패턴의 장단점을 알아보자.

----

### 장단점

----

#### 장점

1. 런타임에 새로운 제품을 추가하고 삭제할 수 있다.
2. 값들을 다양화함으로써 새로운 객체를 명세한다.
3. 구조를 다양화함으로써 새로운 객체를 명세할 수 있다.
이는 2번 장점을 응용한것으로, 객체가 어떤 객체들의 집합을 나타내 줄 때, 런타임에 이를 새 객체로 인식할 수 있게 해준다.
4. 서브클래스의 수를 줄인다.

#### 단점

1. 원형을 구현하는 모든 서브클래스가 clone 메소드를 정의해야 한다.


----

### 클래스 다이어그램

----

![UML](http://imgur.com/P53L2YN.jpg)

서브클래스의 수를 줄인다는 장점 답게 매우 간소화된 다이어그램을 볼 수 있다.

----

### 결론

----

원형 패턴은 원형이 되는 객체를 통해 생성할 객체를 명시해주는 패턴이다.