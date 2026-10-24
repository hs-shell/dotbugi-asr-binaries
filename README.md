# dotbugi-asr-binaries

[dotbugi-mcp](https://github.com/hs-shell/dotbugi-mcp) 의 `dotbugi-mcp asr install` 이 받는 **미리 빌드한 [antirez/qwen-asr](https://github.com/antirez/qwen-asr) 바이너리**를 만드는 저장소입니다. 사용자 PC 에서 컴파일하지 않도록 GitHub Actions 에서 빌드해 Release 에 올립니다. 모델 가중치는 여기 없습니다 (Hugging Face 에서 받음).

## 빌드

| 항목 | 값 |
|---|---|
| upstream | `antirez/qwen-asr` @ `924694251d9e0f18e5d86bbd06aa3ab5f870002d` |
| BLAS | OpenBLAS 0.3.34 정적 링크 (tarball sha256 `cd7e1298…dfed` 검증, `DYNAMIC_ARCH=1`) |
| 플래그 | `-O3 -ffast-math -fno-finite-math-only -DUSE_BLAS -DUSE_OPENBLAS -march=x86-64-v3` |
| 러너 | `ubuntu-22.04` |

| macOS arm64 | 값 |
|---|---|
| BLAS | Apple Accelerate (`-DACCELERATE_NEW_LAPACK`) |
| 플래그 | `-O3 -ffast-math -fno-finite-math-only -DUSE_BLAS -mcpu=apple-m1` |
| 최소 OS | macOS 13.3 (`MACOSX_DEPLOYMENT_TARGET`) |
| 러너 | `macos-14` — 고정 ffmpeg·Qwen3-ASR-0.6B 로 한국어 받아쓰기 스모크 테스트 후 올린다 |

- `-fno-finite-math-only`: upstream 의 `-ffast-math` 만으로는 일부 입력에서 `!!!!` 를 끝없이 출력합니다.
- `-march=x86-64-v3`: AVX2·FMA·BMI1/2·F16C·MOVBE 가 있는 x86-64 CPU (Intel Haswell·AMD Excavator 이후)가 필요합니다.
- **glibc 2.35 이상** (Ubuntu 22.04·Debian 12·Fedora 36 이후)에서 실행됩니다. 동적 의존은 libc·libm 뿐입니다.
- macOS 바이너리의 동적 의존은 시스템 라이브러리(libSystem·Accelerate)뿐입니다. Intel Mac 은 지원하지 않습니다.

워크플로: `.github/workflows/build.yml` — 수동 실행(`tag` 입력) 또는 `asr-v*` 태그 푸시. PR 에서는 빌드·스모크 테스트만 돈다. 액션은 커밋 SHA 로 고정, 기본 권한 `contents: read`, Release 잡만 `contents: write`.

## 검증

```bash
gh release download asr-v2 -R hs-shell/dotbugi-asr-binaries
sha256sum -c SHA256SUMS
```

dotbugi-mcp 는 이 `SHA256SUMS` 값을 코드(`src/asr/pins.ts`)에 고정해 두고, 받은 파일이 일치할 때만 설치합니다.

## 라이선스

- 이 저장소(워크플로·문서): MIT — [`LICENSE`](LICENSE)
- 바이너리에 포함된 코드: qwen-asr (MIT, [`licenses/qwen-asr-LICENSE`](licenses/qwen-asr-LICENSE)), OpenBLAS (BSD-3-Clause, linux-x64 만, [`licenses/OpenBLAS-LICENSE`](licenses/OpenBLAS-LICENSE)). 두 고지는 Release 에도 함께 올라갑니다.
