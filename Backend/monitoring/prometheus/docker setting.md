```yml
prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - /mnt/disk1/monitoring/prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--storage.tsdb.retention.size=10GB'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    restart: unless-stopped
    
# '--storage.tsdb.retention.time=30d' 30일까지 보관
# '--storage.tsdb.retention.size=10GB' 10GB넘으면 오래된것부터 삭제
# '--web.enable-lifecycle'  HTTP API로 종료, 재시작 할 수 있는 옵션
# '--web.enable-admin-api'  관리용 API 엔드포인트 생성
# https://prometheus.io/docs/prometheus/latest/querying/api/

```