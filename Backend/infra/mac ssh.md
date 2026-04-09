---
tags:
  - backend
  - infra
  - networking
---

ssh터널링
```
ssh -L 5432:엔드포인트:5432 aws-entry-1 -N
```

pem키 
```
cd ~/.ssh/pem.pem
```

설정
```
nano ~/.ssh/config
```

```
Host traefik
    HostName 공인IP
    User ubuntu
    IdentityFile ~/.ssh/pem.pem

Host infra
    HostName 내부IP
    User ubuntu
    IdentityFile ~/.ssh/pem.pem
    ProxyJump aws-traefik
```
## 관련 문서
- [[AWS EC2]]
- [[ubuntu]]
- [[openvpn]]
