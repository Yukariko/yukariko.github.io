---
layout: post
title: C++11 move semantics benchmark
categories: [C++]
tags: [C++11]
fullview: false
comments: true
---

계기는 [이곳](https://www.acmicpc.net/board/view/8754)에서 부터 시작했다.

내가 알고있는 표준에 의하면 C++11 이전의 표준에서는
vector의 함수 반환이 vector 내부의 item size에 비례한 시간이 소모된다.

즉 **deep copy**가 이루어진다.

반면 C++11 부터는 move semantics에 의해 **shallow copy**가 이루어져 item size와는
무관한 시간이 소모된다.

과연 정말 그럴까? 실험해보기로 하였다.

--------------

## 실험 환경

--------------

실험에 사용된 머신의 스펙은 아래와 같다.

> | 이름 | 스펙 |
> | --- | --- |
> | CPU | AMD Opteron(tm) Processor 4180 (2600MHz) |
> | RAM | 32GB |
> | OS | Linux Ubuntu 14.04 |
> | COMPILER | g++ 4.8.4 |

실험에 사용된 코드는 다음과 같다.

```c++
#include <bits/stdc++.h>

using namespace std;

const int n = 100000000;

vector<int> f() 
{
        vector<int> a;
        for(int i=1; i < n; i++)
                a.push_back(n / i); 
        return a;
}

void f2(vector<int>& b)
{
        for(int i=1; i < n; i++)
                b.push_back(n / i); 
}

int main()
{
        clock_t begin, end;
        begin = clock();
        vector<int> b = f();
        end = clock();
        cout<<"수행시간 : "<< (double)(end-begin) / CLOCKS_PER_SEC <<'\n';

        begin = clock();
        vector<int> c;
        f2(c);
        end = clock();
        cout<<"수행시간 : "<< (double)(end-begin) / CLOCKS_PER_SEC <<'\n';
        return 0;
}
```

위 코드를 다음의 4가지 컴파일 옵션을 통해 성능을 측정하였다.

`g++ -o test test.cpp -std=c++98`  
`g++ -o test test.cpp -std=c++11`  
`g++ -o test test.cpp -std=c++98 -O2`  
`g++ -o test test.cpp -std=c++11 -O2`

나의 가정은 다음과 같다.

1. 함수의 리턴에서 *deep copy*가 이루어진다면, f함수는 f2함수보다 성능이 나쁠것이다.
2. 함수의 리턴에서 *shallow copy*가 이루어진다면, f함수는 f2함수와 성능이 비슷할것이다.

이제 실험 결과를 보도록 하자.

--------------

## 실험 결과

--------------

> 표준 | f | f2 
> :--- | ---: | ---:
> C++98 | 4.3485s | 4.1647s
> C++11 | 6.6201s | 6.6205s
> C++98 (O2) | 2.3574s | 2.3947s
> C++11 (O2) | 2.5109s | 2.5528s


실험 결과를 보아 내 예상은 적중한 것 같다.

C++98의 경우 f함수가 f2함수에 비해 0.2초가량 성능이 안좋게 나왔고, C++11은 거의 비슷했다.

이는 deep copy와 shallow copy의 차이로 봐도 될 것 같다.

그러나 의외의 결과를 볼 수 있었는데, 전체적인 시간에서
C++11이 C++98보다 2초나 느린 시간을 보였다는 것이다.

이것에 대해서는 단순히 C++11의 구현이 무거워서 라는 생각밖에는 들지 않는다.

한가지 더 눈여겨 볼 점은 O2 최적화를 거쳤을 때 둘 모두 큰 차이가 없었다는 것이다.

이정도는 최적화 해 줬다고 생각하면 되겠지만, 항상 f함수가 더 빨랐다는 점은 주목할 만하다.

--------------

## 결론

--------------

대부분의 경우 컴파일 시 최적화를 수행하기 때문에 어떤 방법을 사용해도 별 문제 없을것 같다.

다만 최적화 시 f함수가 f2함수보다 항상 빠른것이 사실이라면 전자의 방법을 채용하는것이 어떨까?