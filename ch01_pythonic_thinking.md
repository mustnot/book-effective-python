# 1장. 파이썬답게 생각하기

## 1. 사용 중인 파이썬의 버전을 알아두라

```python
>>> import sys
>>> sys.version_info
sys.version_info(major=3, minor=10, micro=8, releaselevel='final', serial=0)
>>> sys.version
'3.10.8 (main, Oct 13 2022, 10:17:43) [Clang 14.0.0 (clang-1400.0.29.102)]'
```

## 2. PEP8 스타일 가이드를 따르라

PEP (Python Enhancement Proposal) #8 : 파이선 개선 제안은 파이썬 코드를 어떤 형식으로 작성할지 알려주는 스타일 가이드이다. 일관된 스타일의 코드는 더 친숙하게 접근하고 더 쉽게 읽을 수 있다.

### 공백

파이썬에서 공백은 중요한 의미가 있다. 파이썬 개발자들은 코드의 의미를 명확히 하는 데 공백이 미치는 영향이 특히 민감하다.

- 탭 대신 스페이스를 사용해 들여쓰기하라.
- 문법적으로 중요한 들여쓰기에는 4칸 스페이스를 사용하라.
- 라인 길이는 79개 문자 이하여야 한다.
- 긴 식을 다음 줄에 이어서 쓸 경우에는 일반적인 들여쓰기보다 4 스페이스를 더 들여써야 한다.
- 파일 안에서 각 함수와 클래스 사이에는 빈 줄을 두 줄을 넣어라.

```python
class StyleGuide1:
	pass

class StyleGuide2:
	pass
```

- 클래스 안에서 메서드와 메서드 사이에는 빈 줄을 한 줄만 넣어라.

```python
class StyleGuide:
	def method1(self):
		pass

	def method2(self):
		pass
```

- 딕셔너리에서 키와 콜론 사이에는 공백을 넣지 앟고, 한 줄 안에 키와 값을 같이 넣는 경우에는 콜론 다음에 스페이스를 하나 넣는다.

```python
{"key1": "value1", "key2": "value2"}
```

- 변수 대입에서 `=` 전후에는 스페이스를 하나씩만 넣는다.
- 타입 표기를 덧붙이는 경우에는 변수 이름과 콜론 사이에 공백을 넣지 않도록 주의하고, 콜론과 타입 정보 사이에는 스페이스를 하나 넣어라.

```python
string_value: str = "string"
```

### 명명 규약

PEP8은 여러 부분에 사용하는 이름을 어떻게 붙일지에 대한 고유 스타일을 제공한다. 이러한 스타일은 어떤 유형에 속하는지 쉽게 구분이 가능하다.

- 함수, 변수, 애트리뷰트(attribute)는 `lowercase_underscore`처럼 소문자와 밑줄을 사용한다.
- 보호돼야 하는 인스턴스 애트리뷰트는 일반적인 애트리뷰트 이름 규칙을 따르되, `_leading_underscore`처럼 밑줄로 시작한다.
- 비공개(private)(한 클래스 안에서만 쓰이고 다른 곳에서는 쓰면 안되는 경우) 인스턴스 애트리뷰트는 일반적인 애트리뷰트 이름 규칙을 따르되, `__leading_underscore`처럼 밑줄 두 개로 시작한다.
- 클래스(예외도 포함한다) `CapitalizedWord`처럼 여러 단어를 이어 붙이되, 각 단어의 첫 글자를 대문자로 만든다.
- 모듈 수준의 상수는 `ALL_CAPS`처럼 모든 글자를 대문자로 하고 단어와 단어 사이를 밑줄로 연결한 형태를 사용한다.
- 클래스에 들어 있는 인스턴스 메서드는 호출 대상 객체를 가리키는 첫 번째 인자의 이름으로 반드시 `self`를 사용해야 한다.
- 클래스 메서드는 클래스를 가리키는 첫 번째 인자의 이름으로 반드시 `cls`를 사용해야 한다.

### 식과 문

- 긍정적인 식을 부정하지 말고 (`if not a is b`) 부정을 내부에 넣어라 (`if a is not b`).
- 빈 컨테이너(container)나 시퀀스(sequence)(`[]`나 `‘’` 등)을 검사할 때는 길이를 0과 비교하지 말라. 빈 컨테이너나 시퀀스 값이 암묵적으로 False로 취급된다는 사실을 통해 `if not 컨테이너` 라는 조건문을 써라.
- 마찬가지로 비어 있지 않은 컨테이너나 시퀀스를 검사할 때도 길이를 비교하기 보다 암묵적으로 True로 취급된다는 사실을 활용하라.
- 한 줄짜리 if 문이나, for, while 루프, except 복합문을 사용하지 말라. 명확성을 위해 각 부분을 여러 줄에 나눠 배치하라.
- 식을 한 줄 안에 다 쓸 수 없는 경우, 식을 괄호로 둘러싸고 줄바꿈과 들여쓰기를 추가해서 읽기 쉽게 만들라.
- 여러 줄에 걸쳐 식을 쓸 때는 줄이 계속된다는 표시를 하는 `\` 문자보다는 괄호를 사용하라.

### 임포트

- import 문을 항상 파일 맨 앞에 위치시켜라.
- 모듈을 임포르할 때는 절대적인 이름(absolute name)을 사용하고, 현 모듈의 경로에 상대적인 이름을 사용하지 말라.
- 반드시 상대적인 경로로 임포트해야 하는 경우에는 `from . import foo` 처럼 명시적인 구문을 사용하라.
- 임포트를 적을 대는 표준 라이브러리 모듈, 서드 파티 모듈, 만든 모듈 순서로 섹션을 나눠라.
- 각 세션에서는 알파벳 순서로 모듈을 임포트하라.

## 3. bytes와 str의 차이를 알아두라

파이썬에서는 문자열 데이터의 시퀀스를 표현하는 bytes와 str 두 가지 타입이 존재한다.

```python
>>> a = b'h\x65ello'
>>> list(a)
[104, 101, 101, 108, 108, 111]
>>> a
b'heello'
>>>
```

위와 같이 bytes 타입의 인스턴스는 부호가 없는 8바이트 데이터가 그대로 들어간다.

```python
>>> a = 'a\u0300 propos'
>>> list(a)
['a', '̀', ' ', 'p', 'r', 'o', 'p', 'o', 's']
>>> a
'à propos'
```

str 인스턴스에는 사람이 사용하는 언어의 문자를 표현하는 유니코드 **코드 포인트**(code point)가 들어 있다.

중요한 사실은 str 인스턴스에는 직접 대응하는 이진 인코딩은 없고, bytes에는 텍스트 인코딩이 없다.

- 유니코드 데이터를 이진 데이터로 변환하려면 str의 encode 메서드를 호출해야 하고, 이진 데이터를 유니코드 데이터로 변환하려면 bytes의 decode 메서드를 호출해야 한다.
- 일반적으로 UTF-8이 시스템 디폴트 인코딩 방식이다.

파이썬 프로그램을 작성할 때 유니코드 데이터를 인코딩하고 디코딩하는 부분을 인터페이스 가장 먼 경계 지점에 위치시켜라. 이런 방식을 **유니코드 샌드위치**라고 부른다. 핵심 부분은 유니코드 데이터가 들어 있는 str을 사용해야 하고, 문자 인코딩에 대해 어떠한 가정도 해서는 안된다.

문자를 표현하는 타입이 둘로 나뉘어 있기 때문에 다음과 같은 두 가지 상황이 자주 발생한다.

- UTF-8(또는 다른 인코딩 방식)로 인코딩된 8비트 시퀀스를 그대로 사용하고 싶다.
- 특정 인코딩을 지정하지 않은 유니코드 문자열을 사용하고 싶다.

두 경우를 변환해주고 입력 값이 코드가 원하는 값과 일치하는지 확신하기 위해 두 가지 도우미 함수가 필요하다.

첫 번째 함수 : bytes나 str 인스턴스를 받아서 항상 str을 반환한다.

```python
def to_str(bytes_or_str):
	if isinstance(bytes_or_str, bytes):
		value = bytes_or_str.decode("utf-8")
	else:
		value = bytes_or_str
	return value

>>> repr(to_str(b"foo"))
'foo'
>>> repr(to_str("bar"))
'bar'
>>> repr(to_str(b"\xed\x95\x9c"))
'한'
```

두 번째 함수 : bytes나 str 인스턴스를 받아서 항상 bytes를 반환한다.

```python
def to_bytes(bytes_or_str):
	if isinstance(bytes_or_str, str):
		value = bytes_or_str.encode("utf-8")
	else:
		value = bytes_or_str
	return value

>>> repr(to_bytes(b"foo"))
b'foo'
>>> repr(to_bytes("bar"))
b'bar'
>>> repr(to_bytes("한글"))
b'\\xed\\x95\\x9c\\xea\\xb8\\x80'
```

이진 8비트 값과 유니코드 문자열을 파이썬에서 다룰 때 꼭 기억해야 할 두 가지 문제점이 있다.

1. bytes와 str이 똑같이 작동하는 것처럼 보이지만 각각의 인스턴스는 서로 호환되지 않기 때문에 전달 중인 문자 시퀀스가 어떤 타입인지 항상 잘 알고 있어야 한다는 점이다.
   - bytes와 str 인스턴스를 연산자에 섞어서 사용할 수 없다.
2. (내장 함수인 `open` 을 호출해 얻은) 파일 핸들과 관련된 연산들이 디폴트로 유니코드 문자열을 요구하고 이진 바이트 문자열을 요구하지 않는다는 것이다.
   - 이진 데이터를 파일에서 읽거나 파일에 쓰고 싶으면 항상 이진 모드(`rb`나 `wb`)로 파일을 열어라
   - 유데이터를 파일에서 읽거나 쓰고 싶을 때는 시스템 디폴트 인코딩에 주의하라.
   - `encoding` 파라미터를 명시적으로 전달하라.

## 4. C 스타일 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라

**형식화**(formatting)는 미리 정의된 문자열에 데이터 값을 끼워 넣어서 사람이 보기 좋은 문자열로 저장하는 과정이다. 문자열을 형식화하는 가장 일반적인 방법은 `%` 형식화 연산자를 사용하는 것이다. 이 연산자 왼쪽에 들어가는 미리 정의된 텍스트 템플릿을 **형식 문자열**이라고 부른다.

형식 문자열은 연산자 왼쪽에 있는 값을 끼워 넣을 자리를 표현하기 위해 `%d` 와 같은 **형식 지정자**(format specifier)를 사용한다. 형식 지정자 문법은 C의 `printf` 함수에서 비롯됐으며 파이썬에 이식됐다.

파이썬은 `%s, %x, %f, %d` 등 C 언어의 `printf`에 사용할 수 있는 대부분의 형식 지정자를 지원하고, 소수점 위치나 패딩, 채워 넣기, 좌우 정렬 등도 제공한다.

하지만, 파이썬에서 C 스타일 형식 문자열을 사용하는 데는 네 가지 문제점이 있다.

1. 형식화 식에서 오른쪽에 있는 `tuple` 내 데이터 값의 순서를 바꾸거나 타입을 바꾸면 타입 변환이 불가능하므로 오류가 발생할 수 있따.
2. 형식화를 하기 전에 값을 살짝 변경해야 한다면, 식을 읽기가 매우 어려워진다.
3. 형식화 문자열에서 같은 값을 여러 번 사용하고 싶다면 튜플에서 같은 값을 여러 번 반복해야한다.
   - 해결하기 위해 튜플 대신 딕셔너리를 사용해 형식화하는 기능이 추가되었다.
4. 형식화 식에 딕셔너리를 사용하면 문장이 번잡스러워진다.

### 내장 함수 format과 str.format

파이썬 3부터는 오래된 C 스타일 형식화 문자열보다 더 표현력이 좋은 **고급 문자열 형식화** 기능이 도입됐다. 이 기능은 format 내장 함수를 통해 파이썬 값에 사용할 수 있다.

```python
>>> a = 1234.5678
>>> formatted = format(a, ",.2f")
>>> print(formatted)
1,234.57

>>> b = "my 문자열"
>>> formatted = format(b, "^20s")
>>> print("*", formatted, "*")
*        my 문자열        *
```

str 타입에 새로 추가된 format 메서드를 호출하면 여러 값에 대해 한꺼번에 이 기능을 적용할 수 있다. 기본적으로 형식화 문자열의 위치 지정자는 format 메서드에 전달된 인자 중 순서상 같은 위치에 있는 인자를 가리킨다.

```python
>>> key, value = "my_var", "1.234"
>>> formatted = "{} = {}".format(key, value)
>>> print(formatted)
my_var = 1.234

>>> formatted = "{:<10} = {:.2f}".format(key, value)
>>> formatted
'my_var     = 1.23'
```

각 위치 지정자에는 콜론 뒤 형식 지정자를 붙여 넣어 문자열에 값을 넣을 때 어떤 형식으로 변환할지 정할 수 있다.

- 모든 형식 지정자에 대한 정보를 보고 싶으면, `help(”FORMATTING”)` 을 콘솔에 입력하면 된다.

- `__format__` 을 사용해 클래스별 형식화 방식을 커스텀화할 수 있다.

또한 위치 지정자 중괄호에 위치 인덱스, 즉 format 메서드에 전달된 인자의 순서를 표현하는 위치 인덱스를 전달할 수도 있다.

```python
>>> formatted = "{1} = {0}".format(key, value)
>>> print(formatted)
1.234 = my_var
```

하지만, 이 기능도 앞에서 설명한 네 번째 문제점인 키가 반복된 경우의 중복을 줄여주지 못한다.

이런 단점과 C 스타일 형식화 식의 문제점이 일부 남아 있으므로, 웬만하면 `str.format` 메서드를 사용하지 말 것을 권장한다.

### 인터폴레이션을 통한 형식 문자열

파이썬 3.6부터는 **인터폴레이션을 통한 형식 문자열**, 짧게는 **f-문자열**이 도입됐다. 새로운 언어 문법에서는 형식 문자열 앞에 f 문자를 붙여야한다.

f-문자열은 형식 문자열의 표현력을 극대화하고, 앞에서 설명한 네 번째 문제점인 형식 문자열에서 키와 값을 불필요하게 중복 지정해야 하는 경우를 없애준다. 또한 식 안에서 현재 파이썬 영역에서 사용할 수 있는 모든 변수를 자유롭게 참조할 수 있도록 허용함으로써 간결함을 제공한다.

```python
>>> formatted = f"{key} = {value}"
>>> print(formatted)
my_var = 1.234
```

그리고 format 함수의 형식 지정자 안에서 콜론 뒤에 사용할 수 있는 내장 미니 언어를 f-문자열에도 동일하게 사용할 수 있다. 역시 유니코드나 repr 문자열로 변환기능 역시 사용할 수 있다.

```python
>>> formatted = f"{key!r:<10} = {value:.2f}"
>>> print(formatted)
'my_var'   = 1.23
```

f-문자열을 사용한 형식화는 C 스타일 형식화 문자열에 `%` 연산자를 사용하는 경우나 str.format 메서드를 사용하는 경우보다 항상 짧다. f-문자열이 제공하는 표현력, 간결성, 명확성을 고려할 때 파이썬 프로그래머가 사용할 수 있는 형식화 옵션 중에서 최고다.

## 5. 복잡한 식을 쓰는 대신 도우미 함수를 작성하다.

파이썬은 문법이 간결하므로 상당한 로직이 들어가는 식도 한 줄로 매우 쉽게 작성할 수 있다.

```python
my_values = {"빨강": ["5"], "파랑": ["0"], "초록": [""]}
```

일부 질의 문자열 파라미터는 여러 값이 들어가 있고, 일부 파라미터는 값이 하나만 들어 있는 등 다양한 경우가 존재하는데, 이 경우 결과 딕셔너리에 `get` 메서드를 사용하면 상황에 따른 다른 값이 반환된다.

```python
>>> my_values.get("빨강")
["5"]
>>> my_values.get("초록")
[""]
>>> my_values.get("투명도")
None
```

이럴 때 파라미터가 없거나 비어 있을 경우 0이 디폴트 값으로 대입되면 좋을 것이다. 이런 로직을 처리하기 위해서는 **if 식**(expression)을 사용하면 좋을 것이다.

```python
>>> red = int(my_values.get("빨강", [""])[0] or 0)
```

하지만 이 코드 역시도 읽기 어렵고 시각적 잡음이 많아 코드를 이해하기 쉽지 않다. 더욱 간결하게 유지하면서 명확하게 표현할 수 있는 if/else 조건식(또는 삼항 조건식)이 있다.

```python
>>> red_str = my_values.get("빨강", [""])
>>> red = int(red_str[0]) if red_str[0] else 0
```

이 코드가 더 좋다. 또한 덜 복잡한 경우라면 if/else 조건식이 코드를 아주 명확하게 해줄 것이다. 하지만 앞의 예제는 다음과 같이 여러 줄로 나눠 쓴 완전한 if/else 문보다는 덜 명확하다.

```python
>>> green_str = my_values.get("초록", [""])
>>> if green_str[0]:
>>> 	green = int(green_str[0])
>>> else:
>>> 	green = 0
```

이 로직을 반복 적용하기 위해서는 꼭 도우미 함수를 작성해야 한다. 단지 두세 번에 불과할지라도.

```python
>>> def get_first_int(values, key, default=0):
>>> 	found = value.get(key, [""])
>>> 	if found[0]:
>>> 		return int(found[0])
>>> 	return default

>>> green = get_first_int(my_values, "초록")
```

이처럼 식이 복잡해지기 시작하면 바로 식을 더 작은 조각으로 나눠서 로직을 도우미 함수로 옮길지 고려해야 한다. 아무리 짧게 줄여 쓰는 것을 좋아한다 해도, 코드를 줄여 쓰는 것보다 가독성을 좋게 하는 것이 더 가치 있다.

## 6. 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라

파이썬에는 값으로 이뤄진 불변(immutable) 순서쌍을 만들어낼 수 있는 tuple 내장 타입이 있다. 가장 짧은 튜플은 딕셔너리의 키-값 쌍과 비슷하게 두 값으로 이뤄진다.

```python
>>> snack_calories = {
...     "감자칩": 140,
...     "팝콘": 80,
...     "땅콩": 190,
... }
>>> items = tuple(snack_calories.items())
>>> items
(('감자칩', 140), ('팝콘', 80), ('땅콩', 190))
```

튜플에 있는 값은 숫자 인덱스를 사용해 접근할 수 있다. 이렇게 튜플이 생성되면, 인덱스를 통해 새 값을 대입해서 튜플을 변경할 수 없다.

```python
>>> items[0] = "타래과"
Traceback ...
TypeError: 'tuple' object does not support item assignment
```

파이썬에는 **언패킹**(unpacking)(풀기) 구문이 존재한다. 언패킹 구문을 사용하면 한 문장 안에서 여러 값을 대입할 수 있다.

```python
>>> item = ("호박엿", "식혜")
>>> first, second = item
>>> print(first, "&", second)
호박엿 & 식혜
```

언패킹은 튜플 인덱스를 사용하는 것보다 시각적인 잡음이 적다. 리스트, 시퀀스, 이터러블(iterable) 안에 여러 계층으로 이터러블이 들어간 경우 등 다양한 패턴을 언패킹 구문을 사용할 수 있다.

언패킹의 용례 중 하나는 for 루프 또는 그와 비슷한 다른 요소(컴프리헨션(comprehension)이나 제너레이터 식)의 대상인 리스트의 원소를 언패킹하는 것이 있다.

```python
>>> snack = [("베이컨", 350), ("도넛", 240)]
>>> for name, calories in snack:
...     print(f"{name}의 칼로리는 {calories}이다")
...
베이컨의 칼로리는 350이다
도넛의 칼로리는 240이다
```

## 7. range보다는 enumerate를 사용하라

range 내장 함수는 어떤 정수 집합을 이터레이션하는 루프가 필요할 때 유용하지만, 값이 포함된 list에 대해 이터레이션을 수행하기에 사용하기에는 투박하다. 그 이유는 list의 길이를 알아야하고, 인덱스를 사용해 배열의 원소에 접근해야 하는 번거로움이 존재한다.

이런 문제를 해결하기 위해 enumerate 내장 함수를 제공한다. enumerate는 이터레이터를 지연 계산 제너레이터(lazy generator)로 감싸, 루프 인덱스와 이터레이터의 다음 값으로 이뤄진 쌍을 넘겨준다.

다음은 enumerate 내장함수와 next 내장함수를 사용해 다음 원소를 가져오는 예이다.

```python
>>> flavor_list = ["바닐라", "초콜릿", "피칸", "딸기"]
>>> it = enumerate(flavor_list)
>>> print(next(it))
(0, "바닐라")
>>> print(next(it))
(1, "초콜릿")
```

enumerate가 넘겨주는 각 쌍을 for 문에서 간결하게 언패킹할 수 있다.

```python
>>> for index, flavor in enumerate(flavor_list):
...     print(f"{index+1}: {flavor}")
...
1: 바닐라
2: 초콜릿
3: 피칸
4: 딸기
```

## 8. 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라

파이썬에서는 관련된 객체가 들어 있는 리스트를 다루는 경우가 자주 있다. 리스트 컴프리헨션을 사용하면 소스 list에서 새로운 list를 파생시키기 쉽다.

```python
>>> names = ["Cecilia", "남궁민수"]
>>> counts = [len(n) for n in names]
>>> print(counts)
[7, 4]
```

만들어진 list의 각 원소는 소스 list에서 같은 인덱스 위치에 있는 원소와 관련이 있다. 두 리스트를 동시에 이터레이션할 경우 name 소스 리스트의 길이를 사용해 이터레이션할 수 있다.

```python
>>> longest_name = None
>>> max_count = 0
>>>
>>> for i in range(len(names)):
...		count = counts[i]
...   if count > max_count:
...     longest_name = names[i]
...     max_count = count
...
>>> print(longest_name)
Cecilia
```

문제는 이 루프가 시각적으로 잡음이 많다. 인덱스를 사용해 names와 counts의 원소를 찾는 과정이 코드를 읽기 어렵게 만든다. 이런 코드를 깔끔하게 만들 수 있도록 파이썬은 zip이라는 내장 함수를 제공한다. zip은 둘 이상의 이터레이터를 지연 계산 제너레이터를 사용해 묶어준다.

```python
>>> for name, count in zip(names, counts):
... 	if count > max_count:
... 		longest_name = name
... 		max_count = count
```

하지만 입력 이터레이터의 길이가 서로 다를 때는 zip은 자신이 감싼 이터레이터 중 어느 하나가 끝날 때까지 튜플을 내놓는다. 즉 가장 짧은 이터레이터를 기준으로 동작한다. 그렇기에 만약 zip에 전달한 리스트의 길이가 같지 않을 것으로 예상한다면 itertools 내장 모듈에 있는 zip_longest를 대신 사용하는 것을 고려하라.

```python
>>> import itertools
>>>
>>> for name, count in itertools.zip_longest(names, counts):
... 	print(f"{name}: {count}")
```

- zip_longest는 존재하지 않는 값을 자신에게 전달된 fillvalue로 대신한다. 디폴트는 None이다.

## 9. for나 while 루프 뒤에 else 블록을 사용하지 말라.

파이썬 루프는 대부분의 다른 프로그래밍 언어가 제공하지 않는 기능을 제공한다. 파이썬에서는 루프가 반복수행하는 내부 블록 다음에 else 블록을 추가할 수 있다.

```python
>>> for i in range(3):
... 	print("Loop", i)
... else:
... 	print("else block!")
...
Loop 0
Loop 1
Loop 2
else blcok!
```

else 블록은 루프가 끝나자마자 실행된다. 여기서 else는 “이 블록 앞의 블록이 실행되지 않으면 이 블록을 실행하라”는 뜻으로 try/except 문에서 except도 마찬가지로 “이 블록 앞의 블록을 시도하다가 예외가 발생하면 이 블록을 실행하라”는 뜻이다. 결국 else 부분은 “루프가 정상적으로 완료되지 않으면 이 블록을 실행하라”는 뜻으로 가정하기 쉽다. 그러나 실제 else 블록은 완전히 반대로 동작한다.

```python
>>> for i in range(3):
...     print("Loop", i)
...     if i == 1:
...         break
... else:
...     print("else block!")
...
Loop 0
Loop 1
```

실제로 루프 안에서 break 문을 사용하면 else 블록이 실행되지 않는다. 또 놀라운 점은 빈 시퀀스에 대한 루프를 실행하면 else 블록이 바로 실행된다.

```python
>>> for x in []:
...     print("이 줄은 실행되지 않음")
... else:
...     print("for else block!")
...
for else block!
```

또한, while 루프의 조건이 처음부터 False인 경우에도 else 블록이 바로 실행된다.

```python
>>> while False:
...     print("이 줄은 실행되지 않음")
... else:
...     print("while else block!")
...
while else block!
```

이런 식의 동작은 루프를 사용해 검색을 수행할 경우, 루프 바로 뒤에 있는 else 블록이 그와 같이 동작해야 유용하기 때문이다. 이러한 방법보다 다음과 같은 방식을 사용할 것이다.

1. 원하는 조건을 찾자마자 빠르게 함수를 반환하는 방식
2. 루프 안에서 원하는 대상을 찾았는지 나타내는 결과 변수를 도입하는 것이다.

## 10. 대입식을 사용해 반복을 피하라

대입식은 영어로 assignment expression이며 **왈러스 연산자**(walrus operator)라고도 부른다. 이 대입식은 파이썬 언어에서 고질적인 코드 중복 문제를 해결하고자 3.8 버전부터 새롭게 도입된 구문으로 바다코끼리 문법이라고 부르기도 한다.

```python
>>> fresh_fruit = {
...     "사과": 10,
...     "바나나": 8,
...     "레몬": 5,
... }
>>> if count := fresh_fruit.get("레몬", 0):
...     make_lemonade(count)
... else:
...     out_of_stock()
...
```

이렇듯 대입식에서는 왈러스 연산자를 사용해 하나의 식 안에서 변수 이름에 값을 대입하면서 이 값을 평가할 수 있고, 중복을 줄일 수 있다. 그리고 대입식이 더 큰 식의 일부분으로 쓰일 때는 괄호로 둘러싸야한다.
