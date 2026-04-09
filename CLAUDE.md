# Obsidian Vault — 개인 지식 베이스

이 디렉토리는 사용자의 개인 개발 지식 정리 vault입니다. 사용자가 질문하면 **먼저 이 vault를 탐색**한 후 답변하세요.

## 답변 규칙

1. **Vault 우선 탐색**: 질문을 받으면 일반 지식으로 바로 답하지 말고, 먼저 vault를 탐색합니다.
2. **탐색 순서**:
   - `Backend/Backend MOC.md` 또는 `리눅스/리눅스 MOC.md` 확인 → 주제 카테고리 파악
   - `Grep`으로 키워드 검색 (frontmatter tags, 본문 모두)
   - 히트한 파일 `Read`
   - 해당 파일의 `[[wiki-link]]`와 `## 관련 문서` 섹션을 따라가서 추가 문맥 수집
   - 필요 시 BFS처럼 2-3 홉까지 탐색
3. **답변 형식**:
   - vault에서 찾은 내용은 파일 경로와 함께 인용 (예: `Backend/DB/Postgres.md:11`)
   - vault에 없는 내용은 일반 지식으로 보충하되, **"vault에는 없음"** 명시
   - 관련 문서 링크를 답변 끝에 제시하여 사용자가 원본을 바로 열 수 있게 함
4. **업데이트 제안**: 질문에 답변했는데 vault에 해당 정보가 없다면, "이 내용을 vault에 추가할까요?"라고 물어봅니다.

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
- 각 파일 하단 `## 관련 문서` 섹션에 이웃 노드 나열

## 탐색 예시

**질문**: "Swarm 환경에서 Postgres 연결이 자꾸 끊기는데?"

1. `Grep "Swarm.*Postgres\|keepalive" --glob "*.md"`
2. `Backend/DB/Postgres.md` → tcp_keepalives 설정 발견
3. 파일의 `## 관련 문서` → `[[Sqlalchemy]]`, `[[Docker Swarm]]` 확인
4. `Backend/DB/Sqlalchemy.md` Read → `pool_pre_ping`, `pool_recycle` 발견
5. 종합 답변 + 두 파일 링크 제시

**질문**: "Redis 메모리 꽉 차면 어떻게 삭제?"

1. `Grep "maxmemory\|eviction" --glob "*.md"`
2. `Backend/Redis/설정.md` → maxmemory-policy 테이블 발견
3. 답변 + `Backend/Redis/설정.md#메모리-관리` 링크

## 주의

- 이 vault는 **사용자가 직접 학습/정리한 내용**이므로 권위 있는 1차 참고 자료입니다.
- 답변이 vault 내용과 일반 지식 사이에서 충돌하면, **vault 내용을 먼저 제시**하고 "최신 정보는 다를 수 있음"을 부연합니다.
- vault는 한국어로 작성되어 있으므로 답변도 기본 한국어로 합니다.
