# 5.4 MySQL의 격리 수준
- 트랜잭션의 격리 수준이란 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경되거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것이다.

||DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
|--------------------|----|---|----|
|**READ UNCOMMITEED**|발생|발생|발생|
|**READ COMMITTED**  |없음|발생|발생|
|**REPEATABLE READ** |없음|없음|발생<br>(InooDB는 없음)|
|**SERIALIZABLE**    |없음|없음|없음|

## 5.4.1 READ UNCOMMITTED
- 홍길동이 INSERT를 하고 있는 도중에 이순신이 SELECT를 하게되면 이순신은 2건의 결과를 반환 받게 된다. 여기서 홍길동이 어떠한 문제가 발생해 INSERT된 내용을 롤백한다고 해도 이순신은 여전히 2건의 결과를 반환 받게 된다.
- 이처럼 어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 불구하고 다른 트랜잭션에세 볼 수 있는 현상을 더티 리드하고 하며 더티 리드가 허용되는 격리 수준이 READ UNCOMMITTED이다.  
![READ UNCOMMITTED](./images/READ%20UNCOOMITTED.png)


## 5.4.2 READ COMMITED
- 이 레벨에서는 더티 리드와 같은 현상은 발생하지 않다. 어떤 트랜잭션에서 데이터를 변경했더라도 COMMIT이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있다.
- 이순신의 SELECT 결과는 Member 테이블이 아닌 언두 영역에 백업된 레코드에서 가져온 내용이다.  
![READ COMMITED](./images/READ%20COMMITED.png)
     

## 5.4.3 REPEATABLE READ
- REPEATABLE READ는 MySQL의 InnoDB 스토리 엔진에서 기본적으로 사용되는 격리 수준이다. 이 격리 수준에서는 READ COMMITTED 격리 수준에서 발생하는 NON-REPEATABLE READ 부정합성은 발생하지 않는다. 
- REPEATABLE READ는 MVCC를 위래 언두 영역에 백업된 이전 데이터를 이용해 동일 트랜잭션 내에서는 동일한 결과를 보여줄 수 있게 보장한다. 사실 READ COMMITTED도 MVCC를 이용해 COMMIT되기 전의 데이터를 보여주는데 READ COMMITTED와 REPEATABLE READ의 차이점은 언두 영역에 백업된 레코드의 여러 버전 가운데 몇 번째 이전 버전까지 찾아 들어가느냐에 달려있다.
- 모든 InnoDB의 트랜잭션은 고유한 트랜잭션 번호(순차적으로 증가하는 값)를 가지며, 언두 영역에 백업된 모든 레코드에는 변경을 발생시킨 트랜잭션 번호가 포함되어 있다. 그리고 언두 영역의 백업된 데이터는 InnoDB 스토리지 엔진이 불필요하다고 판단하는 시점에 주기적으로 삭제한다.
- 아래 그림에서는 홍길동의 트랜잭션 아이디는 13이고 이순신의 트랜잭션 아이디는 10입니다. 이때 홍길동이 광개토대왕의 도시를 서울특별시로 변경하고 커밋을 한다. 그런데 이순신이 2번의 SELECT를 하지만 결과값이 동일한 것을 볼 수 있다. 그 이유는 이순신의 트랜잭션 아이디는 10이므로 10번 트랜잭션 안에서 실행되는 모든 SELECT 쿼리는 트랜잭션 번호가 10보다 작은 트랜잭션 번호에서 변경한 것만 보게된다.  
![REPEATABLE READ](./images/REPEATABLE%20READ.png)


## 5.4.4 SERIALIZABLE
- 가장 단순한 격리 수준이며 동시에 가장 엄격한 격리 수준이다. 그만큼 동시 처리 성능도 다른 트랜잭션 격리 수준보다 떨어진다.