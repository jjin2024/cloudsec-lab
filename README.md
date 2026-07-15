# Cloud Security Lab

KISIA ICT융합산업보안 인력양성과정 — 클라우드 보안 실습 정리

## 과정 개요

- **기간**: 2026.07.13 ~ 07.15 (3일)
- **주관**: KISIA 한국정보보호교육원
- **주제**: 클라우드 거버넌스 / 클라우드 네이티브 보안 / DevSecOps

---

## 실습 구성

### Day 2 — 클라우드 네이티브 및 컨테이너 보안

| 폴더 | 내용 |
|---|---|
| [lab1-aks-psa](./day2/lab1-aks-psa/) | Azure AKS + Pod Security Admission |
| [lab2-cloudformation](./day2/lab2-cloudformation/) | AWS CloudFormation IaC 보안 |

### Day 3 — 실무형 DevSecOps 구축

| 폴더 | 내용 |
|---|---|
| [lab1-github-actions](./day3/lab1-github-actions/) | GitHub Actions DevSecOps 파이프라인 |
| [lab2-container-security](./day3/lab2-container-security/) | Trivy + Syft + Cosign 공급망 보안 |
| [lab3-gatekeeper](./day3/lab3-gatekeeper/) | OPA Gatekeeper 정책 엔진 |
| [lab4-kyverno](./day3/lab4-kyverno/) | Kyverno 정책 엔진 |

---

## 환경

- Ubuntu 24.04 (VMware)
- Azure CLI / kubectl / AWS CLI
- Docker / Minikube / Helm
- Trivy / Syft / Cosign

## 주의사항

- `cosign.key`, `.pem`, AWS credentials 등 민감정보는 포함되지 않음
- `cosign.pub`은 이미지 서명 검증용으로 공개 배포 목적
