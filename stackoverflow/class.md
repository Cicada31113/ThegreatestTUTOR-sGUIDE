# 클래스와 함수는 다르다: 파이썬 클래스 적용 예제 (풀번역)

함수(`def`)를 그냥 `class`로 바꿔 쓰는 건 겉보기에 돌아갈 수도 있지만, 그게 클래스의 의도는 아냐.  
클래스는 **데이터(속성)**와 **함수(메서드)**를 함께 담는 구조야. 아래 예제로 직관적으로 보자.

---

## 1) Ball 클래스

```python
class Ball(object):
    # __init__은 클래스로 인스턴스를 만들 때 자동으로 호출되는 특수 메서드.
    # 여기서는 객체의 초기 상태(데이터)를 설정한다.
    def __init__(self):
        # 인스턴스에 데이터(속성) 추가
        self.position = (100, 100)
        self.velocity = (0, 0)

    # 클래스 안에 함수(메서드)도 넣을 수 있다.
    # 공이 "튕길" 때, 세로 속도를 반대로 바꾼다(중력은 고려하지 않음).
    def bounce(self):
        self.velocity = (self.velocity[0], -self.velocity[1])
```

### 사용해 보기

```python
>>> ball1 = Ball()
>>> ball1
<Ball object at ...>
```

겉으로는 별거 없어 보이지만, **데이터**가 있어서 유용하다:

```python
>>> ball1.position
(100, 100)
>>> ball1.velocity
(0, 0)
>>> ball1.position = (200, 100)
>>> ball1.position
(200, 100)
```

### 글로벌 변수 대신 클래스를 쓰는 이점(독립성)

인스턴스를 하나 더 만들면 서로 **독립적**으로 유지된다:

```python
>>> ball2 = Ball()
>>> ball2.velocity = (5, 10)
>>> ball2.position
(100, 100)
>>> ball2.velocity
(5, 10)
```

`ball1`은 그대로다:

```python
>>> ball1.velocity
(0, 0)
```

### bounce 메서드 동작

```python
>>> ball2.bounce()
>>> ball2.velocity
(5, -10)
```

`ball2`의 속도만 바뀌고, `ball1`은 건드리지 않는다:

```python
>>> ball1.velocity
(0, 0)
```

---

## 2) 적용: 간단한 텍스트 게임 설계

게임을 만든다고 하자. 가장 먼저 떠오르는 건 **방(room)** 이다.  
그럼 방을 만들어 보자. 방은 **이름**을 가진다.

```python
class Room(object):
    # 이번에는 self 외에 인자(name)도 받는다.
    def __init__(self, name):
        self.name = name  # 전달받은 이름을 방의 이름으로 저장
```

인스턴스 생성:

```python
>>> white_room = Room("White Room")
>>> white_room.name
'White Room'
```

이 상태만으로는 “방마다 다른 동작”을 주기 어려우니, **서브클래스**를 만들자.  
서브클래스는 슈퍼클래스의 기능을 **상속**받되, 기능을 **추가**하거나 **오버라이드**할 수 있다.

### 방에서 하고 싶은 것들

- 우리는 **방과 상호작용**하고 싶다.  
- 어떻게 할까?  
- 사용자가 한 줄의 텍스트를 입력하면, 그에 **응답**하도록 하자.

응답 방식은 방마다 다를 수 있으니, 방이 스스로 처리하도록 `interact` 메서드를 만든다.

### WhiteRoom 서브클래스

```python
class WhiteRoom(Room):  # 화이트 룸은 Room의 한 종류
    def __init__(self):
        # 모든 화이트 룸의 이름은 'White Room'으로 고정
        self.name = 'White Room'

    def interact(self, line):
        if 'test' in line:
            print "'Test' to you, too!"
```

실행해 보자:

```python
>>> white_room = WhiteRoom()  # WhiteRoom은 부모의 __init__을 오버라이드했기 때문에 인자를 받지 않음
>>> white_room.interact('test')
'Test' to you, too!
```

### 방 이동(전환)

예전 예시엔 방 사이를 이동하는 기능이 있었다.  
단순화를 위해 전역 변수 `current_room`으로 **현재 방**을 추적하자.¹ 그리고 **RedRoom**도 만들자.

```python
class RedRoom(Room):  # 레드 룸도 Room의 한 종류
    def __init__(self):
        self.name = 'Red Room'

    def interact(self, line):
        global current_room, white_room
        if 'white' in line:
            # WhiteRoom을 새로 만들면 이전 데이터가 사라질 수 있으니,
            # 이미 존재하는 white_room 인스턴스로 이동한다.
            current_room = white_room
```

시험해 보기:

```python
>>> red_room = RedRoom()
>>> current_room = red_room
>>> current_room.name
'Red Room'
>>> current_room.interact('go to white room')
>>> current_room.name
'White Room'
```

> 연습: **WhiteRoom**의 `interact`에도 “red”를 감지해서 **RedRoom**으로 돌아가게 하는 코드를 추가해 봐.

---

## 3) 게임 루프와 초기화

현재 방의 이름을 프롬프트에 표시하면서, 사용자가 입력한 문장을 해당 방의 `interact`로 넘기자.

```python
def play_game():
    global current_room
    while True:
        line = raw_input(current_room.name + '> ')
        current_room.interact(line)
```

게임을 리셋(초기화)하는 함수도 만들자:

```python
def reset_game():
    global current_room, white_room, red_room
    white_room = WhiteRoom()
    red_room = RedRoom()
    current_room = white_room
```

이제 모든 클래스 정의와 위 함수들을 `mygame.py`에 넣었다고 하자. 대화식 프롬프트에서 이렇게 실행할 수 있다:

```python
>>> import mygame
>>> mygame.reset_game()
>>> mygame.play_game()
White Room> test
'Test' to you, too!
White Room> go to red room
Red Room> go to white room
White Room>
```

스크립트를 **직접 실행**해서 바로 게임이 시작되게 하려면, 아래를 파일 맨 아래에 추가한다:

```python
def main():
    reset_game()
    play_game()

if __name__ == '__main__':  # 스크립트로 실행될 때만 동작
    main()
```

---

## 마무리

여기까지가 **클래스의 기초**와, 네 상황(텍스트 게임)에 **어떻게 적용**할 수 있는지에 대한 간단한 소개야.

---

## 각주

1) 전역 변수를 쓰는 것보다 더 좋은 방법들이 있지만, 여기서는 단순화를 위해 전역을 사용했어.

---

## 참고(실행 환경 팁)

- 위 예시 일부는 Python 2 스타일(`raw_input`, `print` 구문)을 사용하고 있어.  
  Python 3에서 실행하려면 아래처럼 바꿔:
  - `raw_input(...)` → `input(...)`
  - `print '...'` → `print('...')`
