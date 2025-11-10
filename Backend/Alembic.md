
데이터베이스 마이그레이션 하는 경우 사용하는 라이브러리

마이그레이션 하는경우

1. 클라우드 사용시 다른 인스턴스, 다른 리전으로 이전
2. 특정 Entity 변경
3. 일부 데이터 명칭 변경

수동으로도 변경할 수 있지만, 어떻게 사용했고, 어떠한점을 변경했는지에 대한 추적이 어려워짐.

우리 서비스는 처음에 이렇게 사용했다가 이러한 점이 필요해서, 혹은 요청이 되서 라는 것들을 기록해야 할 수 있다.

또한 개발자가 Dev, Stage, Prod 등 같은 서비스를 여러개의 환경으로 나누어서 사용하는 경우 어떤 환경에 데이터베이스 마이그레이션이 이루어졌고, 이루어지지 않았는지 확인이 어렵다. 혹여라도 이미 마이그레이션이 진행된 인스턴스에 다시 한 번 마이그레이션이 진행된다면 중복이 발생하고, 이로 인한 데이터 변화에 대해 일부 데이터 소실에 대한 원인이 되기도 하며 오히려 생산성을 떨어뜨리게 되거나 심각하게는 서비스 장시간 장애로도 이어질 수 있음

Python에서는 alembic을 이용해 DB 마이그레이션 버전관리를 제공.


Init Alembic
```bash
alembic init 폴더명
```

version 파일 생성
```bash
# 새로운 revision 추가 
alembic revision 
alembic revision -m "메세지" # 메세지를 추가하여 등록

# alembic에서 변화를 자동으로 감지해서 revision 작성 
alembic revision --autogenerate 
alembic revision --autogenerate -m "메세지" 
# --autogenerate 옵션을 사용하지 않고 revision을 진행하면 
# 직접 변화를 작성해주어야 한다
```

데이터베이스 버전 바꾸기
```bash
# 가장 상단의 버전으로 db upgrade 
alembic upgrade head 
# 가장 하단의 버전으로 db downgrade 
alembic downgrade base 

# N 만큼 upgrade/downgrade 
alembic upgrade/downgrade +-N 

# 해당 revision ID로 upgrade/downgrade 
alembic upgrade/downgrade revisionID
```

변경 기록확인
```bash
alembic history
```
