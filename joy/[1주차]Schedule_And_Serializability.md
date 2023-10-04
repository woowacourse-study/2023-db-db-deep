# Concurrency Control 기초 : Schedule과 Serializability  

## 가정 
>여기 100만원을 가진 조이와 200만원을 가진 도이라는 사람이 있다.  
조이가 도이에게 20만원을 이체할 때, 리버도 자신의 계좌에 30만원을 입금하는 상황을 생각해보자.  
  
![Concurrency Control 기초 1 -1.jpeg](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201%20-1.jpeg)

- 조이가 도이의 계좌의 금액을 확인했다(200만원)
- 도이도 자신의 계좌의 금액을 확인했다 (200만원)
- 도이가 자신의 계좌에 30만원을 입금하고자, 기존 금액에 30만원 더한 금액으로 계좌의 총액을 변경했다(230만원)
- 조이가 도이에게 20만원 이체하고자, 기존에 읽은 계좌 금액에 20만원을 더한 금액으로 도이의 계좌 총액을 변경했다(220만원)
- 정상적인 상황이라면 도이의 계좌 총액은 250만원이어야 한다. 즉, 도이가 변경한 작업이 사라진 것이다.
- 이렇게 업데이트한 데이터가 사라져 버리는 현상을 `Lost Update`라고 한다. 

그렇다면 이러한 문제를 어떻게 해결할 수 있을까?  
관련 개념들과 함께 그 방법들을 알아보자.  

<br><br><br><br>

## Operation
> 트랜잭션 내의 하나의 실행 단위를 Operation이라고 한다.  
> 아래와 같이 간소화하여 나타낼수도 있다.

![Concurrency Control 기초 1-2.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-2.png)

<br><br><br><br>

## Schedule
> 여러 트랜잭션들이 동시에 실행될 때, 각 트랜잭션에 속한 operation들의 실행 순서이다.  
> 각 트랜잭션 내의 operation 들의 순서는 바뀌지 않는다.  

앞선 예시에 적용될 수 있는 스케줄 케이스들을 생각해보고, 각 스케줄 종류에 대해서 알아보자.    
스케줄 종류를 알아보기 위한 스케줄 케이스들은 아래와 같다.  

![Concurrency Control 기초 1-3.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-3.png)

<br><br><br><br>

### Serial Schedule
> 케이스 1과 2 처럼 트랜잭션들이 겹치지 않고 한 번에 하나씩 실행되는 스케줄을 말한다.  

![Concurrency Control 기초 1-4.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-4.png)

한 번에 하나의 트랜잭션만 실행되기 때문에, 좋은 성능을 내기 어렵고 현실적으로 사용할 수 없는 방식이다.  
즉, 동시성이 없어서 빠르게 처리가 불가능하다.  


<br><br><br><br>

### Nonserial Schedule
> 트랜잭션이 겹쳐서(interleaving) 실행되는 스케줄을 말한다.

![Concurrency Control 기초 1-5.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-5.png)

트랜잭션들이 겹쳐서 실행되기 때문에 동시성이 높아져서 같은 시간 동안 더 많은 트랜잭션들을 처리할 수 있다.  
하지만 트랜잭션들이 어떤 형태로 겹쳐서 실행되는지에 따라 이상한 결과가 나올 수 있다.  
(예시의 스케줄 4에서 발생한 Lost Update를 생각해볼 수 있다)  

<br><br><br><br>

## 고민해볼 점
성능을 고려하면 여러 트랜잭션들을 겹쳐서 실행할 수 있으면 좋을 것 같다.  
하지만 잘못하면 이상한 결과가 나올 수 있다.  
동시에 여러 트랜잭션을 실행하면서, 결과는 올바르게 나올 수 있는 방법은 없을까?🤔  

그렇다면 `Serial Schedule`과 동일한(`equivalent`) `Nonserial Schedule`을 실행할 수 있으면 되겠다.  
그럼 먼저 Schedule이 동일하다는 의미가 무엇인지 정의해보자.  


<br><br><br><br>

## Conflict of Two Operations
> 스케줄의 동일성에 대해 알아보기 전에 `conflict`에 대해 먼저 알아보자.  
> 두 개의 `operation`에 대해서 사용하는 개념이다.  
> `conflict operation`은 순서가 바뀌면 결과도 바뀐다.    
> 아래 3가지 조건을 만족하면 `conflict` 상태로 간주한다.  

- 서로 다른 트랜잭션의 소속이다.
- 같은 데이터에 접근한다.
- 최소 하나의 오퍼레이션은 `write`이다.

![Concurrency Control 기초 1-6.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-6.png)

<br><br><br><br>


## Conflict Equivalent for Two Schedule
> 이제 `conflict`를 통해 동일성에 대해 알아보자.
> 서로 다른 스케줄 간에 아래 두 조건을 만족하면 `conflict equivalent 하다고 볼 수 있다.

- 두 스케줄은 같은 트랜잭션들을 가진다.
- 어떤 conflicting operations의 순서도 양쪽 스케줄 모두 동일하다.

<br><br><br><br>

### Conflict Serializable
> serial shedule과 conflict equivalent 일 때 conflict serializable 하다고 한다.
> 앞에서 확인한 4개의 스케줄을 다시 확인해보고, conflict serializable한 스케줄이 있는지 찾아보자!

![Concurrency Control 기초 1-3.png](img%2FConcurrency%20Control%20%EA%B8%B0%EC%B4%88%201-3.png)

위 그림에서 serializable한 스케줄은 case 1과 case2 이다.  
case 3과 case 4를 이 스케줄들과 비교해보자.  
case 3은 case 2와 confrlict equivalent하다. 그러니 conflict serializable 하다고 볼 수 있다.  
case 4는 그 어떤 serial schedule과도 conflict equivalent 하지 않다. 그러니 not conflict serializable 하다.  

<br><br><br><br>

## 고민해볼 점

이전에 우리는 동시성과 일관성을 모두 가져가기 위해 아래와 같은 결론을 내렸다.  
> 그렇다면 `Serial Schedule`과 동일한(`equivalent`) `Nonserial Schedule`을 실행할 수 있으면 되겠다.

이에 대한 해결책으로 conflict serializable 한 nonserial schedule을 허용하는 방법을 생각해볼 수 있다.  
그렇다면 실제 DBMS에서 이 내용을 적용할 수 있을까? 
요청이 들어온 모든 스케줄들을 conflict serializable한지 확인하는 것은 비용이 많이들 것이다.  

다른 방법으로 여러 트랜잭션을 동시에 실행해도 스케줄이 `conflict serializable` 하도록 보장하는 프로토콜을 적용하는 방법이 있다.  

<br><br><br><br>

## Serializabilty
- 어떤 스케줄이 serial schedule과 equivalent 하다면 conflict serializable 하다고 하거나 conflict serializability 한 속성을 가진다고 한다.
- 어떤 스케줄이 view equivalent 하다면 view  serializable 하다고 하거나 view  serializability 한 속성을 가진다고 한다.
 
<br><br><br><br>

## Concurrency Control
> concurreny control 은 그 어떤 스케줄도 serializable하게 만드는 것이다.
> 이 것과 관련된 트랜잭션의 속성이 `isolation` 이다.

isolation을 너무 엄격하게 지한하면 그 만큼 성능이 떨어지고 동시성이 떨어질 것이다.  
완화된 isolation을 제공해야하고, 그 방법으로 격리 수준 제어를 생각해볼 수 있다.  


<br><br><br><br>

## 참고 자료
- [쉬운 코드 - concurrency control 기초 - schedule과 serializability](https://youtu.be/DwRN24nWbEc?si=W_SevurH_fyurcwD)
- [[DB] 동시성 제어(Concurrency control) 1편](https://velog.io/@daen_h/DB-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%A0%9C%EC%96%B4-Concurrency-control-1%ED%8E%B8)



