---
tags:
  - backend
---
## 1. 개요
[Github](https://github.com/minio/minio)
[공식문서](https://docs.min.io/enterprise/aistor-object-store/)

- **MinIO**: 고성능, S3 호환 오브젝트 스토리지 서버
- **언어**: Go
- **용도**: 비정형 데이터 저장 (이미지, 로그, 백업 등)
- **운영 환경**: On-Premise 또는 AWS EC2 등 클라우드 인스턴스
- **특징**
    - 완전한 S3 API 호환
    - 단일 실행 파일(bin)로 배포
    - 수평 확장(Distributed mode)
    - 고가용성 및 Erasure Coding 내장
---
## 2. 기본 구조

| 구성요소                        | 설명                           |
| --------------------------- | ---------------------------- |
| **Object**                  | 파일 단위 데이터                    |
| **Bucket**                  | 오브젝트의 논리적 그룹                 |
| **Access Key / Secret Key** | 인증 자격 증명                     |
| **MinIO Server**            | 스토리지 서버 프로세스                 |
| **Client (mc)**             | 관리 CLI 도구                    |
| **Gateway**                 | S3, NAS 등 외부 스토리지를 MinIO로 래핑 |

## 3. 실행
```bash
docker run -d \
  --name minio \
  --env-file .env \
  -p 9000:9000 \
  -p 9001:9001 \
  -v minio_data1:/data \
  --restart unless-stopped \
  minio/minio:RELEASE.2025-04-08T15-41-24Z \
  server /data --console-address ":9001"
```


데이터 동기화
양쪽 PC 등록
```bash
mc alias set {A이름} {host}:{port} {ACCESS_KEY} {SECRET_KEY}
mc alias set {B이름} {host}:{port} {ACCESS_KEY} {SECRET_KEY}
```

mirror 실행
- --overwrite : 덮어쓰기
- --remove : 소스에 없는 파일은 대상에서 삭제
- --retry : 실패시 재시도
- --skip-errors : 에러 스킵
- --watch --active-active : 양방향 동기화
```bash
mc mirror {A이름}/{bucket} {B이름}/{bucket} --overwirte --remove --retry
```
path단위도 가능
```bash
mc mirror {A이름}/{bucket}/{path} {B이름}/{bucket}/{path} --overwirte --remove --retry
```
