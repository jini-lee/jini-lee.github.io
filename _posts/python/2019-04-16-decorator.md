---
layout: post
title: decorator 
category: python
tags: [python, 파이썬, decorator, closure]
comments: true
---
# Decorator

## Decorator ?
데코레이터란 특별한 함수나 클래스로 동작한다. 이 기능이 가능한 이유는 파이썬은 First-Class function이기 때문이다. 
First-Class function의 대표적인 특징이며 이를 통해 데코레이터라를 사용할 수 있다.  
- 변수나 데이터 구조안에 담을 수 있다.
- 함수의 파라미터로 전달할 수 있다.
- 반환 값으로 사용할 수 있다.(closure)
- 동적으로 프로퍼티 할당이 가능하다.

## Inner function
본격적으로 데코레이터에 들어가기전에 closure의 개념이 들어간 inner function을 살펴보자.
closure란 First-Class function을 지원하는 언어의 네임 바인딩 기술이다. 함수안에 선언된 내부 함수를 반환하고 이를 함수 외부에서 접근이 가능하게 된다.

```python 
# closure example

def parent_func(child):
    print("Parent")
    def first_child():
        return "First child"
    def second_child():
        return "Second child"
    if child == 1:
        return first_child
    else:
        return second_child 
```

위 예제를 보면 부모함수 내부에 선언된 두 개의 자식함수를 조건문에 의해 각 함수를 반환한다.
아래의 예제를 보면서 어떤일이 일어나는지 살펴보자.

```python
>>> first = parent_func(1)
부모함수에서 출력
>>> first
<function __main__.first_child>
>>> first()
첫 번째 자식함수에서 출력
```

위 예제를 보면 first 변수에 부모함수에서 반환한 first_child 함수를 참조하고 first를 실행하면 first_child 함수를 실행하게된다.
이러한 특징을 이용한 디자인 패턴이 나왔고 데코레이터 또한 구현할 수 있게된다. 

## Simple decorator 

```python
def simple_decorator(func):
    def wrapper():
        print('Before the function is called.')
        func()
        print('After the function is called.')
    return wrapper

def say_hello():
    print('Hello!')

@simple_decorator
def say_hi():
    print('Hi!')
```

```python
>>> hello = simple_decorator(say_hello)
>>> hello
<function __main__.wrapper>
>>> hello()
Before the function is called.
Hello!
After the function is called.
>>> say_hi
<function __main__.wrapper>
>>> say_hi()
Before the function is called.
Hi!
After the function is called.
```

say_hello 예제는 일급객체의 특징을 이용해 함수의 파라미터로 함수를 던져 함수 반환을 통해 외부(say_hello)에서 내부 함수(wrapper)를 접근하는 것을 구현한 예제이다.  
say_hi 예제는 데코레이터를 사용했는데 외부 함수(say_hi)위에 데코레이터를 정의하여 구현한 예제이다. simple_decorator(say_hi)

## Decorating functions with arguments

```python
def arg_decorator(func):
    def wrapper(*args, **kwargs):
        print('wrapper: hi! {}'.format(args[0]))
        func(*args, **kwargs)
    return wrapper

@arg_decorator
def say_hi(name):
    print('hi! {}'.format(name))
```

say_hi는 wrapper와 동일하다고 생각해도 무관하다. say_hi에 name값을 주면 wrapper의 파리미터로 전달이 된다. 클로저 특징으로 반환된 wrapper를 say_hi에서 참조하기 때문이다. 

```python
>>> say_hi
<function __main__.wrapper>
>>> say_hi('Cole')
wrapper: hi! cole
hi! cole
```

## Returning values from decorated functions
```python
def return_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)
        return func(*args, **kwargs) # 내부 함수의 반환값으로 외부 함수의 반환값이 동작 
    return wrapper

@return_decorator
def say_hi(name):
    print('Hi! {}'.format(name))
    return 'I\'m tyler'

# return_decorator(say_hi('Cole'))
# say_hi는 wrapper인데 wrapper가 반환을 하지 않으면 호출한 함수의 반환값이 동작하지 않는다. 재귀호출을 생각하면 된다.
```

```python
>>> say_hi('Cole')
Hi! Cole
Hi! Cole
I'm tyler
```

## Functools decorator  
데코레이터를 인터프리터에서 확인해보면 내부함수인 wrapper를 참조하는 것을 확인할 수 있는데, 이는 디버깅시 문제가된다.
따라서 functools 모듈을 이용해서 실제 데코레이터의 함수 정보를 얻어야한다.

```python
import functools

def simple_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        func(*args)
    return wrapper

@simple_decorator
def say_hi(name):
    print('Hi! {}'.format(name))
```

```python
>>> say_hi
<function __main__.say_hi>
>>> help(say_hi)
Help on function say_hi in module __main__:

say_hi(*args, **kwargs)
```

## 정리
python은 first-class object인 언어이다. 이러한 특징으로 closure를 이용해 데코레이터를 구현하다.   
이는 특정 함수의 시작 전 시작 후에 동작하는 로직을 구현할 때 유용하다. 유효성 검사, 로그 관리, 싱글턴 등 
함수가 가지는 실제 코어 로직관 별도로 진행되는 역할을 데코레이터를 통해 구현함으로써 코드의 재사용을 줄이는 좋은 기법이다.


### reference
[realpython](https://realpython.com/primer-on-python-decorators/)