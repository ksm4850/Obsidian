# gRPC Protocol Buffers 작성 가이드

## 1. 기본 문법

### 1.1 파일 구조

```protobuf
// 1. 문법 버전 선언 (필수, 파일 최상단)
syntax = "proto3";

// 2. 패키지 선언
package company.service.v1;

// 3. 옵션 (언어별 설정)
option go_package = "github.com/company/api/service/v1";
option java_package = "com.company.service.v1";
option java_multiple_files = true;

// 4. import
import "google/protobuf/timestamp.proto";
import "common/types.proto";

// 5. 메시지 정의
message MyMessage { ... }

// 6. 서비스 정의
service MyService { ... }
```

### 1.2 스칼라 타입

|Proto 타입|Python|Go|설명|
|---|---|---|---|
|`double`|float|float64|64비트 부동소수점|
|`float`|float|float32|32비트 부동소수점|
|`int32`|int|int32|가변 길이, 음수 비효율|
|`int64`|int|int64|가변 길이, 음수 비효율|
|`uint32`|int|uint32|가변 길이, 부호 없음|
|`uint64`|int|uint64|가변 길이, 부호 없음|
|`sint32`|int|int32|음수에 효율적 (ZigZag)|
|`sint64`|int|int64|음수에 효율적 (ZigZag)|
|`fixed32`|int|uint32|고정 4바이트, 큰 수에 효율|
|`fixed64`|int|uint64|고정 8바이트, 큰 수에 효율|
|`sfixed32`|int|int32|고정 4바이트, 부호 있음|
|`sfixed64`|int|int64|고정 8바이트, 부호 있음|
|`bool`|bool|bool|불리언|
|`string`|str|string|UTF-8 문자열|
|`bytes`|bytes|[]byte|바이너리 데이터|

### 1.3 필드 번호 규칙

```protobuf
message Example {
  // 1-15: 1바이트 인코딩 → 자주 쓰는 필드에 할당
  string id = 1;
  string name = 2;
  int32 status = 3;
  
  // 16-2047: 2바이트 인코딩
  string description = 16;
  
  // 19000-19999: 예약됨 (protobuf 내부용, 사용 금지)
  
  // 최대값: 536,870,911 (2^29 - 1)
}
```

---

## 2. 메시지 정의

### 2.1 기본 메시지

```protobuf
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  int64 created_at = 4;
}
```

### 2.2 optional 필드 (proto3)

```protobuf
message SearchRequest {
  string query = 1;
  
  // optional: 값이 설정되었는지 구분 가능
  optional int32 page_size = 2;
  optional string cursor = 3;
}
```

### 2.3 repeated (배열/리스트)

```protobuf
message UserList {
  repeated User users = 1;           // 일반 리스트
  repeated string tags = 2;          // 스칼라 리스트
  repeated int32 scores = 3 [packed = true];  // packed (기본값)
}
```

### 2.4 map (딕셔너리)

```protobuf
message Config {
  map<string, string> settings = 1;     // string → string
  map<int32, User> user_cache = 2;      // int → message
  
  // 주의: key는 스칼라만 가능 (float, bytes, message 불가)
}
```

### 2.5 중첩 메시지

```protobuf
message Order {
  string id = 1;
  
  // 중첩 정의
  message Item {
    string product_id = 1;
    int32 quantity = 2;
    double price = 3;
  }
  
  repeated Item items = 2;
  
  // 다른 곳에서 참조: Order.Item
}
```

### 2.6 oneof (선택적 필드 그룹)

```protobuf
message PaymentMethod {
  string id = 1;
  
  // 셋 중 하나만 설정 가능
  oneof method {
    CreditCard credit_card = 2;
    BankTransfer bank_transfer = 3;
    Crypto crypto = 4;
  }
}

message ApiResponse {
  oneof result {
    SuccessData data = 1;
    ErrorDetail error = 2;
  }
}
```

---

## 3. Enum 정의

### 3.1 기본 규칙

```protobuf
enum Status {
  // 0번은 반드시 UNSPECIFIED (기본값, 미설정 상태)
  STATUS_UNSPECIFIED = 0;
  STATUS_PENDING = 1;
  STATUS_ACTIVE = 2;
  STATUS_INACTIVE = 3;
  STATUS_DELETED = 4;
}

// 네이밍: ENUM_NAME_VALUE_NAME
enum GpuState {
  GPU_STATE_UNSPECIFIED = 0;
  GPU_STATE_IDLE = 1;
  GPU_STATE_BUSY = 2;
  GPU_STATE_ERROR = 3;
}
```

### 3.2 별칭 허용

```protobuf
enum Priority {
  option allow_alias = true;  // 같은 값 허용
  
  PRIORITY_UNSPECIFIED = 0;
  PRIORITY_LOW = 1;
  PRIORITY_NORMAL = 2;
  PRIORITY_DEFAULT = 2;  // NORMAL과 같은 값
  PRIORITY_HIGH = 3;
}
```

---

## 4. 서비스 정의

### 4.1 RPC 패턴

```protobuf
service GpuService {
  // Unary: 1 요청 → 1 응답
  rpc GetStatus(GetStatusRequest) returns (GetStatusResponse);
  
  // Server Streaming: 1 요청 → N 응답
  rpc WatchStatus(WatchRequest) returns (stream GpuStatus);
  
  // Client Streaming: N 요청 → 1 응답
  rpc UploadData(stream DataChunk) returns (UploadResponse);
  
  // Bidirectional Streaming: N 요청 ↔ N 응답
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}
```

### 4.2 요청/응답 네이밍

```protobuf
// 규칙: {MethodName}Request, {MethodName}Response
service InferenceService {
  rpc RunInference(RunInferenceRequest) returns (RunInferenceResponse);
  rpc BatchInference(BatchInferenceRequest) returns (BatchInferenceResponse);
}

message RunInferenceRequest {
  string model_id = 1;
  bytes input = 2;
}

message RunInferenceResponse {
  bytes output = 1;
  int64 latency_ms = 2;
}
```

---

## 5. 예약어 및 특수 키워드

### 5.1 예약어 목록

```
syntax, import, weak, public, package, option, 
message, enum, service, rpc, returns, stream,
repeated, optional, required (proto2), 
oneof, map, reserved, extensions, extend,
true, false, inf, nan
```

### 5.2 reserved (하위 호환성)

```protobuf
message User {
  reserved 2, 15, 9 to 11;           // 필드 번호 예약
  reserved "old_name", "temp_field"; // 필드 이름 예약
  
  string id = 1;
  string email = 3;
  // 2번은 과거에 사용했으므로 재사용 금지
}
```

### 5.3 deprecated

```protobuf
message User {
  string id = 1;
  string email = 2 [deprecated = true];  // 사용 중단 표시
  string new_email = 3;
}
```

---

## 6. import 규칙

### 6.1 기본 import

```protobuf
// 상대 경로
import "common/types.proto";
import "gpu/monitoring.proto";

// Google Well-Known Types
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";
import "google/protobuf/any.proto";
import "google/protobuf/wrappers.proto";
```

### 6.2 public import (재노출)

```protobuf
// all.proto
import public "user.proto";
import public "order.proto";

// 다른 파일에서 all.proto만 import해도 user, order 접근 가능
```

---

## 7. Google Well-Known Types

### 7.1 자주 쓰는 타입

```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";
import "google/protobuf/wrappers.proto";
import "google/protobuf/any.proto";
import "google/protobuf/struct.proto";

message Example {
  google.protobuf.Timestamp created_at = 1;
  google.protobuf.Duration timeout = 2;
  google.protobuf.Int32Value nullable_count = 3;  // nullable int
  google.protobuf.StringValue nullable_name = 4;  // nullable string
  google.protobuf.Any payload = 5;                // 동적 타입
  google.protobuf.Struct metadata = 6;            // JSON-like
}

// Empty 사용 (파라미터/반환값 없을 때)
service HealthService {
  rpc Ping(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

---

## 8. 파일 구조 및 분리

### 8.1 권장 디렉토리 구조

```
proto/
├── buf.yaml                    # buf 설정
├── buf.gen.yaml               # 코드 생성 설정
│
├── common/                    # 공통 타입
│   ├── types.proto           # 범용 타입 (Pagination 등)
│   ├── error.proto           # 에러 정의
│   └── health.proto          # 헬스체크
│
├── gpu/                       # GPU 도메인
│   └── v1/                   # 버전
│       ├── monitoring.proto  # 모니터링 서비스
│       ├── inference.proto   # 추론 서비스
│       └── types.proto       # GPU 관련 타입
│
└── user/                      # 사용자 도메인
    └── v1/
        ├── user.proto
        └── auth.proto
```

### 8.2 파일 분리 기준

|분리 단위|파일|
|---|---|
|도메인별|`gpu/`, `user/`, `order/`|
|버전별|`v1/`, `v2/`|
|역할별|`service.proto`, `types.proto`|
|공통|`common/error.proto`, `common/types.proto`|

### 8.3 공통 타입 예시

```protobuf
// common/types.proto
syntax = "proto3";
package common;

message Pagination {
  int32 page = 1;
  int32 page_size = 2;
  int32 total = 3;
}

message PageToken {
  string next_token = 1;
  string prev_token = 2;
}
```

```protobuf
// common/error.proto
syntax = "proto3";
package common;

message ErrorDetail {
  int32 code = 1;
  string message = 2;
  string field = 3;
  map<string, string> metadata = 4;
}

enum ErrorCode {
  ERROR_CODE_UNSPECIFIED = 0;
  ERROR_CODE_INVALID_ARGUMENT = 1;
  ERROR_CODE_NOT_FOUND = 2;
  ERROR_CODE_ALREADY_EXISTS = 3;
  ERROR_CODE_PERMISSION_DENIED = 4;
  ERROR_CODE_INTERNAL = 5;
}
```

---

## 9. 네이밍 컨벤션

|요소|스타일|예시|
|---|---|---|
|파일명|snake_case|`gpu_monitoring.proto`|
|패키지|lowercase.dot|`company.gpu.v1`|
|메시지|PascalCase|`GpuStatus`|
|필드|snake_case|`model_id`|
|Enum|SCREAMING_SNAKE|`GPU_STATE_IDLE`|
|서비스|PascalCase|`InferenceService`|
|RPC 메서드|PascalCase|`RunInference`|

---

## 10. 실무 팁

### 10.1 버전 관리

```protobuf
// Breaking Change 시 버전 분리
package company.gpu.v1;  // 기존
package company.gpu.v2;  // 새 버전

// 클라이언트가 점진적 마이그레이션 가능
```

### 10.2 하위 호환성 유지

```protobuf
// ✅ 호환 가능한 변경
- 새 필드 추가 (새 번호로)
- 새 enum 값 추가
- 새 서비스/메서드 추가
- 필드를 optional로 변경

// ❌ Breaking Change (피해야 함)
- 필드 번호 변경
- 필드 타입 변경
- 필드/메시지 이름 변경
- required ↔ optional 변경
- enum 값 번호 변경
```

### 10.3 주석 작성

```protobuf
// 서비스 수준 설명
// GPU 리소스 모니터링 및 추론 서비스
service GpuService {
  // 단일 GPU 상태 조회
  // 
  // 지정된 GPU의 현재 상태를 반환합니다.
  // GPU ID가 존재하지 않으면 NOT_FOUND 에러를 반환합니다.
  rpc GetStatus(GetStatusRequest) returns (GetStatusResponse);
}

message GpuStatus {
  // GPU 고유 식별자 (예: "GPU-0", "GPU-1")
  string gpu_id = 1;
  
  // GPU 메모리 사용량 (bytes)
  // 0이면 측정 불가 상태
  int64 memory_used = 2;
  
  // GPU 사용률 (0-100)
  // -1이면 측정 불가 상태
  int32 utilization = 3;
}
```

### 10.4 buf 설정

```yaml
# buf.yaml
version: v1
name: buf.build/company/api
breaking:
  use:
    - FILE
lint:
  use:
    - DEFAULT
  except:
    - PACKAGE_VERSION_SUFFIX
```

```yaml
# buf.gen.yaml
version: v1
plugins:
  - plugin: buf.build/protocolbuffers/python
    out: gen/python
  - plugin: buf.build/grpc/python
    out: gen/python
```

---

## 11. 완성 예시: GPU 모니터링

```protobuf
// gpu/v1/monitoring.proto
syntax = "proto3";

package company.gpu.v1;

option go_package = "github.com/company/api/gpu/v1;gpuv1";

import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";
import "common/error.proto";

// GPU 모니터링 서비스
service GpuMonitoringService {
  // 전체 GPU 목록 조회
  rpc ListGpus(google.protobuf.Empty) returns (ListGpusResponse);
  
  // 단일 GPU 상태 조회
  rpc GetGpuStatus(GetGpuStatusRequest) returns (GpuStatus);
  
  // GPU 상태 실시간 스트리밍
  rpc WatchGpuStatus(WatchGpuStatusRequest) returns (stream GpuStatus);
}

message ListGpusResponse {
  repeated GpuInfo gpus = 1;
}

message GpuInfo {
  string gpu_id = 1;
  string name = 2;
  int64 total_memory = 3;
}

message GetGpuStatusRequest {
  string gpu_id = 1;
}

message WatchGpuStatusRequest {
  repeated string gpu_ids = 1;
  int32 interval_ms = 2;
}

message GpuStatus {
  string gpu_id = 1;
  int64 memory_used = 2;
  int64 memory_total = 3;
  int32 utilization = 4;
  int32 temperature = 5;
  GpuState state = 6;
  google.protobuf.Timestamp timestamp = 7;
}

enum GpuState {
  GPU_STATE_UNSPECIFIED = 0;
  GPU_STATE_IDLE = 1;
  GPU_STATE_BUSY = 2;
  GPU_STATE_ERROR = 3;
  GPU_STATE_OFFLINE = 4;
}
```

---

## 12. 체크리스트

- [ ] `syntax = "proto3"` 선언
- [ ] 패키지에 버전 포함 (`v1`, `v2`)
- [ ] 필드 번호 1-15는 자주 쓰는 필드에 할당
- [ ] Enum 0번은 `UNSPECIFIED`
- [ ] 삭제된 필드는 `reserved` 처리
- [ ] 모든 메시지/필드에 주석 작성
- [ ] Request/Response 네이밍 통일
- [ ] 공통 타입은 별도 파일로 분리
- [ ] buf lint 통과 확인
- [ ] Breaking change 검증