# 06. 메타클래스와 애트리뷰트

**메타클래스**(meta-class)라는 이름은 어렴풋이 이 개념이 클래스를 넘어서는 것을 암시하는데, 메타클래스를 사용하면 파이썬의 class 문을 가로채서 클래스가 정의될 때마다 특별한 동작을 제공할 수 있다. 메타클래스의 기능으로는 동적으로 애트리뷰트 접근을 커스텀화해주는 내장 기능을 들 수 있다. 또한 파이썬의 객체지향적인 요소와 함께 어우러지면 간단한 클래스를 복잡한 클래스로 쉽게 변환할 수 있다.

이러한 강력한 기능에는 항상 함정이 뒤따른다. 예를 들어 동적인 애트리뷰트로 객체를 오버라이드할 경우 예기치 못한 부작용이 생길 수 있다. 그렇기 때문에 최소 놀람의 법칙(rule of least surprise)을 잘 따르고 잘 정해진 관용어로만 이런 기능을 사용하는 것이 중요하다.

ㅤ

## 44. 세터와 게터 메서드 대신 평범한 애트리뷰트를 사용하라

파이썬 개발자들은 클래스에 게터나 세터 메서드를 명시적으로 정의하곤 한다. 하지만 파이썬에서는 명시적인 세터나 게터 메서드를 구현할 필요가 전혀 없다. 대신 다음 코드와 같이 항상 단순한 공개 애트리뷰트로부터 구현을 시작하라.

```python
class Resistor:
		def __init__(self, ohms):
				self.ohms = ohms
				self.voltage = 0
				self.current = 0
```

나중에 애트리뷰트가 서정될 때 특별한 기능을 수행해야 한다면, 애트리뷰트를 `@poperty` 데코레이터와 대응하는 `setter` 애트리뷰터로 옮겨갈 수 있다.

```python
class VoltageResistance(Resistor):
		def __init__(self, ohms):
				super().__init__(ohms)
				self._voltage = 0

		@property
		def voltage(self):
				return self._voltage

		@voltage.setter
		def voltage(self, voltage):
				self._voltage = voltage
				self.current = self._voltage / self.ohms
```

property에 대해 setter를 지정하면 타입을 검사하거나 클래스를 프로퍼티에 전달된 값에 대한 검증을 수행할 수 있다. 

```python
class BoundedResistance(Resistor):
		def __init__(self, ohms):
				super().__init__(ohms)

		@property
		def ohms(self):
				return self._ohms

		@ohms.setter
		def ohms(self, ohms):
				if ohms <= 0:
						raise ValueError(f"저항 > 0이어야 합니다. 실제 값: {ohms}")
				self._ohms = ohms
```

이런 경우 잘못된 저항값을 대입하면 예외가 발생한다. 또한 생성자에 잘못된 값을 넘기는 경우에도 예외가 발생한다. 이렇듯 세터 메서드는 객체 생성이 끝나기 전에 즉시 저항을 검증하는 코드를 실행한다. 심지어 `@property`를 사용해 부모 클래스에 정의된 애트리뷰트를 불변으로 만들 수도 있다.

게터나 세터를 정의할 때 가장 좋은 정책은 관련이 있는 객체 상태를 `@property.setter` 메서드 안에서만 변경하는 것이다. 동적으로 모듈을 임포트하거나, 아주 시간이 오래 걸리는 도우미 함수를 호출하거나 등 호출하는 쪽에서 예상할 수 없는 부작용을 만들어내면 안된다.

`@property`의 가장 큰 단점은 애트리뷰트를 처리하는 메서드가 하위 클래스 사이에서만 공유될 수 있다는 것이다. 서로 관련이 없는 클래스 사이에 같은 구현을 공유할 수 없다. 하지만 파이썬은 재사용 가능한 프로퍼티 로직을 구현할 때는 물론 다른 여러 용도에도 사용할 수 있는 **디스크립터**(descriptor)를 제공한다.

ㅤ

## 45. 애트리뷰를 리팩터링하는 대신 @property를 사용하라

이 기법은 기존 클래스를 호출하는 코드를 전혀 바꾸지 않고도 클래스 애트리뷰트의 기존 동작을 변경할 수 있기 때문에 아주 유용하다.

```python
from datetime import datetime, timedelta

class Bucket:
		def __init__(self, period):
				self.period_delta = timedelta(seconds=period)
				self.reset_time = datetime.now()
				self.quota = 0

		def __repr__(self):
				return f"Bucket(quota={self.quota})"
```

리키 버킷 알고리즘은 시간을 일정한 간격으로 구분하고, 가용 용량을 소비할 때마다 시간을 검사해서 주기가 달라질 경우에는 이전 주기에 미사용한 가용 용량이 새로운 주기로 넘어오지 못하게 막는다.

```python
def fill(bucket, amount):
		now = datetime.now()
		if (now - bucket.reset_time) > bucket.period_delta:
				bucket.quota = 0
				bucket.reset_time = now
		bucket.quota += amount
```

가용 용량을 소비하는 쪽에서는 어떤 작업을 하고 싶을 때마다 먼저 리키 버킷으로부터 자신의 작업에 필요한 용량을 할당받아야 한다.

```python
def deduct(bucket, amount):
		now = datetime.now()
		if (now - bucket.reset_time) > bucket.period_delta:
				return False
		if bucket.quota - amount < 0:
				return False
		else:
				bucket.quota -= amount
				return True
```

이 클래스를 사용하려면 먼저 버킷에 가용 용량을 미리 저애진 할당량만큼 채워야 한다.

```python
bucket = Bucket(60)
fill(bucket, 100)

>>> print(bucket)
Bucket(quota=100)
```

이런 식의 구현은 버킷이 시작할 때 가용 용량이 얼마인지 알 수 없다는 것이다. 그렇기 때문에 deduct를 호출하는 쪽에서 자신이 차단된 이유가 Bucket에 할당된 가용 용량을 다 소진했기 때문인지, 이번 주기에 아직 버킷에 매 주기마다 재설정하도록 미리 정해진 가용 용량을 추가 받지 못했기 때문인지 알 수 있으면 좋을 것이다.

```python
class NewBucket:
		def __init__(self, period):
				self.period_data = timedelta(seconds=period)
				self.reset_time = datetime.now()
				self.max_quota = 0
				self.quota_consumed = 0

		def __repr__(self):
				return (f"NewBucket(max_quota={self.max_quota}, quota_consumed={self.quota_consumed})")

		@property
		def quota(self):
				return self.max_quota - self.quota_consumed
```

원래의 Bucket 클래스와 인터페이스를 동일하게 제공하기 위해 `@property` 데코레이터가 붙은 메서드를 사용해 클래스의 두 애트리뷰트에서 현재 가용 용량 수준을 매 번 계산하게 한다.

이렇듯 `@property` 를 사용하면 데이터 모델을 점진적으로 개선할 수 있다. 만약 프로퍼티 메서드를 과하게 사용하고 있다면, 클래스와 클래스를 사용하는 모든 코드를 리팩터링하는 것을 고려해야 한다.

ㅤ

## 46. 재사용 가능한 `@property` 메서드를 만들려면 디스크립터를 사용하라

`@property` 내장 기능의 가장 큰 문제점은 재사용성이다. 프로퍼티로 데코레이션하는 메서드를 같은 클래스에 속하는 여러 애트리뷰트로 사용할 수가 없고, 서로 무관한 클래스 사이에서 메서드를 재사용할 수도 없다.

예를 들어 두 클래스 사이에 공통된 프로퍼티가 존재하지만 서로 무관한 클래스일 경우에 프로퍼티를 재사용하기 위해 어쩔 수 없이 두 클래스 모두 메서드를 번거롭게 다시 작성해야 한다.

이런 경우에 파이썬에서는 더 나은 방법으로 **디스크립터**를 사용할 수 있다. 디스크립터 프로토콜은 파이썬 언어에서 애트리뷰트 접근을 해석하는 방법을 정의한다.

```python
class Grade:
		def __get__(self, instance, instance_type):
				...

		def __set__(self, instance, value):
				...

class Exam:
		math_grade = Grade()
		writing_grade = Grade()
		science_grade = Grade()
```

Grade 디스크립터를 다음과 같이 작성할 수 있다. 하지만 문제점이 존재하는데, 바로 메모리를 누수시킨다는 점이다. 이런 문제를 해결하기 위해 파이썬 weakref 내장 모듈을 사용할 수 있다. 이 모듈은 `WeakKeyDictionary` 라는 특별한 클래스를 제공하며, 객체를 저장할 때 일반적인 강한 참조 대신에 약한 참조를 사용한다는 점이다.

```python
from weakref import WeakKeyDictionary

class Grade:
		def __init__(self):
				self._values = WeakKeyDictionary() # change from {}

		def __get__(self, instance, instance_type):
				if instance is None:
						return self
				return self._values.get(instance, 0)

		def __set__(self, instance, value):
				if not (0 <= value <= 100):
						raise ValueError('점수는 0과 100 사이입니다.')
				self._values[instance] = value
```

ㅤ

## 47. 지연 계산 애트리뷰트가 필요하면, __getattr__, __getattribute__, __setattr__를 사용하라

파이썬에서는 `__getattr__` 이라는 특별 메서드를 사용해 동적 기능을 활용할 수 있다. 클래스 안에 `__getattr__` 메서드 정의가 있으면, 이 객체의 인스턴스 딕셔너리에서 찾을 수 없는 애트리뷰트에 접근할 때마다 호출한다.

```python
class LazyRecord:
		def __init__(self):
				self.exists = 5

		def __getattr__(self, name):
				value = f"{name}을 위한 값"
				setattr(self, name, value)
				return value
```

여기서 `__getattr__` 메서드는 `__dict__` 인스턴스 딕셔너리를 변경한다. 이러한 getattr의 기능은 스키마가 없는 데이터에 지연 계산으로 접근하는 등의 활용이 필요할 때 매우 유용하다.

예를 들어 데이터베이스 시스템 안에서 트랜잭션이 필요하다고 할 때, 사용자가 프로퍼티에 접근할 때 상응하는 데이터베이스에 있는 레코드가 유요한지, 그리고 트랜잭션이 여전히 열려 있는지 판단해야 한다. 그러나 기존 애트리뷰트를 확인하는 빠른 경로로 객체의 인스턴스 딕셔너리를 사용하기 때문에 `__getattr__` 훅으로는 안정적으로 만들 수 없다.

이 때 파이썬은 `__getattribute__` 라는 다른 object 훅을 제공한다.

```python
class ValidatingRecord:
		def __init__(self):
				self.exists = 5

		def __getattribute__(self, name):
				print(f"* 호출: __getattr__({name!r})")
				try:
						value = super().__getattribute__(name)
						print(f"* {name!r} 찾음, {value!r} 반환")
				except AttributeError:
						value = f"{name}을 위한 값"
						print(f"* {name!r}를 {value!r}로 설정")
						setattr(self, name, value)
						return value
```

존재하지 않는 프로퍼티에 동적으로 접근하는 경우에는 AttributeError 예외가 발생한다. 

파이선에서 일반적인 기능을 구현하는 코드가 hasattr 내장 함수를 통해 프로퍼티가 존재하는지 검사하는 기능과 getattr 내장 함수를 통해 프로퍼티 값을 꺼내오는 기능에 의존할 때도 있다.

`__getattribute__`와 `__setattr__`의 문제점은 원하든 원하지 않든 어떤 객체의 모든 애트리뷰트에 접근할 때마다 함수가 호출된다는 것이다.

```python
class BrokenDictionaryRecord:
		def __init__(self, data):
				self._data = {}

		def __getattribute__(self, name):
				print(f"* 호출: __getattribute__({name!r})")
				return self._data[name]

>>> data = BrokenRecord = BrokenDictionaryRecord({'foo': 3})
>>> data.foo
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
Traceback (most recent call last):
```

이런 경우 `super().__get_attribute__` 를 호출하여 해결할 수 있다.

```python
class BrokenDictionaryRecord:
		def __init__(self, data):
				self._data = {}

		def __getattribute__(self, name):
				print(f"* 호출: __getattribute__({name!r})")
				data_dict = super().__getattribute__("_data")
				return data_dict[name]
```

ㅤ

## 48. __init_subclass__를 사용해 하위 클래스를 검증하라

메타클래스의 가장 간단한 활용법 중 하나는 어떤 클래스가 제대로 구현됐는지 검증하는 것이다. 이럴 경우 새로운 하위 클래스가 정의될 때마다 이런 검증 코드를 수행하는 신뢰성 있는 방법을 제공하기 때문이다.

어떤 클래스 타입의 객체가 실행 시점에 생성될 때 검증 코드를 `__init__` 메서드 안에서 실행하는 경우도 종종 있다. 프로그램 시작 시 클래스가 정의된 모듈을 처음 임포트할 때와 같은 시점에 검증이 이뤄지기 때문에 예외가 훨씬 더 빨리 발생할 수 있다.

기본적인 경우 메타클래스는 `__new__` 메서드를 통해 자신과 연관된 클래스의 내용을 받는다.

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print(f"* 실행 {name}의 메타 {meta}.__new__")
        print("기반 클래스들: ", bases)
        print(class_dict)
        return type.__new__(meta, name, bases, class_dict)

class MyClass(metaclass=Meta):
    stuff = 123
    
    def foo(self):
        pass

class MySubclass(MyClass):
    other = 567
    
    def bar(self):
        pass

* 실행 MyClass의 메타 <class '__main__.Meta'>.__new__
기반 클래스들:  ()
{'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function MyClass.foo at 0x103d89bc0>}
* 실행 MySubclass의 메타 <class '__main__.Meta'>.__new__
기반 클래스들:  (<class '__main__.MyClass'>,)
{'__module__': '__main__', '__qualname__': 'MySubclass', 'other': 567, 'bar': <function MySubclass.bar at 0x103de2520>}
```

메타클래스는 클래스 이름, 클래스가 상속하는 부모 클래스들, 정의된 모든 클래스 애트리뷰트에 접근할 수 있다. 연관된 클래스가 정의되기 전에 해당 클래스의 모든 파라미터를 검증하려면 Meta.__new__에 기능을 추가해야 한다. 그러나 파이썬 3.6부터는 메타클래스를 정의하지 않고 같은 동작을 구현할 수 있는 더 단순한 구문이 존재한다.

```python
class BetterPolygon:
    sides = None

    def __init_subclass__(cls) -> None:
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError("Polygons need 3+ sides")

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180
```

코드가 훨씬 짧아졌고, 메타 클래스가 완전히 제거되었다. 그리고 메타클래스 방식의 또 다른 문제점은 클래스 정의마다 메타클래스를 단 하나만 지정할 수 있어 새롭게 정의된 값을 검증하기 위해서는 하위 클래스에 값을 설정해야한다. 이 때 `__init_subclass__`는 해당 문제를 해결할 수 있고, 심지어 다이아몬드 상속 같은 복잡한 경우에도 사용할 수 있다. 

ㅤ

## 49. __init_subclass__를 사용해 클래스 확장을 등록하라

메타클래스의 다른 용례로 프로그램이 자동으로 타입을 등록하는 것이 있다. 간단한 식별자를 이용해 그에 해당하는 클래스를 찾는 역검색을 하고 싶을 때 유용하다.

```python
import json

class Serializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({'args': self.args})

class Point2D(Serializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Point2D(%d, %d)' % (self.x, self.y)
```

이러한 접근 방식은 직렬화할 데이터의 타입(Point2D)을 미리 알고 있는 경우에만 사용이 가능하다는 문제가 있다. 이런 방식보다 JSON으로 직렬화할 클래스가 아주 많더라도 공통으로 쓰일 수 있는 하나만 존재하는 것이 이상적이다.

```python
import json

class BetterSerializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({
            'class': self.__class__.__name__,
            'args': self.args,
        })

    def __repr__(self):
        return '{}({})'.format(
            self.__class__.__name__,
            ', '.join(map(repr, self.args))
        )
```

그러면 클래스 이름을 객체 생성자로 다시 연결해주는 매핑을 유지할 수 있다. 매핑을 사용해 구현한 일반적인 deserialize 함수는 register_class를 통해 등록된 모든 클래스에 대해 잘 작동한다.

```python
registry = {}

def register_class(target_class):
    registry[target_class.__name__] = target_class

def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry[name]
    return target_class(*params['args'])
```

여기서 deserialize가 항상 제대로 동작하기 위해선 나중에 역직렬화할 모든 클래스에서 register_class를 호출해야 한다. 그러나 이 방식의 문제점은 register_class 호출을 잊어버릴 수 있다는 것이다.

이렇듯 BetterSerializable의 하위 클래스를 정의했음에도 불구하고, 클래스 정의문 다음에 register_class를 잊어버리면 기능을 제대로 활용할 수 없다. 이런 접근 방법은 실수 하기도 쉬운데, **클래스 데코레이터**도 방금 설명한 경우와 마찬가지로 호출을 잊어버리는 실수를 저지를 수 있다.

이런 실수를 방지하기 위해서 사용하기 좋은 접근 방법은 `__init_subclass__`를 사용하는 것이다.

```python
class BetterRegisteredSerializable(BetterSerializable):
    def __init_subclass__(cls) -> None:
        super().__init_subclass__()
        register_class(cls)
```

ㅤ

## 50. __set_name__으로 클래스 애트리뷰트를 표시하라

메타클래스를 통해 사용할 수 있는 유용한 기능이 한 가지 더 있다. 클래스가 정의된 후 클래스가 실제로 사용되기 이전인 시점에 프로퍼티를 변경하거나 표시할 수 있는 기능이다. 애트리뷰트가 포함된 클래스 내부에서 애트리뷰트 사용을 좀 더 자세히 관찰하고자 디스크립터를 쓸 때 이런 접근 방식을 활용한다.

```python
class Field:
    def __init__(self, name):
        self.name = name
        self.internal_name = '_' + self.name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

컬럼 이름을 Field 디스크립터에 저장하고 나면, `setattr` 내장 함수를 사용해 인스턴스별 상태를 직접 인스턴스 딕셔너리에 저장할 수 있고, 나중에 `getattr`로 인스턴스의 상태를 읽을 수 있다.

```python
class Customer:
    first_name = Field('first_name')
    last_name = Field('last_name')
    prefix = Field('prefix')
    suffix = Field('suffix')
```

하지만 이 클래스 정의는 중복이 많아 보인다. 클래스 안에서 왼쪽에 필드 이름을 이미 정의했는데, 굳이 같은 정보가 들어 있는 문자열을 다시 전달해야 할 이유가 없다.

이런 중복을 줄이기 위해 메타클래스를 사용할 수 있다. 메타클래스를 사용하면 class 문에 직접 훅을 걸어 class 본문이 끝나자마자 필요한 동작을 수행할 수 있다.

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, Field):
                value.name = key
                value.internal_name = '_' + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls

class DatabaseRow(metaclass=Meta):
    pass

class Field:
    def __init__(self):
        self.name = None
        self.internal_name = None

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

메타클래스와 새 DatabaseRow 기반 클래스와 새 Field 디스크립터를 사용한 결과 중복이 제거된다.

```python
class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()
```

이 접근 방법의 문제점은 DatabaseRow를 상속하는 것을 잊어버리거나 클래스 계층 구조로 인한 제약 때문에 어쩔 수 없이 DatabaseRow를 상속할 수 없는 경우, Field 클래스를 프로퍼티에 사용할 수 없다.

이런 문제를 해결하는 방법은 디스크립터에 `__set_name__` 특별 메서드를 사용하는 것이다.

```python
class Field:
    def __init__(self):
        self.name = None
        self.internal_name = None

    def __set_name__(self, owner, name):
        self.name = name
        self.internal_name = '_' + name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')
    
    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

ㅤ

## 51. 합성 가능한 클래스 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라

메타클래스를 사용하면 클래스 생성을 다양한 방법으로 커스텀화할 수 있지만, 여전히 메타클래스로 처리할 수 없는 경우가 있다. 

```python
# effective-python chapter 6 metaclass and attribute
from functools import wraps

def trace_func(func):
    if hasattr(func, 'tracing'):
        return func

    @wraps(func)
    def wrapper(*args, **kwargs):
        result = None
        try:
            result = func(*args, **kwargs)
            return result
        except Exception as e:
            result = e
            raise
        finally:
            print('%s(%r, %r) -> %r' % (func.__name__, args, kwargs, result))

    wrapper.tracing = True
    return wrapper
```

다음과 같이 이 데코레이터를 새 dict 하위 클래스에 속한 여러 특별 메서드에 적용할 수 있다.

```python
class TraceDict(dict):
    @trace_func
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    @trace_func
    def __setitem__(self, *args, **kwargs):
        return super().__setitem__(*args, **kwargs)
```

이 코드의 문제점은 꾸미려는 모든 메서드를 데코레이터를 써서 재정의해야 한다는 것이다. 이런 불필요한 중복으로 인해 가독성도 나빠지고 실수를 저지르기도 쉬워진다.

라이브러리를 이용한 방식 또한 존재하지만, 제약이 많이 존재한다. 이런 문제를 해결하고자 파이썬은 **클래스 데코레이터**를 지원한다. 

```python
def trace(klass):
    for key in dir(klass):
        value = getattr(klass, key)
        if isinstance(value, trace_types):
            wrapped = trace_func(value)
            setattr(klass, key, wrapped)
    return klass

@trace
class TraceDict(dict):
    pass
```

클래스를 확장하면서 합성이 가능한 방법을 찾고 있다면 클래스 데코레이터가 가장 적합한 도구이다.