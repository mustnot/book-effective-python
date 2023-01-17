# 5장. 클래스와 인터페이스

## 37. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

파이썬 내장 딕셔너리 타입을 사용하면 객체의 생명 주기 동안 동적인 내부 상태를 잘 유지할 수 있다. 여기서 동적이라는 말은 미리 알 수 없는 식별자들을 잘 유지해야 한다는 뜻이다. 예를 들어 학생들의 점수를 기록하는데, 학생의 이름은 미리 알 수 없는 상황이라고 하자. 이 때 학생 별로 미리 정의된 애트리뷰트를 사용하는 대신 딕셔너리에 이름을 저장하는 클래스를 정의할 수 있다.

```python
class SimpleGradebook:
    def __init__(self):
        self._grades = {}
    
    def add_student(self, name):
        self._grades[name] = []
    
    def report_grade(self, name, score):
        self._grades[name].append(score)
    
    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)

book = SimpleGradebook()
book.add_student('아이작 뉴턴')
book.report_grades('아이작 뉴턴', 90)
book.report_grades('아이작 뉴턴', 95)
book.report_grades('아이작 뉴턴', 85)

print(book.average_grade('아이작 뉴턴'))
>>>
90.0
```

아직까지는 간단하지만 과하게 확장하면 깨지기 쉬운 코드를 작성할 위험이 있다. 예를 들어 전체 성적이 아니라 과목별 성적을 리스트로 저장하고 싶다고 하자. _grades를 변경해서 학생 이름 key가 value로 성적 dict를 갖게하여 구현한다. 과목이 없는 경우를 처리하기 위해 value는 defaultdict를 사용한다.

```python
from collections import defaultdict

class BySubjectGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name] # {'수학' : [75, 65], '체육' : [90, 95]}
        grade_list = by_subject[subject]
        grade_list.append(grade)

    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count

book = SimpleGradebook()
book.add_student('아이작 뉴턴')
book.report_grades('아이작 뉴턴', '수학', 75)
book.report_grades('아이작 뉴턴', '수학', 65)
book.report_grades('아이작 뉴턴', '체육', 90)
book.report_grades('아이작 뉴턴', '체육', 95)

print(book.average_grade('아이작 뉴턴'))
>>>
81.25
```

여기까지는 감당할만 하다. 그러나 이번엔 성적에 가중치가 들어간다고 해보자. 점점 더 늘어나는 요구 사항에 딕셔너리, 리스트, 튜플 계층이 자꾸 늘어난다면 유지 보수의 '악몽'에 들어가는 셈이다. 코드에서 값을 관리하는 부분이 점점 복작해지고 있음을 깨달은 즉시 해당 기능을 클래스로 분리해야 한다.

### 클래스를 활용해 리팩터링하기

리팩터링을 취할 수 있는 접근 방법은 다양하다. 여기서는 먼저 의존 관계 트리의 맨 밑바닥(점수와 가중치의 튜플)을 점수를 표현하는 클래스로 옮겨갈 수 있다. 하지만 이런 단순한 정보를 표현하는 클래스를 따로 만들면 너무 많은 비용이 드는 것 같다. 게다가 점수는 불변 값이기 때문에 튜플이 더 적당해 보인다. 다음 코드에서는 리스트 안에 점수를 저장하기 위해 튜플을 사용한다.
```python
grades = []
grades.append((95, 0.45))
grades.append((85, 0.55))

total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
average_grade = total / total_weight
```

그러나 이 코드의 문제점은 튜플에 저장된 내부 원소에 위치(index)를 사용해 접근한다는 것이다. 예를 들어 선생님이 메모를 추가해야 해서 점수와 연관시킬 정보가 더 늘어났다고 하자. 이런 경우 모든 원소가 3개인 튜플을 제대로 처리하도록 변경해야 한다. 즉, 특정 인덱스를 무시하기 위해 `_`를 더 많이 사용해야 한다는 뜻이다.
```python
grades = []
grades.append((95, 0.45, 'good'))
grades.append((85, 0.55, 'so so'))

total = sum(score * weight for score, weight, _ in grades) 
total_weight = sum(weight for _, weight, _ in grades) # memo에 해당하는 _ 가 늘어남.
average_grade = total / total_weight
```

원소가 3개 이상인 튜플을 사용한다면 다른 접근 방법을 생각해봐야 한다.

collection 내장 모듈에 있는 namedtuple 타입이 이런 경우에 딱 들어맞는다. namedtuple을 사용하면 작은 불변 데이터 클래스를 쉽게 정의할 수 있다.
```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))

a = Grade(100, 0.45)
a.score
>>>
100
a.weight
>>>
0.45
```

이제 일련의 점수를 포함하는 단일 과목을 표현하는 클래스를 작성할 수 있다.

```python
class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))
    
    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total += grade.weight
        return total / total_weight
```

다음으로 한 학생이 수강하는 과목들을 표현하는 클래스를 작성할 수 있다.

```python
class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)
    
    def get_subject(self, name):
        return self._subjects[name]
    
    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count
```
마지막으로 모든 학생을 저장하는 컨테이너를 만들 수 있다.

```python
class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)
    
    def get_student(self, name):
        return self._students[name]
```

아래와 같이 사용할 수 있다.
```python
book = Gradebook()
albert = book.get_student('알버트 아인슈타인')
math = albert.get_subject('수학')
math.report_grade(75, 0.05)
math.report_grade(65, 0.15)
math.report_grade(70, 0.80) # 아인슈타인 수학 못하네..
gym = albert.get_subject('체육')
gym.report_grade(100, 0.40)
gym.report_grade(85, 0.60)
print(albert.average_grade())
>>>
80.25
```

## 38. 간단한 인터페이스의 경우 클래스 대신 함수를 받아라.

함수는 클래스보다 정의하거나 기술하기가 더 쉬우므로 훅으로 사용하기에는 함수가 이상적이다. 또한 파이썬은 함수를 일급 시민 객체로 취급하기 때문에 훅으로 사용할 수 있다. 함수가 일급 시민 객체라는 말은 함수나 메서드를 다른 함수에 넘기거나 변수 등으로 참조할 수 있다는 의미다.

defaultdict 클래스의 동작을 사용자 정의하고 싶다고 하자.

```python
class CountMissing:
    def __init__(self):
        self.added = 0
    
    def missing(self):
        self.added += 1
        return 0
```
파이썬에서는 일급 함수를 사용해 객체에 대한 CountMissing.missing 메서드를 직접 defaultdict의 디폴트 값 훅으로 전달할 수 있다.
```python
current = {'초록': 12, '파랑': 3}
increments = [
    ('빨강', 5),
    ('파랑', 17),
    ('주황', 9),
]

counter = CounterMissing()
result = defaultdict(counter.missing, current)
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

이 코드에서는 CountMissing 클래스의 목적이 무엇인지 분명히 알기는 어렵다. 누가 CountMissing 객체를 만들까? 누가 missing 메서드를 호출할까? defaultdict 와 사용하는 예제를 보기 전까지는 이 클래스는 수수께끼일 뿐이다.

이런 경우를 더 명확히 표현하기 위해 파이썬에서는 클래스를 __call__ 특별 메서들르 정의할 수 있다. __call__을 사용하면 객체를 함수처럼 호출할 수 있다. 그리고 __call__이 정의된 클래스의 인스턴스에 대해 callable 내장 함수를 호출하면, 다른 일반 함수나 메서드와 마찬가지로 True가 반환된다. 이런 방식으로 정의된 객체를 호출 가능(callable) 객체라고 부른다.

```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0
    
    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
assert counter() == 0
assert callable(counter)

counter = BetterCountMissing()
result = defaultdict(counter, current) # __call__ 에 의존함.
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

이 코드가 CountMissing.missing을 사용한 코드보다 깔끔하다. 무엇보다 좋은 점은 defaultdict가 __call__ 내부에서 어떤 일이 벌어지는지에 대해 전혀 알 필요가 없다는 사실이다. 이처럼 파이썬은 단순한 함수 인터페이스를 만족할 수 있는 여러가지 방법을 제공하며, 원하는 목적에 가장 적합한 방식을 선택하면 된다.

## 39. 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 사용하자

파이썬에서는 객체뿐 아니라 클래스도 다형성을 지원한다. 다형성을 사용하면 계층을 이루는 여러 클래스가 자신에게 맞는 유일한 메서드 버전을 구현할 수 있다. 
맵리듀스 구현을 예시로 살펴보자.

```python
# input data
# 입력 데이터를 읽는 추상 클래스
class InputData:
    def read(self):
        raise NotImplementedError

class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path
    
    def read(self):
        with open(self.path) as f:
            return f.read()

# worker
# 맵리듀스 작업을 수행하는 추상 클래스
class Worker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    
    def map(self):
        raise NotImplementedError
    
    def reduce(self, other):
        raise NotImplementedError

class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')
    
    def reduce(self, other):
        self.result += other.result

```
다음은 도우미 함수를 사용해 객체를 직접 만들고 연결하자. 다음 코드는 디렉터리의 목룍을 얻어서 그 안에 들어있는 파일마다 PathInputData 인스턴스를 만든다.

```python
import os

def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))
```

다음으로 generate_inputs를 통해 만든 InputData 인스턴스를 사용해 LineCountWorker 인스턴스를 만든다.

```python
def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers
```
이제 Worker 인스턴스를 만들었으니 map과 reduce를 호출해 결과를 만들어낸다.

```python
from threading import Thread

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join()
    
    first, *rest = workers
    for worker in rest:
        first.reduce(worker)
    return first.result
```
마지막으로 한 함수 안에서 각 단계를 실행한다.
    
```python
def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
```

깔끔하지만 이 함수의 문제점은 뭘까? 바로 함수가 전혀 제너릭하지 않다는 것이다. 다른 Input 함수나 Worker 하위 클래스를 사용하고 싶다면 각 하위 클래스에 맞게 함수를 수정해야 한다. 이런 문제를 해결하려면 @classmethod 다형성을 사용하면 된다.

```python
class GenericInputData:
    def read(self):
        raise NotImplementedError
    
    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(GenericInputData):
    ...

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    
    def map(self):
        raise NotImplementedError
    
    def reduce(self, other):
        raise NotImplementedError
    
    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers
```

마지막으로 mapreduce 함수가 create_workers를 호출하도록 수정한다.

```python
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
```

## 40. super로 부모 클래스를 초기화해라.

아래의 다중 상속은 예기치 않은 결과를 가져온다.

```python
class MyBaseClass:
    def __init__(self, value):
        self.value = value

class TimesSeven(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value *= 7

class PlusNine(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value += 9

class ThisWay(TimesSeven, PlusNine):
    def __init__(self, value):
        TimesSeven.__init__(self, value)
        PlusNine.__init__(self, value)

foo = ThisWay(5)
print('Should be (5 * 7) + 9 = 44 but is', foo.value)
>>>
Should be (5 * 7) + 9 = 44 but is 14
```
TimesSeven과 PlusNine의 부모 클래스인 MyBaseClass의 생성자를 두 번 호출하여 self.value가 중간에 한 번 초기화되어 예기치않은 결과를 가져왔다.
이러한 문제를 해결하기 위해 파이썬은 super 내장 함수를 제공한다. super를 사용하면 다이아몬드 계층의 공통 상위 클래스를 단 한 번만 호출하도록 보장한다.

```python

class TimesSevenCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value *= 7

class PlusNineCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9

class GoodWay(TimesSevenCorrect, PlusNineCorrect):
    def __init__(self, value):
        super().__init__(value)
    
foo = GoodWay(5)
print('Should be 7 * (5 + 9) = 98 and is', foo.value)
>>>
Should be 7 * (5 + 9) = 98 and is 98
```

호출 순서는 클래스에 대한 MRO 정의를 따른다.
```
>>> GoodWay.mro()
[<class '__main__.GoodWay'>,
 <class '__main__.TimesSevenCorrect'>,
 <class '__main__.PlusNineCorrect'>,
 <class '__main__.MyBaseClass'>,
 <class 'object'>]
```
super에 파라미터는 기본적으로 __class__와 self를 넣어준다. 따라서 super를 호출할 때 파라미터를 넣어주지 않으면 super는 자신의 부모 클래스를 찾아서 호출한다.

## 41. 기능을 합성할 때는 믹스인 클래스를 사용하라.

믹스인은 자식 클래스가 사용할 메서드 몇 개만 정의하는 클래스다. 믹스인 클래스에는 자체 애트리뷰트 정의가 없으므로 믹스인 클래스의 __init__ 메서드를 호출할 필요도 없다.
공통적으로 사용하는 메서드를 믹스인 안에 한 번만 작성해두면 다른 여러 클래스에 적용할 수 있다. 메모리 내에 들어 있는 파이썬 객체를 직렬화에 사용할 수 있도록 딕셔너리로 바꾸고 싶다고 하자.

```python

class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)
    
    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            print(key, value)
            output[key] = self._traverse(key, value)
        return output
    
    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value

class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

tree = BinaryTree(10,
                  left=BinaryTree(7, right=BinaryTree(9)),
                  right=BinaryTree(13, left=BinaryTree(11)))

tree = BinaryTree(10, left=BinaryTree(7, right=BinaryTree(9)), right=BinaryTree(13, left=BinaryTree(11)))
print(tree.to_dict())
>>>
{'value': 10, 'left': {'value': 7, 'left': None, 'right': {'value': 9, 'left': None, 'right': None}}, 'right': {'value': 13, 'left': {'value': 11, 'left': None, 'right': None}, 'right': None}}
```

믹스인은 제너릭 기능을 쉽게 연결할 수 있고 기존 기능을 다른 기능으로 오버라이드해 변경하는 것도 쉽다. 아래 코드는 참조를 저장하여 무한루프에 빠지는 문제를 해결하기 위해 _traverse를 오버라이드 한 것이다.

```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

    def _traverse(self, key, value):
        if (isinstance(value, BinaryTreeWithParent) and
                key == 'parent'):
            return value.value  # Prevent cycles
        else:
            return super()._traverse(key, value) # Call unbound super method
```

## 42. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라

애브리뷰트 앞에 밑줄을 두 개(__)붙이면 비공개 필드가 된다. 하위 클래스나 클래스 외부에서 비공개 애트리뷰트에 직접 접근하는 것은 불가능하지만 비공개 애트리뷰트의 동작은 애트리뷰트 이름을 바꾸는 단순한 방식으로 구현되기 때문에, 바뀐 이름을 호출함으로써 접근 가능하다.

```python
class MyParentObject:
    def __init__(self):
        self.__private_field = 71

    def get_private_field(self):
        return self.__private_field

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()
baz.get_private_field()
>>>
AttributeError: 'MyChildObject' object has no attribute '_MyChildObject__private_field'

assert baz._MyParentObject__private_field == 71
```

이처럼 파이썬은 비공개 애트리뷰트에 대한 가시성을 엄격히 제한하지 않는다. 오히려 비공개 애트리뷰트를 사용하면 확장이나 하위 클래스의 오버라이드를 귀찮게 하고 깨지기 쉽게 만든다.
코드 작성을 제어할 수 없는 하위 클래스에서 이름 충돌이 일어나는 경우를 막고 싶을 때만 비공개 애트리뷰트를 사용할 것을 권한다.

## 43. 커스텀 컨테이너 타입은 collections.abc를 상속하라

파이썬은 커스텀 컨테이너 타입을 만들기 위해 내장 컨테이너 타입(List, Dict 등)을 상속할 수 있다. 하지만 내장 컨테이너 타입이 사용하는 모든 동작을 구현하기 위해서는 컨테이너 타입 자체를 상속하기 보다 collections.abc를 상속하라.

```python
from collections.abc import Sequence

class BadType(Sequence):
    pass

foo = BadType()
>>>
TypeError: Can't instantiate abstract class BadType with abstract methods __getitem__, __len__
```

추상 기반 클래스가 요구하는 모든 메서드를 구현하면 index나 count와 같은 추가 메서드 구현을 거저 얻을 수 있다. 이처럼 커스텀 컨테이너 타입이 collections.abc에 정의된 인터페이스를 상속하면 커스텀 컨테이너 타입이 정상적으로 작동하기 위해 필요한 인터페이스와 기능을 제대로 구현하도록 보장할 수 있다.