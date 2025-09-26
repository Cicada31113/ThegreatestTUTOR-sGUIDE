# Python 실행 원리 & 병렬 프로그래밍 학습 정리

> 오늘 강의에서 다룬 내용들을 정리한 문서입니다.  
> 파이썬의 실행 구조, 인터프리터 vs 컴파일러, 프로세스와 스레드, 동시성(concurrency), GIL, 동기화 이슈 등을 실습 코드와 함께 설명합니다.  

---

## 목차
1. `__name__ == "__main__"` 과 인터프리터
2. `sys.settrace` 를 통한 실행 추적
3. 프로세스 (multiprocessing)
4. 스레드 (threading)
5. 스택 vs 힙 (공유 메모리와 로컬 변수)
6. CPU-bound vs I/O-bound 작업
7. 너무 작은 작업 분할의 비효율
8. 임계구역과 Lock
9. GIL(Global Interpreter Lock)

---

## 1. `__name__ == "__main__"` 과 인터프리터

```python
print("__name__ is:", __name__)

def hello():
    print("hello called")

if __name__ == "__main__":
    hello()
```

- `__name__` 은 파이썬이 실행할 때 붙여주는 특별한 변수입니다.  
  - 현재 파일이 직접 실행되면 `__name__ == "__main__"`.  
  - 다른 모듈에서 import 되면 `__name__ == "모듈명"`.  
- 따라서, `if __name__ == "__main__":` 은 **직접 실행될 때만 동작하는 영역**을 구분합니다.

### 인터프리터 vs 컴파일러
- **컴파일러 언어(C, Java)**: 전체 코드를 기계어로 번역 → 실행 파일 생성 → 실행.
- **인터프리터 언어(Python, JS, Ruby)**: 코드를 한 줄씩 읽어 바로 실행 → 실행 파일 없음.
- 장단점:
  - 컴파일러: 실행 속도 빠름, 하지만 빌드 과정 필요.
  - 인터프리터: 개발/디버깅 편리, 하지만 실행 시마다 해석 → 속도 느림.

---

## 2. `sys.settrace` 를 통한 실행 추적

```python
import sys, linecache

def trace(frame, event, arg):
    if event == "line":
        co = frame.f_code
        lineno = frame.f_lineno
        src = linecache.getline(co.co_filename, lineno).strip()
        print(f"Tracing {co.co_name} at line {lineno}: {src}")
    return trace

def add(x, y):
    z = x + y
    return z

if __name__ == "__main__":
    sys.settrace(trace)
    a = 10
    b = 20
    print("Result of add:", add(a, b))
```

- `sys.settrace()` : **파이썬이 실행하는 모든 줄**을 가로채서 추적할 수 있게 해주는 함수.
- `linecache` : 특정 파일의 특정 줄을 가져오는 모듈.
- 이를 통해 **파이썬 가상머신(CPython)이 바이트코드를 어떻게 한 줄씩 실행하는지** 확인 가능.

---

## 3. 프로세스 (multiprocessing)

```python
from multiprocessing import Process, current_process
import os, time

def work(n):
    print(f"[child] pid={os.getpid()}, name={current_process().name}, arg={n}")
    time.sleep(1)

if __name__ == "__main__":
    print(f"[parent] pid={os.getpid()}, name={current_process().name}")
    ps = [Process(target=work, args=(i,), name=f"p{i}") for i in range(3)]
    for p in ps:
        p.start()   # 반드시 호출해야 자식 프로세스 시작
    for p in ps:
        p.join()    # 부모가 자식 종료까지 기다림
```

- **프로세스** = 실행 중인 프로그램 인스턴스, 독립적인 메모리 공간과 PID(Process ID)를 가짐.  
- `start()` 안 하면 자식 실행 X.  
- `join()` 안 하면 부모가 먼저 종료되어 자식은 **고아 프로세스**가 될 수 있음.

---

## 4. 스레드 (threading)

```python
import threading

shared = {"count": 0}

def worker(n):
    for _ in range(100000):
        shared["count"] += 1
    print(f"[worker {n}] done")

ts = [threading.Thread(target=worker, args=(i,)) for i in range(4)]
[t.start() for t in ts]
[t.join() for t in ts]

print("shared count:", shared["count"])
```

- **스레드** = 하나의 프로세스 안에서 실행되는 여러 흐름.  
- 스레드는 **메모리를 공유**하므로 빠른 통신 가능.  
- 하지만 동시에 접근 시 **데이터 충돌(경쟁 조건)** 발생 → 동기화 필요.

---

## 5. 스택 vs 힙 (메모리 구조)

```python
import threading
global_list = []

def push_many(tag):
    local_count = 0
    for i in range(100000):
        global_list.append((tag, local_count))
        local_count += 1
    print(f"[{tag}] pushed {local_count}")

t1 = threading.Thread(target=push_many, args=("A",))
t2 = threading.Thread(target=push_many, args=("B",))
t1.start(); t2.start()
t1.join(); t2.join()
```

- **스택(Stack)**: 함수 호출 시 생기는 로컬 변수 저장 (스레드마다 독립).  
- **힙(Heap)**: 프로세스 전체에서 공유되는 객체 저장소.  
- 따라서 전역 객체(`global_list`)는 여러 스레드가 동시에 접근 가능 → 충돌 가능.

---

## 6. CPU-bound vs I/O-bound 작업

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def cpu_heavy(n: int) -> int:
    return sum(i*i for i in range(n))

def io_heavy(t: float) -> float:
    time.sleep(t)
    return t

if __name__ == "__main__":
    sizes = [10**6, 10**6, 10**6, 10**6]

    start = time.perf_counter()
    with ProcessPoolExecutor(max_workers=4) as executor:
        list(executor.map(cpu_heavy, sizes))
    print("CPU-bound with processes:", time.perf_counter() - start, "s")

    delays = [1, 1, 1, 1]
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=4) as executor:
        list(executor.map(io_heavy, delays))
    print("I/O-bound with threads:", time.perf_counter() - start, "s")
```

- **CPU-bound 작업** (연산 많은 경우): `ProcessPoolExecutor` (멀티코어 활용).  
- **I/O-bound 작업** (대기 많은 경우): `ThreadPoolExecutor` (스레드 전환 비용 적음).  
- 파이썬은 GIL(Global Interpreter Lock) 때문에 CPU 연산은 멀티스레드 효율이 낮음.

---

## 7. 너무 작은 작업 분할의 비효율

```python
import time
from concurrent.futures import ThreadPoolExecutor

def tiny_work() -> int:
    return 1

if __name__ == "__main__":
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=100) as executor:
        list(executor.map(tiny_work, range(100000)))
    print("tiny tasks with threads:", time.perf_counter() - start, "s")
```

- **작업이 지나치게 잘게 쪼개지면** → 스레드 관리 오버헤드 때문에 더 느려질 수 있음.  

---

## 8. 임계구역과 Lock

```python
import threading

N = 100000
count = 0
lock = threading.Lock()

def race():
    global count
    for _ in range(N):
        count += 1  # 동기화 X → 충돌 발생 가능

def safe():
    global count
    for _ in range(N):
        with lock:  # 동기화 O → 안전
            count += 1

# race 예제 실행
count = 0
ts = [threading.Thread(target=race) for _ in range(4)]
[t.start() for t in ts]; [t.join() for t in ts]
print("race result (unsafe):", count)

# safe 예제 실행
count = 0
ts = [threading.Thread(target=safe) for _ in range(4)]
[t.start() for t in ts]; [t.join() for t in ts]
print("safe result (with lock):", count)
```

- **임계구역(Critical Section)**: 여러 스레드가 동시에 접근하면 문제 생기는 코드 영역.  
- `threading.Lock()` 으로 보호 → 안전하게 공유 자원 접근 가능.  

---

## 9. GIL (Global Interpreter Lock)

```python
import time
from concurrent.futures import ThreadPoolExecutor

def io_task(t: float) -> float:
    time.sleep(t)   # I/O 작업 → GIL 자동 해제
    return t

if __name__ == "__main__":
    delays = [0.5, 0.5, 0.5, 0.5]
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=4) as executor:
        list(executor.map(io_task, delays))
    print("I/O tasks with threads:", time.perf_counter() - start, "s")
```

- **GIL**: 파이썬(CPython)에서 한 번에 하나의 스레드만 실행되도록 하는 잠금 장치.  
- CPU-bound 작업에는 병목 발생.  
- 하지만 **I/O 작업 중에는 GIL 해제** → 멀티스레드 이점 있음.

---
