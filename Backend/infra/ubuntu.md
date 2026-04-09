---
tags:
  - backend
  - infra
  - linux
---

기본 설치해야할것들

```bash
# 패키지 업데이트 
sudo apt update && sudo apt upgrade -y
# Docker 설치 
curl -fsSL https://get.docker.com | sudo sh

# ubuntu 유저에 Docker 권한 부여 
sudo usermod -aG docker ubuntu

# 깃 설치
sudo apt install git -y
```
Docker 설치 후 사용법은 [[Docker]] 참고

자주쓰ㅅ는거
```
ss -tnlp
```

## 관련 문서
- [[Docker]]
- [[AWS EC2]]
- [[mac ssh|SSH 설정]]
- [[리눅스 기본 명령어]]