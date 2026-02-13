
## Swarm 초기화 / 관리

```bash
# 스웜 초기화
docker swarm init

# 스웜 해제
docker swarm leave --force

# 워커 노드 조인 토큰 확인
docker swarm join-token worker

# 매니저 노드 조인 토큰 확인
docker swarm join-token manager
```

## 노드 관리

```bash
# 노드 목록
docker node ls

# 노드 상세 정보
docker node inspect <NODE_ID>

# 노드 라벨 추가
docker node update --label-add key=value <NODE_ID>

# 노드 라벨 제거
docker node update --label-rm key <NODE_ID>

# 노드 유지보수 모드 (새 태스크 할당 중지)
docker node update --availability drain <NODE_ID>

# 노드 활성화
docker node update --availability active <NODE_ID>
```

## 스택 (Stack)

```bash
# 스택 배포
docker stack deploy -c docker-stack.yml <STACK_NAME>

# 여러 compose 파일 병합 배포
docker stack deploy -c base.yml -c override.yml <STACK_NAME>

# 프라이빗 레지스트리 사용 시
docker stack deploy -c docker-stack.yml --with-registry-auth <STACK_NAME>

# 스택 목록
docker stack ls

# 스택 서비스 목록
docker stack services <STACK_NAME>

# 스택 내 태스크(컨테이너) 목록
docker stack ps <STACK_NAME>

# 스택 삭제
docker stack rm <STACK_NAME>
```

## 서비스 (Service)

```bash
# 서비스 목록
docker service ls

# 서비스 상세
docker service inspect <SERVICE_NAME>

# 서비스 로그
docker service logs <SERVICE_NAME>
docker service logs -f <SERVICE_NAME>          # follow
docker service logs --tail 100 <SERVICE_NAME>  # 최근 100줄

# 서비스 스케일
docker service scale <SERVICE_NAME>=3

# 서비스 업데이트 (이미지 변경)
docker service update --image <IMAGE:TAG> <SERVICE_NAME>

# 서비스 강제 재시작
docker service update --force <SERVICE_NAME>

# 서비스 태스크 목록
docker service ps <SERVICE_NAME>

# 실패한 태스크 포함 전체 보기
docker service ps --no-trunc <SERVICE_NAME>

# 서비스 삭제
docker service rm <SERVICE_NAME>
```

## 네트워크 (Network)

```bash
# overlay 네트워크 생성
docker network create --driver overlay <NETWORK_NAME>

# attachable overlay (외부 컨테이너도 연결 가능)
docker network create --driver overlay --attachable <NETWORK_NAME>

# 네트워크 목록
docker network ls

# 네트워크 상세 (연결된 서비스 확인)
docker network inspect <NETWORK_NAME>

# 네트워크 삭제
docker network rm <NETWORK_NAME>
```

## secret / config

```bash
# 시크릿 생성
echo "password" | docker secret create my_secret -
docker secret create my_secret ./secret_file

# 시크릿 목록
docker secret ls

# 시크릿 삭제
docker secret rm <SECRET_NAME>

# 컨피그 생성
docker config create my_config ./config_file

# 컨피그 목록
docker config ls

# 컨피그 삭제
docker config rm <CONFIG_NAME>
```

## 디버깅

```bash
# 특정 서비스가 뜨지 않을 때 원인 확인
docker service ps --no-trunc <SERVICE_NAME>

# 컨테이너 직접 접속
docker exec -it <CONTAINER_ID> sh

# 노드별 실행 중인 컨테이너 확인
docker node ps <NODE_ID>

# 전체 시스템 상태
docker system df
docker system info
```

## restart_policy 옵션

| condition | 동작 |
|-----------|------|
| `any` | 항상 재시작 |
| `on-failure` | 비정상 종료(exit != 0)만 재시작 |
| `none` | 재시작 안함 |

## 참고

- 스택 이름은 서비스/네트워크 이름에 prefix로 붙음: `{stack}_{service}`
- `docker stack deploy`는 변경 사항만 업데이트 (기존 서비스 유지)
- 네트워크에 `name:` 지정하면 스택 prefix 없이 고정 이름 사용 가능


