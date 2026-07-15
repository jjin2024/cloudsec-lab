# Lab 2 — AWS CloudFormation IaC 보안

## 목표
CloudFormation 템플릿에 보안 설정을 코드로 박아두면 배포할 때마다 동일한 보안 수준이 자동 적용됨을 확인.
cfn-lint로 배포 전 정적 검증(PaC)을 수행하고 IMDSv2, S3 보안 설정을 직접 검증.

## 핵심 보안 설정

**EC2 — IMDSv2 강제**
```yaml
MetadataOptions:
  HttpTokens: required
  HttpEndpoint: enabled
```
- IMDSv1은 토큰 없이 메타데이터 접근 가능 → SSRF 공격에 취약
- IMDSv2는 토큰 필수 → 토큰 없으면 401 반환

**보안그룹 — 최소 허용**
```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: <본인 IP>/32   # 본인 IP만 허용
```

**S3 — 퍼블릭 접근 차단 + 암호화 + HTTPS 강제**
```yaml
PublicAccessBlockConfiguration:
  BlockPublicAcls: true
  BlockPublicPolicy: true
  IgnorePublicAcls: true
  RestrictPublicBuckets: true

BucketEncryption:
  ServerSideEncryptionConfiguration:
    - ServerSideEncryptionByDefault:
        SSEAlgorithm: AES256

# HTTPS 강제 (HTTP 접근 Deny)
Condition:
  Bool:
    aws:SecureTransport: false
```

## 실습 흐름

1. IAM User 생성 (`cloudformation-lab-user`) + Access Key 발급
2. `aws configure` 인증 설정
3. cfn-lint 설치 (venv)
4. EC2 Key Pair 생성
5. `template.yaml` 작성 + cfn-lint 검증
6. CloudFormation 스택 생성
7. 보안 설정 검증 (EC2 IMDSv2, S3 암호화/퍼블릭 차단)
8. 리소스 정리

## 검증 명령어

```bash
# cfn-lint 검증 (아무것도 안 뜨면 통과)
cfn-lint template.yaml

# IMDSv2 검증 (EC2 접속 후)
curl -w "%{http_code}\n" http://169.254.254.254/latest/meta-data/
# → 401 나오면 정상
```

## 파일 설명

| 파일 | 설명 |
|---|---|
| `template.yaml` | VPC / EC2 / S3 보안 설정 포함 CloudFormation 템플릿 |

## 주의사항

- AWS Access Key / Secret Key는 절대 커밋하지 말 것
- 실습 후 반드시 스택 삭제 (EC2, S3, EIP 과금 주의)
- S3 버킷은 스택 삭제 전 객체 비워야 삭제 가능
