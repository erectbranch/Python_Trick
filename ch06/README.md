# 6 반복과 이터레이션

## 6.1 파이썬다운 반복문

우선 Python에서 C나 Java처럼 작성한 반복문을 살펴보고 이를 고쳐나가자.

```python
my_items = ['a', 'b', 'c']

i = 0
while i < len(my_items):
    print(my_items[i])
    i += 1
```

---

### 6.1.1 i 초기화 없이 `range(len(...))` 사용

```python
for i in range(len(my_items)):
    print(my_items[i])
```

---

### 6.1.2 불필요한 index 제거

하지만 예제 코드에서 index는 불필요하다.

```python
for item in my_items:
    print(item)
```

거의 pseudo code에 가까운 간결함을 보인다. 

---

### 6.1.3 index가 필요하면 enumerate() 사용

만약 index가 필요하면 내장 함수 `enumerate()`를 사용하는 편이 낫다.

```python
for i, item in enumerate(my_items):
    print(f'{i}: {item}')

# 0: a
# 1: b
# 2: c
```

---

### 6.1.4 굳이 꼭 C 스타일의 반복문을 쓰고 싶다면

```c
for (int i = a; i < n; i += s) {
    // ...
}
```

굳이 위와 같은 C 스타일의 반복문을 쓰고 싶다면, 차라리 `range()`로 간단하게 표현하는 방식을 추천한다.

```python
for i in range(a, n, s):
    # ...
```

---

## 6.2 딕셔너리 반환 예제

다음과 같이 딥러닝 모델 이름과 데이터셋이 정의된 딕셔너리가 있다고 하자.

> 딕셔너리에서는 끝에 key-value 쌍이 없어도 언제나 ','를 붙이는 습관을 들이자.

```python
config = {
    "name": "AlexNet",
    "dataset": "ImageNet",
}
```

위 config 딕셔너리를 조회하는 반복문은 다음과 같다. `items()` 메서드를 통해 key-value 쌍을 얻어서 조회한다.

```python
for key, value in config.items():
    print(f'{key}: {value}')

# 'name: AlexNet'
# 'dataset: ImageNet'
```

---

### 6.2.1 중첩 딕셔너리 반환 예제

만약 아래 예시처럼 nested dictionary 형태라면 어떻게 순회해야 할까?

```python
config = {
    "name": "AlexNet",
    "dataset": "ImageNet",
    "conv_1" : {
        "name": 'Conv2d',
        "in_channels": 3,
        "out_channels": 64,
        "kernel_size": 11,
        "stride": 4,
        "padding": 2,
    }
    "conv_2" : {
        ...
    }
}
```

value가 dictionary type인지 검사하여 unpack하면 문제 없이 순회할 수 있다.

```python
for key, value in config.items():
    if type(value) is dict:
        for nested_key, nested_value in value.items():
            print(f'{nested_key}: {nested_value}')
    else:
        print(f'{key}: {value}')
```

- `if isinstance(value, dict)`로 작성해도 된다.

---

### 6.2.2 중첩 딕셔너리 클래스화

> [Convert a nested dictionary to an Object in Python](https://bobbyhadz.com/blog/python-convert-nested-dictionary-to-object)

이러한 반복문을 이용하면 중첩 딕셔너리를 클래스로 만들 수 있다.

```python
class ModelConfig:
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            if isinstance(value, dict):
                self.__dict__[key] = ModelConfig(**value)
            else:
                self.__dict__[key] = value

net = ModelConfig(**config)

print(net.name)           # AlexNet
print(net.conv_1.name)    # Conv2d
```

---