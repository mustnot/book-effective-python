# 4장. 컴프리헨션과 제너레이터

많은 프로그램이 리스트, 딕셔너리의 키/값 쌍, 집합 처리를 중심으로 만들어진다. 파이썬에서는 **컴프리헨션**(comprehension)이라는 특별한 구문을 사용해 간결하게 이터레이션하면서 원소로부터 파생되는 데이터 구조를 생성할 수 있다.

컴프리헨션 코딩 스타일은 **제너레이터**(generator)를 사용하는 함수로 확장할 수 있다. 제너레이터는 함수가 **점진적**으로 반환하는 값으로 이뤄지는 스트림을 만들어준다.

제너레이터를 사용하면 성능을 향상시키고, 메모리 사용을 줄이고 가독성을 높일 수 있다.

ㅤ

## 27. map과 filter 대신 컴프리헨션을 사용하라.

파이썬은 다른 sequence나 iterable에서 새 리스트를 만들어내는 간결한 구문을 제공하는데, 이런 식을 **리스트 컴프리헨션**이라 부른다.

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = [x**2 for x in a]

>>> print(squares)
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

인자가 하나인 함수를 적용하는 경우가 아니라면 간단한 경우에도 map 내장 함수보다 리스트 컴프리헨션이 더 명확하다. map과 달리 리스트 컴프리헨션은 입력 리스트에서 원소를 쉽게 필터링해 결과에서 원하는 원소를 제거할 수 있다.

```python
even_squares = [x**2 for x in a if x % 2 == 0]

>>> print(even_squares)
[4, 16, 36, 64, 100]
```

filter 내장 함수를 map과 함께 사용해서 같은 결과를 얻을 수 있지만, 이렇게 만든 코드는 읽기가 어렵다.

딕셔너리와 집합에도 리스트 컴프리헨션과 동등한 컴프리헨션이 있다. 이를 사용하면 알고리즘을 작성할 때 딕셔너리나 집합에서 파생된 데이터 구조를 쉽게 만들 수 있다.

```python
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}

>>> print(even_squares_dict)
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}

threes_cubed_set = {x**3 for x in a if x % 3 == 0}

>>> print(threes_cubed_set)
{216, 729, 27}
```

각각의 호출을 적절한 생성자로 감싸 같은 결과를 map과 filter를 사용해 만들 수 있다. 하지만 생성자로 감싸서 작성한 코드는 너무 길기 때문에 가능하면 map과 filter를 사용하는 것을 피해야 한다.

ㅤ

## 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라.

컴프리헨션은 기본적인 사용법 외에도 루프를 여러 수즌으로 내포하도록 허용한다. 예를 들어 리스트 안에 리스트가 들어 있는 형태로 정의한 행렬을 모든 원소가 들어 있는 평평한 단일 리스트로 단순화하고 싶다고 할 때, 컴프리헨션에 하위 식을 두 개 포함시키면 처리가 가능하다.

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]

>>> print(flat)
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

또한 컴프리헨션에서 다중 루프를 사용하는 것도 가능하다. 만약 이런 컴프리헨션 안에 다른 루프가 들어 있으면 코드가 너무 길어져서 여러 줄로 나눠 써야 한다.

```python
my_list = [[[1, 2, 3], [4, 5, 6]], ...]

flat = [
    x for sublist1 in my_list
    for sublist2 in sublist1
    for x in sublist2
]
```

이 정도가 되면 다중 컴프리헨션이 다른 대안에 비해 더 길어진다. 이 코드는 일반 루프문을 사용해 변환하면 더 명확해진다.

```python
flat = []
for sublist1 in my_list:
    for sublist2 in sublist1:
        flat.extend(sublist2)
```

이렇듯 컴프리헨션에 들어가는 하위 식이 세 개 이상이 되지 않게 제한하라는 규칙을 지켜야한다. 컴프리헨션이 이보다 더 복잡해지면 일반 if와 for문을 사용하고 도우미 함수를 작성해 읽기 쉽게 해야한다.

ㅤ

## 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라.

컴프리헨션에서 같은 계산을 여러 위치에서 공유하는 경우가 흔하다.

```python
stock = {
    '못': 125,
    '나사못': 35,
    '나비너트': 8,
    '와셔': 24,
}

order = ['나사못', '못', '와셔']

def get_batches(count, size):
    return count // size

result = {}
for name in order:
    count = stock.get(name, 0)
    batches = get_batches(count, 8)
    if batches:
        result[name] = batches

>>> print(result)
{'나사못': 4, '못': 15, '와셔': 3}
```

여기서 딕셔너리 컴프리헨션을 사용하면 이 루프의 로직을 더 간결하게 표현할 수 있다.

```python
found = {name: get_batches(stock.get(name, 0), 8)
        for name in order
        if get_batches(stock.get(name, 0), 8)}

>>> print(found)
{'나사못': 4, '못': 15, '와셔': 3}
```

보다 짧지만 `get_batches` 함수가 반복된다는 단점이 있다. 이런 단점을 왈러스 연산자를 사용하여 간소화 할 수 있다.

```python
found = {name: batches for name in order
        if (batches := get_batches(stock.get(name, 0), 8))}
```

이렇게 대입식을 컴프리헨션의 값 식에 사용해도 문법적으로 올바르다. 여기서 컴프리헨션에서 대입식을 조건에만 사용하는 것을 권장한다.

ㅤ

## 30. 리스트를 반환하기보다는 제너레이터를 사용하라.

시퀀스를 결과로 만들어내는 함수를 만들 때 가장 간단한 선택은 원소들이 모인 리스트를 반환하는 것이다.

```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 진공관'
result = index_words(address)

>>> print(result[:10])
[0, 8, 18, 23, 28, 33, 39]
```

하지만 index_words 함수에는 두 가지 문제점이 있다.

1. 코드에 잡음이 많고 핵심을 알아보기 어렵다.
   - 새로운 결과를 찾을 때마다 append 메서드를 호출한다. 메서드 호출이 너무 덩어리가 크기 때문에 추가될 값(index + 1)의 중요성을 희석해버린다.
2. 반환하기 전에 리스트에 모든 결과를 다 저장해야 한다.
   - 입력이 매우 크면 프로그램이 메모리를 소진해서 중단될 수 있다.

이 함수를 개선하는 방법은 **제너레이터**를 사용하는 것이다. 제너레이터는 `yield` 식을 사용하는 함수에 의해 만들어진다.

```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

it = index_words_iter(address)

>>> print(next(it))
0
>>> print(next(it))
8
```

이 함수가 호출되면 제너레이터 함수가 실제로 실행되지 않고 즉시 이터레이터를 반환한다. 이터레이터가 `next` 내장 함수를 호출할 때마다 이터레이터는 제너레이터 함수를 다음 yield 식까지 진행시킨다.

반환하는 리스트와 상호작용하는 코드가 사라졋으므로 index_words_iter 함수가 훨씬 읽기 쉽지만 결과는 yield 식에 의해 전달된다.

```python
result = list(index_words_iter(address))

>>> print(result[:10])
[0, 8, 18, 23, 28, 33, 39]
```

또한 같은 함수를 제너레이터 버전으로 만들면 두 번째 문제를 해결할 수 있다. 제너레이터는 메모리 크기를 어느 정도 제한할 수 있으므로 입력 길이가 아무리 길어도 쉽게 처리할 수 있다.

여기서 제너레이터를 정의할 때 한 가지 알아둬야 할 점이 있다. 제너레이터가 반환하는 이터레이터에 상태가 있기 때문에 호출하는 쪽에서 재사용이 불가능하다는 사실이다.

ㅤ

## 31. 인자에 대해 이터레이션할 때는 방어적이 돼라.

객체가 원소로 들어 있는 리스트를 함수가 파라미터로 받았을 때, 이 리스트를 여러 번 이터레이션하는 것이 중요할 때가 종종 있다.

```python
def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize(visits)

>>> print(percentages)
[11.538461538461538, 26.923076923076923, 61.53846153846154]
>>> assert sum(percentages) == 100.0
(True)
```

이 코드의 규모 확장성을 높이려면 파일에서 데이터를 읽어야 한다. 파일을 읽는 함수를 재사용할 수도 있으므로 제너레이터를 정의하면 다음과 같다.

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt')
percentages = normalize(it)

>>> print(percentages)
[]
```

이런 현상이 일어난 이유는 이터레이터가 결과를 단 한번만 만들어내기 때문이다. StopIteration 예외가 발생한 이터레이터나 제너레이터를 다시 이터레이션하면 아무 결과도 얻을 수 없다.

```python
it = read_visits('my_numbers.txt')

>>> print(list(it))
[15, 35, 80]
>>> print(list(it))
[]
```

여기서 문제는 이미 소진된 이터레이터에 대해 이터레이션을 수행해도 아무런 오류가 발생하지 않는다. 이런 경우 출력이 없는 이터레이터와 이미 소진돼버린 이터레이터를 구분할 수 없다. 이 문제를 해결하기 위해 입력 이터레이터를 명시적으로 소진시키고 이터레이터의 전체 내용을 리스트에 넣을 수 있다.

```python
def normalize_copy(numbers):
    numbers = list(numbers) # Copy the iterator
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

it = read_visits('my_numbers.txt')
percentages = normalize_copy(it)

>>> print(percentages)
[11.538461538461538, 26.923076923076923, 61.53846153846154]
>>> assert sum(percentages) == 100.0
(True)
```

하지만 이런 접근 방식의 문제점은 입력 이터레이터의 내용을 복사하면서 메모리를 많이 사용할 수 있다는 것이다. 이런 문제를 해결하는 다른 방법은 호출될 때마다 새로운 이터레이터를 반환하는 함수를 받는 것이다.

```python
def normalize_func(get_iter):
    total = sum(get_iter()) # New iterator
    result = []
    for value in get_iter(): # New iterator
        percent = 100 * value / total
        result.append(percent)
    return result
```

더 나은 방법은 **이터레이터 프로토콜**(iterator protocol)을 구현한 새로운 컨테이너 클래스를 제공하는 것이다.

이터레이터 프로토콜은 파이썬의 for 루프나 그와 연관된 식들이 컨테이너 타입의 내용을 방문할 때마다 사용하는 절차다. 파이썬에서 `for x in foo` 와 같은 구문을 사용하면 실제로는 `iter(foo)`를 호출한다. iter 내장 함수는 `__iter__` 라는 특별 메서드를 호출한다. 또한 `__iter__` 메서드는 반드시 이터레이터 객체를 반환해야 한다.

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits('my_numbers.txt')
percentages = normalize(visits)

>>> print(percentages)
[11.538461538461538, 26.923076923076923, 61.53846153846154]
>>> assert sum(percentages) == 100.0
(True)
```

이렇게 작성한 새로운 컨테이너 타입을 원래의 normalize 함수에 넘기면 코드를 전혀 바꾸지 않아도 함수가 잘 작동한다. 여기서 이제는 파라미터로 받은 값이 단순한 이터레이터가 아니라도 잘 동작하는 함수나 메서드를 작성할 수 있다. 따라서 입력값이 제대로 동작하는지 반복적으로 이터레이션할 수 없는 인자를 받은 경우 TypeError를 발생시켜 인자를 거부할 수 있다.

```python
def normalize_defensive(numbers):
    if iter(numbers) is numbers:
        raise TypeError('Must supply a container')
    ...
```

다른 대안으로는 collections.abc 내장 모듈의 isinstance를 사용해 잠재적인 문제를 검사할 수 있는 Iterator 클래스를 제공한다.

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):
        raise TypeError('Must supply a container')
    ...
```

ㅤ

## 32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라.

리스트 컴프리헨션의 문제점은 입력 시퀀스와 같은 수의 원소가 들어 있는 리스트 인스턴스를 만들어낼 수 있다는 것이다. 이는 입력이 작으면 큰 문제가 되지 않지만, 입력이 커지면 메모리를 상당히 많이 사용하고 그로 인해 중단될 수 있다.

```python
value = [len(x) for x in open('my_file.txt')]

>>> print(value)
[100, 57, 15, 1, 12, 75, 5, 86, 89, 11]
```

이 때 파일이 아주 크거나 절대로 끝나지 않는 네트워크 소켓이라면 리스트 컴프리헨션을 사용하는 것이 문제가 될 수 있다. 이 문제를 해결하기 위해 파이썬은 **제너레이터 식**(generator expression)을 제공한다.

제너레이터 식은 리스트 컴프리헨션과 제너레이터를 일반화한 것이다. 제너레이터 식을 실행해도 출력 시퀀스 전체가 실체화되지는 않는다. 그 대신 제너레이터 식에 들어 있는 식으로부터 원소를 하나씩 만들어내는 이터레이터가 생성된다.

```python
it = (len(x) for x in open('my_file.txt'))

>>> print(it)
<generator object <genexpr> at 0x10a2d2a40>
```

반환된 제너레이터에서 다음 값을 가져오면(next 내장함수를 사용) 제너레이터 식에서 다음 값을 얻어올 수 있다. 제너레이터 식을 사용하면 메모리를 모두 소모하는 것을 염려할 필요 없이 원소를 원하는 대로 가져와 소비할 수 있다.

```python
>>> print(next(it))
4
>>> print(next(it))
3
```

제너레이터 식의 또 다른 강력한 특징은 두 제너레이터 식을 합성할 수 있다.

```python
it = (int(x) for x in open('my_file.txt'))
roots = ((x, x**0.5) for x in it)

>>> print(next(roots))
(15, 3.872983346207417)
```

이 이터레이터를 전진시킬 때마다 내부의 이터레이터도 전진되면서, 도미노처럼 연쇄적으로 루프가 실행돼 조건식을 평가하고 입력과 출력을 서로 주고 받는다. 이 모든 과정이 가능한 메모리를 효율적으로 사용하면서 이뤄진다.

ㅤ

## 33. yield from을 사용해 여러 제너레이터를 합성하라.

제너레이터에는 여러 장점이 있고, 제너레이터에서 발생할 수 있는 일반적인 문제를 해결할 방법이 있다.

예를 들어 제너레이터를 사용해 화면의 이미지를 움직이게 하는 프로그램이 존재하고 원하는 시각적인 효과를 얻기 위해 처음에 이미지가 빠르게 이동하고, 잠시 멈춘 다음, 다시 이미지가 느리게 이동해야 한다.

```python
def move(period, speed):
    for i in range(period):
        yield speed

def pause(delay):
    for i in range(delay):
        yield 0

def animate():
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

def render(delta):
    print(f"delta={delta:.1f}")

def run(func):
    for delta in func():
        render(delta)

>>> run(animate)
delta=5.0
delta=5.0
delta=5.0
delta=5.0
delta=0.0
delta=0.0
delta=0.0
delta=3.0
delta=3.0
```

이 코드의 문제점은 animate가 너무 반복적이라는 것이다. for 문과 yield 식이 반복되면서 잡음이 늘고 가독성이 줄어든다. 이 예제는 제너레이터를 겨우 3개만 내포시켰는데도 코드가 명확하지 못하다.

해법은 `yield from` 식을 사용하는 것으로, 이는 고급 제너레이터 기능으로 제어를 부모 제너레이터에게 전달하기 전에 내포된 제너레이터가 모든 값을 내보낸다.

```python
def animate_composed():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)
```

이전 프로그램과 같지만 코드가 더 명확하고 더 직관적이다. `yield from`은 근본적으로 파이썬 인터프리터가 대신 for 루프를 내포시키고 yield 식으로 처리하도록 만든다. 이로 인해 성능도 더 좋아진다.

ㅤ

## 34. send로 제너레이터에 데이터를 주입하지 말라.

yield 식을 사용하면 제너레이터 함수가 간단하게 이터레이션이 가능한 출력 값을 만들어낼 수 있다. 하지만 이렇게 만들어내는 채널은 단방향이다. 제너레이터가 데이터를 내보내면서 다른 데이터를 받아들일 때 직접 쓸 수 있는 방법이 없는 것처럼 보여, 이 때 양방향 통신이 있다면 많은 도움이 될 것이다.

예를 들어, 소프트웨어 라디오를 사용해 신호를 보낸다고 하자.

```python
import math

def wave(amplitude, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        yield output

def transmit(output):
    if output is None:
        print("Output is None")
    else:
        print("Output is {:.3f}".format(output))

def run(it):
    for output in it:
        transmit(output)

>>> run(wave(3.0, 8))
Output is 0.000
Output is 2.121
Output is 3.000
Output is 2.121
Output is 0.000
Output is -2.121
Output is -3.000
Output is -2.121
```

기본 파형을 생성하는 한 이 코드는 잘 동작한다. 하지만, 별도의 입력을 사용해 진폭을 지속적으로 변경해야 한다면 쓸모가 없다. 우리에게는 제너레이터를 이터레이션할 때마다 진폭을 변조할 수 있는 방법이 필요하다.

파이썬 제너레이터는 send 메서드를 지원한다. 이 메서드는 yield 식을 양방향 채널로 격상시켜주어 입력을 제너레이터에 스트리밍하는 동시에 출력을 내보낼 수 있다.

```python
def my_generator():
    received = yield 1
    print("Received: ", received)

it = iter(my_generator())
output = next(it)

>>> print("Output: ", output)
Output: 1
```

하지만, for 루프나 next 내장 함수로 제너레이터를 이터레이션하지 않고 send 메서드를 호출하면, 제너레이터가 재개될 때 yield가 send에 전달된 파라미터 값을 반환한다.

```python
it = iter(my_generator())
output = it.send(None)

>>> print("Output: ", output)
Output: 1

>>> try:
>>>     it.send('안녕!')
>>> except StopIteration:
>>>     pass
Received: 안녕!
```

이렇듯 제너레이터는 send 메서드를 사용해 데이터를 주입할 수 있다. 제너레이터는 send로 주입된 값을 yield 식을 반환하는 값을 통해 받으며, 이 값을 변수에 저장해 활용할 수 있다. send와 yield from 식을 함께 사용하면 제너레이터의 출력에 None이 나타날 수 있다. 그렇기에 합성할 제너레이터들의 입력으로 이터레이터를 전달하는 방식이 send를 사용하는 방식보다 낫다. send는 가급적 사용하지 말라.

ㅤ

## 35. 제너레이터 안에서 throw로 상태를 변화시키지 말라.

제너레이터는 안에서 Exception을 다시 던질 수 있는 throw 메서드가 존재한다. throw가 동작하는 방식은 간단하다. 어떤 제너레이터에 대해 throw가 호출되면 이 제너레이터는 값을 내놓은 yield로부터 평소처럼 제너레이터 실행을 계속하는 대신 throw가 제공한 Exception을 다시 던진다.

```python
class MyError(Exception):
    pass

def my_generator():
    yield 1
    yield 2
    yield 3

>>> it = my_generator()
>>> print(next(it))
1
>>> print(next(it))
2
>>> print(it.throw(MyError('test error')))
MyError: test error
```

throw를 호출하여 제너레이터에 예외를 주입해도, 제너레이터는 try/except 복합문을 사용해 마지막으로 실행된 yield 문을 둘러쌈으로써 이 예외를 잡아낼 수 있다.

```python
def my_generator():
    yield 1

    try:
        yield 2
    except MyError:
        print('catch MyError')
    else:
        yield 3

    yield 4

>>> it = my_generator()
>>> print(next(it))
1
>>> print(next(it))
2
>>> print(it.throw(MyError('test error')))
catch MyError
4
```

이 기능은 제너레이터와 제너레이터를 호출하는 쪽 사이에 양방향 통신 수단을 제공한다. 경우에 따라 이 양방향 수단이 유용할 수도 있다. 이렇게 throw 메서드를 사용하면 제너레이터가 마지막으로 실행한 yield 식의 위치에서 예외를 다시 발생시킬 수 있다. 하지마 throw를 사용하면 가독성이 나빠진다. 예외를 잡아내고 다시 발생시키는 데 준비 코드가 필요하여 내포 단계가 깊어지기 때문이다.

제너레이터에 예외적인 동작을 제공하는 더 나은 방법은 `__iter__` 메서드를 구현하는 클래스를 사용하면서 예외적인 경우에 상태를 전이시키는 것이다.

ㅤ

## 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라.

itertools 내장 모듈에는 이터레이터를 조직화하거나 사용할 때 쓸모 있는 여러 함수가 들어 있다. 복잡한 이터레이션 코드를 작성하고 있다는 사실을 깨달을 때마다 혹시 쓸만한 기능이 없는지 itertools 문서를 다시 살펴보라.

**여러 이터레이터 연결하기**

- chain : 여러 이터레이터를 하나의 순차적인 이터레이터로 합치고 싶을 때 chain을 사용한다.
- repeat : 한 값을 계속해서 반복해 내놓고 싶을 때 repeat을 사용한다. 이터레이터가 값을 내놓는 횟수를 제한하려면 repeat의 두 번째 인자로 최대 횟수를 지정하면 된다.
- cycle : 어떤 이터레이터가 내놓은 원소들을 계속 반복하고 싶을 때 사용한다.
- te : 한 이터레이터를 병렬적으로 두 번째 인자로 지정된 개수의 이터레이터로 만들고 싶을 때 사용한다.
- zip_longest : zip 내장함수의 변종으로 여러 이터레이터 중 짧은 쪽 이터레이터의 원소를 다 사용한 후 fillvalue로 지정한 값을 채워 넣어준다.

**이터레이터에서 원소를 거르기**

- islice : 이터레이터를 복사하지 않으면서 원소 인덱스를 이용해 슬라이싱하고 싶을 때 사용한다.
- takewhile : 이터레이터에서 주어진 술어(predicate)가 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다.
- dropwhile : takewhile의 반대로, 이터레이터에서 주어진 술어가 False를 반환하는 첫 번째 원소를 찾을 때까지 이터레이터의 원소를 건너뛴다.
- filterfalse : filter 내장 함수의 반대로 주어진 이터레이터에서 False가 반환하는 모든 원소를 돌려준다.

**이터레이터에서 원소의 조합 만들어내기**

- accumulate : 파라미터를 두 개 받는 함수를 반복 적용하면서 이터레이터 원소를 값 하나로 줄여준다.
- product : 하나 이상의 이터레이터에 들어 있는 아이템들의 데카르트 곱을 반환한다.
- permutations : 이터레이터의 원소들로부터 만들어낸 길이가 N인 순열을 돌려준다.
- combinations : 이터레이터가 내놓은 원소들로부터 만들어낸 길이 N인 조합을 돌려준다.
- combinations_with_replacement : combination와 같지만 원소의 반복을 허용한다. (중복 조합)
