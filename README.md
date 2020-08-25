# Iterator, Generator

## Iterator

### 이터레이터(iterator)는 값을 차례대로 꺼낼 수 있는 객체(object)이다.

for문을 반복할때 for i in range(100):은 0부터 99까지 연속된 숫자를 만들어낸다.

사실은 이는 0부터 99까지 만드는 것이 아닌 0부터 99까지 차례대로 꺼내는 이터레이터를 만드는 것이다.

연속된 숫자가 클때 미리 만들어 놓으면 메모리를 많이 사용하게 된다.

```python
>>> dir([1, 2, 3])
['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
```



리스트중 `__iter__` 이 존재하는데 이를 호출하면 iterator가 나온다.

```python
>>> [1, 2, 3].__iter__()
<list_iterator object at 0x03616630>
```



리스트의 이터레이터를 저장한뒤 `__next__` 를 호출해보면 요소를 차례대로 꺼낼 수 있다.

```python
>>> it = [1, 2, 3].__iter__()
>>> it.__next__()
1
>>> it.__next__()
2
>>> it.__next__()
3
>>> it.__next__()
Traceback (most recent call last):
  File "<pyshell#48>", line 1, in <module>
    it.__next__()
StopIteration
```

it에서 `__next__` 를 호출할 때마다 리스트에 들어있는 1, 2, 3이 나온다.
그리고 3 다음에 `__next__` 를 호출하면 StopIteration 예외가 발생한다.


리스트뿐만 아니라 문자열, 딕셔너리, 세트도 __iter__를 호출하면 이터레이터가 나온다.

```python
>>> 'Hello, world!'.__iter__()
<str_iterator object at 0x03616770>
>>> {'a': 1, 'b': 2}.__iter__()
<dict_keyiterator object at 0x03870B10>
>>> {1, 2, 3}.__iter__()
<set_iterator object at 0x03878418>
```



### Iterator 생성

```python
class Counter:
    def __init__(self, stop):
        self.current = 0    # 현재 숫자 유지, 0부터 지정된 숫자 직전까지 반복
        self.stop = stop    # 반복을 끝낼 숫자
 
    def __iter__(self):
        return self         # 현재 인스턴스를 반환
 
    def __next__(self):
        if self.current < self.stop:    # 현재 숫자가 반복을 끝낼 숫자보다 작을 때
            r = self.current            # 반환할 숫자를 변수에 저장
            self.current += 1           # 현재 숫자를 1 증가시킴
            return r                    # 숫자를 반환
        else:                           # 현재 숫자가 반복을 끝낼 숫자보다 크거나 같을 때
            raise StopIteration         # 예외 발생
 
for i in Counter(3):
    print(i, end=' ')
    
========================
1, 2, 3
```



### Iterarot는  언패킹이 가능하다

```python
>>> a, b, c = Counter(3)
>>> print(a, b, c)
0 1 2
>>> a, b, c, d, e = Counter(5)
>>> print(a, b, c, d, e)
0 1 2 3 4
```



### 인덱스로 접근할 수 있는 Iterator(__getitem__)

- `__getitem__` 만 있어도 Iterator의  구현이 가능하다

```python

class 이터레이터이름:
    def __getitem__(self, 인덱스):
        코드
        
========================================

class Counter:
    def __init__(self, stop):
        self.stop = stop
 
    def __getitem__(self, index):
        if index < self.stop:
            return index
        else:
            raise IndexError
 
print(Counter(3)[0], Counter(3)[1], Counter(3)[2])
 
for i in Counter(3):
    print(i, end=' ')
    
==============================
0 1 2
0 1 2
```



## Generator

### 제너레이터는 이터레이터를 생성해주는 함수이다.

이터레이터는 클래스에서 `__iter__` , `__next__` 또는 `__getitem__` 메서드를 구현해야 하지만 제너레이터는 `yield` 라는 키워드만 사용하면 끝이다.

제너레이터는 발생자라고 부르기도 한다.



### 함수 안에서  yield를 사용하면 함수는 제너레이터가 되며 yield에는 값(변수)을 지정한다.

```python
def number_generator():
    yield 0
    yield 1
    yield 2
 
for i in number_generator():
    print(i, end=" ")
    
===================
0 1 2
```



### yield의 동작 과정

- 변수 = next(제너레이터객체)

```python
def number_generator():
    yield 0    # 0을 함수 바깥으로 전달하면서 코드 실행을 함수 바깥에 양보
    yield 1    # 1을 함수 바깥으로 전달하면서 코드 실행을 함수 바깥에 양보
    yield 2    # 2를 함수 바깥으로 전달하면서 코드 실행을 함수 바깥에 양보
 
g = number_generator()
 
a = next(g)    # yield를 사용하여 함수 바깥으로 전달한 값은 next의 반환값으로 나옴
print(a)       # 0
 
b = next(g)
print(b)       # 1
 
c = next(g)
print(c)       # 2

====================
0
1
2
```

yield를 사용하여 바깥으로 전달한 값은 next 함수( `__next__` 메서드)의 반환값으로 나온다

따라서 next(g)의 반환값을 출력해보면 yield에 지정한 값 0, 1, 2가 차례대로 나온다

즉, 제너레이터 함수가 실행되는 중간에 next로 값을 가져온다


### Generator 생성

```python
def number_generator(stop):
    n = 0              # 숫자는 0부터 시작
    while n < stop:    # 현재 숫자가 반복을 끝낼 숫자보다 작을 때 반복
        yield n        # 현재 숫자를 바깥으로 전달
        n += 1         # 현재 숫자를 증가시킴
 
for i in number_generator(3):
    print(i)
======================
0
1
2
```



### yield에서 함수 호출하기

```python
def upper_generator(x):
    for i in x:
        yield i.upper()    # 함수의 반환값을 바깥으로 전달
 
fruits = ['apple', 'pear', 'grape', 'pineapple', 'orange']
for i in upper_generator(fruits):
    print(i)
============================
APPLE
PEAR
GRAPE
PINEAPPLE
ORANGE
```

리스트 fruits에 들어있는 문자열이 모두 대문자로 출력한다 

yield i.upper()와 같이 yield에서 함수(메서드)를 호출하면 해당 함수의 반환값을 바깥으로 전달한다

즉, yield에 무엇을 지정하든 결과만 바깥으로 전달한다 (함수의 반환값, 식의 결과).



### yield from으로 값을 여러 번 바깥으로 전달하기

```python
def number_generator():
    x = [1, 2, 3]
    for i in x:
        yield i
 
for i in number_generator():
    print(i)
=============================
1
2
3
```

이런 경우에는 매번 반복문을 사용하지 않고, yield from을 사용하면 된다

 yield from에는 반복 가능한 객체, 이터레이터, 제너레이터 객체를 지정한다

- yield from 반복가능한 객체
- yield from 이터레이터
- yield from 제너레이터 객체

### yield from에 리스트를 지정해서 숫자 1, 2, 3을 바깥으로 전달

```python
def number_generator():
    x = [1, 2, 3]
    yield from x    # 리스트에 들어있는 요소를 한 개씩 바깥으로 전달
 
for i in number_generator():
    print(i)
==================
1
2
3
```



### yield from에 제너레이터 객체 지정하기

```python
def number_generator(stop):
    n = 0
    while n < stop:
        yield n
        n += 1
 
def three_generator():
    yield from number_generator(3)    # 숫자를 세 번 바깥으로 전달
 
for i in three_generator():
    print(i)
===========================
0
1
2
```