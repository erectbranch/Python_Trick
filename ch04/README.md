# 4 클래스와 객체 지향 프로그래밍

## 4.1 객체 비교: `is` vs `==`

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

## 4.2 `__str__` vs `__repr__`

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

### 4.2.1 `__str__`

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

### 4.2.2 `__repr__`

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

