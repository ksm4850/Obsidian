
Python 코드를 C 언어 코드로 변환한 후, 이를 다시 기계어로 컴파일하는 AOT 컴파일러이다.

<details>
<summary>상세</summary>

단순히 Python 인터프리터와 스크립트, 라이브러리 등을 한꺼번에 패키징할 뿐인 PyInstaller와 달리, Nuitka는 실제 컴파일 과정을 거친다는 특징이 있다.
<br/>
파이썬은 예로부터 일반 사용자에게 프로그램을 배포하는 것이 매우 까다로운 것으로 악명높았다. 아무리 PyPI를 통해 배포를 받는 것이 타 언어에 비해 쉽다고 해도, **터미널을 열고 명령어를 작성하는 것** 그 자체가 최종 사용자들에게는 크나큰 벽이다. 그러나 파이썬 커뮤니티에서는 이에 대해 별 반응이 없고, PSF에서도 이를 진지하게 해결해주려는 모습을 보이지 않아 개발자들이 각자도생을 해야만 했다.  
 <br/>
그 결과물로 나왔던 Py2exe, PyInstaller, cx_Freeze 등의 기존 패키징 도구는 Python 스크립트와 CPython 런타임을 하나의 실행 프로그램으로 단순히 묶어버리는 단순무식한 방법인데다, 그 와중에도 서드파티 프로그램이라 동적으로 로드되는 모듈이나 일부 패키지와 제대로 호환되지 않는 잠재적 문제가 존재한다. 이 탓에 진지하게 배포를 생각해야 할 정도로 프로그램이 커지면, 정작 배포의 시도가 거의 불가능해지는 모순까지 발생하였다.  
<br/>
Nuitka는 이런 기존 패키징 도구들의 문제를 해결하기 위해 나타난 패키징 도구 중 하나로, 비슷한 시기 주목받는 GraalPy 등과 함께 **아예 네이티브 실행을 추구하기 위해 Python을 다른 언어로 컴파일하는** 특징이 있다. 비록 사용하기 복잡하다는 단점이 있지만, 잘만 사용하면 깔끔한 배포가 가능하다는 점에서 큰 주목을 받고 있다.

</details>

설치
```bash
pip install nuitka
```
OS별 C 컴파일러 설치
**Windows:**
```bash
# Visual Studio Build Tools 다운로드
# https://visualstudio.microsoft.com/downloads/
# 또는
choco install mingw
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install build-essential
sudo apt-get install ccache  # 빌드 속도 향상
```

**macOS:**
```bash
xcode-select --install
```

