
로컬 포트포워딩
```bash
ssh -L {로컬포트}:{타겟호스트}:{타겟포트} -N -f {ssh설정명}
# -L : Local Port Forwarding
# - N : 명령실행없이 포워딩만
# -f : 백그라운드실행
```
