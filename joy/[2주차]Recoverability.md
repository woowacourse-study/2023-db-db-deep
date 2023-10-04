# Concurrency Control 기초 2 - Recoverability

## 학습 주제
- Recoverable Schedule
- Cascadeless Schedule
- Strict Schedule


<br><br><br><br>

## 가정
> 여기 100만원을 가진 `조이`와 200만원을 가진 `리버`라는 사람이 있다.  
> `조이`가 `리버`에게 20만원을 이체할 때, `리버`도 자신의 계좌에 30만원을 입금하는 상황을 생각해보자.

![Concurrency Control 기초 2-1.jpeg](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%202-1.jpeg)

위 상황은 어떤 문제가 발생한 것일까?  
좀 더 자세하게 살펴보자.  

![Concurrency Control 기초 2-2.jpeg](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%202-2.jpeg)

- 리버가 먼저 자신의 계좌에 30만원을 입금하여 리버 계좌의 총 금액이 230만원이 되었다. 하지만 아직 커밋(반영)하지는 않았다.
- 그 사이에 조이가 리버에게 20만원 이체하여 230 + 20 으로 총 금액을 250만원으로 만들고 커밋(반영)하였다.
- 그런데 리버가 커밋하기 전에 어떤 문제가 발생하여 롤백(작업 취소)을 진행하였다.
- 정상적인 상황이라면 리버의 계좌는 총 금액이 220만원 이어야 한다. 그런데 여전히 총 금액이 250만원인 문제가 발생했다.  

위와 같은 현상이 발생하는 스케줄을 `Unrecoverable Schedule`이라고 한다.  

<br><br><br><br>
## Unrecoverable Schedule
-  Schedule 내에서 커밋된 트랜잭션이 롤백된 트랜잭션이 `write` 했었던 데이터를 읽은 경우에 해당한다. 즉, 유효하지 않은 데이터를 읽는 경우를 말한다.  
-  롤백을 해도 이전 상태로 회복이 불가능할 수 있기 때문에, 이런 스케줄은 DBMS가 허용하면 안 된다.
-  이 스케줄로 데이터 불일치가 발생할 수 있다.
-  이렇게 Schedule 내에서 이미 다른 트랜잭션이 커밋된 상태이면 durablity 속성 때문에 롤백이 불가능하다.

이러한 이유로 `Unrecoverable Schedule`이 아닌 `Recoverable Schedule`을 허용해야 한다.  

<br><br><br><br>
## Recoverable Schedule
- 같은 스케줄 내에서 어떤 트랜잭션도 자신이 읽은 데이터를 `write`한 트랜잭션이 먼저 `commit`/`rollback` 전까지 커밋 하지 않은 경우에 해당한다. 
- 의존성이 있을 경우, 의존의 대상이 되는 트랜잭션(예시 `트랜잭션1`)이 종료될 때까지 의존하는 트랜잭션(예시 `트랜잭션2`)은 커밋하면 안 된다.
  - 예시에서 `write(리버_balance = 230)` 경우를 생각해볼 수 있음  
- 롤백할 때 이전 상태로 온전히 돌아갈 수 있기 때문에 DBMS는 이런 스케줄만 허용해야 한다.

<br><br><br><br>
## Cascading Rollback
- 하나의 트랜잭션이 롤백하면 의존성이 있는 다른 트랜잭션도 롤백 해야 한다.
- 하지만 여러 트랜잭션의 롤백이 연쇄적으로 일어나면 처리하는 비용이 많이 든다.

그럼 데이터를 `write`한 트랜잭션이 `commit`/`rollback`한 뒤에 데이터를 읽는 스케줄(`Cascadeless Schedule`)만 허용하면 어떨까?  
그럼 의존하는 트랜잭션이 없기 때문에 그냥 롤백 하면 된다!  

<br><br><br><br>
## Cascadeless Schedule 
- `avoid cascading rollback` 이라고도 한다.  
- 스케줄 내에서 어떤 트랜잭션도 커밋 되지 않은 트랜잭션들이 `write` 한 데이터는 읽지 않는 경우를 말한다.  

`Cascadeless Schedule`은 약간의 문제가 발생할 수 있다.  
아래와 같은 문제를 생각해 볼 수 있다.  

> 어느 피자집의 피자 가격이 3만원이다.
> 이 피자 가격을 2만원으로 낮추려는데, 직원은 실수로 1만원으로 낮추고 사장님은 2만원으로 낮추는 상황을 생각해보자.  

```text
피자 가격: 3만원

-----------------------  transaction 1 (직원)
t1_write(pizza = 1만원)
........................ transaction 2 (사장)
t2_write(pizza = 2만원)
t2_commit
........................
t1_abort
------------------------

피자 가격: 3만원
```

`Casecadeless Schedule`을 사용했지만 사장님 작업(트랜잭션2)의 결과 사라지는 문제가 발생한다.  
이 문제를 해결하기 위해 `Stric Schedule`을 사용할 수 있다.  

<br><br><br><br>
## Strict Schedule 
- 스케줄 내에서 어떤 트랜잭션도 커밋 되지 않은 트랜잭션들이 `write`한 데이터는 쓰지도 읽지도 않는 경우
- 롤백 할 때 recovery가 쉽다.  

```text
피자 가격: 3만원

-----------------------  transaction 1 (직원)
t1_write(pizza = 1만원)
t1_abort
------------------------
........................ transaction 2 (사장)
t2_write(pizza = 2만원)
t2_commit
........................

피자 가격: 2만원
```

<br><br><br><br>
## 정리

![Concurrency Control 기초 2-3.jpeg](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%202-3.jpeg)


- `Unrecoverable Schedule`은 롤백이 발생했을 때 회복 불가능할 수  있기 때문에, DBMS에서 허용하면 안된다.
- `Recoverable Schedule`은 롤백이 발생했을 때 회복 가능하기 때문에 이런 스케줄은 DBMS에서 허용해줘도 된다. 
- `Recoverable schedule` 중 `cascade rollback`이 발생하지 않도록 하는 스케줄이 `Cascadeless Schedule` 이다.
- `Cascadeless Schedule` 중에서도 더 엄격한 스케줄을 `Strict Schedule` 이라고 한다 (`read` X /`write` X).
- `Recoverable Schedule`은 `Recoverability` 속성을 가진다.
- 동시성 제어(Concurrency Control)는 `Serializability`와 `Recoverability`를 제공한다.
- 이와 관련된 트랜잭션의 속성이 `isolation`이다.

<br><br><br><br>

## 참고 자료
[[쉬운 코드] concurrency control 기초: recoverability](https://youtu.be/89TZbhmo8zk?si=TidfuP_oQAy-lDmg)
