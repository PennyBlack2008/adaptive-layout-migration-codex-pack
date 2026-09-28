# iPhone Duo Migration Team — Repository Rules

이 저장소에서 사용자가 iPhone Duo, 듀얼 패널, 폴더블·확장형 화면, 분할 화면, 적응형 iOS 레이아웃 마이그레이션을 요청하면 이 문서와 `prompts/`의 역할 프롬프트를 적용한다.

## 목적

현재 프로젝트와 설치된 SDK가 실제로 지원하는 범위 안에서 iOS 앱을 다양한 창 크기와 화면 구성에 적응시키고, 넓은 화면에서 유용한 2열 UX 후보를 식별한다. 특정 기기 이름, 힌지, 자세, 화면 분리 API가 실제 SDK나 공식 문서에서 확인되지 않으면 이를 발명하거나 추정 구현하지 않는다.

## 기본 설정

| 설정 | 기본값 | 의미 |
|---|---|---|
| `WORKFLOW_MODE` | `SAFE_AUTO` | 안전 변경은 자동 진행, 위험·모호 변경은 승인 대기 |
| `ARTIFACT_DIR` | `.codex/duo-migration` | 역할 간 공유 문서와 진행 상태 |
| `MAX_FIX_ATTEMPTS_PER_FINGERPRINT` | `2` | 같은 실패 증상에 대한 구현 시도 상한 |
| `MAX_TOTAL_RETURN_LOOPS` | `3` | 구현 단계로 되돌아가는 전체 횟수 상한 |
| `ALLOW_COMMIT` | `false` | 명시적 요청 전에는 커밋하지 않음 |
| `ALLOW_PUSH_OR_PR` | `false` | 이 워크플로에서는 push·PR·merge 금지 |

사용자가 다른 값을 명시하면 `progress.md`에 기록하고 그 값을 따른다. 저장소의 기존 `AGENTS.md`, 보안 규칙, 브랜치 보호, 빌드 규칙이 더 엄격하면 그 규칙이 우선한다.

## 역할과 권한

| 역할 | 프롬프트 | 소스 읽기 | 소스 수정 | 테스트 실행 | Git 쓰기 |
|---|---|---:|---:|---:|---:|
| supervisor | `prompts/supervisor.md` | 요약·diff만 | 금지 | 결과 확인만 | 금지 |
| project-analyzer | `prompts/project-analyzer.md` | 허용 | 금지 | 비파괴 진단만 | 금지 |
| migration-architect | `prompts/migration-architect.md` | 허용 | 금지 | 금지 | 금지 |
| implementer | `prompts/implementer.md` | 허용 | 허용 | 허용 | 기본 금지 |
| migration-reviewer | `prompts/migration-reviewer.md` | 허용 | 금지 | 금지 | 금지 |
| qa-engineer | `prompts/qa-engineer.md` | 허용 | 금지 | 허용 | 금지 |

supervisor만 `ARTIFACT_DIR`의 문서와 진행 상태를 기록할 수 있다. implementer는 승인된 계획에 포함된 프로덕션 코드와 테스트 파일만 수정할 수 있다. 다른 역할은 보고서를 부모 에이전트에 Markdown으로 반환하며 파일을 직접 고치지 않는다.

## 공통 절대 규칙

1. 사용자의 기존 수정과 미추적 파일을 보존한다. 무관한 변경을 되돌리거나 정리하지 않는다.
2. push, PR 생성, merge, release, 배포, 원격 브랜치 수정은 금지한다.
3. 패키지·Xcode·Swift 도구 버전, deployment target을 임의로 올리지 않는다.
4. entitlement, privacy manifest, signing, capability, 데이터 모델·마이그레이션은 승인된 계획과 사용자 승인 없이는 변경하지 않는다.
5. 실제 SDK, 컴파일러, 공식 문서 또는 저장소 코드에서 확인하지 못한 타입·메서드·기기 동작을 만들지 않는다.
6. `UIScreen.main.bounds`, 고정 orientation, 기기 모델명 같은 전역 가정은 창·컨테이너 기반 레이아웃 근거로 교체할 수 있을 때만 교체한다.
7. 넓은 화면이라고 무조건 2열 UI로 바꾸지 않는다. 정보 구조와 사용자 작업 흐름이 2열을 정당화해야 한다.
8. 승인된 `02-migration-plan.md` 밖의 개선은 구현하지 않고 `DEVIATION_REQUEST`로 보고한다.
9. 완료 기준을 만족하는 최소 diff를 만든다. 일반화, 헬퍼, 방어 코드, 리팩터링은 계획에 명시된 경우만 추가한다.
10. 자격증명 입력, 유료 동작, 데이터 삭제·전송, 되돌릴 수 없는 QA는 실행하지 않는다.

## 위험 분류

### SAFE

- 기존 UX를 유지하는 제약 조건·우선순위 수정
- 창 또는 컨테이너 크기에 따른 단순 레이아웃 전환
- 기존 design token을 사용한 간격·최대 너비 보정
- 기존 테스트 범위 안의 회귀 수정
- 접근성 크기와 회전에 대한 잘림·겹침 수정

### RISKY

- `NavigationStack`과 `NavigationSplitView` 사이의 구조 변경
- 멀티윈도우·scene 생성 또는 상태 복원 도입
- 데이터 소유권, 선택 상태, 라우팅, 딥링크 변경
- 공개 API, persistence, analytics, 권한, entitlement 변경
- deployment target 또는 빌드 설정 변경
- 넓은 화면에만 존재하는 새 기능·사용자 흐름

### AMBIGUOUS

- 제품 의도나 디자인 근거가 없는 좌우 패널 분리
- SDK에서 확인되지 않은 Duo·힌지·자세 전용 동작
- 여러 UX가 비슷한 장단점을 갖고 사용자의 제품 결정이 필요한 경우

`SAFE_AUTO`에서는 SAFE만 자동 구현한다. RISKY는 supervisor가 정확한 변경·영향·대안을 보여주고 승인을 기다린다. AMBIGUOUS는 구현하지 않고 추천 목록에 남긴다.

## 역할 간 산출물 계약

모든 보고서에는 `Status`, `Evidence`, `Decision`, `Open questions`가 있어야 한다. 사실과 추정을 분리하고, 가능한 경우 파일 경로와 줄 번호 또는 명령 결과를 근거로 남긴다.

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

`02-migration-plan.md`는 implementer의 유일한 변경 명세다. supervisor는 architect 원문을 바꾸지 않고 문서 맨 위의 `Supervisor approval`만 갱신한다. 구현 전 이 값이 `APPROVED_SAFE` 또는 `APPROVED_BY_USER`여야 한다.

## 워크플로 상태

```text
INIT
→ ANALYZED
→ PLANNED
→ APPROVED_SAFE | WAITING_USER_APPROVAL
→ IMPLEMENTED
→ REVIEW_PASS | REVIEW_RETURN
→ QA_PASS | QA_RETURN | QA_BLOCKED
→ COMPLETE
```

각 전이는 `progress.md`에 시간, 입력 산출물, 결정, 회귀 카운터와 함께 기록한다. 중단 후 재개할 때 supervisor는 `progress.md`를 먼저 읽고 마지막 완료 단계 다음부터 계속한다.

## 회귀 루프

리뷰 또는 QA 실패는 다음의 fingerprint로 기록한다.

```text
<surface>::<observable symptom>::<failing check or scenario>
```

- 새 fingerprint: implementer에 한 번 수정 요청하고 총 복귀 횟수 `+1`.
- 같은 fingerprint가 두 번째 나타남: 추가 패치를 중단한다.
- fingerprint와 무관하게 구현 복귀가 누적 3회: 추가 패치를 중단한다.
- 중단 시 새 project-analyzer 에이전트를 이전 가설 없이 호출한다. 제공 입력은 원래 분석, 승인 계획, 현재 diff, 실패 증거뿐이다.
- 새 분석 결과를 `regression-NN.md`에 저장하고 migration-architect가 계획을 수정한다.
- 수정 계획에 RISKY 또는 AMBIGUOUS가 있거나 원래 완료 기준이 바뀌면 사용자 결정을 기다린다.

## 역할 호출

이 워크플로는 역할 분리를 위해 별도 에이전트 컨텍스트를 순차적으로 사용한다. supervisor는 각 역할을 호출할 때 해당 프롬프트 경로, 정확한 입력 파일, 작업 범위, 금지 사항, 반환 형식을 메시지에 적는다. 병렬 실행은 읽기 전용 조사 범위가 서로 독립적일 때만 허용하며, 같은 파일을 수정하는 implementer를 둘 이상 동시에 실행하지 않는다.

에이전트 협업 기능이 없으면 역할을 한 세션에서 흉내 내지 말고, 사용자가 별도 세션으로 각 프롬프트를 실행할 수 있도록 현재 산출물과 다음 실행 문구를 제공하고 멈춘다.
