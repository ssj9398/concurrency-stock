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