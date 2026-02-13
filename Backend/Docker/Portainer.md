
## 1. Portainer란?

Docker 및 Kubernetes 환경을 **웹 UI로 관리**할 수 있는 오픈소스 컨테이너 관리 도구. CLI 없이 컨테이너, 이미지, 볼륨, 네트워크 등을 시각적으로 관리할 수 있다.

- **CE (Community Edition)**: 무료, 소규모/개인 서버에 적합
- **BE (Business Edition)**: 유료, RBAC·레지스트리 관리 등 엔터프라이즈 기능 포함

---

## 2. 설치

### 2-1. Docker 환경 설치

```bash
# 볼륨 생성
docker volume create portainer_data

# 컨테이너 실행
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

|포트|용도|
|---|---|
|9443|웹 UI (HTTPS)|
|8000|Edge Agent 통신 (선택)|

### 2-2. Docker Compose 설치

```yaml
# docker-compose.yml
version: "3.8"
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - "8000:8000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

```bash
docker compose up -d
```

### 2-3. 초기 설정

1. `https://localhost:9443` 접속
2. admin 계정 생성 (비밀번호 12자 이상)
3. Environment → **Local** 선택

> ⚠️ 최초 접속을 5분 내에 하지 않으면 보안상 초기화됨. 컨테이너 재시작 필요.

---

## 3. 주요 메뉴 및 기능

### 3-1. Dashboard

현재 환경의 전체 현황 한눈에 확인 (컨테이너 수, 이미지 수, 볼륨, 네트워크 등).

### 3-2. Containers

|기능|설명|
|---|---|
|목록 조회|실행 중/중지된 컨테이너 확인|
|로그 확인|`docker logs` 와 동일|
|Console|컨테이너 내부 쉘 접속 (`docker exec -it`)|
|Inspect|컨테이너 상세 설정 확인|
|Start/Stop/Restart/Remove|컨테이너 제어|

### 3-3. Images

- 로컬 이미지 목록 확인
- 이미지 pull / 삭제
- 사용하지 않는 이미지 정리

### 3-4. Volumes

- 볼륨 생성 / 삭제
- 사용 중인 볼륨 확인
- 볼륨 브라우저로 내부 파일 확인 (BE 전용)

### 3-5. Networks

- Docker 네트워크 목록 조회
- 커스텀 네트워크 생성
- 컨테이너 간 네트워크 연결 관리

### 3-6. Stacks

Docker Compose와 동일한 기능을 웹 UI에서 관리.

```yaml
# 예시: Portainer에서 Stack 배포
version: "3.8"
services:
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    restart: always
```

**Stack 배포 방법:**

1. Stacks → Add Stack
2. Web editor에 docker-compose.yml 내용 입력
3. Deploy the stack 클릭

### 3-7. App Templates

미리 정의된 템플릿으로 빠르게 앱 배포 (Nginx, Redis, PostgreSQL 등).

---

## 4. 실전 예시

### 예시 1: PostgreSQL + pgAdmin Stack 배포

Stacks → Add Stack에서 아래 내용 입력:

```yaml
version: "3.8"
services:
  postgres:
    image: postgres:16
    container_name: my-postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: my-pgadmin
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin1234
    ports:
      - "5050:80"
    depends_on:
      - postgres

volumes:
  pg_data:
```

### 예시 2: FastAPI 앱 컨테이너 관리

```yaml
version: "3.8"
services:
  fastapi-app:
    image: my-fastapi-app:latest
    container_name: fastapi-app
    restart: always
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://admin:secretpassword@postgres:5432/mydb
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    container_name: postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: mydb
    volumes:
      - pg_data:/var/lib/postgresql/data

volumes:
  pg_data:
```

### 예시 3: 컨테이너 로그 확인

1. Containers 메뉴 진입
2. 대상 컨테이너 클릭
3. **Logs** 탭 클릭
4. Auto-refresh 켜면 실시간 로그 확인 가능

### 예시 4: 컨테이너 내부 접속

1. Containers → 대상 컨테이너 클릭
2. **Console** 탭 클릭
3. Command: `/bin/bash` 또는 `/bin/sh` 선택
4. **Connect** 클릭

---

## 5. 자주 쓰는 관리 명령어

|작업|CLI|Portainer|
|---|---|---|
|컨테이너 목록|`docker ps -a`|Containers 메뉴|
|로그 확인|`docker logs <name>`|컨테이너 → Logs|
|쉘 접속|`docker exec -it <name> bash`|컨테이너 → Console|
|스택 배포|`docker compose up -d`|Stacks → Add Stack|
|이미지 정리|`docker image prune`|Images → Remove unused|
|볼륨 확인|`docker volume ls`|Volumes 메뉴|

---

## 6. 업데이트

```bash
# 기존 컨테이너 중지 및 삭제
docker stop portainer
docker rm portainer

# 최신 이미지 pull
docker pull portainer/portainer-ce:latest

# 재실행 (볼륨은 유지되므로 설정 보존)
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

---

## 7. 팁

- **SSL 인증서**: 기본은 self-signed. 실서비스에선 `--sslcert`, `--sslkey` 옵션으로 인증서 지정
- **원격 Docker 관리**: Environment → Add Environment로 원격 Docker 호스트 추가 가능
- **백업**: `portainer_data` 볼륨만 백업하면 설정 전체 복원 가능
- **Agent 모드**: 원격 서버에 Portainer Agent만 설치하고 중앙에서 관리 가능

```bash
# Agent 설치 (원격 서버)
docker run -d \
  -p 9001:9001 \
  --name portainer_agent \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```