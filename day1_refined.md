# Day 1 정리

0. **파이썬은 객체 참조를 전달한다.**  
- Q1) 불변/가변이 왜 다른 결과를 내놓는가  
- Q2) 프로그램/프로세스 차이  
- Q3) 전역변수/지역변수 차이  
- Q4) Call by Value / Call by Reference / Call by Object Reference 차이  

---

## Q1. 불변/가변이 왜 다른 결과를 내놓는가

### (1) 불변 객체 (Immutable Objects)
- 한번 생성된 객체의 값을 변경할 수 없음.  
- 값을 바꾸면 기존 객체를 수정하는 것이 아니라 **새 객체를 생성**함.  
- 대표: `int`, `float`, `str`, `tuple`  

```python
a = 10
print(id(a))  # a가 가리키는 객체 주소 출력
a = 20        # 새로운 객체 20 생성 → a가 이를 가리킴
print(id(a))  # 새로운 주소 출력
```

### (2) 가변 객체 (Mutable Objects)
- 생성된 객체의 값을 변경할 수 있음.  
- 동일 객체를 참조하는 다른 변수에도 영향이 감.  
- 대표: `list`, `dict`, `set`  

```python
list1 = [1, 2, 3]
list2 = list1      # 같은 리스트 객체 참조
list1[0] = 100     # 리스트 내용 수정
print(list2)       # [100, 2, 3]
```

✅ **정리**  
- 불변 객체: 값 변경 시 **새 객체 생성** → 원래 객체 불변.  
- 가변 객체: 값 자체를 변경 가능 → 같은 객체 참조 시 다른 변수도 영향.  

---

## Q2. 프로그램 / 프로세스 차이

### 프로그램 (Program)
- 실행 전 정적인 명령어 집합.  
- 저장 위치: 디스크(HDD/SSD).  
- 자원 사용: 없음.  
- 수명: 파일이 삭제될 때까지 존재.  
- 예: `.exe`, `.py` 파일.  

### 프로세스 (Process)
- 실행 중인 프로그램(메모리에 올라간 상태).  
- 저장 위치: RAM.  
- 자원 사용: CPU, 메모리, 입출력.  
- 수명: 실행 시작 → 종료까지.  
- 예: 실행 중인 프로그램 인스턴스.  

---

## Q3. 전역변수 / 지역변수 차이

파이썬에서 차이는 **스코프(scope)** 와 **생존기간(lifetime)**.

### 전역변수 (Global Variable)
- 함수 밖에서 선언됨.  
- 프로그램 전체에서 접근 가능.  
- 함수 내에서 수정하려면 `global` 키워드 필요.  

```python
x = 10

def func():
    global x
    x = 20   # 전역변수 수정

func()
print(x)  # 20
```

### 지역변수 (Local Variable)
- 함수 내에서 선언됨.  
- 함수가 종료되면 메모리에서 사라짐.  
- 같은 이름의 전역변수보다 **지역변수가 우선**됨.  

```python
def func():
    y = 10
    print(y)

func()
print(y)   # 오류 (함수 밖 접근 불가)
```

---

## Q4. Call by Value / Reference / Object Reference

### (1) Call by Value (값에 의한 호출)
- 인자의 **값 복사본**을 전달.  
- 함수 내 변경이 원본에 영향 없음.  
- 예: C, Java 기본형.  

### (2) Call by Reference (참조에 의한 호출)
- 인자의 **메모리 주소**를 전달.  
- 함수 내 변경이 원본에도 영향.  
- 예: C++ 참조(`&`).  

### (3) Call by Object Reference (파이썬 방식)
- 파이썬은 **객체의 참조를 복사**하여 전달.  
- 가변 객체 → 원본 영향 O  
- 불변 객체 → 원본 영향 X  

```python
def change_val(x):
    x = 10   # 불변 타입: 새로운 객체 대입

a = 5
change_val(a)
print(a)    # 5 (변화 없음)


def change_list(lst):
    lst[0] = 100   # 가변 타입: 리스트 수정

my_list = [1, 2, 3]
change_list(my_list)
print(my_list)     # [100, 2, 3]
```

---

### 추가 예시

```python
def change_value(x, value):
    x = value
    print("x:", x, "in change_value")

x = 10
change_value(x, 20)
print("x:", x, "in main")
```

```python
def func(i):
    li[0] = 'I am your father!'

def func2(li):
    li = ['I am your father!', 2, 3, 4]  
    # 새로운 리스트 할당 → 원본 영향 없음

li = [1, 2, 3, 4]
func2(li)
print(li)   # [1, 2, 3, 4]
```
