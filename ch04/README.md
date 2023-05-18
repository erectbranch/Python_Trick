# 4 클래스와 객체 지향 프로그래밍

## 4.1 객체 비교: is vs ==

비교 연산자 `is`와 `==`의 차이를 알아보자.

- `is`: 같은 메모리 주소를 가리키는지 확인.

- `==`: 값이 같은지 확인.

```python
a = [1, 2, 3]
b = a            # 동일한 메모리 주소를 가리킨다.
c = list(a)      # c: a의 복사본
```

위 예시에서 a와 b, a와 c를 비교하면 `is`, `==`의 결과가 다르게 나온다.

```python
# is 비교
a is b   # True
a is c   # False

# == 비교
a == b   # True
a == c   # True
```

---

## 4.2 \_\_str\_\_ vs \_\_repr\_\_

Python에서 class를 만들고 `print({객체명})`을 입력 시 memory address가 출력된다.

```python
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

my_car = Car('red', 37281)
print(my_car)                   # <__console__.Car object at 0x109b73da0>
```

이를 방지할 수 있는 게 `__str__`, `__repr__`인데, 정답부터 말하면 `__repr__`을 쓰는 편이 낫다.

> double underscore(`__`)로 표시한 method를 magic method(dunder method)라고 부른다. 참고로 dunder는 double under의 약자.

---

### 4.2.1 \_\_str\_\_

우선 `__str__`을 추가한 버전을 보자.

```python
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __str__(self):
        return f'a {self.color} car'

my_car = Car('red', 37281)
print(my_car)                   # a red car
my_car                          # <__console__.Car object at 0x109b73da0>
```

이번에는 my_car를 입력 시 memory address를 출력한다.

---

### 4.2.2 \_\_repr\_\_

다음은 `__repr__`을 추가한 버전이다.

- `__str__` method를 추가하지 않으면, `__str__`을 사용해야 할 때도 `__repr__`을 사용한다. 따라서 최소한의 구현으로 `__repr__`을 두는 것이 좋다.

```python
class Car:
    def __init__(self, color, milege):
        self.color = color
        self.milege = milege

    def __repr__(self):
        return '__repr__ for Car'

    def __str__(self):
        return '__str__ for Car'


my_car = Car('red', 37281)
print(my_car)                      # __str__ for Car
'{}'.format(my_car)                # __str__ for Car
my_car                             # __repr__ for Car
```

이번에는 my_car를 입력 시, `__repr__`에서 정의한 대로 출력이 반환되었다.

> 참고로 list, dictionary data type은 언제나 `__repr__` 결과를 사용한다.

`__repr__` 함께 사용하면 유용한 attribute가 있다. 바로 항상 class의 이름을 담는 `__class__.__name__`이다.

```python
    def __repr__(self):
        return (f'{self.__class__.__name__}('
                f'{self.color!r}, {self.mileage!r})')    # !r 변환 플래그로 str 대신 repr을 사용한다.
```

---

## 4.3 custom exception class 정의하기

사용자 정의 예외 클래스를 만들면 예외에서 더 많은 정보를 얻을 수 있다. 즉, 디버깅이 훨씬 쉬워진다.

우선 별달리 정의한 예외 클래스 없이 `ValueError` 예외를 발생시켜 보자.

```python
def validate(name):
    if len(name) < 10:
        raise ValueError

validate('joe')    # ValueError
```

이 경우 (1) 무엇이 잘못되어 발생한 에러인지 고민하고 (2) `validate()` 코드를 자세하게 분석하고 나서야 `ValueError` 원인을 알 수 있을 것이다.

이번에는 사용자 정의 예외 클래스를 만들어 보자.

```python
class NameTooShortError(ValueError):
    pass

def validate(name):
    if len(name) < 10:
        raise NameTooShortError

validate('joe')    # NameTooShortError
```

이처럼 기본 에러 클래스에서 파생시켜서 에러 클래스의 종류를 늘리면 된다.

---

## 4.4 copy.deepcopy

생략

---

## 4.5 abstract base class

**Abastact Base Class**(ABC, 추상 클래스)는 상속 클래스가 특정 method를 반드시 구현하도록 강제한다.

우선 ABC 없이 서브 클래스가 기본 클래스의 인터페이스 메서드를 구현하도록 강제해 보자.

```python
class Base:
    def foo(self):
        raise NotImplementedError()

    def bar(self):
        raise NotImplementedError()

class Concrete(Base):
    def foo(self):
        return 'foo() called'

    # def bar가 있어야 한다.

c = Concrete()
c.foo    # 'foo() called'
c.bar    # NotImplementedError
```

이러한 구현의 문제는, 인터페이스 메서드를 까먹고 구현하지 않아도 `bar()` 메서드 호출 전까지는 에러가 발생하지 않는다는 점이다.

게다가 Base()를 인스턴스화할 수도 있다.

```
b = Base()
b.foo    # NotImplementedError
```

`abc` 모듈을 이용하면 더 잘 해결할 수 있다.

```python
from abc import ABCmeta, abstractmethod

class Base(metaclass=ABCMeta):
    @abstractmethod
    def foo(self):
        pass
    
    @abstractmethod
    def bar(self):
        pass

class Concrete(Base):
    def foo(self):
        pass

    # def bar가 있어야 한다.

c = Concrete()                  # TypeError: "Can't instantiate abstract class Concrete with abstract methods bar"
```

---

## 4.6 namedtuple

**tuple**은 불변이므로 생성하면 수정할 수 없다.

```python
tup = ('hello', object(), 42)
tup             # ('hello', <object object at 0x105e76b70>, 42)
tup[2] = 23     # "'tuple' object does not support item assignment"
```

위 예시에서 튜플의 문제점을 알 수 있다. 튜픙른 개별 속성에 이름을 붙일 수 없고, 오로지 정수 index를 써서 데이터에 접근해야 한다. 이러한 문제를 해결하려는 것이 **namedtuple**이다.

```python
from collections import namedtuple
Car = namedtuple('Car', 'color mileage')    # Car라는 이름의 namedtuple을 생성한다.
```

tuple은 tuple인데 'Car'라는 이름의 tuple이 됐다. 그런데 'color mileage'를 space로 구분해서 전달했다. 이는 namedtuple의 factory function이 `split()`을 통해 list로 쪼개주기 때문이다.

```python
# 이 코드와 동일하다.
Car = namedtuple('Car', [
    'color',
    'mileage'
    ]
)
```

실제로 사용해 보면 왜 편리한지 알 수 있다. 내부적으로는 일반 class와 동일하게 생성되지만, memory 사용량이 일반 클래스보다 적으며 가독성 면에서 효율적이다.

```python
my_car = Car('red', 3812.4)
my_car.color        # 'red'
my_car.mileage      # 3812.4

# 일반 튜플처럼 정수 index로도 접근하거나 내용물을 볼 수 있다.
my_car[0]           # 'red'
tuple(my_car)       # ('red', 3812.4)
print(*my_car)      # red 3812.4 (튜플 풀기)
```

---

### 4.6.1 namedtuple 상속하기

`._fields` 메서드를 통해 namedtuple도 상속할 수 있다.

> 보통 `_`는 private를 뜻하나, namedtuple은 다른 tuple field와 충돌을 방지하기 위해 사용했을 뿐이다.

```python
Car = namedtuple('Car', 'color mileage')
ElectricCar = namedtuple(
    'ElectricCar', Car._fields + ('charge',)    # Car의 필드에 'charge'를 추가한다.
)

ElectricCar('red', 1234, 45.0)    # ElectricCar(color='red', mileage=1234, charge=45.0)
```

---

## 4.7 class variable vs instance variable

Python에서는 클래스 변수와 인스턴스 변수를 구별한다.

```python
class Model:
    model_type = "CNN"            # class variable

    def __init__(self, name):
        self.name = name    # instance variable

alexnet = CNN('alexnet')
alexnet.model_type      # 'CNN'
mobilenet = CNN('mobilenet')
```

클래스 자체를 통해 인스턴스 변수에 접근하면 어떻게 될까?

```python
Model.name     # AttributeError: "type object 'Model' has no attribute 'name'"
```

오류가 뜬다. 그렇다면 "CNN"에서 "Transformer"로 변경할 수 없을까?

```python
Model.model_type = "Transformer"

alexnet.model_type      # 'Transformer'
mobilenet.model_type    # 'Transformer'
```

변경하자 모든 인스턴스의 `model_type` 클래스 변수가 'Transformer'로 바뀌어 버렸다. alexnet 변수에서만 바꾸면 어떨까?

```python
alexnet.model_type = "CNN"

alexnet.model_type      # 'CNN'
alexnet.__class__.model_type    # 'Transformer'
```

이는 단순히 인스턴스 변수가 클래스 변수을 가리고 있는 것과 마찬가지다. `__class__`를 통해서 기존의 클래스 변수도 여전히 접근할 수 있다. 버그의 원인이 될 수 있기 때문에 언제나 주의해야 한다.

---


