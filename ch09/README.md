# 9 typing과 정적 타입 검사

> [파이썬 Typing 파헤치기 - 심화편](https://sjquant.tistory.com/69)

---

## 9.1 애너테이션, mypy

> [mypy](https://github.com/python/mypy): static type checker for Python

> `pip install mypy`

`mypy`와 패러미터, 변수 타입의 annotation을 통해, 버그를 쉽게 방지할 수 있다.

```python
def subtract(a: int, b: int) -> int:
    return a - b
```

일부러 버그를 만들어 보자.

```bash
$ python3 -m mypy --strict subtract_ex.py    # error: Argument 2 to "subtract" has imcompatible type "str"; expected "int"
```

---

### 9.1.1 mypy 클래스 오류 검출

다음은 버그가 있는 'Counter' 클래스다.

```python
class Counter:
    def __init__(self):
        self.value = 0
    
    def add(self, offset: int):
        value += offset        # self. 누락

    def get(self) -> int:
        self.value             # return 누락
```

mypy를 사용하여 오류를 검출할 수 있다.

```bash
$ python3 -m mypy --strict counter_ex.py     # error: Name: 'value' is not defined
                                             # error: Missing return statement
```

---

## 9.2 typing

---

### 9.2.1 typing.Union: 여러 타입 애너테이션

만약 인자로 여러 타입을 사용하고 싶다면, `typing.Union`을 사용하면 된다.

```python
from typing import Union

def message(msg: Union[str, bytes, None]) -> str:
    ...  
```

---

### 9.2.2 typing.Optional: Union[<타입>, None] 대체

`typing.Optional`은 `Union[<타입>, None]`을 대체할 수 있다.

> 명시적으로 None 값을 사용하는 인자를 받을 경우 적합하다.

```python
from typing import Optional

def message2(msg: Optional[str]) -> str:
    ...
```

---

### 9.2.3 typing.List, Tuple, Dict

리스트, 튜플, 딕셔너리의 인자를 애너테이션할 때는 `typing.List`, `typing.Tuple`, `typing.Dict`를 사용하면 된다.

> 파이썬 3.9부터는 typing 모듈을 import하지 않고도 사용할 수 있다.

```python
from typing import List, Tuple, Dict

names: List[str]
location: Tuple[int, int, int]
counts: Dict[str, int]
```

---

### 9.2.4 typing.Callable

인자가 함수일 경우, `typing.Callable`을 사용하면 된다.

```python
def checker(func: typing.Callable[[str, int], bool]) -> None:
    ...
```

---


## 9.3 annotations: 전방 참조 방지

클래스 애너테이션 사용 시, 앞에 있는 클래스(FirstClass)가 뒤에 있는 클래스(SecondClass)가 실제로 정의되기 전에 사용하는 문제가 발생할 수 있다.(forward referencing)

> 이러한 문제는 mypy로 검출하지 못하므로 주의해야 한다.

```python
class FirstClass:
    def __init__(self, value: SecondClass) -> None:    # name: 'SecondClass' is not defined
        self.value = value

class SecondClass:
    def __init__(self, value: int) -> None:
        self.value = value

second = SecondClass(5)
first = FirstClass(second)
```

해결 방법은 `from __future__ import annotations`를 사용하는 것이다.

> Python 3.7 이상부터 사용 가능(Python 4에서 이 기능은 기본으로 제공될 예정이다.)

> 사용이 불가능하다면, SecondClass를 문자열로 작성(value: 'SecondClass')하는 방법으로 우회할 수 있다.

```python
from __future__ import annotations

class FirstClass:
    def __init__(self, value: SecondClass) -> None:
        self.value = value

class SecondClass:
    def __init__(self, value: int) -> None:
        self.value = value

second = SecondClass(5)
first = FirstClass(second)
```

---

## 9.4 typing.Generator, Iterator

> [Real Python: How to Use Generators and yield in Python](https://realpython.com/introduction-to-python-generators/)

제너레이터 역할을 하는 함수는 typing.Generator, Iterator를 통해 애너테이션할 수 있다. 

- yield만 하는 경우: typing.Iterator[YieldType] 사용
  
  다음은 csv 파일을 읽어서 row를 반환하는 제너레이터다.

```python
def csv_reader(file_name: str) -> Iterator[str]:
    for row in open(file_name, "r"):
        yield row
```

- send와 return이 있을 경우: typing.Generator[YieldType, SendType, ReturnType] 사용

  다음 함수는 두 가지 기능을 수행한다.

  - `float` 타입 수를 받아서, `int`으로 rounding하여 yield한다.
  
  - 음수를 받을 경우 StopIteration 예외를 발생시면서, `str` 타입의 "Done"을 반환한다.

```python
def echo_round() -> Generator[int, float, str]:
    sent = yield 0
    while sent >= 0:
        sent = yield round(sent)
    return "Done"

a = echo_round()
a.send(None)    # Start
                # 0
a.send(3.5)     # 4
a.send(-3)      # Error
                # StopIteration: Done
```

---

## 9.5 typing.TypeVar: 제네릭 타입

`typing.TypeVar`를 사용하여, 내가 원하는 일반화된 타입을 구현할 수 있다.(제네릭 타입) 

```python
from typing import TypeVar

T = TypeVar('T', bound=Union[int, str, bytes])

def batch_iter(data: Sequence[T], size: int) -> Iterable[Sequence[T]]:
    for i in range(0, len(data), size):
        yield data[i:i+size]
```

---