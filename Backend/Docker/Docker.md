---
tags: [backend, docker]
---
[[lazydocker]]
Docker 관리: [[Portainer]] | 데몬 설정: [[docker daemon]]
자주쓰는 명령어

```
컨테이너별 네트워크 io 트래픽 확인
docker stats --no-stream --format "table {{.Name}}\t{{.NetIO}}"
```

```
docker compose up -d --force-recreate
```
```bash
docker pull 주소
docker push 이름
```

```bash
# 원본을 짧은 이름으로 태그 
docker tag registry.gitlab.com/myteam/backend:latest backend:latest
```
```bash
docker buildx 빌드 \  
--provenance= false \  
-t " $IMAGE_NAME : $TAG " \  
-f examples/example01/target.Dockerfile \  
--platform linux/amd64,linux/arm64 \  
--target "debian" \  
--push \  
--build-arg "GITLAB_PROXY= ${CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX} /" .
```

멀티노드 배포는 [[Docker Swarm]] 참고.

## 관련 문서
- [[Docker Swarm]]
- [[Portainer]]
- [[docker daemon]]
- [[lazydocker]]
- [[dozzle]]