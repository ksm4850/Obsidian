gpu 리소스 사용하고싶을떄
{
  "data-root": "/mnt/disk1/docker",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime"
    }
  },
  "default-runtime": "nvidia"
}


## Docker daemon.json의 runtime 설정 설명

### `runtimes`

Docker 컨테이너가 실행될 때 사용할 **OCI 런타임**을 등록하는 설정입니다.

기본적으로 Docker는 `runc`라는 런타임을 사용합니다. 이건 CPU/메모리 같은 일반 리소스만 컨테이너에 할당합니다.

`nvidia` 런타임(`nvidia-container-runtime`)을 등록하면, 컨테이너 내부에서 **호스트의 NVIDIA GPU에 접근**할 수 있게 됩니다. 이 런타임은 컨테이너 시작 시 GPU 드라이버, CUDA 라이브러리 등을 자동으로 마운트해주는 역할을 합니다.

json

```json
"runtimes": {
  "nvidia": {                          // "nvidia"라는 이름으로 런타임 등록
    "path": "nvidia-container-runtime"  // 실제 실행할 바이너리 경로
  }
}
```

이렇게 등록하면 `docker run --runtime=nvidia ...` 형태로 특정 컨테이너에만 선택적으로 GPU 런타임을 적용할 수 있습니다.

### `default-runtime`

**모든 컨테이너에 기본 적용되는 런타임**을 지정합니다.

- 설정하지 않으면 기본값은 `runc`
- `"default-runtime": "nvidia"`로 설정하면, `--runtime` 플래그 없이 `docker run`만 해도 **자동으로 nvidia 런타임이 적용**됩니다

### 실질적인 차이 정리

|설정|효과|
|---|---|
|`runtimes`만 등록|`--runtime=nvidia` 명시해야 GPU 사용 가능|
|`default-runtime`까지 설정|모든 컨테이너가 기본으로 GPU 접근 가능|

### 언제 `default-runtime`을 설정하나?

- GPU 서버에서 **거의 모든 컨테이너가 GPU를 사용**하는 경우 (ML/DL 전용 서버 등)
- `docker-compose`에서 매번 runtime 지정하기 번거로울 때

반대로 GPU를 일부 컨테이너만 쓴다면, `default-runtime`은 `runc`로 두고 필요한 컨테이너에만 `--runtime=nvidia`를 명시하는 게 더 안전합니다. GPU가 불필요한 컨테이너에까지 nvidia 런타임이 적용되면 nvidia-container-runtime이 없는 환경에서 문제가 생길 수 있기 때문입니다.