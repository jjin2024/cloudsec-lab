# Lab 1 — GitHub Actions DevSecOps 파이프라인

## 목표
코드 push 한 번으로 코드 정적 분석 → 취약점 스캔 → 의존성 업데이트까지 자동화되는 DevSecOps 파이프라인 구축.
보안을 개발 프로세스에 녹여내는 DevSecOps의 핵심 원리 체득.

## 파이프라인 구성

```
코드 push
    ↓
GitHub Actions Demo (환경 확인)
    ↓
CodeQL (정적 분석 — 코드 취약점)
    ↓
Security Pipeline / Snyk (의존성 취약점 스캔)
    ↓
Dependabot (패키지 자동 업데이트 PR)
```

## 워크플로우 파일

| 파일 | 도구 | 역할 |
|---|---|---|
| `github-actions-demo.yml` | GitHub Actions | 환경 정보 출력, 첫 실행 확인 |
| `codeql.yml` | CodeQL | JS 코드 정적 분석 및 취약점 탐지 |
| `security.yml` | Snyk | npm 의존성 패키지 취약점 스캔 |

## 주요 설정

**CodeQL**
- `language: javascript` — JS 기준, Python/Java/Go 등으로 변경 가능
- JS 코드 없으면 에러 → `package.json` + `index.js` 추가 후 통과

**Snyk**
- `SNYK_TOKEN`을 GitHub Secrets에 등록해야 동작
- Organization 없이는 일부 기능 제한

**Dependabot**
- `.github/dependabot.yml`로 설정
- npm 패키지 매주 검사 → 업데이트 가능한 패키지 있으면 PR 자동 생성

## PAT 권한 주의

workflow 파일 push 시 PAT에 아래 권한 필요:
- `repo`
- `workflow`
- `write:packages`
