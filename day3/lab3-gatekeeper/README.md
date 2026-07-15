# Lab 3 — OPA Gatekeeper 정책 엔진

## 목표
쿠버네티스 Admission 단계에서 Gatekeeper로 정책을 강제하는 흐름 체험.
ConstraintTemplate(Rego)과 Constraint의 관계를 이해하고 레이블 없는 Deployment를 차단.

## Gatekeeper 개념

### 역할
쿠버네티스 API 서버와 실제 리소스 생성 사이에 위치하는 **Admission Webhook**.
`kubectl apply` 시 API 서버가 Gatekeeper에 먼저 검사 요청 → 통과하면 배포, 위반하면 차단.

### 동작 흐름
```
kubectl apply -f bad-pod.yaml
    ↓
API 서버가 Gatekeeper Admission Webhook 호출
    ↓
Gatekeeper가 Constraint 검사
    ↓
owner 레이블 없음 → ConstraintTemplate의 Rego가 violation 반환
    ↓
차단 (Error: admission webhook denied)
```

### ConstraintTemplate vs Constraint

| | ConstraintTemplate | Constraint |
|---|---|---|
| 역할 | 정책 "틀" 정의 | 실제 정책 적용 |
| 내용 | Rego로 "어떤 조건이 위반인가" | "어느 namespace, 어느 리소스에" |
| 재사용 | 여러 Constraint에서 재사용 가능 | ConstraintTemplate 1개에 종속 |

## 실습 흐름

1. Gatekeeper 설치 (Helm)
2. demo namespace 생성
3. ConstraintTemplate 적용 (Rego로 "owner 레이블 없으면 violation")
4. Constraint 적용 (demo namespace Deployment에 적용)
5. bad-pod → 차단 확인
6. good-pod (owner: devsecops-team) → 통과 확인
7. 정리

## 설치

```bash
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm repo update
helm install gatekeeper gatekeeper/gatekeeper -n gatekeeper-system --create-namespace
kubectl -n gatekeeper-system get pods
```

## 파일 설명

| 파일 | 설명 |
|---|---|
| `constrainttemplate-requiredlabels.yaml` | Rego로 "지정 레이블 없으면 violation" 정책 틀 정의 |
| `constraint-owner.yaml` | demo namespace Deployment에 owner 레이블 필수 정책 적용 |
| `bad-pod.yaml` | owner 레이블 없음 → 차단 |
| `good-pod.yaml` | owner: devsecops-team 있음 → 통과 |

## 주의사항

- Kyverno와 함께 사용 시 충돌 가능 → 전환 전 반드시 정리
- minikube 재시작 후 NotFound 에러는 정상 (이미 삭제된 리소스)
