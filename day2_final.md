# 네임스페이스란?
- 네임스페이스란 이름(변수, 함수 등)을 객체에 매핑한 (사물함에 있는 서류철에 집어넣고 네임태그 붙여놓은 것처럼) 컨테이너로, 파이썬에선 딕셔너리처럼(dict) 동작함. 동일한 이름이라도 다른 네임스페이스에 존재하면 충돌 없이 사용할 수 있음

- 대표적인 네임스페이스 종류:
    빌트인 네임스페이스: 파이썬이 실행되는 동안 항상 존재, 내장 함수·예외·자료형 등이 속함
    글로벌 네임스페이스: 모듈 단위로 생성, 프로그램이 실행되는 동안 유지
    로컬 네임스페이스: 함수 호출 시 생성, 함수 종료 시 소멸
    엔클로징(비지역/nonlocal) 네임스페이스: 중첩 함수가 있을 때 적용

- 스코프는 이름을 참조할 수 있는 코드의 범위입니다. 네임스페이스와 스코프는 밀접하게 관련되어 있음
- 파이썬의 LEGB 규칙에 따라 이름을 찾는 순서: Local → Enclosing → Global → Built-in
- 네임스페이스 관리는 globals()와 locals() 함수로 할 수 있는데,
    globals()는 글로벌 네임스페이스의 참조를,
    locals()는 현 로컬 네임스페이스의 사본을 반환합니다.
    함수 내부에서 외부 네임스페이스의 변수를 수정하려면
    global 키워드는 글로벌 변수 수정에,
    nonlocal 키워드는 가장 가까운 비지역(엔클로징) 변수 수정에 사용함. 

**엔클로징(enclosing scope)**은 파이썬에서 함수가 중첩(함수 안에 함수)되어 있을 때 등장하는 개념입니다.
함수 내부에 또 다른 함수가 정의되면,
내부 함수는 자신의 로컬 스코프(Local Scope) 밖이지만 글로벌 스코프(Global Scope)와는 다른 중간 단계의 스코프, 즉 엔클로징 스코프를 가집니다.
이 엔클로징 스코프는 “가장 가까운 바깥 함수의 함수 몸체 범위”를 뜻합니다.
LEGB 규칙(이름 조회 순서)에서 E는 Enclosing(엔클로징)을 의미합니다.
예를 들어, 다음과 같이 함수가 중첩되어 있을 때:

def outer():
    x = 10
    def inner():
        print(x)  # outer의 변수 x에 접근 가능 (엔클로징)
    inner()

여기서 outer 함수의 변수 x는 내부 함수 inner에서 접근할 수 있는데,
그 이유가 바로 x가 inner의 "엔클로징 스코프"에 있기 때문입니다.

정리하면:
엔클로징은 “내부 함수가 바깥(가장 가까운) 함수의 네임스페이스/스코프에 접근하는 것”을 의미합니다.
즉, 로컬도 글로벌도 아니고, 중간에 낀 바깥 함수의 영역을 가리킴.

결론
파이썬 네임스페이스의 종류와 동작 방식, 스코프와의 관계, 이름 검색(LEGB), 그리고 외부 변수 수정 방법(global, nonlocal)에 대해 이해하면, 더 구조적이고 깔끔한 코드를 작성할 수 있습니다.

# 1급 객체는 변수로도 사용이 가능하고, 함수의 리턴값으로도 사용이 가능하다
파이썬에서 함수는 1급 객체
파이썬의 함수는 1급 객체야. 즉, 함수도 변수에 담을 수 있고, 인자로 넘길 수 있고, 리턴값으로 쓸 수 있어.

🧩 1급 객체 조건

변수에 담을 수 있다
→ 함수든 뭐든 변수에 넣어서 다룰 수 있어야 함

함수의 인자로 넘길 수 있다
→ 다른 함수에 전달할 수 있어야 함

함수의 반환값(return)으로 돌려줄 수 있다
→ 함수가 다른 함수를 리턴할 수도 있어야 함

이 세 가지가 가능하면 그걸 1급 객체라고 불러.

def say_hello():
    return "Hello"

# 1) 변수에 담기
greet = say_hello
print(greet())   # Hello

# 2) 인자로 넘기기
def run_func(f):
    print(f())

run_func(say_hello)   # Hello

# 3) 리턴값으로 쓰기
def outer():
    def inner():
        return "Hi"
    return inner   # 함수 자체를 리턴

new_func = outer()
print(new_func())   # Hi

📌 1급 객체 vs 2급 객체

1급 객체

아까 말했듯이, 변수에 담기 / 함수 인자로 전달 / 함수 리턴값으로 반환 → 다 가능

파이썬에서는 int, str, list, function, class 전부 다 1급 객체임

2급 객체

어떤 언어에서는 “변수에 담을 수는 있지만, 리턴값으로는 못 쓴다” 같은 제한이 있는 애들을 2급 객체라고 불렀어.

예전 언어나 제한된 환경에서 종종 있었음 (예: 초기의 FORTRAN 함수는 변수에 담을 수 없었음)

⚙️ __new__ vs __init__

__new__(cls, ...)

객체(인스턴스)를 “생성”하는 단계

메모리에 새로운 빈 인스턴스를 만들고 리턴함

실제로는 object.__new__가 기본 구현이라 웬만하면 안 건드림

이때 **새 네임스페이스(빈 방)**가 만들어짐

__init__(self, ...)

이미 만들어진 인스턴스를 “초기화”하는 단계

self라는 빈 방에 가구(속성)를 채우는 역할

리턴값은 항상 None

__new__ : 아파트 분양사무소 → 새 집을 짓고 빈 방 하나를 배정
__init__: 이사 업체 → 빈 방에 가구랑 짐을 넣어서 살 수 있게 셋업

class Toy:
    def __new__(cls):
        print("🧱 __new__ : 장난감 본체를 만든다")
        toy = super().__new__(cls)   # 빈 장난감 틀을 생성
        return toy

    def __init__(self):
        print("🎨 __init__ : 장난감을 색칠한다")
        self.color = "red"

t = Toy()
print("색깔:", t.color)

# __new__ 를 배웠으니 싱글톤을 알아보자.  싱글톤이란? (Singleton)

프로그램 전체에서 객체를 딱 한 번만 생성해서 공유하게 하는 패턴이야.

예: 데이터베이스 연결, 설정(config) → 여러 개 만들 필요 없음.

class Singleton:
    _instance = None   # 클래스 변수 (공용 저장소)

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:  # 아직 인스턴스가 없으면
            print("🆕 새 인스턴스 생성")
            cls._instance = super().__new__(cls)
        else:
            print("♻️ 기존 인스턴스 재사용")
        return cls._instance

    def __init__(self, value):
        self.value = value

a = Singleton(10)
print("a.value:", a.value)

b = Singleton(20)
print("b.value:", b.value)

print(a is b)   # 같은 객체인지 확인

🆕 새 인스턴스 생성
a.value: 10
♻️ 기존 인스턴스 재사용
b.value: 20
True

첫 번째 Singleton(10) → __new__가 새 인스턴스를 만들어 _instance에 저장.

두 번째 Singleton(20) → 이미 _instance가 있으니까 새로 안 만들고 그걸 그대로 반환.

그래서 a와 b는 같은 객체(True).

마지막 b.value = 20은 같은 인스턴스를 가리키니까 a.value도 20으로 바뀜.

# 쉐도잉이란?

class A:
    x = 1   # 클래스 변수 (모든 인스턴스가 공유)

a = A()    # 인스턴스 생성

print("A.x:", A.x)   # 1
print("a.x:", a.x)   # 1 (아직 인스턴스에 x가 없으니까 클래스 변수 x=1을 참조)

print("A.__dict__:", A.__dict__)
print("a.__dict__:", a.__dict__)

A.x: 1
a.x: 1
A.__dict__: {'__module__': '__main__', 'x': 1, '__dict__': <attribute ...>, '__weakref__': <attribute ...>, '__doc__': None}
a.__dict__: {}

⚙️ 동작 과정

A.x = 1

x는 클래스 A의 클래스 변수로 저장됨 → A.__dict__ 안에 들어 있음

모든 인스턴스(a, b 등)는 기본적으로 이 값을 공유

a = A()

새로운 인스턴스 생성

아직 a.__dict__ 안에는 아무 속성 없음

print(a.x)

파이썬은 a.__dict__ → A.__dict__ 순서로 탐색

인스턴스에 x가 없으니까 클래스 변수 A.x = 1을 찾아서 출력

⚡ 쉐도잉(Shadowing)
a.x = 99

여기서 중요한 건 클래스 변수 A.x를 바꾼 게 아니라, 인스턴스 a에 새로운 속성 x를 만든 것

이제 a.__dict__에 'x': 99가 추가됨

print("A.x:", A.x)   # 여전히 1
print("a.x:", a.x)   # 이제 99 (자기 인스턴스 변수 우선)
print("a.__dict__:", a.__dict__)  # {'x': 99}

📝 정리

A.x → 클래스 변수 (모든 인스턴스가 공유)

a.x → 먼저 자기 인스턴스 변수(a.__dict__)를 찾고, 없으면 클래스 변수(A.__dict__) 참조

a.x = 99 → 인스턴스에 새로운 x를 만들어서 클래스 변수와 “이름이 같아져” 버림 → 이걸 쉐도잉(shadowing) 이라고 부름

# 메서드 바인딩 & 디스크립터

class M:
    def f(self, x):
        return (self, x)   # self와 인자 x를 같이 돌려줌

m = M()

print(M.f)   # 클래스에서 직접 함수 꺼내기
print(m.f)   # 인스턴스에서 메서드 꺼내기

who, val = m.f(10)
print(who is m, val)

⚙️ 동작 단계

M.f

클래스에서 f를 직접 꺼내면 그냥 일반 함수 객체가 나와.

출력: <function M.f at 0x...>

m.f

인스턴스에서 꺼내면 바운드(bound) 메서드 객체가 나와.

출력: <bound method M.f of <__main__.M object at 0x...>>

여기서 "bound"라는 건 self 자리에 이미 m이 결합됐다는 뜻.

m.f(10) 실행

실제 내부 동작은 M.f(m, 10) 과 같아. (self = m, x = 10)

f 함수가 (self, x)를 반환하니까 (m, 10)이 돌아옴.

그래서 who is m → True, val → 10.

📝 핵심 포인트

클래스 속성에 있는 함수 객체는 자동으로 디스크립터(descriptor) 역할을 해.

인스턴스로 접근할 때(m.f), 파이썬은 함수의 __get__ 메서드를 호출해서 self가 자동으로 바인딩된 bound method를 만들어 줘.

그래서 m.f(10)은 사실 M.f(m, 10) 호출과 똑같이 동작해.

🏠 비유

M.f : “요리 레시피 종이” (그냥 함수)

m.f : “요리사 m이 이미 정해진 레시피” (self가 붙음)

m.f(10) : “요리사 m에게 재료 10을 줘서 요리를 시킴” → 결과 (m, 10)

# 스택프레임(stack frame) 힙(heap) 이란? (힙합아님)
import inspect

def foo(a):
    b = [a, a+1]
    frame = inspect.currentframe()
    print("locals in foo:", frame.f_locals)

res = foo(10)
print("after call, res:", res)
print("globals in main:", globals())

⚙️ 실행 흐름

함수 호출 → 스택 프레임 생성

res = foo(10)

파이썬은 foo라는 함수를 호출하면 스택 프레임이라는 “작업 공간”을 하나 만듦.

이 프레임 안에는 함수의 **지역 변수들(locals)**이 들어감.
→ 여기선 a = 10, b = [10, 11], frame = <frame object ...>

inspect.currentframe()

현재 실행 중인 스택 프레임 객체를 가져옴.

frame.f_locals를 찍으면, 지금 함수 안에서 관리 중인 지역 변수 dict가 나옴:

locals in foo: {'a': 10, 'b': [10, 11], 'frame': <frame object ...>}

함수 종료

foo는 return문이 없으니까 None 반환.

스택 프레임은 호출이 끝나면서 사라짐.

그래서 res = None

글로벌 네임스페이스 확인

print("globals in main:", globals())

globals()는 현재 모듈(여기서는 main)의 전역 변수 dict.

이 안에는 foo, inspect, res 같은 전역 이름들이 있음.

🏠 스택 프레임 vs 힙

스택 프레임(stack frame)

함수 호출될 때 생김 → 지역 변수(a, b, frame)가 여기 들어감

함수가 끝나면 사라짐

힙(heap)

실제 객체(b = [10, 11]라는 리스트)는 힙 메모리에 있음

스택에는 "힙에 있는 객체를 가리키는 이름표"만 들어감

[호출 시: foo(10)]

스택 프레임(foo)
 ├─ a → 10
 ├─ b → [힙 객체 @0x1234]
 └─ frame → <frame object>

힙
 └─ [10, 11]   (리스트 실제 데이터)

[호출 끝나면 foo 스택 프레임은 제거됨,
 힙 객체는 참조 없으면 가비지 컬렉션됨]

📝 정리

함수 호출 → 스택 프레임 만들어져서 지역 변수 관리

지역 변수들이 가리키는 실제 데이터(리스트, dict, 객체 등)는 힙에 있음

함수 끝나면 스택 프레임은 사라지고, 힙 객체는 참조가 남아있을 때만 계속 살아남음

#  클래스 상속
class A:
    pass

class B(A):   # B는 A를 상속받음
    pass

#  디자인 패턴 (객체지향의 응용)

자주 반복되는 설계 문제를 해결하는 “템플릿/관용구”

싱글톤 패턴

인스턴스를 단 하나만 만들도록 제한

설정, DB 연결 같은 곳에 사용

팩토리 패턴

객체 생성 로직을 별도의 메서드/클래스로 분리

“무엇을 만들지”만 결정하고 “어떻게 만들지”는 팩토리에게 맡김

# IS - A 관계

🟢 Is-A 관계 (상속, Inheritance)

정의: **“~은 ~이다”**라는 관계

클래스가 다른 클래스를 상속받을 때 성립

파생 클래스(자식)는 기반 클래스(부모)의 속성과 메서드를 물려받음

📌 특징

부모가 가진 속성/기능을 자식이 그대로 사용 가능

자식은 필요하면 기능을 재정의(오버라이딩) 할 수 있음

“일반적인 개념 → 구체적인 개념” 구조

class Animal:
    def speak(self):
        print("소리를 낸다")

class Dog(Animal):   # Dog is-a Animal
    def speak(self):
        print("멍멍")

class Cat(Animal):   # Cat is-a Animal
    def speak(self):
        print("야옹")

d, c = Dog(), Cat()
d.speak()  # 멍멍
c.speak()  # 야옹

Dog는 Animal이다.

Cat은 Animal이다

Animal
   │
   ├─ Dog      (Dog is-a Animal)
   └─ Cat      (Cat is-a Animal)

# HAS - A 관계
🟢 Has-A 관계 (포함, Composition)

정의: **“~은 ~을 가진다”**라는 관계

클래스 안에 다른 클래스의 인스턴스를 멤버 변수로 포함할 때 성립

“전체 → 부분” 구조

📌 특징

코드 재사용: 필요한 기능만 가져다 씀

클래스 간 결합도를 낮추기 쉬움 → 유지보수 유리

상속보다 더 유연함

class Engine:
    def start(self):
        print("엔진 시동 켬")

class Car:
    def __init__(self):
        self.engine = Engine()   # Car has-a Engine
    
    def drive(self):
        self.engine.start()
        print("차가 달린다")

c = Car()
c.drive()

Car
 └─ Engine   (Car has-a Engine)

컴퓨터(Computer)
 ├─ CPU
 ├─ Memory
 └─ Disk

Car 안에 Engine 객체가 들어있음

Computer 안에 CPU, Memory, Disk가 들어있음

“~은 ~을 가진다”

| 구분   | Is-A (상속)        | Has-A (포함)       |
| ---- | ---------------- | ---------------- |
| 정의   | “\~은 \~이다”       | “\~은 \~을 가진다”    |
| 구조   | 부모 → 자식          | 전체 → 부분          |
| 사용 예 | Dog is-a Animal  | Car has-a Engine |
| 재사용  | 부모 기능을 물려받음      | 필요한 객체를 포함해서 사용  |
| 유연성  | 강한 결합 (부모-자식 밀접) | 더 유연, 낮은 결합      |

👉 정리하면:

Is-A → 상속 (계층 구조, 일반 → 구체)

Has-A → 포함 (부품 조립, 전체 → 부분)

# IS - A // HAS - A 선택 기준

🟢 Is-A (상속) 선택 기준

관계가 **“~은 ~이다”**로 자연스럽게 설명될 때

공통 기능을 재사용하고 싶을 때

자식 클래스가 부모 클래스의 행동을 확장하거나 수정해야 할 때

🔎 예시

Dog is-a Animal

Square is-a Shape

Student is-a Person

❗️ 주의

남발하면 강한 결합이 생겨서 유지보수 어려워짐

부모 바꾸면 자식들이 줄줄이 깨짐 → 취약한 구조

🟢 Has-A (포함/Composition) 선택 기준

관계가 **“~은 ~을 가진다”**로 자연스럽게 설명될 때

기능을 조립해서 사용하고 싶을 때

여러 클래스에서 같은 객체를 공유하거나 독립적으로 관리하고 싶을 때

유연성과 낮은 결합도가 필요할 때

🔎 예시

Car has-a Engine

House has-a Door

Computer has-a CPU

🟠 실무 가이드 (상속 vs 포함)

“재사용만 필요하다” → 포함(Has-A)이 더 낫다

“진짜로 상속 관계가 자연스럽다” → 상속(Is-A)을 쓴다

디자인 원칙: 구성(Has-A)을 상속(Is-A)보다 선호하라 (Composition over Inheritance)