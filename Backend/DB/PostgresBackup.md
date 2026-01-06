```sh
docker run -d --name postgres_backup_local -e POSTGRES_HOST=223.130.155.240 -e POSTGRES_PORT=5432 -e POSTGRES_DB=slam -e POSTGRES_USER=slam -e POSTGRES_PASSWORD=pilab-240 -e SCHEDULE=@hourly -e BACKUP_KEEP_DAYS=1 -e POSTGRES_EXTRA_OPTS=-Fc -e BACKUP_SUFFIX=.dump -v $(pwd)/backups:/backups --restart unless-stopped prodrigestivill/postgres-backup-local:16
```


- -T
	-  터미널 할당 비활성화
	- 파이프나 리다이렉션 사용시 필수
- -U
	- User : 접속할 PogsteSQL사용자명
- -d
	- database : 복원대상 데이터베이스 이름
- `--if-exists`
	- 삭제할 객체가 없어도 에러 무시
	- `--clean`과 함께 사용 
	- 안전하게 보원가능
- `--clean`
	- 복원 전에 기존 객체 삭제
	- DROP 문 먼저 실행 후 CREATE 실행
	- 깨끗한 상태로 복원
```sh
docker-compose exec -T postgres pg_restore -U {유저명} -d {DB명} --if-exists --clean < {dump파일 위치}
```