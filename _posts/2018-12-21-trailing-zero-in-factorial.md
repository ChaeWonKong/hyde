---
title: "N 팩토리얼에서 1의자리부터 연속하는 0의 개수 구하기"
date: "2018-12-21"
categories:
- Algorithm
excerpt: |
  파이썬에서 재귀호출과 동적계획법 사용하는 방법 익히기. 동적계획법과 재귀호출을 이용해 N!에서 숫자 뒤에 붙는 0의 개수를 반환하는 함수를 작성해본다.
feature_text: |
  ## N 팩토리얼에서 1의자리부터 연속하는 0의 개수 구하기
  동적계획법과 재귀호출을 이용해 N!에서 숫자 뒤에 붙는 0의 개수를 반환하는 함수를 작성해본다.
feature_image: "https://images.unsplash.com/photo-1509228468518-180dd4864904?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1950&q=80"
image: "https://images.unsplash.com/photo-1509228468518-180dd4864904?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1950&q=80"
---

```python
def count_trailing_zero(n):
    """Return number of zeros in n!"""
    cache = [-1] * (n+1)
    
    def count(n, div):
        """Return counts of how many times n could be divided by div"""      
        if n % div != 0:
            return 0
        elif cache[n] != -1:
            return cache[n]
        else:
            return count(n//div, div) + 1
    
    counts = 0
    for i in range(1, n+1):
        cache[i] = count(i, 5)
        counts += cache[i]
        
    return counts
```

## N!에서 1의자리부터 연속하는 0의 개수 구하기
동적계획법과 재귀호출을 이용해 N!에서 숫자 뒤에 붙는 0의 개수를 반환하는 함수를 작성해본다.

## 규칙 파악
먼저 1의자리부터 연속하는 0의 개수를 구하려면 어떤 규칙을 알아야 하는지 살펴본다. 0이 늘어나기 위해서는 반드시 2와 5의 곱이 있어야 한다. 예를들어, 5!은 "5\*4\*3\*2\*1이다. 5가 1개, 2가 1개 4가 1개 곱해졌다. 즉, 5는 1개, 2는 3개 곱해진 셈이다. 따라서 0의 개수는 1개이다. 실제로 5!은 120으로 1의자리부터 연속하는 0의 개수는 1개이다.

2가 아무리 많이 곱해져 있더라도 5가 2의 개수보다 적게 곱해져 있다면 5의 개수 이상만큼 0이 연속하는 것은 불가능하다. 즉, 0의 개수는 min(5의 곱해진 개수, 2의 곱해진 개수)이다.

여기서 하나의 트릭이 있다. 팩토리얼은 1부터 N까지 연속한 숫자의 곱으로 표현된다. 짝수는 모두 2로 나눠지므로 2를 포함한다. 짝수는 숫자 2개 당 1개가 존재한다. 즉, 무조건 절반 이상의 곱해진 숫자는 2를 포함한다. 

한편, 5는 다섯 번에 한번 등장한다. 따라서 무조건 5의 개수는 2의 개수보다 적을 수밖에 없다. 따라서 굳이 min()함수를 이용해 최소를 구할 필요가 없이 무조건 5의 개수만 카운트 하면 0의 개수를 알 수 있다.

## 재귀함수를 위한 일반항 찾기
우섯 숫자들을 살펴보자. N!에서 모든 숫자가 5로 한 번만 나뉘는 것은 아니다. 만약 N이 125 라고 할 때, 125!에서 25, 50, 75, 100은 5로 2번, 125는 5로 3번 나뉜다. N!을 구성하는 1부터 N까지의 모든 수에 대해서 5로 나뉘는 수가 있다면, 5로 더 이상 나뉘지 않을 때까지 연속해서 나누며 한번 나눠질 때마다 5의 개수를 count해줘야 한다. 뭔가 재귀의 냄새가 난다.

250을 예로 들어보자.
```python
counts = 0

# 시작
250 % 5 = 0 
250 / 5 = 50
counts += 1
# count = 1

50 % 5 = 0
50 / 5 = 10
counts += 1
# count = 2

10 % 5 = 0
10 / 5 = 2
counts += 1
# count = 3

2 % 5 = 2
# 종료

print(counts)
# 3
```

받은 숫자가 5로 나뉜다면 5로 나눠주고 counts에 1을 더한다. 이후 그 숫자에 대해 똑같은 작업을 진행한다. 즉, 다음과 같은 일반항이 성립한다.

```python
if n % 5 == 0:
    f(n) = f(n/5) + 1
```

탈출조건은 n을 5로 나눈 나머지가 0이 아닌 경우가 된다.

## 재귀함수 구현하기
일반항을 활용해 재귀함수를 먼저 구현해본다.
```python
def count(n, div):
    if n % div != 0:
        return 0
    return count(n // 5, div) + 1
```

탈출조건은 n을 div로 나눈 나머지가 0이 아닌 경우이다. 이 경우엔 나눗셈을 진행하지 않았으므로 counts를 늘려줄 필요가 없다. 따라서 0을 반환한다.

이외의 경우에는 일반항에서처럼 count(n//5, div) +1을 반환한다.

## 전체적인 함수 구현하기
이제 재귀함수를 활용해 전체적인 함수를 구현해보자.

```python
def count_trailing_zero(n):

    def count(n, div):
        if n % div != 0:
            return 0
        return count(n//div, div) + 1
    
    counts = 0
    for i in range(1, n+1):
        counts += count(i, 5)
        
    return counts
```

N!은 1부터 N까지의 연속하는 모든 숫자의 곱이다. 따라서 연속하는 각 숫자에 대해 각각 count함수를 실행해 5로 나눠지는 횟수를 세고 이 결과를 counts라는 변수에 더하며 저장해 나간다.

```python
for i in range(1, n+1):
    counts += count(i, 5)
```

이후 counts를 반환하는 return문을 작성하면 함수는 완성된다.

## 리팩토링

여기서 살펴볼점은 count 재귀함수는 동적계획법의 여지가 있다는 것이다. 100!에서 100을 생각해보자. 100은 20과 5의 곱이며, 20은 4와 5의 곱이다. 100을 5로 나누는 재귀호출 함수를 실행하다 보면 반드시 20을 5로 나누는 재귀호출을 실행하게 된다.

여기서 100!의 특징은, 1부터 100까지 모든 수의 곱으로 이루어졌다는 것이다. 100!의 1의자리부터 연속하는 0의 개수를 구하기 위해서는 반드시 20에 대한 재귀호출을 통해 5로 나눠지는 횟수를 카운트 했을 터다. 이미 한 번 한 계산을 또 한다는 것은 뭔가 비효율적이다.

이렇게 생각해볼 수 있다. 어차피 1부터 차근차근 100까지의 모든 수를 재귀호출을 통해 5로 나눠볼 텐데, 한번 나눠서 나온 결과 값을 배열에 기록해두고, 이후 재귀호출에서 해당 값을 찾아 쓰면 어떨까? 즉 기록하며 풀면 어떨까? 그렇게 한다면 같은 연산을 반복하지 않으니 성능은 향상될 것이다.

이것이 바로 동적계획법이다.

## 동적계획법 도입하기
우선 0부터 n+1까지의 cache라는 배열을 생각해보자. 배열의 각 원소는 -1로 초기화 해본다.
```python
cache = [-1] * (n+1)
```
여기서 n+1까지 두는 이유는 간단하다. 계산의 편리함을 위해 0을 padding으로 두었다. 즉 실제로 우리가 기록해나가는 것은 1부터 n까지의 index이다.

cache가 존재할 때, 앞서 만든 재귀호출 함수는 다음과 같이 리팩토링 할 수 있다.

```python
def count(n, div):
    if n % div != 0:
        return 0
    elif cache[n] != -1:
        return cache[n]
    else:
        return count(n // div, div) + 1
```
elif문이 추가 되었다. 만약 cache[n]이 -1이 아니라면, 즉 이미 계산한 값이 저장되어 있다면 cache[n]을 반환하라는 것이다.

전체적인 알고리즘은 다음과 같아진다.

```python
def count_trailing_zero(n):
    cache = [-1] * (n+1)
    
    def count(n, div):
        if n % div != 0:
            return 0
        elif cache[n] != -1:
            return cache[n]
        else:
            return count(n//div, div) + 1
    
    counts = 0
    for i in range(1, n+1):
        cache[i] = count(i, 5)
        count += cache[i]
        
    return counts
```

for문을 살펴보자.
```python
for i in range(1, n+1):
    cache[i] = count(i, 5)
    counts += cache[i]
```
우선 count 함수를 실행한 값을 바로 counts에 더하는 것이 아니라, cache에 저장한다. 이후 cache에 저장된 값을 counts에 더하는 식으로 for문이 변경되었다. 이를 통해 재귀호출을 통해 찾은 결과값은 cache에 저장되고, 재귀호출시에는 cache를 탐색해 답을 반환하는 동적계획법 구현이 완성되었다.

## 마치며
오늘은 N!의 1의 자리부터 연속한 0의 개수를 세는 간단한 알고리즘을 통해 재귀호출과 동적계획법에 대해 살펴보았다. 

재귀호출은 아름답고 간결하지만 depth가 깊어지거나 복잡해지만 알고리즘의 성능이 크게 떨어지기도 한다. 동적계획법을 사용할 수 있다면, 동적계획법을 도입해 한번 계산한 값은 저장해두고 때때로 불러옴으로써 성능을 개선할 수 있다.