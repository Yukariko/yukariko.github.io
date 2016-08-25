---
layout: post
title: 디자인 패턴 - 팩토리 메소드(Factory Method)
categories: [DesignPattern]
tags: [DesignPattern, FactoryMethod, UML]
fullview: false
comments: true
---

----

### 정의

----

객체를 생성하는 인터페이스는 미리 정의하되, 인스턴스를 만들 클래스의 결정은 서브클래스 쪽에서 내리는 패턴.

----

### 상황

----

이전 포스트 [추상 팩토리](https://yukariko.github.io/designpattern/2016/08/19/abstract-factory.html),
[빌더](https://yukariko.github.io/designpattern/2016/08/20/builder.html)에서 객체 생성에 대한 이야기를 했었다.

추상 팩토리는 여러 관련있는 객체들을 모아 그 객체 각각을 생성 하는 방법을 제공하는 패턴이었고,
빌더는 하나의 통합된 객체를 만들기 위한 방법을 제공하는 패턴이었다.

이번에는 다른 관점으로 별찍기를 바라보도록 하자.

우선 PrintStar의 맵 생성 코드를 다시 한번 확인해보자.

```csharp

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

```

역시 위 코드에서 주목해야 할 점은 Star와 Blank를 직접 생성하는것으로 인한 낮은 확장성과 유연성이다.

그래서 이전 포스트에서는 객체의 생성의 책임을 다른 클래스로 분리하여 이를 해결했었다.

하지만 객체 생성의 책임을 PrintStar가 가지고 있어야 한다면? 그러면서도 하드 코딩을 피하고 싶다면 어떻게 해야 할까?

생각보다 답은 간단하다. `객체를 생성하는 메소드를 미리 정의해 놓고, 상속을 통해 이 메소드를 구현하는 것`이다.

바로 코드를 살펴보자.

```csharp

class PrintStar
{
    const int MapSize = 10;
    Item[,] map = new Item[MapSize, MapSize];

    public virtual Item MakeStar()
    {
        return new Star();
    }

    public virtual Item MakeBlank()
    {
        return new Blank();
    }

    public PrintStar()
    {
        map[3, 3] = MakeStar();
        map[4, 2] = MakeStar();
        map[4, 4] = MakeStar();
        map[5, 3] = MakeStar();
        for (int i = 0; i < MapSize; i++)
            for (int j = 0; j < MapSize; j++)
                if (map[i, j] == null)
                    map[i, j] = MakeBlank();
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

class PrintMoneyStar : PrintStar
{
    public override Item MakeStar()
    {
        return new MoneyStar();
    }
}

```

이전과 다른점은 MakeStar와 MakeBlank 메소드를 통해 객체를 생성하고, 다른 객체를 생성해야 할 때에는 상속을 통해 이를 정의한다는 점이다.

이제 PrintStar는 객체 생성의 책임을 지면서도 하드코딩을 피하면서 확장성을 가질 수 있게 되었다.

팩토리 메소드 패턴은 특히 추가될 객체가 무엇인지 모르는 상태로 구현을 진행해야 할 때 유용한 방법이다.

이제 팩토리 메소드의 장단점을 알아보자.

----

### 장단점

----

팩토리 메소드의 장단점은 비교적 확실하다.

#### 장점

1. 클래스(PrintStar)가 사용자의 코드(MoneyStar)에 종속되지 않도록 해 준다. 
2. 객체를 직접 생성하는 것보다 훨씬 응용성이 높아진다.

#### 단점

1. 제품 객체(Item)를 하나만 만들더라도 클래스(PrintStar)를 상속해야 할 수 있다.
2. 팩토리 메소드는 부모 클래스에서 처리 할 일을 모두 정의하고, 상속은 기존 클래스를 확장하지 않는다. 이는 상속의 개념을 제대로 반영하지 못한것.

----

### 클래스 다이어그램

----

![UML](http://imgur.com/BBoBqif.jpg)

클래스 다이어그램을 그려보았다.

이번 다이어그램을 그리면서 생긴 의문은 다음과 같다.

1. 팩토리 메소드로 구현된 서브 클래스 PrintMoneyStar는 MakeStar밖에 오버라이드 하지 않는다.
그래도 Star, Blank 모두에 Dependency를 가진다고 봐야하는가?
2. PrintStar와 Item의 상관관계는 서브 클래스인 PrintMoneyStar도 마찬가지라고 봐야 할듯 싶다. 하지만 팩토리 메소드에서는
서브 클래스에서 생성 방법만 재정의하기 때문에 따로 관계를 정의하지 않았다. 어떻게 해야 정확한걸까?

----

### 결론

----

팩토리 메소드 패턴은, 어떤 객체를 생성하게 될 지 모르는 상황에서도 구현을 진행하고, 실제 객체 생성은 서브 클래스에서 구현해주는 패턴이다.
