# 파이썬 네임스페이스 · 스코프 · 일급 객체 · 객체지향 요약 (복습용)

## 목차
- [1. 네임스페이스와 스코프](#1-네임스페이스와-스코프)
  - [1.1 네임스페이스 종류](#11-네임스페이스-종류)
  - [1.2 LEGB 이름 탐색 규칙](#12-legb-이름-탐색-규칙)
  - [1.3 globals()·locals(), global·nonlocal](#13-globalslocals-globalnonlocal)
  - [1.4 엔클로징(Enclosing) 스코프](#14-엔클로징enclosing-스코프)
- [2. 일급 객체](#2-일급-객체)
- [3. __new__ vs __init__](#3-__new__-vs-__init__)
- [4. 싱글톤 패턴](#4-싱글톤-패턴)
- [5. 쉐도잉(Shadowing)](#5-쉐도잉shadowing)
- [6. 메서드 바인딩 & 디스크립터](#6-메서드-바인딩--디스크립터)
- [7. 스택 프레임 · 힙](#7-스택-프레임--힙)
- [8. 클래스 상속 (Is-A)](#8-클래스-상속-is-a)
- [9. Has-A (포함/구성)](#9-has-a-포함구성)
- [10. 상속 vs 포함 선택 가이드](#10-상속-vs-포함-선택-가이드)
- [11. 한눈 요약](#11-한눈-요약)

---

## 1. 네임스페이스와 스코프
네임스페이스는 “이름 → 객체” 매핑 컨테이너야. 파이썬에선 사실상 딕셔너리처럼 동작하고, 같은 이름이라도 **서로 다른 네임스페이스**에 있으면 충돌 없이 공존해.

### 1.1 네임스페이스 종류
- **Built-in**: 파이썬 실행 내내 존재. 내장 함수/예외/형 등이 포함됨.
- **Global**: 모듈 단위 전역. 모듈이 살아있는 동안 유지.
- **Local**: 함수 호출 시 생성, 함수 종료 시 소멸.
- **Enclosing**: 중첩 함수에서 “가장 가까운 바깥 함수” 범위.

### 1.2 LEGB 이름 탐색 규칙
이름을 찾을 때 순서는 **L → E → G → B**야.
1) Local → 2) Enclosing → 3) Global → 4) Built-in

### 1.3 globals()·locals(), global·nonlocal
- `globals()`는 **전역 네임스페이스**에 대한 참조를 돌려줘.
- `locals()`는 **현재 로컬 네임스페이스의 사본**을 돌려줘.
- 함수 안에서 바깥 변수 수정하려면:
  - `global`: 전역 변수 수정
  - `nonlocal`: 가장 가까운 **엔클로징** 변수 수정

### 1.4 엔클로징(Enclosing) 스코프
내부 함수는 자기 로컬이 아니고 전역도 아닌, “가장 가까운 바깥 함수”의 이름에 접근할 수 있어.

```python
def outer():
    x = 10
    def inner():
        print(x)  # outer의 x를 읽어옴 (Enclosing)
    inner()
```

---

## 2. 일급 객체
파이썬의 함수는 일급 객체라서 **변수에 담기 / 인자로 전달 / 반환값으로 사용**이 다 가능해.

```python
def say_hello():
    return "Hello"

# 1) 변수에 담기
greet = say_hello
print(greet())  # Hello

# 2) 인자로 넘기기
def run_func(f):
    print(f())
run_func(say_hello)  # Hello

# 3) 함수가 함수를 리턴
def outer():
    def inner():
        return "Hi"
    return inner

new_func = outer()
print(new_func())  # Hi
```

---

## 3. __new__ vs __init__
- `__new__(cls, ...)`: **인스턴스 생성 단계**. 보통 `object.__new__`를 호출해서 “빈 인스턴스”를 만들어 반환함. 이때 **새 네임스페이스(빈 방)**가 생김.
- `__init__(self, ...)`: **초기화 단계**. 이미 생성된 인스턴스에 속성을 채움. 항상 `None` 반환.

비유: `__new__` = 새 집 짓고 방 배정 / `__init__` = 이사 와서 가구 배치.

```python
class Toy:
    def __new__(cls):
        print("__new__: 본체 생성")
        toy = super().__new__(cls)
        return toy

    def __init__(self):
        print("__init__: 속성 초기화")
        self.color = "red"

t = Toy()
print(t.color)  # red
```

---

## 4. 싱글톤 패턴
전체 프로그램에서 **단 하나의 인스턴스**만 유지하고 싶을 때 쓰는 패턴이야.

```python
class Singleton:
    _instance = None  # 클래스 변수

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, value):
        self.value = value

a = Singleton(10)
b = Singleton(20)
print(a is b)       # True (같은 객체)
print(a.value)      # 20 (같은 인스턴스라 값 공유)
```

---

## 5. 쉐도잉(Shadowing)
인스턴스에 **동일한 이름의 속성**을 만들면, 클래스 변수와 “그림자처럼 이름이 겹쳐” 인스턴스 쪽이 우선돼.
클래스 변수가 그림자처럼 가려진다고 해서 쉐도잉 이라불러.

```python
class A:
    x = 1  # 클래스 변수

a = A()
print(A.x, a.x)  # 1, 1

a.x = 99         # 인스턴스에 x 생성(쉐도잉)
print(A.x, a.x)  # 1, 99
print(a.__dict__)  # {'x': 99}
```

탐색 순서: `a.__dict__` → `A.__dict__`.

---

## 6. 메서드 바인딩 & 디스크립터
클래스에 있는 “함수”는 디스크립터로 동작해서, 인스턴스로 접근하면 **self가 결합된 바운드 메서드**가 돼.

```python
class M:
    def f(self, x):
        return (self, x)

m = M()
print(M.f)  # <function ...> (그냥 함수)
print(m.f)  # <bound method ...> (self=m로 바인딩)

who, val = m.f(10)  # 내부적으로 M.f(m, 10)
print(who is m, val)  # True 10
```

---

## 7. 스택 프레임 · 힙
- **스택 프레임**: 함수 호출 시 생성되는 실행 컨텍스트. 지역 변수들이 여기에 올라왔다가 함수가 끝나면 함께 사라져.
- **힙**: 실제 객체(리스트, dict 등) 저장 공간. 스택에는 힙 객체를 가리키는 **참조**만 있음.

```python
import inspect

def foo(a):
    b = [a, a+1]
    frame = inspect.currentframe()
    print("locals in foo:", frame.f_locals)

res = foo(10)     # 스택 프레임 생성 → 지역변수 a,b,frame
print(res)        # None (return 없음)
```

요점: 함수 끝나면 스택 프레임은 사라지고, 힙 객체는 참조가 남아있을 때만 유지됨.

---

## 8. 클래스 상속 (Is-A)
“**~은 ~이다**” 관계가 자연스러울 때 상속을 써.

```python
class Animal:
    def speak(self):
        print("소리를 낸다")

class Dog(Animal):  # Dog is-a Animal
    def speak(self):
        print("멍멍")

class Cat(Animal):  # Cat is-a Animal
    def speak(self):
        print("야옹")
```

---

## 9. Has-A (포함/구성)
“**~은 ~을 가진다**” 관계면 포함을 써. 결합도를 낮추고 유연해.

```python
class Engine:
    def start(self):
        print("엔진 시동")

class Car:
    def __init__(self):
        self.engine = Engine()  # Car has-a Engine
    def drive(self):
        self.engine.start()
        print("달린다")
```

---

## 10. 상속 vs 포함 선택 가이드

| 구분 | Is-A (상속) | Has-A (포함/구성) |
|---|---|---|
| 자연어 판별 | “~은 ~이다” | “~은 ~을 가진다” |
| 구조 | 일반 → 구체 | 전체 → 부분 |
| 재사용 | 부모 기능을 물려받음 | 필요한 객체를 조립 |
| 결합도 | 상대적으로 강함 | 낮음, 유연 |
| 권장 | **진짜** 상속 관계일 때 | 단순 재사용·확장 대부분 |

실무 원칙: **구성을 상속보다 선호**하라 (Composition over Inheritance).

---

## 11. 한눈 요약
- 네임스페이스 = 이름→객체 매핑. 종류: Built-in/Global/Local/Enclosing.
- 이름 탐색 = **LEGB** 순서.
- 바깥 변수 수정 = `global` / `nonlocal`.
- 함수는 **일급 객체**: 변수에 담고/인자로 넘기고/리턴 가능.
- `__new__`는 생성, `__init__`은 초기화.
- 싱글톤은 인스턴스 하나만 유지.
- **쉐도잉**: 인스턴스 속성이 클래스 변수 이름을 가리면 인스턴스 쪽이 우선.
- 인스턴스에서 꺼낸 메서드는 **바운드 메서드**(self 결합).
- 스택 프레임은 호출 시 생성·종료 시 소멸, 실제 객체는 힙에 존재.
- **Is-A**면 상속, **Has-A**면 포함. 대체로 **포함을 먼저 고려**해.
