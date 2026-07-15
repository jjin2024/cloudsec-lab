# Lab 2 — 컨테이너 공급망 보안 (Trivy + Syft + Cosign)

## 목표
컨테이너 이미지 취약점 스캔 → SBOM 생성 → 이미지 서명 및 Attestation까지 공급망 보안 전체 흐름 체험.

## 공급망 보안 흐름

```
이미지 빌드
    ↓
Trivy로 취약점 스캔
    ↓
Trivy / Syft로 SBOM 생성
    ↓
Cosign으로 이미지 서명 (cosign sign)
    ↓
Cosign으로 SBOM Attestation 첨부 (cosign attest)
    ↓
배포 시 서명 검증 (cosign verify / cosign verify-attestation)
```

## 핵심 개념

### SBOM (Software Bill of Materials)
컨테이너 이미지 안에 들어있는 패키지, 라이브러리, 버전 정보를 목록화한 문서.
형식: CycloneDX, SPDX

### cosign sign vs cosign attest (자주 헷갈림)

| | `cosign sign` | `cosign attest` |
|---|---|---|
| 대상 | 이미지 자체 | SBOM 등 메타데이터 |
| 의미 | 내가 빌드한 이미지가 맞다 | 이미지 구성 요소 목록이 맞다 |
| 검증 | `cosign verify` | `cosign verify-attestation` |

→ 둘 다 개인키로 서명, 공개키로 검증 (비대칭 암호화)

### CR에서 pull할 때 서명 검증
자동으로 확인되지 않음. Kyverno `verifyImages` 정책으로 강제해야 함.

## 주요 명령어

```bash
# 취약점 스캔
trivy image nginx:1.14
trivy image nginx:1.14 | grep -E "CRITICAL|HIGH" | wc -l

# SBOM 생성
trivy image --format cyclonedx --output sbom.cdx.json <IMAGE>
syft <IMAGE> -o spdx-json > sbom.spdx.json

# 키 생성
cosign generate-key-pair

# 이미지 서명 / 검증
cosign sign --key cosign.key <IMAGE_URI>
cosign verify --key cosign.pub <IMAGE_URI>

# SBOM Attestation 생성 / 검증
cosign attest --key cosign.key --predicate sbom.cdx.json --type cyclonedx <IMAGE_URI>
cosign verify-attestation --key cosign.pub --type cyclonedx <IMAGE_URI>
```

## 파일 설명

| 파일 | 설명 |
|---|---|
| `Dockerfile` | Node.js 앱 컨테이너 이미지 빌드용 |
| `.dockerignore` | 개인키, SBOM 등 이미지에 포함되면 안 되는 파일 제외 |
| `cosign.pub` | 이미지 서명 검증용 공개키 |

## 주의사항

- `cosign.key` (개인키)는 절대 커밋하지 말 것 → `.gitignore`에 포함
- `.dockerignore`에서 `*.json` 쓰면 `package.json`도 제외되므로 SBOM 파일만 명시적으로 제외할 것
