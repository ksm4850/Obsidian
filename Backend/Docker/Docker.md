[[lazydocker]]
자주쓰는 명령어

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