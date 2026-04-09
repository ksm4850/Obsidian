# Obsidian Vault — 개인 지식 베이스

이 디렉토리는 사용자의 개인 개발 지식 정리 vault입니다. 사용자가 질문하면 **먼저 이 vault를 탐색**한 후 답변하세요.

## 답변 규칙

1. **Vault 우선 탐색**: 질문을 받으면 일반 지식으로 바로 답하지 말고, 먼저 vault를 탐색합니다.
2. **탐색 순서**:
   - `Backend/Backend MOC.md` 또는 `리눅스/리눅스 MOC.md` 확인 → 주제 카테고리 파악
   - `Grep`으로 키워드 검색 (frontmatter tags, 본문 모두)
   - 히트한 파일 `Read`
   - 해당 파일의 본문 내 `[[wiki-link]]`를 따라가서 추가 문맥 수집
   - 필요 시 BFS처럼 2-3 홉까지 탐색
3. **토큰 절약 탐색 규칙** (중요):
   - **Grep 먼저, Read는 최소화**: `Grep`에 `-n`, `-C 3` 옵션으로 매칭 라인만 확인. 파일 전체 Read는 꼭 필요할 때만.
   - **큰 파일 분할 읽기**: `Backend/gRPC/gRPC 가이드 기초.md`처럼 수천 줄짜리 파일은 `offset`/`limit`으로 관련 섹션만 읽기. 절대 전체를 한 번에 Read 하지 말 것.
   - **탐색 홉 제한**: 시작 파일에서 최대 **2홉**까지만 관련 문서를 따라감. 3홉부터는 관련성이 급락하므로 중단.
   - **MOC 먼저**: 주제가 애매하면 `Backend MOC.md`나 `리눅스 MOC.md`를 먼저 훑어 탐색 범위를 좁힌 후 Grep 실행.
   - **중복 Read 금지**: 이미 읽은 파일을 다시 읽지 않음.
4. **답변 형식**:
   - vault에서 찾은 내용은 파일 경로와 함께 인용 (예: `Backend/DB/Postgres.md:11`)
   - vault에 없는 내용은 일반 지식으로 보충하되, **"vault에는 없음"** 명시
   - 관련 문서 링크를 답변 끝에 제시하여 사용자가 원본을 바로 열 수 있게 함
5. **업데이트 제안**: 질문에 답변했는데 vault에 해당 정보가 없다면, "이 내용을 vault에 추가할까요?"라고 물어봅니다.

## Vault 구조

- `Backend/` — 백엔드 개발 전반
  - `Docker/` — Docker, Swarm, Portainer, daemon, lazydocker, dozzle
  - `DB/` — Postgres, TimescaleDB, Sqlalchemy, PostgresBackup
  - `Python/` — Alembic, Cython, Nuitka, Socket.io
  - `Redis/` — 설정, 명령어, 동시성
  - `infra/` — AWS EC2, AWS 알림, mac ssh, ubuntu
  - `monitoring/` — Grafana, Prometheus, dozzle
  - `통신/` — IPC, openvpn, 클라우드플레어 터널링
  - `gRPC/` — 가이드, proto 작성, 참고 사이트
  - `Git/` — submodule
  - `MinIo.md` — S3 호환 오브젝트 스토리지
- `리눅스/` — Linux 기본 명령어, 권한, 파일시스템, LVM, 포트
- `Backend/Backend MOC.md`, `리눅스/리눅스 MOC.md` — **Map of Content, 탐색 시작점**

## 링크 규약

- `[[파일명]]` — 다른 파일 참조
- `[[파일명|표시텍스트]]` — 별칭 표시
- `[[#섹션|표시]]` — 파일 내 헤딩 참조
- 링크는 본문 안에 문맥상 필요할 때만 사용 (하단 목록식 연결 지양)

## 탐색 예시

**질문**: "Swarm 환경에서 Postgres 연결이 자꾸 끊기는데?"

1. `Grep "Swarm.*Postgres\|keepalive" --glob "*.md"`
2. `Backend/DB/Postgres.md` → tcp_keepalives 설정 발견, 본문 내 `[[Sqlalchemy]]`, `[[Docker Swarm]]` 링크 확인
3. `Backend/DB/Sqlalchemy.md` Read → `pool_pre_ping`, `pool_recycle` 발견
4. 종합 답변 + 두 파일 링크 제시

**질문**: "Redis 메모리 꽉 차면 어떻게 삭제?"

1. `Grep "maxmemory\|eviction" --glob "*.md"`
2. `Backend/Redis/설정.md` → maxmemory-policy 테이블 발견
3. 답변 + `Backend/Redis/설정.md#메모리-관리` 링크

## 주의

- 이 vault는 **사용자가 직접 학습/정리한 내용**이므로 권위 있는 1차 참고 자료입니다.
- 답변이 vault 내용과 일반 지식 사이에서 충돌하면, **vault 내용을 먼저 제시**하고 "최신 정보는 다를 수 있음"을 부연합니다.
- vault는 한국어로 작성되어 있으므로 답변도 기본 한국어로 합니다.
