# iPhone Duo Migration Team for Codex

Codex가 iOS 프로젝트를 조사하고, 적응형 레이아웃 마이그레이션을 계획하고, 최소 범위로 구현한 뒤, 계획 정합성 리뷰와 QA까지 수행하도록 만드는 역할 분리 프롬프트 팩입니다.

`iPhone Duo`는 이 팩의 프로젝트 이름입니다. 프롬프트는 존재가 확인되지 않은 기기 특성이나 API를 가정하지 않습니다. 먼저 현재 Xcode SDK와 프로젝트에서 지원 근거를 확인하고, 전용 API가 없으면 일반적인 적응형 레이아웃과 멀티태스킹 대응만 수행합니다.

## 구성

```text
iphone-duo-migration-codex-pack/
├── AGENTS.md
└── prompts/
    ├── supervisor.md
    ├── project-analyzer.md
    ├── migration-architect.md
    ├── implementer.md
    ├── migration-reviewer.md
    └── qa-engineer.md
```

## 설치

이 폴더의 `AGENTS.md`와 `prompts/`를 대상 iOS 저장소 루트에 복사합니다. 기존 `AGENTS.md`가 있다면 덮어쓰지 말고, 이 파일의 내용을 기존 규칙 아래에 병합하십시오. 충돌 시 저장소의 기존 안전·빌드·배포 규칙이 우선입니다.

## 원클릭 실행 문구

Codex에서 대상 저장소를 연 뒤 다음과 같이 요청합니다.

```text
prompts/supervisor.md를 읽고 iPhone Duo migration workflow를 SAFE_AUTO 모드로 끝까지 실행해줘.
현재 사용자 변경은 보존하고, push·PR·패키지 업그레이드는 하지 마.
```

`SAFE_AUTO`는 호환성 중심의 낮은 위험 변경만 자동 적용합니다. 내비게이션 구조, 멀티윈도우, 데이터 흐름, 권한·entitlement처럼 제품 동작을 바꾸는 결정은 사용자 승인을 기다리며, 근거가 불충분한 Duo 전용 UX는 추천으로만 남깁니다.

## 산출물

실행 중 생성되는 문서는 대상 저장소의 `.codex/duo-migration/` 아래에 저장됩니다.

```text
.codex/duo-migration/
├── progress.md
├── 01-project-analysis.md
├── 02-migration-plan.md
├── 03-implementation-report.md
├── 04-review-report.md
├── 05-qa-report.md
└── regression-01.md
```

프로덕션 코드는 implementer만 수정합니다. 분석가·설계자·리뷰어·QA는 소스 코드를 수정하지 않습니다. supervisor는 작업 문서와 진행 상태만 기록합니다.

## 동작 원칙

- 역할별 권한과 입출력을 분리합니다.
- 승인된 계획에 없는 변경은 좋아 보이더라도 이탈로 처리합니다.
- 구현은 완료 기준을 만족하는 최소 diff를 목표로 합니다.
- 리뷰는 추가뿐 아니라 삭제 가능한 코드도 찾습니다.
- 같은 실패가 반복되면 임시 패치를 멈추고 새 분석 컨텍스트에서 원인을 다시 조사합니다.
- 실제 SDK·문서·컴파일러가 확인하지 못한 API나 하드웨어 동작은 사용하지 않습니다.
