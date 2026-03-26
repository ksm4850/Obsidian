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