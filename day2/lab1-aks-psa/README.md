# Lab 1 — Azure AKS + Pod Security Admission (PSA)

## 목표
PSA의 3단계 보안 레벨과 enforce/warn/audit 모드 차이를 직접 확인.
보안 컨텍스트 없는 Pod가 restricted 정책에서 차단되는 흐름을 체험.

## PSA 개념

**3단계 보안 레벨**

| 레벨 | 설명 |
|---|---|
| `privileged` | 제한 없음 |
| `baseline` | 명백히 위험한 설정만 차단 |
| `restricted` | 보안 컨텍스트 명시 필수 |

**3가지 모드**

| 모드 | 효과 |
|---|---|
| `enforce` | 위반 Pod 실행 차단 |
| `warn` | 경고 메시지 출력 (실행은 됨) |
| `audit` | 감사 로그 기록 (실행은 됨) |

**restricted 통과 필수 조건**
```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

## 실습 흐름

1. Azure CLI + kubectl 설치
2. AKS 클러스터 생성 (koreacentral, 노드 1개)
3. `psa-test` namespace 생성 + PSA 레이블 적용
4. 일반 nginx Pod → baseline 통과, restricted 경고 확인
5. 보안 컨텍스트 갖춘 Pod → 경고 없이 통과
6. enforce=restricted 상향 → bad-pod 차단, secure-pod 통과
7. 리소스 그룹 삭제

## 주요 명령어

```bash
# namespace에 PSA 레이블 적용
kubectl label --overwrite ns psa-test \
  pod-security.kubernetes.io/enforce=baseline \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted

# 레이블 확인
kubectl get ns psa-test --show-labels

# AKS 클러스터 삭제 (리소스 그룹 통째로)
az group delete --name myResourceGroup --yes --no-wait
```

## 파일 설명

| 파일 | 설명 |
|---|---|
| `nginx-deploy.yaml` | 보안 컨텍스트 없는 일반 nginx (baseline 통과, restricted 경고) |
| `baseline-compliance.yaml` | 일부 보안 컨텍스트 적용 (restricted 경고 없음) |
| `bad-pod.yaml` | 보안 설정 없음 → restricted enforce 시 차단 |
| `secure-pod.yaml` | restricted 통과 조건 전부 충족 |
