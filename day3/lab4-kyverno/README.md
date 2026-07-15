# Lab 4 — Kyverno 정책 엔진

## 목표
YAML 기반 Kyverno로 validate(차단)와 mutate(자동 주입) 정책을 작성하고 적용.
Gatekeeper와 달리 Rego 없이 YAML만으로 정책을 표현하는 방식 체험.

## Kyverno 개념

### 정책 종류

| 종류 | 역할 | 이번 실습 |
|---|---|---|
| validate | 조건 미충족 리소스 차단 | team 레이블 없는 Pod 차단 |
| mutate | 리소스 배포 시 자동 설정 주입 | imagePullPolicy: Always 자동 주입 |
| generate | 리소스 생성 시 연관 리소스 자동 생성 | 미실습 |

### Gatekeeper와 비교

| | Gatekeeper | Kyverno |
|---|---|---|
| 정책 언어 | Rego | YAML |
| 정책 구조 | ConstraintTemplate + Constraint | ClusterPolicy 하나 |
| 진입장벽 | 높음 | 낮음 |
| mutate | 복잡 | 간단 |
| 이미지 서명 검증 | 미지원 | verifyImages 지원 |

## 실습 흐름

1. Kyverno 설치 (Helm)
2. validate 정책 — team 레이블 없는 Pod 차단
3. bad-pod → 차단, good-pod → 통과 확인
4. validate 정책 삭제
5. mutate 정책 — imagePullPolicy: Always 자동 주입
6. Deployment 생성 후 자동 주입 확인
7. 전체 정리

## 설치

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
kubectl get pods -n kyverno
# admission-controller까지 모두 Running 확인 후 진행
```

## 파일 설명

| 파일 | 종류 | 설명 |
|---|---|---|
| `policy-require-labels.yaml` | validate | team 레이블 없는 Pod 차단 |
| `policy-imagepull-always.yaml` | mutate | imagePullPolicy: Always 자동 주입 |
| `bad-pod.yaml` | 테스트 | team 레이블 없음 → 차단 |
| `good-pod-validate.yaml` | 테스트 | team: devsecops 있음 → 통과 |
