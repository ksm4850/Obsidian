---
tags:
  - backend
  - grpc
  - python
---
## 목차

1. [gRPC 개요](<#1. gRPC 개요>)
2. [핵심 개념](<#2. 핵심 개념>)
3. [Protocol Buffers](<#3. Protocol Buffers>)
4. [통신 패턴 4가지](<#4. 통신 패턴 4가지>)
5. [Python + gRPC 환경 구축](<#5. Python + gRPC 환경 구축>)
6. [실전 구현](<#6. 실전 구현>)
7. [심화 주제](<#7. 심화 주제>)
8. [성능 최적화](<#8. 성능 최적화>)
9. [보안](<#9. 보안>)
10. [모범 사례 및 안티패턴](<#10. 모범 사례 및 안티패턴>)
11. [FastAPI와 gRPC 통합](<#11. FastAPI와 gRPC 통합>)
12. [트러블슈팅](<#12. 트러블슈팅>)

---

## 1. gRPC 개요

### 1.1 gRPC란?

gRPC(gRPC Remote Procedure Call)는 Google이 개발한 고성능, 오픈소스 RPC 프레임워크다. 클라이언트 애플리케이션이 마치 로컬 객체의 메서드를 호출하듯이 다른 머신의 서버 애플리케이션 메서드를 직접 호출할 수 있게 해준다.

### 1.2 REST vs gRPC 비교

| 특성 | REST | gRPC |
|------|------|------|
| 프로토콜 | HTTP/1.1 (HTTP/2 가능) | HTTP/2 |
| 데이터 포맷 | JSON (텍스트) | Protocol Buffers (바이너리) |
| API 계약 | OpenAPI/Swagger (선택) | .proto 파일 (필수) |
| 스트리밍 | 제한적 (WebSocket 별도 필요) | 네이티브 양방향 지원 |
| 브라우저 지원 | 완전 지원 | 제한적 (gRPC-Web 필요) |
| 페이로드 크기 | 상대적 큼 | 작음 (2-10배 차이) |
| 코드 생성 | 선택적 | 필수 (자동 생성) |
| 디버깅 | 쉬움 (JSON 가독성) | 어려움 (바이너리) |

### 1.3 gRPC를 선택해야 하는 경우

**적합한 경우:**
- 마이크로서비스 간 내부 통신 (서비스 메시 내부)
- 실시간 양방향 스트리밍 (채팅, 게임, 라이브 피드)
- 대용량 데이터 스트리밍 (파일 업로드, 로그 수집)
- 다중 언어 환경에서 타입 안전한 API 계약 필요
- 낮은 지연시간이 중요한 내부 API

**부적합한 경우 (REST 추천):**
- 브라우저가 주요 클라이언트 (gRPC-Web은 제약이 많음)
- 단순 CRUD API (gRPC 오버헤드가 큼)
- API 디버깅/테스트 편의성이 중요할 때
- 외부 공개 API (REST가 범용적)
- 캐싱이 중요한 경우 (HTTP 캐싱 활용 어려움)

### 1.4 HTTP/2의 이점

gRPC는 HTTP/2를 기반으로 하며 다음 이점을 제공한다:

- **멀티플렉싱**: 단일 TCP 연결에서 여러 요청/응답을 동시 처리 (HOL blocking 해결)
- **헤더 압축**: HPACK을 통한 효율적인 헤더 압축
- **바이너리 프레이밍**: 효율적인 파싱과 낮은 오버헤드
- **스트림 우선순위**: 중요한 요청에 우선순위 부여 가능

---

## 2. 핵심 개념

### 2.1 아키텍처 구성요소

```
┌─────────────┐                    ┌─────────────┐
│   Client    │                    │   Server    │
│             │                    │             │
│  ┌───────┐  │    HTTP/2 +        │  ┌───────┐  │
│  │ Stub  │◄─┼───Protobuf────────►│  │Service│  │
│  └───────┘  │                    │  └───────┘  │
└─────────────┘                    └─────────────┘
```

**주요 구성요소:**
- **Service Definition**: `.proto` 파일에 정의된 서비스 인터페이스
- **Stub (Client)**: 서버 메서드를 호출하는 클라이언트 측 프록시
- **Server**: 서비스 구현체를 호스팅
- **Channel**: 클라이언트와 서버 간의 연결

### 2.2 gRPC 생명주기

1. **채널 생성**: 클라이언트가 서버와 연결
2. **Stub 생성**: 채널을 통해 Stub 인스턴스화
3. **메서드 호출**: Stub을 통해 원격 메서드 호출
4. **직렬화**: 요청 메시지를 Protobuf로 직렬화
5. **전송**: HTTP/2를 통해 바이너리 데이터 전송
6. **역직렬화**: 서버에서 메시지 역직렬화
7. **처리**: 서비스 로직 실행
8. **응답**: 동일한 과정을 거쳐 응답 반환

### 2.3 메타데이터 (Metadata)

메타데이터는 gRPC 호출에 첨부되는 키-값 쌍으로, HTTP 헤더와 유사하다.

**용도:**
- 인증 토큰 전달
- 요청 추적 ID
- 커스텀 헤더 정보

**종류:**
- **Initial Metadata**: 요청/응답 시작 시 전송
- **Trailing Metadata**: 응답 완료 시 전송 (상태 정보 포함)

---

## 3. Protocol Buffers

### 3.1 Protocol Buffers란?

Protocol Buffers(Protobuf)는 Google이 개발한 언어 중립적, 플랫폼 중립적인 데이터 직렬화 메커니즘이다. gRPC의 기본 IDL(Interface Definition Language)이자 메시지 교환 포맷이다.

### 3.2 .proto 파일 문법

```protobuf
// proto3 문법 사용 선언
syntax = "proto3";

// 패키지 선언 (네임스페이스 충돌 방지)
package myservice;

// 메시지 정의
message User {
  int32 id = 1;              // 필드 번호 1
  string name = 2;           // 필드 번호 2
  string email = 3;          // 필드 번호 3
  repeated string roles = 4; // 반복 필드 (배열)
  optional string bio = 5;   // 선택적 필드
  
  // 중첩 메시지
  message Address {
    string street = 1;
    string city = 2;
  }
  Address address = 6;
}

// 서비스 정의
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
  rpc CreateUsers(stream User) returns (CreateUsersResponse);
  rpc Chat(stream Message) returns (stream Message);
}
```

### 3.3 스칼라 타입

| Protobuf 타입 | Python 타입 | 설명 |
|--------------|-------------|------|
| double | float | 64비트 부동소수점 |
| float | float | 32비트 부동소수점 |
| int32 | int | 32비트 정수 |
| int64 | int | 64비트 정수 |
| uint32 | int | 부호 없는 32비트 |
| uint64 | int | 부호 없는 64비트 |
| bool | bool | 불리언 |
| string | str | UTF-8 문자열 |
| bytes | bytes | 바이트 배열 |

### 3.4 필드 번호 규칙

- **1-15**: 1바이트로 인코딩 (자주 사용하는 필드에 할당)
- **16-2047**: 2바이트로 인코딩
- **19000-19999**: 예약됨 (사용 불가)
- 한 번 사용한 번호는 재사용 금지 (하위 호환성)

### 3.5 고급 타입

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "google/protobuf/any.proto";
import "google/protobuf/wrappers.proto";

// Enum 타입
enum Status {
  STATUS_UNSPECIFIED = 0;  // 기본값은 0
  STATUS_ACTIVE = 1;
  STATUS_INACTIVE = 2;
}

// Oneof - 여러 필드 중 하나만 설정 가능
message SearchRequest {
  oneof query {
    string text_query = 1;
    int32 id_query = 2;
  }
}

// Map 타입
message Project {
  string name = 1;
  map<string, string> labels = 2;  // key-value 맵
}

// Well-Known Types 사용
message Event {
  string name = 1;
  google.protobuf.Timestamp created_at = 2;  // 타임스탬프
  google.protobuf.Any payload = 3;           // 동적 타입
  google.protobuf.StringValue nullable_field = 4; // nullable
}
```

### 3.6 하위 호환성 유지

**안전한 변경:**
- 새 필드 추가 (새 번호 사용)
- 필드 제거 (번호 예약)
- 필드 이름 변경 (번호 유지)

**위험한 변경:**
- 필드 번호 변경
- 필드 타입 변경
- required → optional 변경 (proto2)

```protobuf
message User {
  reserved 2, 15, 9 to 11;  // 번호 예약
  reserved "old_field_name"; // 이름 예약
  
  int32 id = 1;
  string name = 3;
}
```

---

## 4. 통신 패턴 4가지

### 4.1 Unary RPC (단일 요청-응답)

가장 기본적인 패턴. 클라이언트가 하나의 요청을 보내고 하나의 응답을 받는다.

```protobuf
service Greeter {
  rpc SayHello(HelloRequest) returns (HelloResponse);
}
```

```
Client                 Server
  │                      │
  │──── Request ────────►│
  │                      │
  │◄──── Response ───────│
  │                      │
```

**사용 사례:**
- 사용자 조회
- 데이터 생성/수정
- 인증 요청

> 💡 Unary RPC만 사용한다면 REST와 큰 차이 없음. 마이크로서비스 내부 통신에서 성능이 중요할 때 의미 있음.

### 4.2 Server Streaming RPC

클라이언트가 하나의 요청을 보내고, 서버가 스트림으로 여러 응답을 반환한다.

```protobuf
service StockService {
  rpc SubscribePrice(StockRequest) returns (stream PriceUpdate);
}
```

```
Client                 Server
  │                      │
  │──── Request ────────►│
  │                      │
  │◄──── Response 1 ─────│
  │◄──── Response 2 ─────│
  │◄──── Response 3 ─────│
  │◄──── ... ────────────│
  │◄──── Response N ─────│
  │                      │
```

**사용 사례:**
- 실시간 주가 업데이트
- 로그 스트리밍
- 대용량 데이터 페이지네이션

### 4.3 Client Streaming RPC

클라이언트가 스트림으로 여러 요청을 보내고, 서버가 하나의 응답을 반환한다.

```protobuf
service UploadService {
  rpc UploadFile(stream FileChunk) returns (UploadStatus);
}
```

```
Client                 Server
  │                      │
  │──── Request 1 ──────►│
  │──── Request 2 ──────►│
  │──── Request 3 ──────►│
  │──── ... ────────────►│
  │──── Request N ──────►│
  │                      │
  │◄──── Response ───────│
  │                      │
```

**사용 사례:**
- 파일 업로드
- 대량 데이터 전송
- IoT 센서 데이터 수집

### 4.4 Bidirectional Streaming RPC

양쪽 모두 스트림을 사용하여 독립적으로 데이터를 주고받는다.

```protobuf
service ChatService {
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}
```

```
Client                 Server
  │                      │
  │──── Message 1 ──────►│
  │◄──── Message A ──────│
  │──── Message 2 ──────►│
  │──── Message 3 ──────►│
  │◄──── Message B ──────│
  │◄──── Message C ──────│
  │──── Message 4 ──────►│
  │                      │
```

**사용 사례:**
- 실시간 채팅
- 게임 서버 통신
- 실시간 협업 도구

---

## 5. Python + gRPC 환경 구축

### 5.1 필수 패키지 설치

```bash
# 기본 gRPC 패키지
pip install grpcio grpcio-tools

# 추가 유틸리티
pip install grpcio-reflection  # 서버 리플렉션
pip install grpcio-health-checking  # 헬스 체크
pip install grpcio-status  # 상세 에러 상태
```

### 5.2 프로젝트 구조

```
project/
├── protos/
│   └── user_service.proto
├── generated/
│   ├── __init__.py
│   ├── user_service_pb2.py      # 메시지 클래스
│   └── user_service_pb2_grpc.py # 서비스 클래스
├── server/
│   ├── __init__.py
│   ├── main.py
│   └── services/
│       └── user_service.py
├── client/
│   ├── __init__.py
│   └── main.py
└── Makefile
```

### 5.3 Proto 파일 컴파일

```bash
# 단일 파일 컴파일
python -m grpc_tools.protoc \
  -I./protos \
  --python_out=./generated \
  --grpc_python_out=./generated \
  ./protos/user_service.proto

# 여러 파일 컴파일 (Makefile 권장)
```

**Makefile 예시:**

```makefile
PROTO_DIR = protos
OUT_DIR = generated

.PHONY: proto clean

proto:
	python -m grpc_tools.protoc \
		-I$(PROTO_DIR) \
		--python_out=$(OUT_DIR) \
		--grpc_python_out=$(OUT_DIR) \
		$(PROTO_DIR)/*.proto

clean:
	rm -f $(OUT_DIR)/*_pb2.py $(OUT_DIR)/*_pb2_grpc.py
```

### 5.4 Import 문제 해결

생성된 파일의 import 경로 문제는 흔히 발생한다.

**문제 상황:**
```python
# user_service_pb2_grpc.py 에서
import user_service_pb2 as user__service__pb2  # 절대 경로
```

**해결 방법 1: PYTHONPATH 설정**
```bash
export PYTHONPATH="${PYTHONPATH}:./generated"
```

**해결 방법 2: 상대 경로로 수정**
```bash
# sed를 사용한 자동 수정
sed -i 's/import \(.*\)_pb2/from . import \1_pb2/' generated/*_pb2_grpc.py
```

**해결 방법 3: betterproto 사용**
```bash
pip install betterproto[compiler]
# 더 파이썬다운 코드 생성
```

---

## 6. 실전 구현

### 6.1 Proto 파일 정의

```protobuf
// protos/user_service.proto
syntax = "proto3";

package user;

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

// 사용자 메시지
message User {
  int32 id = 1;
  string username = 2;
  string email = 3;
  UserStatus status = 4;
  google.protobuf.Timestamp created_at = 5;
}

enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0;
  USER_STATUS_ACTIVE = 1;
  USER_STATUS_INACTIVE = 2;
  USER_STATUS_BANNED = 3;
}

// 요청/응답 메시지
message GetUserRequest {
  int32 user_id = 1;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
}

message UpdateUserRequest {
  int32 user_id = 1;
  optional string username = 2;
  optional string email = 3;
  optional UserStatus status = 4;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  string next_page_token = 2;
}

message DeleteUserRequest {
  int32 user_id = 1;
}

// 서비스 정의
service UserService {
  // Unary
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
  
  // Server Streaming
  rpc ListUsers(ListUsersRequest) returns (stream User);
  
  // Client Streaming
  rpc BatchCreateUsers(stream CreateUserRequest) returns (ListUsersResponse);
  
  // Bidirectional Streaming
  rpc SyncUsers(stream User) returns (stream User);
}
```

### 6.2 서버 구현

```python
# server/services/user_service.py
import grpc
from concurrent import futures
from datetime import datetime
from typing import Iterator

from google.protobuf.empty_pb2 import Empty
from google.protobuf.timestamp_pb2 import Timestamp

# 생성된 모듈
from generated import user_service_pb2 as pb2
from generated import user_service_pb2_grpc as pb2_grpc


class UserServiceServicer(pb2_grpc.UserServiceServicer):
    """UserService 구현체"""
    
    def __init__(self):
        # 간단한 인메모리 저장소
        self._users: dict[int, pb2.User] = {}
        self._next_id = 1
    
    def GetUser(
        self,
        request: pb2.GetUserRequest,
        context: grpc.ServicerContext
    ) -> pb2.User:
        """단일 사용자 조회"""
        user_id = request.user_id
        
        if user_id not in self._users:
            context.abort(
                grpc.StatusCode.NOT_FOUND,
                f"User {user_id} not found"
            )
        
        return self._users[user_id]
    
    def CreateUser(
        self,
        request: pb2.CreateUserRequest,
        context: grpc.ServicerContext
    ) -> pb2.User:
        """사용자 생성"""
        # 중복 체크
        for user in self._users.values():
            if user.email == request.email:
                context.abort(
                    grpc.StatusCode.ALREADY_EXISTS,
                    f"Email {request.email} already exists"
                )
        
        # 타임스탬프 생성
        now = Timestamp()
        now.FromDatetime(datetime.utcnow())
        
        user = pb2.User(
            id=self._next_id,
            username=request.username,
            email=request.email,
            status=pb2.USER_STATUS_ACTIVE,
            created_at=now
        )
        
        self._users[self._next_id] = user
        self._next_id += 1
        
        return user
    
    def UpdateUser(
        self,
        request: pb2.UpdateUserRequest,
        context: grpc.ServicerContext
    ) -> pb2.User:
        """사용자 수정"""
        user_id = request.user_id
        
        if user_id not in self._users:
            context.abort(
                grpc.StatusCode.NOT_FOUND,
                f"User {user_id} not found"
            )
        
        user = self._users[user_id]
        
        # 선택적 필드 업데이트
        if request.HasField('username'):
            user.username = request.username
        if request.HasField('email'):
            user.email = request.email
        if request.HasField('status'):
            user.status = request.status
        
        return user
    
    def DeleteUser(
        self,
        request: pb2.DeleteUserRequest,
        context: grpc.ServicerContext
    ) -> Empty:
        """사용자 삭제"""
        user_id = request.user_id
        
        if user_id not in self._users:
            context.abort(
                grpc.StatusCode.NOT_FOUND,
                f"User {user_id} not found"
            )
        
        del self._users[user_id]
        return Empty()
    
    def ListUsers(
        self,
        request: pb2.ListUsersRequest,
        context: grpc.ServicerContext
    ) -> Iterator[pb2.User]:
        """Server Streaming: 사용자 목록 스트리밍"""
        page_size = request.page_size or 10
        
        for i, user in enumerate(self._users.values()):
            if i >= page_size:
                break
            yield user
    
    def BatchCreateUsers(
        self,
        request_iterator: Iterator[pb2.CreateUserRequest],
        context: grpc.ServicerContext
    ) -> pb2.ListUsersResponse:
        """Client Streaming: 대량 사용자 생성"""
        created_users = []
        
        for request in request_iterator:
            now = Timestamp()
            now.FromDatetime(datetime.utcnow())
            
            user = pb2.User(
                id=self._next_id,
                username=request.username,
                email=request.email,
                status=pb2.USER_STATUS_ACTIVE,
                created_at=now
            )
            
            self._users[self._next_id] = user
            created_users.append(user)
            self._next_id += 1
        
        return pb2.ListUsersResponse(users=created_users)
    
    def SyncUsers(
        self,
        request_iterator: Iterator[pb2.User],
        context: grpc.ServicerContext
    ) -> Iterator[pb2.User]:
        """Bidirectional Streaming: 사용자 동기화"""
        for incoming_user in request_iterator:
            # 수신된 사용자 처리
            if incoming_user.id in self._users:
                # 기존 사용자 업데이트
                self._users[incoming_user.id] = incoming_user
                yield incoming_user
            else:
                # 새 사용자 생성
                self._users[incoming_user.id] = incoming_user
                yield incoming_user
```

### 6.3 서버 실행

```python
# server/main.py
import grpc
from concurrent import futures
import logging
import signal
import sys

from grpc_reflection.v1alpha import reflection
from grpc_health.v1 import health, health_pb2, health_pb2_grpc

from generated import user_service_pb2 as pb2
from generated import user_service_pb2_grpc as pb2_grpc
from server.services.user_service import UserServiceServicer

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


def serve():
    # 서버 생성
    server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=10),
        options=[
            ('grpc.max_send_message_length', 50 * 1024 * 1024),
            ('grpc.max_receive_message_length', 50 * 1024 * 1024),
        ]
    )
    
    # 서비스 등록
    pb2_grpc.add_UserServiceServicer_to_server(
        UserServiceServicer(),
        server
    )
    
    # 헬스 체크 서비스 등록
    health_servicer = health.HealthServicer()
    health_pb2_grpc.add_HealthServicer_to_server(health_servicer, server)
    health_servicer.set(
        pb2.DESCRIPTOR.services_by_name['UserService'].full_name,
        health_pb2.HealthCheckResponse.SERVING
    )
    
    # 리플렉션 활성화 (개발/디버깅용)
    SERVICE_NAMES = (
        pb2.DESCRIPTOR.services_by_name['UserService'].full_name,
        reflection.SERVICE_NAME,
        health.SERVICE_NAME,
    )
    reflection.enable_server_reflection(SERVICE_NAMES, server)
    
    # 포트 바인딩
    server.add_insecure_port('[::]:50051')
    server.start()
    logger.info("Server started on port 50051")
    
    # Graceful Shutdown
    def shutdown(signum, frame):
        logger.info("Shutting down gracefully...")
        server.stop(grace=5)
        sys.exit(0)
    
    signal.signal(signal.SIGTERM, shutdown)
    signal.signal(signal.SIGINT, shutdown)
    
    server.wait_for_termination()


if __name__ == '__main__':
    serve()
```

### 6.4 클라이언트 구현

```python
# client/main.py
import grpc
import logging
from typing import Iterator

from generated import user_service_pb2 as pb2
from generated import user_service_pb2_grpc as pb2_grpc

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class UserClient:
    """gRPC UserService 클라이언트"""
    
    def __init__(self, host: str = 'localhost', port: int = 50051):
        self.channel = grpc.insecure_channel(f'{host}:{port}')
        self.stub = pb2_grpc.UserServiceStub(self.channel)
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
    
    def close(self):
        self.channel.close()
    
    # Unary 호출
    def get_user(self, user_id: int) -> pb2.User:
        request = pb2.GetUserRequest(user_id=user_id)
        return self.stub.GetUser(request)
    
    def create_user(self, username: str, email: str) -> pb2.User:
        request = pb2.CreateUserRequest(
            username=username,
            email=email
        )
        return self.stub.CreateUser(request)
    
    def update_user(
        self,
        user_id: int,
        username: str = None,
        email: str = None
    ) -> pb2.User:
        request = pb2.UpdateUserRequest(user_id=user_id)
        if username:
            request.username = username
        if email:
            request.email = email
        return self.stub.UpdateUser(request)
    
    def delete_user(self, user_id: int) -> None:
        request = pb2.DeleteUserRequest(user_id=user_id)
        self.stub.DeleteUser(request)
    
    # Server Streaming
    def list_users(self, page_size: int = 10) -> Iterator[pb2.User]:
        request = pb2.ListUsersRequest(page_size=page_size)
        for user in self.stub.ListUsers(request):
            yield user
    
    # Client Streaming
    def batch_create_users(
        self,
        users: list[tuple[str, str]]
    ) -> pb2.ListUsersResponse:
        def generate_requests():
            for username, email in users:
                yield pb2.CreateUserRequest(
                    username=username,
                    email=email
                )
        
        return self.stub.BatchCreateUsers(generate_requests())
    
    # Bidirectional Streaming
    def sync_users(
        self,
        users: Iterator[pb2.User]
    ) -> Iterator[pb2.User]:
        return self.stub.SyncUsers(users)


# 사용 예시
def main():
    with UserClient() as client:
        # 사용자 생성
        user = client.create_user("john", "john@example.com")
        logger.info(f"Created user: {user}")
        
        # 사용자 조회
        fetched = client.get_user(user.id)
        logger.info(f"Fetched user: {fetched}")
        
        # 사용자 수정
        updated = client.update_user(user.id, username="john_updated")
        logger.info(f"Updated user: {updated}")
        
        # 대량 생성
        batch_users = [
            ("user1", "user1@example.com"),
            ("user2", "user2@example.com"),
            ("user3", "user3@example.com"),
        ]
        result = client.batch_create_users(batch_users)
        logger.info(f"Batch created {len(result.users)} users")
        
        # 스트리밍 조회
        logger.info("Listing users:")
        for u in client.list_users(page_size=5):
            logger.info(f"  - {u.username}: {u.email}")


if __name__ == '__main__':
    main()
```

---

## 7. 심화 주제

### 7.1 인터셉터 (Interceptors)

인터셉터는 gRPC 호출의 전/후에 공통 로직을 삽입하는 미들웨어다.

**서버 인터셉터:**

```python
# interceptors/logging_interceptor.py
import grpc
import time
import logging
from typing import Callable, Any

logger = logging.getLogger(__name__)


class LoggingInterceptor(grpc.ServerInterceptor):
    """요청/응답 로깅 인터셉터"""
    
    def intercept_service(
        self,
        continuation: Callable,
        handler_call_details: grpc.HandlerCallDetails
    ) -> grpc.RpcMethodHandler:
        method = handler_call_details.method
        logger.info(f"Incoming request: {method}")
        
        start_time = time.perf_counter()
        
        # 실제 핸들러 호출
        response = continuation(handler_call_details)
        
        elapsed = time.perf_counter() - start_time
        logger.info(f"Request {method} completed in {elapsed:.3f}s")
        
        return response


class AuthInterceptor(grpc.ServerInterceptor):
    """인증 인터셉터"""
    
    def __init__(self, valid_tokens: set[str]):
        self._valid_tokens = valid_tokens
    
    def intercept_service(
        self,
        continuation: Callable,
        handler_call_details: grpc.HandlerCallDetails
    ) -> grpc.RpcMethodHandler:
        # 메타데이터에서 토큰 추출
        metadata = dict(handler_call_details.invocation_metadata or [])
        token = metadata.get('authorization', '')
        
        if not token.startswith('Bearer '):
            return self._create_abort_handler(
                grpc.StatusCode.UNAUTHENTICATED,
                "Missing or invalid authorization header"
            )
        
        token_value = token[7:]  # "Bearer " 제거
        
        if token_value not in self._valid_tokens:
            return self._create_abort_handler(
                grpc.StatusCode.UNAUTHENTICATED,
                "Invalid token"
            )
        
        return continuation(handler_call_details)
    
    def _create_abort_handler(
        self,
        code: grpc.StatusCode,
        details: str
    ) -> grpc.RpcMethodHandler:
        def abort(request, context):
            context.abort(code, details)
        
        return grpc.unary_unary_rpc_method_handler(abort)


# 서버에 인터셉터 적용
server = grpc.server(
    futures.ThreadPoolExecutor(max_workers=10),
    interceptors=[
        LoggingInterceptor(),
        AuthInterceptor({'valid_token_123'}),
    ]
)
```

**클라이언트 인터셉터:**

```python
# interceptors/client_interceptors.py
import grpc
import logging
import time

logger = logging.getLogger(__name__)


class AuthClientInterceptor(
    grpc.UnaryUnaryClientInterceptor,
    grpc.UnaryStreamClientInterceptor,
    grpc.StreamUnaryClientInterceptor,
    grpc.StreamStreamClientInterceptor,
):
    """인증 헤더 추가 인터셉터"""
    
    def __init__(self, token: str):
        self._token = token
    
    def _add_auth_metadata(self, client_call_details):
        metadata = list(client_call_details.metadata or [])
        metadata.append(('authorization', f'Bearer {self._token}'))
        
        return grpc.ClientCallDetails(
            client_call_details.method,
            client_call_details.timeout,
            metadata,
            client_call_details.credentials,
            client_call_details.wait_for_ready,
            client_call_details.compression,
        )
    
    def intercept_unary_unary(self, continuation, client_call_details, request):
        new_details = self._add_auth_metadata(client_call_details)
        return continuation(new_details, request)
    
    def intercept_unary_stream(self, continuation, client_call_details, request):
        new_details = self._add_auth_metadata(client_call_details)
        return continuation(new_details, request)
    
    def intercept_stream_unary(self, continuation, client_call_details, request_iterator):
        new_details = self._add_auth_metadata(client_call_details)
        return continuation(new_details, request_iterator)
    
    def intercept_stream_stream(self, continuation, client_call_details, request_iterator):
        new_details = self._add_auth_metadata(client_call_details)
        return continuation(new_details, request_iterator)


class RetryInterceptor(grpc.UnaryUnaryClientInterceptor):
    """재시도 인터셉터"""
    
    RETRIABLE_CODES = {
        grpc.StatusCode.UNAVAILABLE,
        grpc.StatusCode.DEADLINE_EXCEEDED,
    }
    
    def __init__(self, max_retries: int = 3, backoff_ms: int = 100):
        self._max_retries = max_retries
        self._backoff_ms = backoff_ms
    
    def intercept_unary_unary(self, continuation, client_call_details, request):
        for attempt in range(self._max_retries + 1):
            try:
                return continuation(client_call_details, request)
            except grpc.RpcError as e:
                if e.code() not in self.RETRIABLE_CODES:
                    raise
                if attempt == self._max_retries:
                    raise
                
                sleep_time = self._backoff_ms * (2 ** attempt) / 1000
                logger.warning(
                    f"Retry {attempt + 1}/{self._max_retries} "
                    f"after {sleep_time:.2f}s due to {e.code()}"
                )
                time.sleep(sleep_time)


# 클라이언트에 인터셉터 적용
channel = grpc.intercept_channel(
    grpc.insecure_channel('localhost:50051'),
    AuthClientInterceptor('my_token'),
    RetryInterceptor(max_retries=3),
)
stub = pb2_grpc.UserServiceStub(channel)
```

### 7.2 에러 처리

**gRPC 상태 코드:**

| 코드 | 설명 | HTTP 매핑 |
|------|------|-----------|
| OK | 성공 | 200 |
| CANCELLED | 취소됨 | 499 |
| UNKNOWN | 알 수 없는 에러 | 500 |
| INVALID_ARGUMENT | 잘못된 인자 | 400 |
| DEADLINE_EXCEEDED | 타임아웃 | 504 |
| NOT_FOUND | 리소스 없음 | 404 |
| ALREADY_EXISTS | 이미 존재 | 409 |
| PERMISSION_DENIED | 권한 없음 | 403 |
| RESOURCE_EXHAUSTED | 리소스 부족 | 429 |
| FAILED_PRECONDITION | 사전조건 실패 | 400 |
| ABORTED | 작업 중단 | 409 |
| OUT_OF_RANGE | 범위 초과 | 400 |
| UNIMPLEMENTED | 구현되지 않음 | 501 |
| INTERNAL | 내부 에러 | 500 |
| UNAVAILABLE | 서비스 불가 | 503 |
| DATA_LOSS | 데이터 손실 | 500 |
| UNAUTHENTICATED | 인증 필요 | 401 |

**상세 에러 정보 (Rich Error Model):**

```python
# 서버 측 - 상세 에러 반환
from grpc_status import rpc_status
from google.protobuf import any_pb2
from google.rpc import status_pb2, error_details_pb2


def create_user_with_validation(request, context):
    errors = []
    
    if len(request.username) < 3:
        errors.append(
            error_details_pb2.BadRequest.FieldViolation(
                field="username",
                description="Username must be at least 3 characters"
            )
        )
    
    if '@' not in request.email:
        errors.append(
            error_details_pb2.BadRequest.FieldViolation(
                field="email",
                description="Invalid email format"
            )
        )
    
    if errors:
        bad_request = error_details_pb2.BadRequest(field_violations=errors)
        
        detail = any_pb2.Any()
        detail.Pack(bad_request)
        
        status = status_pb2.Status(
            code=grpc.StatusCode.INVALID_ARGUMENT.value[0],
            message="Validation failed",
            details=[detail]
        )
        
        context.abort_with_status(rpc_status.to_status(status))


# 클라이언트 측 - 상세 에러 파싱
from grpc_status import rpc_status


try:
    response = stub.CreateUser(request)
except grpc.RpcError as e:
    status = rpc_status.from_call(e)
    
    for detail in status.details:
        if detail.Is(error_details_pb2.BadRequest.DESCRIPTOR):
            bad_request = error_details_pb2.BadRequest()
            detail.Unpack(bad_request)
            
            for violation in bad_request.field_violations:
                print(f"Field '{violation.field}': {violation.description}")
```

### 7.3 데드라인과 타임아웃

```python
# 클라이언트 측 타임아웃 설정
try:
    response = stub.GetUser(
        request,
        timeout=5.0  # 5초 타임아웃
    )
except grpc.RpcError as e:
    if e.code() == grpc.StatusCode.DEADLINE_EXCEEDED:
        print("Request timed out")


# 서버 측 데드라인 확인
def GetUser(self, request, context):
    # 남은 시간 확인
    remaining = context.time_remaining()
    
    if remaining < 0.5:  # 0.5초 미만이면 빠른 경로
        return self._get_from_cache(request.user_id)
    
    # 일반 처리
    return self._get_from_db(request.user_id)
```

### 7.4 취소 (Cancellation)

```python
# 클라이언트 측 취소
import threading


def cancel_after_delay(call, delay):
    time.sleep(delay)
    call.cancel()


call = stub.ListUsers.future(request)

# 3초 후 취소
cancel_thread = threading.Thread(
    target=cancel_after_delay,
    args=(call, 3.0)
)
cancel_thread.start()

try:
    for user in call:
        print(user)
except grpc.RpcError as e:
    if e.code() == grpc.StatusCode.CANCELLED:
        print("Request was cancelled")


# 서버 측 취소 감지
def ListUsers(self, request, context):
    for user in self._users.values():
        if context.is_active():  # 취소 여부 확인
            yield user
        else:
            return
```

### 7.5 압축

```python
# 서버 측 압축 설정
server = grpc.server(
    futures.ThreadPoolExecutor(max_workers=10),
    compression=grpc.Compression.Gzip,
    options=[
        ('grpc.default_compression_algorithm', grpc.Compression.Gzip.value),
        ('grpc.default_compression_level', 2),  # 0-3
    ]
)


# 클라이언트 측 압축 설정
channel = grpc.insecure_channel(
    'localhost:50051',
    compression=grpc.Compression.Gzip
)

# 또는 호출별 압축
response = stub.CreateUser(
    request,
    compression=grpc.Compression.Gzip
)
```

---

## 8. 성능 최적화

### 8.1 연결 관리

```python
# 채널 풀링
class ChannelPool:
    def __init__(self, target: str, pool_size: int = 5):
        self._channels = [
            grpc.insecure_channel(target)
            for _ in range(pool_size)
        ]
        self._index = 0
    
    def get_channel(self) -> grpc.Channel:
        channel = self._channels[self._index]
        self._index = (self._index + 1) % len(self._channels)
        return channel
    
    def close_all(self):
        for channel in self._channels:
            channel.close()


# Keep-alive 설정
channel = grpc.insecure_channel(
    'localhost:50051',
    options=[
        ('grpc.keepalive_time_ms', 10000),           # 10초마다 ping
        ('grpc.keepalive_timeout_ms', 5000),         # ping 응답 대기 5초
        ('grpc.keepalive_permit_without_calls', True), # 호출 없어도 keepalive
        ('grpc.http2.min_time_between_pings_ms', 10000),
    ]
)
```

### 8.2 메시지 크기 설정

```python
# 대용량 메시지 처리
MAX_MESSAGE_SIZE = 100 * 1024 * 1024  # 100MB

# 서버
server = grpc.server(
    futures.ThreadPoolExecutor(max_workers=10),
    options=[
        ('grpc.max_send_message_length', MAX_MESSAGE_SIZE),
        ('grpc.max_receive_message_length', MAX_MESSAGE_SIZE),
    ]
)

# 클라이언트
channel = grpc.insecure_channel(
    'localhost:50051',
    options=[
        ('grpc.max_send_message_length', MAX_MESSAGE_SIZE),
        ('grpc.max_receive_message_length', MAX_MESSAGE_SIZE),
    ]
)
```

### 8.3 동시성 최적화

```python
# 서버 스레드 풀 크기 조정
import os

cpu_count = os.cpu_count() or 4
worker_count = cpu_count * 2 + 1  # I/O 바운드 작업용

server = grpc.server(
    futures.ThreadPoolExecutor(max_workers=worker_count),
    maximum_concurrent_rpcs=100,  # 동시 RPC 제한
)


# 비동기 서버 (aio)
import grpc.aio


async def serve():
    server = grpc.aio.server()
    pb2_grpc.add_UserServiceServicer_to_server(
        AsyncUserServiceServicer(),
        server
    )
    server.add_insecure_port('[::]:50051')
    await server.start()
    await server.wait_for_termination()


class AsyncUserServiceServicer(pb2_grpc.UserServiceServicer):
    async def GetUser(self, request, context):
        user = await self._async_get_user(request.user_id)
        return user
```

### 8.4 로드 밸런싱

```python
# 클라이언트 측 로드 밸런싱
channel = grpc.insecure_channel(
    'dns:///my-service.example.com:50051',
    options=[
        ('grpc.lb_policy_name', 'round_robin'),
        # 또는 'pick_first' (기본값)
    ]
)


# 여러 서버 주소 지정
channel = grpc.insecure_channel(
    'ipv4:///10.0.0.1:50051,10.0.0.2:50051,10.0.0.3:50051',
    options=[
        ('grpc.lb_policy_name', 'round_robin'),
    ]
)
```

### 8.5 벤치마킹

```python
# 간단한 벤치마크
import time
import statistics
from concurrent.futures import ThreadPoolExecutor, as_completed


def benchmark_grpc(stub, iterations=1000, concurrency=10):
    latencies = []
    
    def single_call():
        start = time.perf_counter()
        stub.GetUser(pb2.GetUserRequest(user_id=1))
        return time.perf_counter() - start
    
    with ThreadPoolExecutor(max_workers=concurrency) as executor:
        futures = [executor.submit(single_call) for _ in range(iterations)]
        
        for future in as_completed(futures):
            latencies.append(future.result())
    
    return {
        'total_requests': iterations,
        'avg_latency_ms': statistics.mean(latencies) * 1000,
        'p50_latency_ms': statistics.median(latencies) * 1000,
        'p99_latency_ms': statistics.quantiles(latencies, n=100)[98] * 1000,
        'requests_per_second': iterations / sum(latencies),
    }
```

---

## 9. 보안

### 9.1 TLS/SSL 암호화

```python
# 인증서 생성 (개발용)
# openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes


# 서버 측 TLS 설정
with open('server.key', 'rb') as f:
    private_key = f.read()
with open('server.crt', 'rb') as f:
    certificate_chain = f.read()

server_credentials = grpc.ssl_server_credentials(
    [(private_key, certificate_chain)]
)

server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
pb2_grpc.add_UserServiceServicer_to_server(UserServiceServicer(), server)
server.add_secure_port('[::]:50051', server_credentials)


# 클라이언트 측 TLS 설정
with open('server.crt', 'rb') as f:
    trusted_certs = f.read()

channel_credentials = grpc.ssl_channel_credentials(
    root_certificates=trusted_certs
)

channel = grpc.secure_channel(
    'localhost:50051',
    channel_credentials
)
```

### 9.2 상호 TLS (mTLS)

```python
# 서버: 클라이언트 인증서도 검증
with open('server.key', 'rb') as f:
    server_key = f.read()
with open('server.crt', 'rb') as f:
    server_cert = f.read()
with open('ca.crt', 'rb') as f:
    ca_cert = f.read()

server_credentials = grpc.ssl_server_credentials(
    [(server_key, server_cert)],
    root_certificates=ca_cert,
    require_client_auth=True  # 클라이언트 인증 필수
)


# 클라이언트: 인증서 제공
with open('client.key', 'rb') as f:
    client_key = f.read()
with open('client.crt', 'rb') as f:
    client_cert = f.read()
with open('ca.crt', 'rb') as f:
    ca_cert = f.read()

channel_credentials = grpc.ssl_channel_credentials(
    root_certificates=ca_cert,
    private_key=client_key,
    certificate_chain=client_cert
)

channel = grpc.secure_channel('localhost:50051', channel_credentials)
```

### 9.3 토큰 기반 인증

```python
# 메타데이터로 토큰 전달
class AuthMetadataPlugin(grpc.AuthMetadataPlugin):
    def __init__(self, token: str):
        self._token = token
    
    def __call__(self, context, callback):
        metadata = (('authorization', f'Bearer {self._token}'),)
        callback(metadata, None)


# 클라이언트 설정
call_credentials = grpc.metadata_call_credentials(
    AuthMetadataPlugin('my_jwt_token')
)

# TLS와 결합
composite_credentials = grpc.composite_channel_credentials(
    channel_credentials,  # TLS 크리덴셜
    call_credentials      # 토큰 크리덴셜
)

channel = grpc.secure_channel('localhost:50051', composite_credentials)
```

### 9.4 Rate Limiting

```python
# 서버 측 Rate Limiter
import time
from collections import defaultdict
from threading import Lock


class RateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self._max_requests = max_requests
        self._window = window_seconds
        self._requests = defaultdict(list)
        self._lock = Lock()
    
    def is_allowed(self, client_id: str) -> bool:
        now = time.time()
        
        with self._lock:
            # 오래된 요청 제거
            self._requests[client_id] = [
                t for t in self._requests[client_id]
                if now - t < self._window
            ]
            
            if len(self._requests[client_id]) >= self._max_requests:
                return False
            
            self._requests[client_id].append(now)
            return True


class RateLimitInterceptor(grpc.ServerInterceptor):
    def __init__(self, max_requests: int = 100, window_seconds: int = 60):
        self._limiter = RateLimiter(max_requests, window_seconds)
    
    def intercept_service(self, continuation, handler_call_details):
        # 클라이언트 식별 (예: IP 또는 토큰)
        metadata = dict(handler_call_details.invocation_metadata or [])
        client_id = metadata.get('x-client-id', 'anonymous')
        
        if not self._limiter.is_allowed(client_id):
            def abort(request, context):
                context.abort(
                    grpc.StatusCode.RESOURCE_EXHAUSTED,
                    "Rate limit exceeded"
                )
            return grpc.unary_unary_rpc_method_handler(abort)
        
        return continuation(handler_call_details)
```

---

## 10. 모범 사례 및 안티패턴

### 10.1 Proto 파일 설계

**✅ 좋은 예:**

```protobuf
// 명확한 네이밍
message CreateUserRequest {
  string username = 1;
  string email = 2;
}

// 응답 래핑 (페이지네이션, 메타데이터 확장 가능)
message ListUsersResponse {
  repeated User users = 1;
  string next_page_token = 2;
  int32 total_count = 3;
}

// Enum 첫 번째 값은 UNSPECIFIED
enum Status {
  STATUS_UNSPECIFIED = 0;
  STATUS_ACTIVE = 1;
}
```

**❌ 나쁜 예:**

```protobuf
// 직접 반복 필드 반환 (확장 불가)
rpc ListUsers(Empty) returns (stream User);

// 너무 일반적인 메시지
message Request {
  string type = 1;
  bytes data = 2;
}

// Enum 첫 값이 0이 아님
enum Status {
  ACTIVE = 1;  // 0이 없으면 기본값 문제
}
```

### 10.2 서비스 설계

gRPC는 REST와 달리 **스트리밍과 복잡한 RPC 패턴**에서 강점을 가진다. 단순 CRUD는 REST로도 충분하다.

**✅ gRPC에 적합한 설계:**

```protobuf
// 실시간 스트리밍 서비스
service TradingService {
  // 실시간 주가 구독 (Server Streaming)
  rpc SubscribePrices(SubscribeRequest) returns (stream PriceUpdate);
  
  // 대량 주문 처리 (Client Streaming)
  rpc BatchOrders(stream Order) returns (BatchResult);
  
  // 실시간 거래 (Bidirectional)
  rpc LiveTrade(stream TradeRequest) returns (stream TradeResponse);
}

// 마이크로서비스 내부 통신 (고성능 필요)
service RecommendationService {
  // ML 모델 호출 - 지연시간 중요
  rpc GetRecommendations(UserContext) returns (stream Product);
}

// 대용량 데이터 처리
service DataPipelineService {
  // 파일 청크 업로드
  rpc UploadData(stream DataChunk) returns (UploadResult);
  
  // 처리 결과 스트리밍
  rpc ProcessAndStream(ProcessRequest) returns (stream ProcessedItem);
}
```

**⚠️ gRPC가 굳이 필요 없는 경우:**

```protobuf
// 단순 CRUD - REST로도 충분함
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (Empty);
}
// → 브라우저에서 직접 호출해야 하면 REST가 나음
```

**❌ 나쁜 예:**

```protobuf
// 모든 것을 하나의 서비스에
service APIService {
  rpc DoEverything(Request) returns (Response);
}

// 너무 세분화된 서비스
service GetUserService {
  rpc Get(GetRequest) returns (User);
}
service CreateUserService {
  rpc Create(CreateRequest) returns (User);
}
```

### 10.3 에러 처리

**✅ 좋은 예:**

```python
def GetUser(self, request, context):
    try:
        user = self._repo.get(request.user_id)
        if not user:
            context.abort(
                grpc.StatusCode.NOT_FOUND,
                f"User {request.user_id} not found"
            )
        return user
    except DatabaseError as e:
        context.abort(
            grpc.StatusCode.INTERNAL,
            "Database error occurred"
        )
    except ValidationError as e:
        context.abort(
            grpc.StatusCode.INVALID_ARGUMENT,
            str(e)
        )
```

**❌ 나쁜 예:**

```python
def GetUser(self, request, context):
    # 모든 에러를 UNKNOWN으로 처리
    try:
        return self._repo.get(request.user_id)
    except Exception:
        context.abort(grpc.StatusCode.UNKNOWN, "Error")
```

### 10.4 버전 관리

**✅ 좋은 예:**

```protobuf
// 패키지에 버전 포함
package mycompany.userservice.v1;

// 또는 서비스에 버전 포함
service UserServiceV1 { ... }
service UserServiceV2 { ... }

// 필드 예약으로 하위 호환성
message User {
  reserved 3, 5;  // 삭제된 필드 번호
  reserved "old_field";  // 삭제된 필드 이름
  
  int32 id = 1;
  string name = 2;
}
```

### 10.5 스트리밍 주의사항

**✅ 좋은 예:**

```python
# 스트리밍에서 청크 단위 처리
def UploadFile(self, request_iterator, context):
    chunks = []
    total_size = 0
    MAX_SIZE = 100 * 1024 * 1024  # 100MB
    
    for chunk in request_iterator:
        total_size += len(chunk.data)
        
        if total_size > MAX_SIZE:
            context.abort(
                grpc.StatusCode.RESOURCE_EXHAUSTED,
                "File too large"
            )
        
        chunks.append(chunk.data)
    
    return UploadResponse(bytes_received=total_size)
```

**❌ 나쁜 예:**

```python
# 전체 스트림을 메모리에 로드
def UploadFile(self, request_iterator, context):
    all_data = b''.join(chunk.data for chunk in request_iterator)
    # 메모리 부족 위험
```

---

## 11. FastAPI와 gRPC 통합

### 11.1 아키텍처 패턴

일반적인 구성: 외부는 REST, 내부는 gRPC

```
                    ┌─────────────┐
    HTTP/REST       │   FastAPI   │       gRPC
    ──────────────► │   Gateway   │ ────────────────►
    Browser/Mobile  │             │   Internal Services
                    └─────────────┘
```

**이 구조를 쓰는 이유:**
- 브라우저는 gRPC 직접 호출 어려움 → REST로 노출
- 내부 서비스 간은 gRPC로 고성능 통신
- FastAPI가 API Gateway 역할

### 11.2 통합 구현

```python
# main.py - FastAPI + gRPC 동시 실행
import asyncio
import grpc.aio
from fastapi import FastAPI
from contextlib import asynccontextmanager

from generated import user_service_pb2_grpc as pb2_grpc
from server.services.user_service import UserServiceServicer


# gRPC 클라이언트 (FastAPI에서 사용)
class GRPCClient:
    def __init__(self):
        self.channel = None
        self.stub = None
    
    async def connect(self, target: str):
        self.channel = grpc.aio.insecure_channel(target)
        await self.channel.channel_ready()
        self.stub = pb2_grpc.UserServiceStub(self.channel)
    
    async def close(self):
        if self.channel:
            await self.channel.close()


grpc_client = GRPCClient()


@asynccontextmanager
async def lifespan(app: FastAPI):
    # 시작 시
    await grpc_client.connect('localhost:50051')
    yield
    # 종료 시
    await grpc_client.close()


app = FastAPI(lifespan=lifespan)


# REST 엔드포인트가 gRPC 서비스 호출
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    from generated import user_service_pb2 as pb2
    
    request = pb2.GetUserRequest(user_id=user_id)
    response = await grpc_client.stub.GetUser(request)
    
    return {
        "id": response.id,
        "username": response.username,
        "email": response.email,
    }


@app.post("/users")
async def create_user(username: str, email: str):
    from generated import user_service_pb2 as pb2
    
    request = pb2.CreateUserRequest(username=username, email=email)
    response = await grpc_client.stub.CreateUser(request)
    
    return {
        "id": response.id,
        "username": response.username,
        "email": response.email,
    }


# gRPC 서버와 FastAPI 동시 실행
async def serve_grpc():
    server = grpc.aio.server()
    pb2_grpc.add_UserServiceServicer_to_server(
        UserServiceServicer(),
        server
    )
    server.add_insecure_port('[::]:50051')
    await server.start()
    await server.wait_for_termination()


async def main():
    import uvicorn
    
    # gRPC 서버 태스크
    grpc_task = asyncio.create_task(serve_grpc())
    
    # FastAPI 서버
    config = uvicorn.Config(app, host="0.0.0.0", port=8000)
    server = uvicorn.Server(config)
    
    await asyncio.gather(
        grpc_task,
        server.serve()
    )


if __name__ == '__main__':
    asyncio.run(main())
```

### 11.3 의존성 주입

```python
# dependencies.py
from fastapi import Depends
from functools import lru_cache


class GRPCClientManager:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    async def get_user_stub(self):
        # 채널 풀에서 스텁 반환
        return grpc_client.stub


@lru_cache()
def get_grpc_manager() -> GRPCClientManager:
    return GRPCClientManager()


# 라우터에서 사용
@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    grpc: GRPCClientManager = Depends(get_grpc_manager)
):
    stub = await grpc.get_user_stub()
    # ...
```

---

## 12. 트러블슈팅

### 12.1 일반적인 에러

| 에러 | 원인 | 해결 |
|------|------|------|
| `UNAVAILABLE: Connection refused` | 서버 미실행 또는 포트 문제 | 서버 실행 확인, 방화벽 확인 |
| `DEADLINE_EXCEEDED` | 타임아웃 | 타임아웃 값 증가, 서버 성능 확인 |
| `UNIMPLEMENTED` | 메서드 미구현 | proto 재컴파일, 서비스 등록 확인 |
| `RESOURCE_EXHAUSTED` | 메시지 크기 초과 | `max_message_length` 옵션 조정 |
| `INVALID_ARGUMENT` | 잘못된 요청 | 요청 메시지 검증 |
| `INTERNAL` | 서버 내부 에러 | 서버 로그 확인 |

### 12.2 디버깅 도구

**grpcurl (CLI 도구):**

```bash
# 설치
brew install grpcurl  # macOS

# 서비스 목록 (리플렉션 필요)
grpcurl -plaintext localhost:50051 list

# 메서드 목록
grpcurl -plaintext localhost:50051 list user.UserService

# 메서드 호출
grpcurl -plaintext -d '{"user_id": 1}' \
  localhost:50051 user.UserService/GetUser
```

**grpc_cli:**

```bash
# 서비스 확인
grpc_cli ls localhost:50051

# 메서드 호출
grpc_cli call localhost:50051 user.UserService.GetUser \
  "user_id: 1"
```

### 12.3 로깅 설정

```python
import logging
import grpc

# gRPC 내부 로깅 활성화
os.environ['GRPC_VERBOSITY'] = 'DEBUG'
os.environ['GRPC_TRACE'] = 'all'

# Python 로깅
logging.basicConfig(level=logging.DEBUG)

# 또는 프로그래밍 방식
grpc.aio.init_grpc_aio()  # aio 초기화
```

### 12.4 성능 문제 진단

```python
# 지연시간 측정
import time


class LatencyInterceptor(grpc.ServerInterceptor):
    def intercept_service(self, continuation, handler_call_details):
        start = time.perf_counter()
        response = continuation(handler_call_details)
        elapsed = time.perf_counter() - start
        
        if elapsed > 1.0:  # 1초 이상
            logging.warning(
                f"Slow RPC: {handler_call_details.method} "
                f"took {elapsed:.2f}s"
            )
        
        return response
```

### 12.5 메모리 누수 체크

```python
# 스트리밍에서 제너레이터 정리
def ListUsers(self, request, context):
    try:
        for user in self._iterate_users():
            if not context.is_active():
                break
            yield user
    finally:
        # 리소스 정리
        self._cleanup()
```

---

## 요약

### gRPC를 쓰는 진짜 이유

| 강점 | 설명 |
|------|------|
| **스트리밍** | 양방향 실시간 통신 (REST로 불가능) |
| **성능** | 바이너리 직렬화 + HTTP/2 멀티플렉싱 |
| **타입 안전성** | .proto 파일로 계약 강제, 컴파일 타임 검증 |
| **코드 생성** | 클라이언트/서버 stub 자동 생성 |

### gRPC가 필요 없는 경우

- 브라우저 직접 호출 → REST
- 단순 CRUD → REST
- 외부 공개 API → REST
- 디버깅 편의성 중요 → REST

### 핵심 정리

1. **스트리밍이 필요하면** → gRPC
2. **마이크로서비스 내부 통신** → gRPC 고려
3. **브라우저/외부 API** → REST
4. **단순 CRUD** → REST로 충분

**학습 순서:**
1. Proto 문법 숙지
2. Unary RPC 구현
3. 스트리밍 패턴 학습 (gRPC의 핵심)
4. 인터셉터 및 보안
5. 프로덕션 배포

---

## 참고 자료

- [gRPC 공식 문서](https://grpc.io/docs/)
- [Protocol Buffers 가이드](https://protobuf.dev/)
- [gRPC Python 예제](https://github.com/grpc/grpc/tree/master/examples/python)
- [Awesome gRPC](https://github.com/grpc-ecosystem/awesome-grpc)

## 관련 문서
- [[proto 작성]]
- [[gRPC 참고 사이트]]
- [[IPC (Inter-Process Communication)|IPC]]
