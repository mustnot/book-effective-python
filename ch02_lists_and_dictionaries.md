# 2장. 리스트와 딕셔너리

많은 프로그램은 사람보다 기계가 하기에 더 적합한 반복적인 작업을 자동화하고자 만들어졌다. 파이썬에서는 이런 작업을 조직적으로 관리하는 가장 일반적인 방법은 리스트 타입을 쓰는 것이다.

리스트를 자연스럽게 보완할 수 있는 타입이 딕셔너리 타입이다. 딕셔너리 타입은 검색에 사용할 키와 키에 연관된 값을 저장한다. (일반적으로 해시 테이블(hash table)이나 연관 배열(associative array)이라 부르는 데이터 구조 안에 값을 저장한다) 딕셔너리는 (분활상환 복잡도로) 상수 시간에 원소를 삽입하고 찾을 수 있다. 따라서 동적인 정보를 관리하는 데는 딕셔너리가 가장 이상적이다.

## 11. 시퀀스를 슬라이싱하는 방법을 익혀라.

파이썬에는 시퀀스를 여러 조각으로 나누는 슬라이싱 구문이 있다. 슬라이싱을 사용하면 최소한의 노력으로 시퀀스에 들어 잇는 아이템의 부분집합에 쉽게 접근할 수 있다. 특정한 파이썬 클래스에도 `__getitem__`과 `__setitem__`으로 특별 메서드를 구현하면 된다.

슬라이싱 구문의 기본 형태는 리스트[시작:끝]이다. 여기서 시작 인덱스에 있는 원소는 슬라이스에 포함되지만, 끝 인덱스에 있는 원소는 포함되지 않는다.

- 리스트의 맨 앞부터 슬라이싱할 때는 시각적 잡음을 없애기 위해 0을 생략해야 한다.
- 리스트의 끝까지 슬라이싱할 때는 쓸데없이 끝 인덱스를 적지 말라.
- 리스트의 끝에서부터 원소를 찾고 싶을 때는 음수 인덱스를 사용하면 된다.

특히 슬라이싱할 때 리스트의 인덱스 범위를 넘어가는 시작과 끝 인덱스는 조용히 무시된다. 이런 동작 방식으로 Input 동작을 다룰 때 원하는 최대 길이를 쉽게 지정할 수 있다.

```python
first_twenty_items = a[:20]
last_twenty_items = a[-20:]
```

반면에 같은 인덱스에 직접 접근하면 예외가 발생한다.

```python
>>> a[20]
Traceback ...
IndexError: list index out of range
```

리스트를 슬라이싱한 결과는 완전히 새로운 리스트이며, 원래 리스트에 대한 참조는 그대로 유지된다. 즉 슬라이싱한 결과로 얻은 리스트를 변경해도 원래 리스트는 바뀌지 않는다.

```python
>>> a = ["a", "b", "c", "d", "e", "f", "g", "h"]
>>> b = a[3:]
>>> print(f"이전: {b}")
이전: ['d', 'e', 'f', 'g', 'h']
>>> b[1] = 99
>>> print(f"이후: {b}")
이후: ['d', 99, 'f', 'g', 'h']
>>> print(f"변화 없음: {a}")
변화 없음: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
```

슬라이싱에서 시작과 끝 인덱스를 모두 생략하면 원래 리스트를 복사한 새 리스트를 얻는다.

```python
>>> b = a
>>> b is a
True
>>> b = a[:]
>>> b is a
False
```

## 12. 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라.

기본 슬라이싱 외에 파이썬은 리스트[시작:끝:증가값]으로 일정한 간격을 두고 슬라이싱 할 수 있는 특별한 구문을 제공한다. 이를 스트라이드(stride)라고 한다.

스트라이드를 사용하면 시퀀스를 슬라이싱하면서 매 n번째 원소만 가져올 수 있다. 예를 들어 스트라이드를 사용해 리스트에서 인덱스가 짝수인 그룹과 홀수인 그룹을 쉽게 나눌 수 있다.

하지만, 스트라이드를 사용하는 구문은 종종 예기치 못한 동작이 일어나서 버그를 야기할 수 있다는 단점이 존재한다. 예를 들어 파이썬에서 바이트 문자열을 역으로 뒤집는 가장 일반적인 기법은 -1을 증가값으로 사용해 문자열을 슬라이싱하는 것이다.

```python
>>> x = b'mongoose'
>>> y = x[::-1]
>>> print(y)
b'esoognom'
```

유니코드 문자열에서도 물론 이런 기법이 잘 작동한다. 하지만, 유니코드 데이터를 UTF-8로 인코딩한 문자열에서는 이 코드가 작동하지 않는다.

그럼 -1 말고 다른 음수 증가값은 유용할까?

```python
>>> x = ["a", "b", "c", "d", "e", "f", "g", "h"]
>>> x[::2]
['a', 'c', 'e', 'g']
>>> x[::-2]
['h', 'f', 'd', 'b']
```

여기서 `::2`는 ‘시작부터 매 두 번째 원소를 선택한다’는 뜻이다. 또한, `::-2`는 ‘끝에서 시작해 앞으로 가면서 매 두 번째 원소를 선택한다’는 뜻이다. 그렇다면 `2::2`는 무슨 뜻일까? 그리고 `-2::-2`, `-2:2:-2`, `2:2:-2` 사이에는 어떤 차이가 있을까?

중요한 점은 슬라이싱 구문에 스트라이딩까지 들어가면 아주 혼란스럽다는 것이다. 각 괄호 안에 수가 세 개나 들어 있으면 코드 밀도가 너무 높아서 읽기 어렵다. 게다가 증가값에 따라 시작값과 끝값이 어떤 역할을 하는지 불분명하다. 이런 문제를 방지하기 위해 시작값이나 끝값을 증가값과 함께 사용하지 말 것을 권한다.

시작이나 끝 인덱스와 함께 증가값을 사용해야 한다면, 스트라이딩한 결과를 변수에 대입한 다음 슬라이싱하라.

```python
>>> y = x[::2]
>>> z = y[1:-1]
```

프로그램이 이 두 단계 연산에 필요한 시간과 메모리를 감당할 수 없다면, itertools 내장 모듈의 `islice` 메서드를 고려하라.

## 13. 슬라이싱보다는 나머지 모두 잡아내는 언패킹을 사용하라.

기본 언패킹의 한 가지 한계점은 언패킹할 시퀀스의 길이를 미리 알고 있어야한다는 것이다.

```python
>>> car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
>>> car_ages_descending = sorted(car_ages, reverse=True)
>>> oldest, second_oldest = car_ages_descending
Traceback ...
ValueError: too many values to unpack (expected 2)
```

이렇듯 기본 언패킹으로 리스트 맨 앞에서 원소를 두 개 가져오면 실행 시점에 예외가 발생한다. 파이썬을 처음 사용하는 사람은 이런 상황에서 인덱스와 슬라이싱을 자주 사용한다.

```python
>>> oldest = car_ages_descending[0]
>>> second_oldest = car_ages_descending[1]
>>> others = car_ages_descending[2:]
>>> print(oldest, second_oldest, others)
20 19 [15, 9, 8, 7, 6, 4, 1, 0]
```

물론 코드는 잘 작동하지만, 모든 인덱스와 슬라이스로 인해 시각적 잡음이 많다. 실제로 이런 식으로 시퀀스의 원소를 여러 하위 집합으로 나누면 1 차이 나는 인덱스로 인한 오류(off-by-one error)를 만들어내기 쉽다.

이런 상황을 더 잘 다룰 수 있도록 파이썬은 **별표 식**(starred expression)을 사용해 모든 값을 담는 언패킹을 할 수 있게 지원한다.

```python
>>> oldest, second_oldest, *others = car_ages_descending
>>> print(oldest, second_oldest, others)
20 19 [15, 9, 8, 7, 6, 4, 1, 0]
```

더 잛고, 읽기 쉽고, 여러 줄 사이에 인덱스 경계 값이 어긋나서 오류가 발생할 여지도 없다.

## 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라.

list 내장 타입에는 리스트의 원소를 여러 기준에 따라 정렬할 수 있는 sort 메서드가 들어 있다. 기본적으로 sort는 리스트의 내용을 원소 타입에 따른 자연스러운 순서를 사용해 오름차순으로 정렬한다.

```python
>>> numbers = [93, 86, 11, 68, 70]
>>> numbers.sort()
>>> print(numbers)
[11, 68, 70, 86, 93]
```

sort 메서드는 자연스럽게 순서를 정할 수 있다. 대부분의 내장 타입(문자열, 부동소수점 등)에서 잘 작동한다.

그럼, sort는 객체를 어떻게 처리할까?

```python
class Tool:
	def __init__(self, name, weight):
		self.name = name
		self.weight = weight

	def __repr__(self):
		return f"Tool({self.name}, {self.weight})"

tools = [
	Tool("수준계", 3.5),
	Tool("해머", 1.25),
	Tool("스크류드라이버", 0.5),
	Tool("끌", 0.25),
]

>>> tools.sort()
Traceback ...
TypeError: '<' not supported between instances of 'Tool' and 'Tool'
```

sort 메서드가 호출하는 객체 비교 특별 메서드가 정의돼 있지 않으므로 이런 타입의 객체는 정렬할 수 없다. 만약, 클래스에 정수와 마찬가지로 자연스러운 순서가 있어야하는 경우에는 필요한 특별 메서드를 정의하면 별도의 인자를 넘기지 않고 sort를 쓸 수 있다.

정렬에 사용하고 싶은 애트리뷰트가 객체에 들어 있는 경우가 많고, 이런 상황을 지원하기 위해 sort에는 key라는 파라미터가 존재한다. 여기서 key는 함수여야 한다.

```python
>>> print("before:", repr(tools))
before: [Tool(수준계, 3.5), Tool(해머, 1.25), Tool(스크류드라이버, 0.5), Tool(끌, 0.25)]
>>> tools.sort(key=lambda x: x.name)
>>> print("after: ", repr(tools))
after:  [Tool(끌, 0.25), Tool(수준계, 3.5), Tool(스크류드라이버, 0.5), Tool(해머, 1.25)]
```

name과 마찬가지로 weight을 기준으로 정렬도 가능하다.

이처럼 key로 전달된 람다 함수 내부에서는 원소 애트리뷰트에 접근하거나 인덱스를 써서 값을 얻거나, 동작하는 다른 모든 식을 사용할 수 있다.

만약, weight으로 먼저 정렬한 다음에 name으로 정렬하고 싶다면 어떻게 해야 할까? 파이썬에서 가장 쉬운 해법은 tuple 타입을 쓰는 것이다. 튜플은 기본적으로 비교 가능하며 자연스러운 순서가 정해져 있다. 이는 sort에 필요한 `__lt__` 정의가 들어 있다는 뜻이다.

```python
tools = [
	Tool("수준계", 3.5),
	Tool("해머", 3.5),
	Tool("스크류드라이버", 0.5),
	Tool("끌", 0.25),
]

>>> print("before:", repr(tools))
before: [Tool(수준계, 3.5), Tool(해머, 3.5), Tool(스크류드라이버, 0.5), Tool(끌, 0.25)]
>>> tools.sort(key=lambda x: (x.name, x.weight))
>>> print("after: ", repr(tools))
after:  [Tool(끌, 0.25), Tool(수준계, 3.5), Tool(스크류드라이버, 0.5), Tool(해머, 3.5)]
```

그렇다면, 정렬 조건이 동일한 값은 어떻게 될까? 파이썬에서는 이런 상황을 위해 **안정적인**(stable) 정렬 알고리즘을 제공한다. 즉 리스트 타입의 sort 메서드는 key 함수가 변환하는 값이 서로 같은 경우 리스트에 들어 있던 원래 순서를 그대로 유지해준다.

## 15. 딕셔너리 삽입 순서에 의존할 때는 조심하라.

파이썬 3.5 이전에는 딕셔너리에 대해 이터레이션을 수행하면 키를 임의의 순서로 돌려줬으며, 이터레이션 순서는 원소가 삽입된 순서와 일치하지 않았다.

```python
# python 3.5
baby_names = {
    "cat": "kitten",
    "dog": "puppy",
}

>>> print(baby_names)
{"dog": "puppy", "cat": "kitten"}
```

이런 일이 발생하는 이유는 예전의 딕셔너리 구현이 내장 hash 함수와 파이썬 인터프리터가 시작할 때 초기화되는 난수 seed를 사용하는 해시 테이블 알고리즘으로 만들어졌기 때문이다.

하지만, 파이썬 3.6부터는 딕셔너리가 삽입 순서를 보존하도록 동작이 개선됐고, 파이썬 3.7부터는 아예 파이썬 언어 명세에 이 내용이 포함됐다.

```python
# after python 3.7
baby_names = {
    "cat": "kitten",
    "dog": "puppy",
}

>>> print(baby_names)
{"cat": "kitten", "dog": "puppy"}
```

딕셔너리가 삽입 순서를 유지하는 방식은 이제 파이썬 언어 명세의 일부가 됐다.

<aside>
💡 **OrderedDict vs dict**
collections 내장 모듈에 삽입 순서를 유지해주는 OrderedDict가 존재했고, python 3.7 이후부터는 표준 dict의 동작과 비슷하기는 하지만 OrderedDict와 dict는 성능 특성이 많이 다르다.
만약 키 삽입과 popitem 호출이 빈번하다면 OrderedDict가 성능적으로 더 낫다.

</aside>

하지만 딕셔너리 처리 시 삽입 순서 동작이 항상 성립한다고 가정하면 안된다. 파이썬에서는 개발자가 list, dict 등의 표준 **프로토콜**(protocol)을 흉내내는 커스텀 컨테이너 타입을 쉽게 정의할 수 있다.

파이썬은 정적 타입 지정 언어가 아니기 때문에 대부분의 경우 코드는 엄격한 클래스 계층보다는 객체의 동작이 객체의 실질적인 타입을 결정하는 **덕 타이핑**에 의존하며 이로 인해 가끔 어려운 함정에 빠질 수 있다.

## 16. in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 get을 사용하라.

딕셔너리와 상호작용하는 세 가지 기본 연산은 키나 키에 연관된 값에 접근하고, 대입하고, 삭제하는 것이다. 딕셔너리의 내용은 동적이므로 어떤 키에 접근하거나 키를 삭제할 때 그 키가 딕셔너리가 없을 수 있다.

```python
counters = {
    'pumpernickel': 2,
    'sourdough': 1
}
```

여기서 카운터를 증가시키려면 먼저 키가 딕셔너리에 존재하는지 살펴본 후, 키가 없으면 디폴트 카운터 값인 0을 딕셔너리에 넣고 그 카운터를 증가시켜야한다. 하지만 이렇게 처리할 경우 키를 두 번 읽고, 키에 대한 값을 한 번 대입해야 한다.

```python
key = 'wheat'

if key in counters:
    count = counters[key]
else:
    count = 0

>>> counters[key] = count + 1
```

혹은 KeyError 예외를 활용하는 방법으로 키를 한 번만 읽고, 값을 한 번만 대입하면 되므로 더 효율적이다.

```python
try:
    count = counters[key]
except KeyError:
    count = 0
```

딕셔너리에서는 이런 식으로 키가 존재하면 그 값을 가져오고 존재하지 않으면 디폴트 값을 반환하는 흐름이 꽤 자주 일어난다. 그래서 dict 내장 타입에는 이런 작업을 수행하는 get 메서드가 들어 있다.

```python
counters[key] = counters.get(key, 0) + 1
```

만약, 딕셔너리에 지정된 값이 리스트처럼 더 복잡한 값이라면 어떻게 해야 할까?

```python
votes = {
    "baguette": ["Bob", "Alice"],
    "ciabatta": ["Coco", "Deb"],
}

key = "brioche"
who = "Elmer"

if key in votes:
		names = votes[key]
else:
		votes[key] = names = []

>>> names.append(who)
```

여기서 이중 대입문인 `votes[key] = names = []` 는 키 대입을 두 줄이 아니라 한 줄로 처리한다.

디폴트 값으로 빈 리스트를 딕셔너리에 넣고 나면 참조를 통해 리스트 내용을 변경할 수 있어, append를 호출한 다음 리스트를 다시 딕셔너리에 대입할 필요가 없다.

get을 이용한 방법으로 개선이 가능하다.

```python
if (names := votes.get(key)) is None:
		votes[key] = names = []

>>> names.append(who)
```

하지만 dict 타입은 이 패턴을 더 간단히 사용할 수 있게 해주는 setdefault 메서드를 제공한다. setdefault는 딕셔너리에 키를 사용해 값을 가져오려고 시도한다. 이 메서드는 키가 없으면 제공받은 디폴트 값을 키에 연관시켜 딕셔너리에 대입한 다음, 키에 연관된 값을 반환한다.

```python
names = votes.setdefault(key, [])
names.append(who)
```

get과 대입식을 사용하는 것보다 더 짧다. 하지만 이 접근 방법은 가독성이 그다지 좋지 않다. 메서드 이름인 setdefault는 이 메서드의 동작을 직접적으로 분명히 드러내지 못한다. 만약 이렇듯 setdefault를 사용하는 것이 딕셔너리 키를 처리하는 지름길인 경우, setdefault보다는 defaultdict를 사용하는 것이 충분할 수 있다.

## 17. 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라.

직접 만들지 않은 딕셔너러를 다룰 때 키가 없는 경우 처리하는 방법에는 여러 가지가 있다. get메서드를 이용하는 방법, in과 KeyError, 경우에 따라서는 setdefault가 가장 빠른 길일 수도 있다.

```python
visits = {
    'Mexico': {'Tulum', 'Puerto Vallarta'},
    'Japan': {'Hakone'},
}

>>> visits.setdefault('France', set()).add('Arles')

if (japan := visits.get('Japan')) is None:
    visits['Japan'] = japan = set()

>>> japan.add('Kyoto')
```

여기서 만약 직접 딕셔너리 생성을 제어할 수 있다면 어떨까? 예를 들어 클래스 내부에서 상태를 유지하기 위해 딕셔너리 인스턴스를 사용할 때가 이런 경우에 해당한다.

```python
class Visits:
    def __init__(self):
        self.data = {}

    def add(self, country, city):
        city_set = self.data.setdefault(country, set())
        city_set.add(city)
```

새로 만든 해당 클래스는 setdefault 호출의 복잡도를 제대로 감춰주며, 더 나은 인터페이스를 제공한다.

```python
>>> visits = Visits()
>>> visits.add('Russia', 'Yekaterinburg')
>>> visits.add('Tajikistan', 'Dushanbe')
>>> print(visits.data)
{'Russia': {'Yekaterinburg'}, 'Tajikistan': {'Dushanbe'}}
```

하지만 visits.add 메서드 구현은 여전히 setdefault라는 메서드 이름이 헷갈리기 때문에 코드 동작을 바로 이해하기 어렵다. 이 때 사용 가능한 클래스로 collections 내장 모듈에 있는 defaultdict 클래스가 존재한다. defaultdict 클래스는 키가 없을 때 자동으로 디폴트 값을 저장하여 이런 용법을 간단히 처리할 수 있게 해준다.

```python
from collections import defaultdict

class Visits:
    def __init__(self):
        self.data = defaultdict(set)

    def add(self, country, city):
        self.data[country].add(city)

>>> visits = Visits()
>>> visits.add('England', 'Bath')
>>> visits.add('England', 'London')
>>> print(visits.data)
defaultdict(<class 'set'>, {'England': {'London', 'Bath'}})
```

add 메서드가 아주 많이 호출되면 집합 생성에 따른 비용도 커지지만, 불필요한 set이 만들어지는 경우가 없다.

## 18. `__missing__` 을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라.

내장 dict 타입의 setdefault 메서드는 키가 없는 경우 짧은 코드로 처리할 수 있게 해준다. 그 중 대부분은 defaultdict 클래스를 이용하면 처리하는데 더 용이하다. 하지만 setdefault나 defaultdict 모두 사용하기 적당하지 않은 경우도 있다.

예를 들어 파일 시스템에 있는 SNS 프로필 사진을 관리하는 프로그램을 작성한다고 가정할 때, 필요할 때 파일을 읽고 쓰기 위해 프로필 사진의 경로와 열린 파일 핸들을 연관시켜주는 딕셔너리가 필요하다.

```python
pictures = {}
path = 'profile_1234.png'

if (handle := pictures.get(path)) is None:
    try:
        handle = open(path, 'rb')
    except OSError:
        print('cannot open', path)
        raise
    else:
        pictures[path] = handle

>>> handle.seek(0)
>>> image_data = handle.read()
```

위와 같은 로직은 굉장히 복잡하고, 로직이 복잡해질수록 블록 깊이가 더 깊어지는 단점이 있다.

이렇듯 내부 상태를 관리하려 한다면 프로필 사진의 상태를 관리하기 위해 defaultdict를 쓸 수 있다고 가정할 수 있다.

```python
from collections import defaultdict

def open_picture(profile_path):
    try:
        return open(profile_path, 'a+b')
    except OSError:
        print(f"Can't open {profile_path}")
        raise

>>> pictures = defaultdict(open_picture)
>>> handle = pictures[path]
>>> handle.seek(0)
>>> handle.read()
Traceback ...
TypeError: open_picture() missing 1 required positional argument: 'profile_path'
```

문제는 defaultdict 생성자에 전달한 함수는 인자를 받을 수 없다. 이렇듯 setdefault와 defaultdict 모두 필요한 기능을 제공하지 못하는 경우 파이썬은 다른 해법을 내장해 제공한다.

dict 타입의 하위 클래스를 만들고 `__missing__` 특별 메서드를 구현하면 키가 없는 경우를 처리하는 로직을 커스텀화할 수 있다.

```python
class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)
        self[key] = value
        return value

>>> pictures = Pictures()
>>> handle = pictures[path]
>>> handle.seek(0)
>>> image_data = handle.read()
```

picturers[path]라는 딕셔너리 접근에서 path가 딕셔너리에 없으면 `__missing__` 메서드가 호출된다. 이 메서드는 키에 해당하는 디폴트 값을 생성해 딕셔너리에 넣어준 다음 호출한 쪽에 그 값을 반환해야 한다.
