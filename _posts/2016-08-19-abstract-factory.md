---
layout: post
title: 디자인 패턴 - 추상 팩토리(Abstract Factory)
categories: [DesignPattern]
tags: [DesignPattern, AbstractFactory, UML]
fullview: false
comments: true
---

----

### 정의

----

구체적인 클래스를 지정하지 않고, 관련성을 갖는 객체들의 집합을 생성하거나

서로 독립적인 객체들의 집합을 생성할 수 있는 인터페이스를 제공하는 패턴.

----

### 상황

----

어떤 2차원 공간에 별을 찍는 프로그램을 만든다고 해보자.
(~~왜 이런짓을 하는지는 묻지 말자..~~)

먼저 2차원 공간을 나타낼 map이라는 변수가 있고, map은 2차원 배열로 이루어져 있으며,
배열의 각 요소는 별인지 공백인지를 저장할 Item 객체를 저장한다.

코드로 확인해보자.

```c#

class Item
{
    public virtual void print() { }
}
class Star : Item
{
    public override void print() { Console.Write("*"); }
}
class Blank : Item
{
    public override void print() { Console.Write(" "); }
}

class PrintStar
{
    const int MapSize = 10;
    Item[,] map = new Item[MapSize, MapSize];
    public PrintStar()
    {
        map[3, 3] = new Star();
        map[4, 2] = new Star();
        map[4, 4] = new Star();
        map[5, 3] = new Star();
        for (int i = 0; i < MapSize; i++)
            for (int j = 0; j < MapSize; j++)
                if (map[i, j] == null)
                    map[i, j] = new Blank();
    }

    public void print()
    {
        for(int i=0; i < MapSize; i++)
        {
            for(int j=0; j < MapSize; j++)
                    map[i, j].print();
            Console.WriteLine();
        }
    }
}

```

위에 설명하진 않았지만, 코드를 보면 별을 찍는 PrintStar라는 클래스가 있고,
생성자에서 map을 구성하고 있다.

map의 출력은 PrintStar 클래스의 print 메소드를 통해 이루어진다.

위 코드는 10 * 10 맵에 별 4개를 사각형 모양으로 찍게된다.

main의 코드를 확인해 보자.

```c#

class Test
{
    static void Main(string[] args)
    {
        PrintStar printStar = new PrintStar();
        printStar.print();
    }
}

```

여기까진 간단하다. Star 객체와 Blank 객체가 Item으로 표시되는 다형성을 이해할 수 있으면 된다.

그런데 이 프로그램을 공개 배포했더니 대박이 났다! (~~픽션입니다~~)

그래서 제작자는 별찍기 버전 2를 만들기로 결정했다.

버전 2에서는 사용자가 2개의 맵 중에서 하나를 선택할 수 있다.

두번째 맵에는 기존의 별인 Star 대신 $ 모양을 갖는 MoneyStar를 출력할 것이다.

코드를 통해 확인해보자.

```c#

class Item
{
    public virtual void print() { }
}
class Star : Item
{
    public override void print() { Console.Write("*"); }
}
class Blank : Item
{
    public override void print() { Console.Write(" "); }
}

class MoneyStar : Star
{
    public override void print() { Console.Write("$"); }
}

class PrintStar
{
    const int MapSize = 10;
    Item[,] map = new Item[MapSize, MapSize];
    public PrintStar(int mode)
    {
        if(mode == 1)
        {
            map[3, 3] = new MoneyStar();
            map[4, 2] = new MoneyStar();
            map[4, 4] = new MoneyStar();
            map[5, 3] = new MoneyStar();
        }
        else
        {
            map[3, 3] = new Star();
            map[4, 2] = new Star();
            map[4, 4] = new Star();
            map[5, 3] = new Star();
        }
        for (int i = 0; i < MapSize; i++)
            for (int j = 0; j < MapSize; j++)
                if (map[i, j] == null)
                    map[i, j] = new Blank();
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

이 코드는 PrintStar의 생성자에 mode를 추가하여
mode가 1이면 MoneyStar를, 그 외엔 Star를 출력한다.

이 코드의 문제점이 보이는가?

그렇다. **바로 별의 종류가 늘어날 때마다 생성자에 하드코딩을 해야 하는것이 문제이다.**
(~~물론 다른 문제들도 있긴하다~~)

이렇게 되면 새로운 별이 추가될 때 마다 기존의 PrintStar 클래스를 수정해야 한다.

이는 재사용성의 측면에서 볼 때 매우 문제가 된다.

어떻게 하면 이 문제를 해결할 수 있을까?

한가지 해결 방법은 생성자에 Star 객체를 받아서 그 객체를 복사해서 사용하는 것이다.

이 경우 새로운 별이 추가되어도 그 객체를 전달하는것으로 기존의 클래스를 수정하지 않아도 된다.

좋은 방법이지만, 만약 공백의 모양도 바뀌게 된다면 똑같은 문제가 발생할 수 있다.

이런 문제까지 해결하기 위해 나온것이 바로 **추상 팩토리(Abstract Factory) 패턴**이다.

원리는 간단하다. 처음의 해결방법이 객체 하나만을 받아오는 것이었다면, 이번엔 여러 객체를
생성하는 방법을 나타내는 객체를 받아오는 것이다.

코드를 살펴보자.

```c#

// 위 클래스 생략

class StarFactory
{
    public virtual Star CreateStar() { return new Star(); }
    public virtual Blank CreateBlank() { return new Blank(); }
}

class MoneyStarFactory : StarFactory
{
    public override Star CreateStar() { return new MoneyStar(); }
}

class PrintStar
{
    const int MapSize = 10;
    Item[,] map = new Item[MapSize, MapSize];
    public PrintStar(StarFactory factory)
    {
        map[3, 3] = factory.CreateStar();
        map[4, 2] = factory.CreateStar();
        map[4, 4] = factory.CreateStar();
        map[5, 3] = factory.CreateStar();
        for (int i = 0; i < MapSize; i++)
            for (int j = 0; j < MapSize; j++)
                if (map[i, j] == null)
                    map[i, j] = factory.CreateBlank();
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

Star나 MoneyStar 같은 클래스의 정의는 앞에 포함했기 때문에 생략했다.

main의 구현도 살펴보자.

```c#

class Test
{
    static void Main(string[] args)
    {
        //PrintStar printStar = new PrintStar(new StarFactory());
        PrintStar printStar = new PrintStar(new MoneyStarFactory());
        printStar.print();
    }
}

```

PrintStar 객체를 생성할 때, 팩토리 객체를 넘겨주는것으로 객체 생성의 책임을 팩토리 객체에 위임했다.

이제 새로운 종류의 Star나 Blank를 만들더라도, 새로운 팩토리를 넘겨주는것으로 기존의 코드를 수정하지 않아도 된다.

전에 비해 훨씬 깔끔하지 않은가? 하지만 추상 팩토리를 이용하는것이 항상 옳은것만은 아니다.

다음을 통해 추상 팩토리의 장단점을 알아보자.

----

### 장단점

----

#### 장점

1. 구체적인 클래스를 분리한다.
    * 팩토리를 통해 객체를 생성하는 과정과 책임을 캡슐화 했기 때문에, 구체적인 구현 클래스가 사용자에게서 분리된다.
2. 제품군을 쉽게 대체할 수 있도록 한다.
    * 위에서 보았던 StarFactory와 MoneyStarFactory의 사용이 그 예이다. 코드 한줄로 제품군을 대체할 수 있다.
3. 제품 사이의 일관성을 증진시킨다.
    * 제품이 하나의 군 안에서 함께 동작하도록 설계되어 있다면, 다른 제품과 동시에 사용되지 않게 막아야 할 필요가 있다.
    * 팩토리의 사용은 이 점을 아주 쉽게 보장할 수 있다.

#### 단점

1. 새로운 종류의 제품을 제공하기 힘들다.
    * 여기서 새로운 종류란, Star 클래스를 상속하는 여러 종류의 Star가 아니라 더 넓은 범위의 새로운 종류를 의미한다.
    * 예를들어 Star와 Blank 말고 폭탄을 추가하고 싶다면, 기존의 모든 팩토리에 폭탄의 생성을 추가해야 한다.
    * 이는 재사용성을 해치는 단점으로 작용하게 된다.

----

### 클래스 다이어그램

----

![abstrcact_factory_uml](http://i.imgur.com/vdZeJVu.jpg)

뭔가 많이 엉성하지만.. UML 다이어그램을 그려보았다.

그려보면서 느낀거지만 지금의 구조가 제대로 된 것인가에 대한 의문이 남는다.

1. Factory의 Create 메소드의 반환은 Item이어야 하지 않을까?
2. PrintStar는 팩토리 객체를 계속 가지고 있지 않기때문에 Dependency로 표기하였는데 맞는것일까?
3. 팩토리가 늘어날수록 다이어그램이 매우 지저분해질것 같은데 이럴땐 어떻게 하면 좋을까?

----

### 결론

----

추상 팩토리는 클래스에게서 객체 생성에 대한 책임을 분리하는 것으로 재사용성을 지켜주어
클래스를 더 유연하게 만들어 주는 패턴이다.

하지만 단점도 존재하니 잘 고려하여 사용하는것이 중요하겠다.

----

### 참고

----

위 예제에서 문제가 될 수 있는 부분은 객체의 생성부분 뿐만은 아니다.

앞으로 다룰 패턴에서 이 코드의 다른 문제에 대해서도 살펴볼것이기 때문에 위 코드를 꼭 읽어보자.