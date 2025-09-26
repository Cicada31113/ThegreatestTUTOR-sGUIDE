# Python 실행 원리 & 동시성(프로세스/스레드) 심화 노트

> 오늘 강의 필기 정리본 (보완/수정 포함).  
> 인터프리터 vs 컴파일러, CPython 동작 방식, 프로세스·스레드, 힙/스택, GIL, 컨텍스트 스위칭, 임계구역/락, 작업 분할 전략까지.

---

## 목차
1. [인터프리터 vs 컴파일러 (핵심 비교)](#sec-1)
2. [CPython은 어떻게 실행되는가 (바이트코드·가상머신·sys/linecache)](#sec-2)
3. [프로세스 작동 원리 (current_process, getpid, start/join, target/args)](#sec-3)
4. [스레드: 실타래 비유, 장단점·한계, `_`(언더스코어) 의미](#sec-4)
5. [스택 프레임 vs 힙: 무엇이 공유되고 무엇이 분리되는가](#sec-5)
6. [CPU-bound vs I/O-bound, Executors, asyncio, GIL, 컨텍스트 스위칭](#sec-6)
7. [작업을 너무 잘게 쪼개면 왜 느려지나 (오버헤드, 대안 전략)](#sec-7)
8. [임계구역과 Lock: race vs safe, `with lock`의 실제 동작](#sec-8)
9. [GIL은 언제/왜 해제되는가: `time.sleep`이 주는 효과](#sec-9)

---

<a id="sec-1"></a>
## 1. 인터프리터 vs 컴파일러 (핵심 비교)

### 🎯 학습 포인트
- 두 실행 모델의 차이를 **실행 시점/산출물/성능/디버깅** 관점에서 비교.
- “파이썬은 인터프리터 언어” 라는 말의 정확한 의미를 잡는다.

### 📖 개념 정리
| 구분 | 컴파일러(예: C/C++) | 인터프리터(예: Python/JS) |
|---|---|---|
| 실행 전 변환 | 전체 소스를 **기계어**로 변환(빌드) | **줄 단위/블록 단위**로 해석하며 바로 실행 |
| 산출물 | 실행 파일(또는 바이너리 라이브러리) | 별도 실행 파일 없음(파이썬 런타임이 필요) |
| 성능 | 대체로 **빠름**(미리 최적화) | **상대적으로 느림**(매 실행 시 해석) |
| 디버깅/개발 | 빌드 필요(피드백 루프 길어지기 쉬움) | 즉시 실행/빠른 실험·디버깅에 유리 |
| 배포 | 런타임 없이 배포 가능(대체로) | 런타임 필요(CPython 등 인터프리터) |

> 주의: “파이썬=순수 인터프리트만 한다”는 오해가 많다. **CPython은 내부적으로 바이트코드로 “컴파일”** 후 **가상머신이 인터프리트**한다. (2번에서 상세)

---

<a id="sec-2"></a>
## 2. CPython은 어떻게 실행되는가 (바이트코드·가상머신·sys/linecache)

### 📌 실습 코드: 한 줄 실행 흐름 추적
```python
import sys, linecache

def trace(frame, event, arg):
    if event == "line":
        co = frame.f_code          # 코드 객체
        lineno = frame.f_lineno    # 현재 줄 번호
        src = linecache.getline(co.co_filename, lineno).strip()
        print(f"Tracing {co.co_name} at line {lineno}: {src}")
    return trace  # 계속 추적하려면 트레이스 함수를 그대로 반환

def add(x, y):
    z = x + y
    return z

if __name__ == "__main__":
    sys.settrace(trace)            # 전역 트레이스 훅 설치
    a = 10
    b = 20
    print("Result of add:", add(a, b))
```

### 🎯 학습 포인트
- **CPython 파이프라인**: 소스 → 파서/AST → **바이트코드**(.pyc 캐시) → **가상머신(평가 루프)**.
- `sys.settrace`로 **줄 단위 이벤트**를 관찰하여 “인터프리트되는 느낌”을 체감.
- `linecache`로 **파일의 특정 줄**을 안전하게 읽어 설명 메시지 구성.

### 📖 개념·모듈 설명
- **CPython**: 표준 파이썬 구현체. 소스를 **바이트코드로 컴파일** 후 **바이트코드를 인터프리트**한다. `__pycache__/모듈명.cpython-xy.pyc` 캐시 사용.
- **`sys.settrace(callable)`**: CPython이 **함수 호출/줄 실행/예외** 등 이벤트마다 콜백을 호출하게 함. 디버거/커버리지 도구의 기반.
- **`linecache.getline(path, lineno)`**: 파일을 매번 열지 않고 **캐시**에서 특정 줄을 가져옴.

### ⚠️ 주의·팁
- 트레이스는 **오버헤드가 큼**. 실서비스 코드에 상시 켜두지 말 것.
- 파이썬 프로그램은 “인터프리트 언어”라 불리지만, **실제론 ‘컴파일(바이트코드) + 인터프리트’ 혼합 모델**임.

---

<a id="sec-3"></a>
## 3. 프로세스 작동 원리 (current_process, getpid, start/join, target/args)

### 📌 보정된 실습 코드
```python
from multiprocessing import Process, current_process
import os, time

def work(n):
    # 자식 프로세스의 문맥: 고유 PID/이름, 인자 n 확인
    print(f"[child] pid={os.getpid()}, name={current_process().name}, arg={n}")
    time.sleep(1)

if __name__ == "__main__":
    print(f"[parent] pid={os.getpid()}, name={current_process().name}")

    ps = [Process(target=work, args=(i,), name=f"p{i}") for i in range(3)]
    for p in ps:
        p.start()    # 🟢 새 프로세스를 실제로 시작 (spawn/fork)
    for p in ps:
        p.join()     # 🟢 자식 종료까지 부모가 대기 (정리/동기화)
```

### 🎯 학습 포인트
- **프로세스 = 독립 주소 공간 + 고유 PID**. 부모/자식은 메모리를 **공유하지 않는다**.
- `start()`가 있어야 **자식이 실제로 실행**된다. `join()`은 **부모가 대기**하는 동기화 지점.

### 📖 키 API 설명
- **`multiprocessing.current_process()`**: 현재 실행 중인 **Process 객체**(이름, PID 등) 반환.
- **`os.getpid()`**: 현재 OS 프로세스 **PID(정수)**.
- **`Process(target=..., args=(...,), kwargs={...}, name="...")`**: 새 프로세스 **생성자**.  
  - `target`: 자식에서 실행할 **함수**  
  - `args/kwargs`: 해당 함수에 넘길 인자  
  - `name`: 디버깅/로그용 식별 이름
- **`start()`**: 자식 **실행 개시(필수)**.  
- **`join()`**: 자식이 **끝날 때까지 대기**. 미호출 시 부모가 먼저 종료될 수 있고, 자식 정리 타이밍 제어가 어렵다.
- **`time.sleep(sec)`**: 해당 프로세스의 **실행을 잠시 멈춤**(OS 스케줄러에게 CPU 양보).

### 👀 부모 vs 자식의 “움직임 차이”
- **PID**: 부모/자식 **서로 다름** → OS가 다른 작업으로 취급.
- **메모리**: 부모의 객체 변경이 자식에게 **자동 전파되지 않음**. 데이터 교환은 **파이프/큐/공유메모리** 필요.
- **스케줄링**: OS가 **병렬**로 실행 가능(코어가 여럿일 때).

---

<a id="sec-4"></a>
## 4. 스레드: 실타래 비유, 장단점·한계, `_`(언더스코어) 의미

### 📌 실습 코드 (공유 딕셔너리 카운팅)
```python
import threading

shared = {"count": 0}

def worker(n):
    for _ in range(100_000):     # _ : 값을 쓰지 않을 루프 변수(관용적 이름)
        shared["count"] += 1
    print(f"[worker {n}] done")

ts = [threading.Thread(target=worker, args=(i,)) for i in range(4)]
[t.start() for t in ts]
[t.join() for t in ts]

print("shared count:", shared["count"])
```

### 🎯 학습 포인트
- **스레드 = 하나의 프로세스 안의 여러 실행 흐름** (주소공간 공유).
- 장점: **공유 메모리로 통신이 빠름**, 생성/전환 비용이 프로세스보다 저렴.
- 한계: **경쟁 조건**(race), **데드락**, **GIL로 인한 CPU 작업 병렬성 한계(CPython)**.

### 📖 상세 설명
- **장점**
  - 같은 프로세스의 **힙**을 공유 → 객체 전달이 **참조만으로** 가능.
  - I/O-bound 작업에 특히 유리(대기 중 다른 스레드가 일함).
- **한계/주의**
  - **GIL** 때문에 CPython에서 **CPU-bound**는 멀티스레드 이득이 작음(6번 참고).
  - 공유 데이터에 **동시 접근**하면 순서/일관성이 깨질 수 있음 → Lock 등 동기화 필수(8번 참고).
- **`_` 언더스코어**: “이 변수는 쓰지 않을 것”이라는 **관용적 이름**일 뿐, 파이썬 문법 키워드는 아님.  
  - 루프 인덱스를 쓰지 않을 때 `for _ in range(n)` 처럼 사용.

---

<a id="sec-5"></a>
## 5. 스택 프레임 vs 힙: 무엇이 공유되고 무엇이 분리되는가

### 📌 실습 코드 (전역 리스트에 다중 스레드가 push)
```python
import threading
global_list = []

def push_many(tag):
    local_count = 0                  # ← 각 스레드의 "스택 프레임(로컬)"에 존재
    for _ in range(100_000):
        global_list.append((tag, local_count))  # ← 프로세스 힙(공유 객체) 조작
        local_count += 1
    print(f"[{tag}] pushed {local_count}")

t1 = threading.Thread(target=push_many, args=("A",))
t2 = threading.Thread(target=push_many, args=("B",))
t1.start(); t2.start()
t1.join(); t2.join()
```

### 🎯 학습 포인트
- **스택 프레임(로컬 변수)**: 스레드마다 **분리**. (함수 호출 컨텍스트)
- **힙(객체 저장소)**: **프로세스 단위로 공유**. 모든 스레드가 접근 가능.

### 📖 상세 설명
- CPython에서 **로컬 변수**는 각 스레드의 **프레임 객체**에 저장되어 서로 섞이지 않음.
- 리스트/딕셔너리/사용자 객체 등은 **힙(공유)** 에 있음 → 여러 스레드가 같은 객체를 바라본다.
- 위 예제의 `list.append`는 CPython C 레벨에서 **원자성 보장되는 편**이라 보통 충돌로 망가지진 않지만,  
  **복합 연산**(읽고-수정하고-쓰는 연속 동작)은 **락 없이** 하면 쉽게 레이스가 난다.  
  (예: `x = x + 1` 은 원자적이지 않음. 8번에서 자세히)

---

<a id="sec-6"></a>
## 6. CPU-bound vs I/O-bound, Executors, asyncio, GIL, 컨텍스트 스위칭

### 📌 보정/확장된 벤치 코드
```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def cpu_heavy(n: int) -> int:
    # 연산량 큰 작업 (CPU-bound)
    return sum(i*i for i in range(n))

def io_heavy(t: float) -> float:
    # I/O 대기 (time.sleep로 시뮬레이션) → 대기 중 GIL 해제
    time.sleep(t)
    return t

def bench_cpu_with_processes(sizes):
    start = time.perf_counter()
    with ProcessPoolExecutor(max_workers=4) as ex:
        list(ex.map(cpu_heavy, sizes))
    return time.perf_counter() - start

def bench_cpu_with_threads(sizes):
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=4) as ex:
        list(ex.map(cpu_heavy, sizes))
    return time.perf_counter() - start

def bench_io_with_threads(delays):
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=4) as ex:
        list(ex.map(io_heavy, delays))
    return time.perf_counter() - start

def bench_io_with_processes(delays):
    start = time.perf_counter()
    with ProcessPoolExecutor(max_workers=4) as ex:
        list(ex.map(io_heavy, delays))
    return time.perf_counter() - start

if __name__ == "__main__":
    sizes  = [10**6]*4
    delays = [1.0]*4

    print("CPU (processes):", bench_cpu_with_processes(sizes), "s")
    print("CPU (threads)  :", bench_cpu_with_threads(sizes),   "s")
    print("IO  (threads)  :", bench_io_with_threads(delays),   "s")
    print("IO  (processes):", bench_io_with_processes(delays), "s")
```

### 🎯 학습 포인트
- **CPU-bound**: 연산이 병목 → **ProcessPool**로 코어 병렬 활용(GIL 회피).
- **I/O-bound**: 대기가 병목 → **ThreadPool** 유리(대기 중 GIL 해제로 동시성↑).
- **컨텍스트 스위칭**: 실행 주체 전환 비용(스레드 < 프로세스). 너무 자주 발생하면 느려짐(7번 참고).

### 📖 용어·API 설명
- **`concurrent.futures`**: 고수준 풀 API. `ThreadPoolExecutor`, `ProcessPoolExecutor` 제공.
- **`max_workers`**: 풀의 최대 작업자 수. 과도하게 크면 컨텍스트 스위칭/스케줄링 오버헤드↑.
- **`time.perf_counter()`**: **모노토닉 고해상도 타이머**. 경과 시간 측정에 권장.
- **GIL(Global Interpreter Lock)**: CPython에서 **한 번에 하나의 스레드만 바이트코드 실행** 가능하게 하는 전역 잠금.  
  단, **블로킹 I/O 구간**과 일부 C 확장 모듈은 **GIL을 해제**하여 동시성 확보.
- **`asyncio`**: **협력형 코루틴** 기반 동시성. `await` 지점에서 제어 양보. CPU-bound에는 부적합, **대규모 I/O**에 적합.

> 실무 팁: “CPU는 프로세스, I/O는 스레드/asyncio” 를 1차 선택지로 삼고, **프로파일링**으로 검증.

---

<a id="sec-7"></a>
## 7. 작업을 너무 잘게 쪼개면 왜 느려지나 (오버헤드, 대안 전략)

### 📌 보정된 실습 코드 (작은 작업 vs 배치)
```python
import time
from concurrent.futures import ThreadPoolExecutor

def tiny_work(_) -> int:
    # 아주 작은 일 (오버헤드가 상대적으로 큼)
    return 1

def batched_work(batch_size: int) -> int:
    # 작은 일을 묶어서 한 쓰레드가 한 번에 처리
    s = 0
    for _ in range(batch_size):
        s += 1
    return s

if __name__ == "__main__":
    N = 100_000

    # 1) 작은 작업 N개를 그대로 스레드풀에 던짐
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=100) as ex:
        list(ex.map(tiny_work, range(N)))
    t_tiny = time.perf_counter() - start
    print("tiny tasks individually:", t_tiny, "s")

    # 2) 작은 작업을 묶어서(배치) 개수를 줄임
    batch = 100
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=100) as ex:
        list(ex.map(batched_work, [batch]*(N//batch)))
    t_batch = time.perf_counter() - start
    print("batched tasks:", t_batch, "s")
```

### 🎯 학습 포인트
- **작은 작업 개수가 너무 많으면** 스케줄링/큐잉/컨텍스트 스위칭/퓨처 관리 **오버헤드**가 지배.
- **배치(batch)** 로 묶거나, **벡터화/일괄 처리**로 호출 횟수를 줄이면 성능이 크게 나아진다.

### 📖 왜 느려지나?
- **큐 푸시/팝 + 잠금 비용** (스레드풀 내부 구현).
- **작업자 깨우기/컨텍스트 스위칭**.
- **함수 호출/파이썬 프레임 생성/소멸** 비용이 **작업 자체보다 커짐**.

---

<a id="sec-8"></a>
## 8. 임계구역과 Lock: race vs safe, `with lock`의 실제 동작

### 📌 실습 코드 (경쟁 조건 재현 → Lock으로 해결)
```python
import threading

N = 100_000
count = 0
lock = threading.Lock()

def race():
    global count
    for _ in range(N):
        # 비원자적(Load → Add → Store)이라 중간에 스위칭되면 꼬임
        count += 1

def safe():
    global count
    for _ in range(N):
        with lock:      # __enter__ 시 lock.acquire(), __exit__ 시 lock.release()
            count += 1

# 1) 레이스 (락 없음)
count = 0
ts = [threading.Thread(target=race) for _ in range(4)]
[t.start() for t in ts]; [t.join() for t in ts]
print("race result (unsafe):", count)

# 2) 안전 (락 사용)
count = 0
ts = [threading.Thread(target=safe) for _ in range(4)]
[t.start() for t in ts]; [t.join() for t in ts]
print("safe result (with lock):", count)
```

### 🎯 학습 포인트
- **임계구역**: 여러 스레드가 **동시에** 들어오면 **불일치/깨짐**이 발생하는 코드 영역.
- `threading.Lock()` 으로 **서로 배타적**으로 실행되게 만들어 일관성을 보장.

### 📖 상세 설명
- `count += 1` 은 바이트코드로 **여러 단계**로 나뉘며 **원자적이지 않다**.  
  → GIL이 있어도 **바이트코드 사이에 스위칭**이 일어나면 값이 유실.
- **`with lock:`** 는 컨텍스트 매니저 프로토콜:  
  - 진입 시 `lock.acquire()`  
  - 블록 종료(정상/예외) 시 **반드시** `lock.release()`  
- 상황에 따라 `RLock`(재귀 락), `Semaphore`, `Condition` 등 고급 동기화 도구를 사용.

---

<a id="sec-9"></a>
## 9. GIL은 언제/왜 해제되는가: `time.sleep`이 주는 효과

### 📌 실습 코드 (I/O 대기에서 GIL 해제 체감)
```python
import time
from concurrent.futures import ThreadPoolExecutor

def io_task(t: float) -> float:
    time.sleep(t)   # 블로킹 호출 동안 C 레벨에서 GIL을 풀고 대기
    return t

if __name__ == "__main__":
    delays = [0.5, 0.5, 0.5, 0.5]
    start = time.perf_counter()
    with ThreadPoolExecutor(max_workers=4) as ex:
        list(ex.map(io_task, delays))
    print("I/O tasks with threads:", time.perf_counter() - start, "s")
```

### 🎯 학습 포인트
- **GIL(Global Interpreter Lock)**: CPython 인터프리터가 **동시에 하나의 스레드만** 바이트코드를 실행하도록 보장하는 전역 락.
- 하지만 **블로킹 I/O**(예: `time.sleep`, 소켓/파일 I/O) 구간에서 CPython C 코드가 **GIL을 해제**(`Py_BEGIN_ALLOW_THREADS`)하므로,  
  **다른 스레드가 실행**될 수 있다 → I/O-bound에 스레드가 유리.

### 📖 상세 설명
- **해제되는 경우**:  
  - `time.sleep()`, 파일/소켓 I/O, 일부 C 확장(NumPy의 대형 연산 등)이 내부적으로 GIL을 잠시 풀고 대기/계산.  
- **CPU-bound**는?  
  - 파이썬 순수 연산은 대부분 GIL을 쥔 채 실행 → 스레드 병렬성 이득 작음.  
  - 해결책: **multiprocessing**(프로세스 분리) 또는 **C 확장/NumPy**로 GIL 바깥에서 계산.

---

## 부록) 자주 언급된 모듈/함수 한눈에
- **`sys.settrace(fn)`**: 줄 실행/호출/예외 이벤트를 후킹. (디버깅·커버리지 기반)
- **`linecache.getline(path, n)`**: 파일의 n번째 줄을 캐시에서 읽음.
- **`os.getpid()`**: 현재 프로세스 PID.
- **`multiprocessing.current_process()`**: 현재 프로세스 객체(이름/PID).
- **`Process(target, args, kwargs, name)`**: 새 프로세스 생성자.
- **`Thread(target, args, kwargs, name)`**: 새 스레드 생성자.
- **`start()/join()`**: 실행 시작/종료 대기(동기화).
- **`ThreadPoolExecutor/ProcessPoolExecutor`**: 스레드/프로세스 풀. `max_workers`로 병렬도 제어.
- **`time.perf_counter()`**: 경과 시간 측정용 고해상도 타이머.
- **`threading.Lock()`**: 상호배제 락. `with lock:` 으로 안전하게 사용.
