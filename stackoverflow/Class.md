# 클래스와 함수의 차이: 파이썬 클래스 기초 튜토리얼

함수와 클래스는 매우 다르다.\
단순히 `def`를 `class`로 바꿔도 얼추 동작할 수는 있지만, 그것이 클래스의
본래 목적은 아니다.\
클래스는 **데이터(속성)**와 **함수(메서드)**를 함께 담는 설계도다.

------------------------------------------------------------------------

## Ball 클래스 예제

``` python
class Ball(object):
    # __init__은 인스턴스를 만들 때 자동 호출되는 특수 메서드다.
    # 여기서는 공의 초기 위치와 속도를 지정한다.
    def __init__(self):
        self.position = (100, 100)
        self.velocity = (0, 0)

    # 공이 튀길 때, 세로 속도를 반전시키는 메서드
    def bounce(self):
        self.velocity = (self.velocity[0], -self.velocity[1])
```

### 사용 예시

``` python
>>> ball1 = Ball()
>>> ball1
<Ball object at ...>

>>> ball1.position
(100, 100)
>>> ball1.velocity
(0, 0)

>>> ball1.position = (200, 100)
>>> ball1.position
(200, 100)
```

### 독립성 확인

``` python
>>> ball2 = Ball()
>>> ball2.velocity = (5, 10)
>>> ball2.position
(100, 100)
>>> ball2.velocity
(5, 10)

>>> ball1.velocity
(0, 0)
```

### bounce 메서드 실행

``` python
>>> ball2.bounce()
>>> ball2.velocity
(5, -10)
```

ball1은 영향을 받지 않는다:

``` python
>>> ball1.velocity
(0, 0)
```

------------------------------------------------------------------------

## Room 클래스와 상속

게임을 만든다고 하면, "방(Room)"이라는 개념이 필요하다.\
방은 이름을 가진다.

``` python
class Room(object):
    def __init__(self, name):
        self.name = name
```

### 인스턴스 생성

``` python
>>> white_room = Room("White Room")
>>> white_room.name
'White Room'
```

하지만 이것만으로는 다양한 기능을 구현하기 어렵다.\
따라서 상속을 이용해 기능을 추가/변경한 서브클래스를 만들어보자.

------------------------------------------------------------------------

## WhiteRoom 클래스

``` python
class WhiteRoom(Room):
    def __init__(self):
        # 모든 WhiteRoom은 이름이 'White Room'이다.
        self.name = 'White Room'

    def interact(self, line):
        if 'test' in line:
            print "'Test' to you, too!"
```

### 사용 예시

``` python
>>> white_room = WhiteRoom()
>>> white_room.interact('test')
'Test' to you, too!
```

------------------------------------------------------------------------

## RedRoom 클래스

``` python
class RedRoom(Room):
    def __init__(self):
        self.name = 'Red Room'

    def interact(self, line):
        global current_room, white_room
        if 'white' in line:
            # 기존 WhiteRoom을 새로 만들지 않고 그대로 이동
            current_room = white_room
```

### 방 이동 테스트

``` python
>>> red_room = RedRoom()
>>> current_room = red_room
>>> current_room.name
'Red Room'

>>> current_room.interact('go to white room')
>>> current_room.name
'White Room'
```

연습 문제: WhiteRoom에도 `interact`를 수정하여 다시 RedRoom으로 돌아갈
수 있게 만들어보자.

------------------------------------------------------------------------

## 게임 실행 루프

사용자가 입력한 문자열을 받아서, 현재 방의 `interact` 메서드를 실행한다.

``` python
def play_game():
    global current_room
    while True:
        line = raw_input(current_room.name + '> ')
        current_room.interact(line)
```

------------------------------------------------------------------------

## 게임 리셋 함수

게임을 다시 시작할 수 있도록 초기화한다.

``` python
def reset_game():
    global current_room, white_room, red_room
    white_room = WhiteRoom()
    red_room = RedRoom()
    current_room = white_room
```

------------------------------------------------------------------------

## 전체 실행 예시

``` python
>>> import mygame
>>> mygame.reset_game()
>>> mygame.play_game()
White Room> test
'Test' to you, too!
White Room> go to red room
Red Room> go to white room
White Room>
```

------------------------------------------------------------------------

## 스크립트 실행 가능하게 만들기

``` python
def main():
    reset_game()
    play_game()

if __name__ == '__main__':  # 스크립트로 실행할 경우만 동작
    main()
```

------------------------------------------------------------------------

## 요약

-   클래스는 데이터(속성)와 기능(메서드)을 묶는 구조다.
-   `__init__`은 객체 초기화를 담당한다.
-   여러 인스턴스는 서로 독립적으로 동작한다.
-   상속을 통해 부모 클래스의 기능을 확장할 수 있다.
-   간단한 텍스트 기반 방 이동 게임을 구현할 수 있다.
