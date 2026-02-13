[[TimescaleDB]]
[[PostgresBackup]]



## PostgreSQL tcp_keepalives 설정

OS 레벨 TCP keepalive를 PostgreSQL 연결에 적용하는 설정. Swarm 네트워크가 유휴 연결을 몰래 끊는 걸 감지하는 역할.

postgres.conf
- **`tcp_keepalives_idle = 60`**: 연결이 60초간 유휴 상태이면 keepalive 패킷 전송 시작. 기본값은 OS 따라 다르지만 보통 7200초(2시간)라서 Swarm 환경에선 너무 김.
- **`tcp_keepalives_interval = 10`**: keepalive 패킷에 응답이 없으면 10초 간격으로 재전송.
- **`tcp_keepalives_count = 3`**: 3번 연속 응답이 없으면 연결이 끊어진 것으로 판단하고 서버 쪽에서 연결 정리.