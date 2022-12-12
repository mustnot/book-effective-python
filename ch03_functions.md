# 3장. 함수

# 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라.

언패킹을 사용하면 함수가 둘 이상의 값을 반환할 수 있다.
```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)

    return minimum, maximum

lengths = [63, 73, 72, 60, 61, 71]

minimum, maximum = get_stats(lengths)
```
함수는 응답 값을 튜플에 넣어 반환하는 식으로 작동한다. 이 함수를 호출한 코드는 반환받은 튜플을 두 변수에 대입해서 언패킹한다.

그러나 이런 방식은 리턴하는 응답 값 수가 많으면 순서를 혼동하기 쉽고 나중에 알아내기 어려운 버그를 만들어 낼 수 있다. 그러므로 최대 3개까지만 언패킹할 수 있도록 한다. 더 많은 값을 언패킹해야 한다면 경량 클래스나 namedtuple을 사용하고 함수도 이런 값을 반환하게 만드는 것이 낫다.

# 20. None을 반환하기보다는 예외를 발생시켜라.

예외대신 `None`을 리턴하는 방식은 `False`와 동등하게 해석되어 오류를 야기하기 쉽다.
```python
def careful_devide(a,b):
    try:
        return a / b
    except ZeroDivisionError:
        return None
         
x,y = 0, 5
result = careful_divide(x, y)
if not result:
    print('잘못된 입력')

>>>
잘못된 입력 #실행되면 안되는데 실행됨.
```
이 코드는 응답 값이 0일 때와 None일 때 모두 동일하게 동작한다.

실수할 가능성을 줄이는 방법은 두 가지다.

1.반환 값에 성공여부를 추가하는 튜플을 반환한다.
```python
def careful_devide(a,b):
    try:
        return True, a / b
    except ZeroDivisionError:
        return False, None
```
그러나 이 방식은 첫번째 인자를 고려하지 않으면 똑같은 오류를 발생시킨다는 문제가 있다.

2.특별한 경우에는 결코 None을 반환하지 않는 것이다.
```python
def careful_devide(a,b):
    try:
        return a / b
    except ZeroDivisionError:
        raise ValueError('잘못된 입력')
```
이 접근법을 확장해서 타입 애너테이션을 사용해 볼 수도 있다. 함수의 반환 값이 항상 float이라고 지정할 수 있고, 그에 따라 None이 결코 반환되지 않음을 알릴 수 있다.
이제 입력, 출력, 예외적인 동작이 모두 명확해졌다.

# 21. 변수 영역과 클로저의 상호작용 방식을 이해하라.

숫자로 이뤄진 list를 정렬하되, 정렬한 리스트의 앞쪽에는 우선순위를 부여한 몇몇 수자를 위치시켜야 한다고 가정하자.
```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        reutnr (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)

>>>
[2, 3, 5, 7, 1, 4, 6, 8]
```
이 코드로 확인하는 파이썬 동작 방식은 다음과 같다.
1.파이썬이 클로저를 지원: 클로저란 자신이 정의된 영역 밖의 변수를 참조하는 함수다. 클로저로 인해 `helper` 함수가 `sort_priority` 함수의 `group` 인자에 접근할 수 있다.

2.파이썬에서 함수가 일급 시민 객체임: 일급 시민 객체는 함수를 직접 가리킬 수 있고, 변수에 대입하거나 다른 함수에 인자로 전달할 수 있으며, if문에서 함수를 비교 또는 반환하는 것 등이 가능함을 의미한다. 이 성질로 인해 sort 메서드는 클로저 함수를 key 인자로 받을 수 있다.

3.파이썬에는 시퀀스를 비교하는 구체적인 규칙이 있음: 파이썬은 시퀀스를 비교할 때 순서대로 원소를 비교해 두 값이 같으면 그 다음 원소로 넘어가며 모든 원소를 비교한다. 이로 인해 helper 클로저가 반환하는 튜플이 서로 다른 두 그룹을 정렬하는 기준 역할을 할 수 있다.

추가로, group 내 원소를 발견했음을 확인하는 flag를 추가한다고 가정하자.
```python
def sort_priority(values, group):
    found = False
    def helper(x):
        if x in group:
            found = True # 될까?
            return (0, x)
        reutnr (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print('발견:', found)
print(numbers)

>>>
발견: False #안된다.^^
[2, 3, 5, 7, 1, 4, 6, 8]
```
group 원소를 찾았음에도 found는 False이다. 왜 이럴까?

파이썬은 변수가 현재 영역에 이미 정의돼 있다면 그 변수의 값만 바뀐다. 하지만 변수가 현재 영역에 정의돼 있지 않다면 변수 대입을 변수 정의로 취급한다.
다시 말해, helper 함수의 클로저 안에서 이 대입문은 helper 영역 안에서 새로운 변수를 정의하는 것으로 취급되지, sort_priority2 안에서 기존 변수에 값을 대입하는 것으로 취급되지는 않는다. 이 문제는 영역 지정 버그(Scope bug)라고도 불린다.

이 문제를 해결하기 위해서는 `nonlocal`을 사용하면 된다. nonlocal 문이 지정된 변수에 대해서는 아래의 변수 참조 영역 규칙에 따라 대입될 변수의 영역이 결정된다.
1. 현재 함수의 영역
2. 현재 함수를 둘러싼 영역(현재 함수를 둘러싸고 있는 함수 등)
3. 현재 코드가 들어있는 모듈의 영역(global scope)
4. 내장 영역(built-in scope)

단, nonlocal은 모듈(3~4)수준 영역까지 변수 이름을 찾지는 않는다.
```python
def sort_priority(values, group):
    found = False
    def helper(x):
        nonlocal found
        if x in group:
            found = True # 될까?
            return (0, x)
        reutnr (1, x)
    values.sort(key=helper)
```
but, 간단한 함수 외에는 nonlocal을 사용하지 말라고 경고한다.

# 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라.

위치 인자를 가변적(*args)으로 받을 수 있다면 함수 호출이 더 깔끔해진다.
```python
def log(message, *values):
    if not values:
        print(message)
    else:
        values_str = ','.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는', 1, 2)
log('안녕')

>>>
내 숫자는: 1, 2
안녕
```
가변적 위치 인자를 받는 데는 2가지 문제점이 있다.

1. 제너레이터의 데이터를 전부 튜플로 변환하므로 메모리 부하가 발생할 수 있다. 가변적 위치 인자에 전달되는 데이터는 항상 튜플로 변환되는데, 제너레이터를 넘기면 제너레이터의 모든 원소를 얻기 위해 반복한다. 이렇게 만들어지는 튜플은 제너레이터의 모든 값을 포함하며 메모리를 많이 사용할 수 있다. 그러므로 전달하려는 데이터가 충분히 작은 경우에 사용할 수 있도록 한다.
2. 함수에 새로운 위치 인자를 추가하면 해당 함수를 호출하는 모든 코드를 변경해야 한다. 이런 가능성을 완전히 없애려면 *args를 사용하는 함수를 확장할 때는 키워드 기반의 인자만 사용해야 한다. 더 방어적으로는 타입 애너테이션을 사용해도 된다.

# 23. 키워드 인자로 선택적인 기능을 제공하라.
파이썬 함수에서는 키워드를 사용해 데이터를 전달할 수 있다. 이 때 키워드를 넘기는 순서는 고려하지 않는다.
```python
def remainder(number, divisor):
    return number % divisor

remainder(number=20, divisor=7)
remainder(number=7, divisor=20)
```

** 인자를 여러번 사용할 수도 있다. 다만 딕셔너리에 겹치는 키가 없어야 한다.
```python
my_kwargs = {
    'number': 20,
}

other_kwargs = {
    'divisor': 7,
}

assert remainder(**my_kwargs, **other_kwargs) == 6
```
아무 키워드 인자나 받는 함수를 만들고 싶다면 파라미터로 `**kwargs` 를 사용하라.

키워드 인자를 사용할 때의 장점은 다음과 같다.
1. 함수의 의미가 명확해진다.
2. 함수 정의에서 디폴트 값을 지정할 수 있다.
3. 하위 호환성을 제공하면서 함수 파라미터를 확장할 수 있다.

# 24. None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라.

키워드 인자의 기본값은 함수가 정의되는 시점에 단 한 번만 정의된다.
```python
def log(message, when=datetime.now()):
    print(f'{when}: {message}')

log('안녕')
sleep(1)
log('다시 안녕') #1초 뒤 다시 호출!

>>>
2022-12-11 11:29:11: 안녕!
2022-12-11 11:29:11: 다시 안녕! # but 시간 값이 안 바뀜.
```

동적인 값을 기본 값으로 하고 싶을 때는 보통 기본 값을 None으로 지정하고 실제 동작을 독스트링에 문서화하는 것이다.
```python
def log(message, when=None):
    """메세지와 타임스탬프를 로그에 넘긴다.

    Args:
        message: 출력할 메시지.
        when: 메시지가 발생한 시각. 디폴트값은 현재 시간이다.
    """
    if when is None:
        when = datetime.now()
    print(f'{when}: {message}')
```

인자가 가변적인 경우 특히 중요하다. 데이터 디코딩에 실패하면 디폴트로 빈딕셔너리를 반환한다고 가정하자.
```python
import json

def decode(data, default={})
    try:
        return json.loads(data)
    except:
        return default

foo = decode('wrong data')
foo['stupp'] = 5
bar = decode('wrong data')
bar['meep'] = 1

print('Foo:', foo)
print('Bar:', bar)

>>>
Foo: {'stuff': 5, 'meep': 1}
Bar: {'stuff': 5, 'meep': 1}
```
이 코드의 문제점 역시, 디폴트 값이 모듈을 로드하는 시점에 단 한 번만 정의되기 때문에 default에 지정된 딕셔너리가 모두 공유된다. 마찬가지로 디폴트 값을 None으로 지정하고 함수의 독스트링에 동작 방식을 기술하도록 한다.

# 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라.

복잡한 함수의 경우 키워드만 사용하는 인자를 사용해서 의도를 명확히 밝히는 것이 좋다.
```python 
# 위치 기반 인자
def safe_division(number, divisor, ignore_overflow, ignore_zero_division):
    pass

# 키워드 기반 인자
def safe_division_c(number, divisor, *, ignore_overflow=False, ignore_zero_division=False):
    pass
```
가운데 `*` 기호는 위치 인자의 마지막과 키워드만 사용하는 인자의 시작을 구분해준다. 이 때 키워드 인자를 사용하지 않으면 에러가 발생한다.

또는 인자의 이름이 바뀔 것을 고려하여 위치로만 지정하는 인자를 사용할 수도 있다. 위치로만 지정하는 인자는 반드시 위치만 사용해야 하고 키워드 인자로는 쓸 수 없다.
```python
def safe_division_d(number, divisor, /, *, ignore_overflow=False, ignore_zero_division=False):
    pass
```
인자 목록의 `/`기호는 위치로만 지정하는 인자의 끝을 표시한다.

# 26. functools.wrap을 사용해 함수 데코레이터를 정의하라.

데코레이터는 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다. 이는 데코레이터가 자신이 감싸고 있는 함수의 입력 인자, 반환 값, 함수에서 발생한 오류에 접근할 수 있음을 의미한다.
```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r} {kwargs!r}) -> {result!r} ')
        return result
    return wrapper


@trace
def fibonacci(n):
    if n in (0, 1):
        return n
    return (fibonacci(n-2) + fibonacci(n-1))
```
이 데코레이터 함수는 아래와 같은 의미이다.
```python
fibonacci = trace(fibonacci)
```

그러나 trace 함수는 자신의 본문에 정의된 wrapper 함수를 반환한다. 이런 동작은 인트로스펙션을 하는 도구에서 문제가 된다.
** 인트로스펙션: 실행 시점에 프로그램이 어떻게 실행되는지 관찰하는 것.

이 문제는 functools 내장 모듈에 정의된 wraps 도우미 함수를 사용하는 것이다.
```python
from functools import wraps

def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        ...
        return wrapper

@trace
def fibonacci(n):
    ...
```
wraps를 사용하면 함수의 모든 애트리뷰트를 제대로 복사해서 함수가 제대로 작동하도록 해준다.