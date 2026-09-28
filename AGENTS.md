# Adaptive Layout Migration Team — Repository Rules

사용자가 폴더블, 듀얼 패널, 대화면, 분할 화면 또는 동적 창 크기 대응을 요청하면 `prompts/supervisor.md`를 진입점으로 사용한다. supervisor는 대상 스택과 기기를 확인하고 관련 프로필 하나만 읽는다.

## 프로필 라우팅

| 대상 | 필수 프로필 | 추가 QA 문서 |
|---|---|---|
| SwiftUI/UIKit 기반 iOS adaptive·Duo 대응 | `profiles/ios-duo.md` | 프로필 내부 매트릭스 |
| React Native 기반 Android·Galaxy Fold 대응 | `profiles/react-native-android-foldable.md` | `qa/galaxy-fold-matrix.md` |

사용자가 프로필을 지정하면 그대로 사용한다. 자동 감지 결과가 혼합되거나 여러 프로필이 필요하면 분석까지만 진행하고 구현 전에 범위를 확인한다. 선택하지 않은 프로필은 읽지 않는다.

## 기본 설정

| 설정 | 기본값 |
|---|---|
| `WORKFLOW_MODE` | `SAFE_AUTO` |
| `ARTIFACT_DIR` | `.codex/adaptive-layout-migration` |
| `MAX_FIX_ATTEMPTS_PER_FINGERPRINT` | `2` |
| `MAX_TOTAL_RETURN_LOOPS` | `3` |
| `ALLOW_COMMIT` | `false` |
| `ALLOW_PUSH_OR_PR` | `false` |

저장소의 기존 안전·빌드·배포 규칙이 더 엄격하면 그 규칙이 우선한다.

## 역할 경계

| 역할 | 프롬프트 | 소스 수정 | 검사 실행 | Git 쓰기 |
|---|---|---:|---:|---:|
| supervisor | `prompts/supervisor.md` | 금지 | 결과 확인만 | 금지 |
| project-analyzer | `prompts/project-analyzer.md` | 금지 | 비파괴 진단 | 금지 |
| migration-architect | `prompts/migration-architect.md` | 금지 | 금지 | 금지 |
| implementer | `prompts/implementer.md` | 승인 범위만 | 허용 | 기본 금지 |
| migration-reviewer | `prompts/migration-reviewer.md` | 금지 | 금지 | 금지 |
| qa-engineer | `prompts/qa-engineer.md` | 금지 | 허용 | 금지 |

supervisor만 `ARTIFACT_DIR`의 보고서와 진행 상태를 기록한다. implementer만 승인 계획에 포함된 소스와 테스트를 수정한다. 다른 역할은 Markdown 보고서를 부모 에이전트에 반환한다.

## 공통 절대 규칙

1. 사용자의 기존 수정과 미추적 파일을 보존한다.
2. push, PR, merge, release, 배포는 실행하지 않는다.
3. 패키지, 도구 버전, deployment target, target SDK, entitlement, signing, capability는 승인 계획과 사용자 승인 없이 변경하지 않는다.
4. 실제 SDK, 컴파일러, 공식 문서 또는 저장소에서 확인하지 못한 API와 기기 동작을 만들지 않는다.
5. 물리 기기명이나 전역 화면 크기보다 현재 창·컨테이너 크기를 우선한다.
6. 넓은 화면이라고 무조건 2열로 바꾸지 않는다. 정보 구조와 작업 흐름이 정당화해야 한다.
7. 승인된 `02-migration-plan.md` 밖의 개선은 구현하지 않고 `DEVIATION_REQUEST`로 보고한다.
8. 완료 기준을 만족하는 최소 diff를 만든다. 계획에 없는 일반화·헬퍼·리팩터링은 추가하지 않는다.
9. 자격증명 입력, 유료 동작, 데이터 삭제·전송 등 되돌릴 수 없는 QA는 실행하지 않는다.

## 위험 분류

### SAFE

- 기존 UX를 유지하는 제약·flex·spacing·최대 너비 수정
- 현재 창 크기에 따른 단순 표시·배치 전환
- 기존 상태 구조와 navigation을 보존하는 compact fallback 수정
- 기존 테스트 범위 안의 회귀 수정

### RISKY

- navigation, scene/activity lifecycle, state ownership 또는 deep link 변경
- 새 native module, platform dependency 또는 빌드 설정 추가
- persistence, analytics, 권한, entitlement, manifest 동작 변경
- 넓은 화면이나 posture에서만 존재하는 새 사용자 흐름

### AMBIGUOUS

- 제품 근거가 없는 패널 분리
- SDK나 공식 문서에서 확인되지 않은 기기 전용 동작
- 여러 UX가 비슷한 장단점을 가져 제품 결정이 필요한 경우

`SAFE_AUTO`에서는 SAFE만 자동 구현한다. RISKY는 정확한 영향과 대안을 제시하고 승인을 기다린다. AMBIGUOUS는 추천으로만 남긴다. 선택한 프로필의 분류가 더 엄격하면 프로필을 따른다.

## 산출물 계약

```text
ARTIFACT_DIR/
  progress.md
  01-project-analysis.md
  02-migration-plan.md
  03-implementation-report.md
  04-review-report.md
  05-qa-report.md
  regression-NN.md
```

모든 보고서는 `Status`, `Evidence`, `Decision`, `Open questions`를 포함하고 사실과 추정을 분리한다. `02-migration-plan.md`가 implementer의 유일한 변경 명세다. supervisor는 architect 본문을 바꾸지 않고 문서 위의 `Supervisor approval`만 `PENDING`, `APPROVED_SAFE`, `APPROVED_BY_USER` 중 하나로 관리한다.

## 상태와 회귀 루프

```text
INIT → PROFILE_SELECTED → ANALYZED → PLANNED
→ APPROVED_SAFE | WAITING_USER_APPROVAL
→ IMPLEMENTED → REVIEW_PASS | REVIEW_RETURN
→ QA_PASS | QA_RETURN | QA_BLOCKED → COMPLETE
```

실패 fingerprint는 `<surface>::<observable symptom>::<failing check>` 형식이다. 같은 fingerprint가 두 번째 나타나거나 구현 복귀가 누적 3회면 패치를 멈춘다. 새 project-analyzer를 이전 가설 없이 호출해 원래 분석, 승인 계획, 현재 diff, 실패 증거만 제공한다. 결과를 `regression-NN.md`에 저장하고 architect가 계획을 다시 작성한다.

## 역할 호출

역할은 별도 에이전트 컨텍스트에서 순차 실행한다. supervisor는 역할 프롬프트, 선택 프로필, 정확한 입력 파일, 허용 쓰기, 금지 행동, 반환 형식을 호출 메시지에 적는다. 같은 파일을 수정하는 implementer를 동시에 둘 이상 실행하지 않는다.

에이전트 협업 기능이 없으면 한 세션에서 역할을 흉내 내지 않는다. 현재 산출물과 다음 프롬프트를 제공하고 사용자가 별도 세션에서 이어가도록 한다.
