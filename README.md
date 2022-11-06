## 환경세팅

1. mysql 설치 및 실행
```
docker pull mysql
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 --name mysql mysql 
docker ps
```

2. mysql 데이터베이스 생성
```
docker exec -it mysql bash
mysql -u root -p
create database stock_example;
use stock_example;
```

#### 아래와 같은 오류가 날 경우
docker: no matching manifest for linux/arm64/v8 in the manifest list entries.
```
docker pull --platform linux/x86_64 mysql
```

### 문제점
1. Excutors란?
- 멀티쓰레드이용을 위해 사용
- 비동기로 실행하는 작업을 단순화하여 사용 할수 있게 해주는 자바의 API

2. CountDownLatch란?
- 100개의 요청이 끝날때 까지 기다려야 하므로 CountDownLatch 사용
- 다른쓰레드에서 수행중인 작업이 완료될때까지 대기할수있도록 도와주는 클래스


3. Race Condition이 발생
- 하나의 쓰레드 완료 후 다른쓰레드가 접근할수 있게 해결 할수있음

4. synchronized
- 손쉽게 하나의 쓰레드만 접근하게 할수있음
- 그래도 안됨 이유는 @Transactional 어노테이션 때문
    - 수량 감소 method호출 후 업데이트를 하기 직전에 다른쓰레드가 해당 method에 접근이 가능해진다.
    - 쉽게설명하면 수량감소method를 호출 후 udpate쿼리가 날아가기전 다른 쓰레드가 접근 할 수 있어서

5. java synchronized 문제점
- 서버가 1대일때는 되는듯싶으나 여러대의 서버를 사용하게되면 사용하지 않았을때와 동일한 문제가 발생
- 인스턴스단위로 thread-safe 이 보장이 되고, 여러서버가 된다면 여러개의 인스턴스가 있는것과 동일하기때문

### 해결방법
#### Database를 활용하여 레이스컨디션 해결해보기
1. Optimistic Lock
- lock 을 걸지않고 문제가 발생할 때 처리
- 대표적으로 version column 을 만들어서 해결하는 방법
#### 장점
- Optimistic의 경우 별도의 락을 잡지않으므로 Pessimistic보다 성능상 이점이 있다.
#### 단점
- update 실패했을경우 재시도 해주는 로직을 직접 개발자가 작성해주어야 한다.
- 충돌이 잦은경우 Pessimistic이 더 좋을 수 있다.
2. Pessimistic Lock (exclusive lock)
- 다른 트랜잭션이 특정 row 의 lock 을 얻는것을 방지
- A 트랜잭션이 끝날때까지 기다렸다가 B 트랜잭션이 lock 을 획득
- 특정 row 를 update 하거나 delete 할 수 있습니다.
- 일반 select 는 별다른 lock 이 없기때문에 조회는 가능합니다.
- row or table 단위로 건다
#### 장점
- 충돌이 빈번하게 일어난다면 Optimistic Lock보다 성능좋다.
- 락을 통해 Update를 제어하기 때문에 데이터 정합성이 어느정도 보장된다.
#### 단점
- 별도의 락을 걸기때문에 성능감소가 있을 수 있다.

3. named Lock 활용하기
- 이름과 함께 lock을 획득 해당 lock 은 다른세션에서 획득 및 해제가 불가능
- 메타데이터의 마킹을 하는 방법

```
https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_exclusive_lock
https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html
https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html
```