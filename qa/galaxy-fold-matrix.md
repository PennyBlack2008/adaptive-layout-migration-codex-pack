# Galaxy Fold QA Matrix

React Native Android foldable profile의 QA 시나리오다. 승인 계획과 실제 변경에 관련된 항목만 실행한다. 프로젝트의 기존 breakpoint가 있으면 해당 값을 사용하고 경계 전후를 검사한다.

## 환경 기록

각 실행에 다음을 기록한다.

- device/emulator 모델
- Android API와 target SDK
- app build variant
- React Native architecture와 Expo/bare workflow
- 실제 window width/height
- fold posture source와 지원 여부
- Samsung 설정 중 결과에 영향을 주는 항목

## 필수: 기본 창 크기

| ID | Window | 절차 | 기대 결과 |
|---|---|---|---|
| GF-W01 | compact `<600` | cover 또는 좁은 emulator/split window에서 핵심 흐름 실행 | single-pane navigation, 잘림·가로 스크롤 없음 |
| GF-W02 | medium `600..<840` | 펼친 세로 또는 동등 window에서 실행 | 계획된 medium 표현, compact 기능 유지 |
| GF-W03 | expanded `>=840` | 펼친 가로 또는 동등 window에서 실행 | 계획된 multi-pane/max-width, 과도한 stretch 없음 |
| GF-W04 | breakpoint 전후 | 각 breakpoint보다 약간 작은/큰 폭으로 resize | flicker·loop 없이 한 번 전환, state 유지 |
| GF-W05 | compact height | 넓지만 낮은 landscape/pop-up window | 높이 부족 시 2열·컨트롤 배치가 사용 가능 |

## 필수: runtime continuity

| ID | 상태 | 절차 | 기대 결과 |
|---|---|---|---|
| GF-C01 | navigation | 상세 화면에서 fold/unfold 또는 resize | 같은 콘텐츠·경로 유지 |
| GF-C02 | selection | 목록 항목 선택 후 size class 전환 | 선택 유지, compact back behavior 정상 |
| GF-C03 | form | 입력 중 fold/unfold | 입력값·focus 정책이 계획과 일치 |
| GF-C04 | scroll | 긴 목록 중간에서 전환 | 허용 오차 안에서 위치·항목 context 유지 |
| GF-C05 | modal | modal/sheet가 열린 상태에서 전환 | crash·고아 overlay 없음 |
| GF-C06 | keyboard | keyboard가 열린 상태에서 전환 | 입력 영역 가시성과 inset 정상 |

## 필수: multi-window와 재개

| ID | 모드 | 절차 | 기대 결과 |
|---|---|---|---|
| GF-M01 | split-screen | divider로 window를 연속 resize | 실시간 재배치, stale width 없음 |
| GF-M02 | pop-up/freeform | 작은 window로 변경 후 확대 | compact fallback과 복귀 정상 |
| GF-M03 | background/resume | 전환 중 background 후 resume | state와 dimension 최신화 |
| GF-M04 | activity recreation | QA 환경에서 안전하게 가능한 경우 recreation | navigation·입력 상태가 계획대로 복원 |
| GF-M05 | DeX | 프로젝트 범위에 포함될 때만 resizable window 실행 | mouse/keyboard와 resize에서 핵심 흐름 정상 |

## 조건부: posture와 hinge

전용 API가 구현·승인된 경우만 실행한다. JS window width만 있는 구현에서는 이 섹션을 `NOT_APPLICABLE`로 둔다.

| ID | Posture | 절차 | 기대 결과 |
|---|---|---|---|
| GF-P01 | FLAT | 펼친 상태 진입 | 일반 adaptive layout, 잘못된 hinge gap 없음 |
| GF-P02 | HALF_OPENED tabletop | horizontal fold로 전환 | 계획된 상·하 배치와 controls 접근성 |
| GF-P03 | HALF_OPENED book | vertical fold로 전환 | 계획된 좌·우 배치와 navigation 연속성 |
| GF-P04 | occluding hinge | hinge가 콘텐츠를 가리는 환경 | 주요 콘텐츠·터치 타깃이 hinge와 겹치지 않음 |
| GF-P05 | posture events | 여러 번 FLAT/HALF_OPENED 전환 | 중복 listener·render loop·stale event 없음 |

## 접근성과 시각 회귀

| ID | 환경 | 기대 결과 |
|---|---|---|
| GF-A01 | 시스템 글꼴 기본/큰 크기 | text clipping과 주요 action 손실 없음 |
| GF-A02 | TalkBack 영향 범위 | pane 전환 후 focus 순서와 label 유지 |
| GF-A03 | RTL 영향 범위 | pane·navigation 방향과 content order 정상 |
| GF-A04 | light/dark | 새 surface·divider·scrim 대비 정상 |
| GF-A05 | safe area/system bars | edge-to-edge/inset에서 주요 UI 겹침 없음 |

## 성능·안정성

| ID | 검사 | 기대 결과 |
|---|---|---|
| GF-S01 | 연속 resize | JS error, ANR, render loop 없음 |
| GF-S02 | fold/unfold 반복 | crash, memory/listener 증가 징후 없음 |
| GF-S03 | list-heavy screen | column/pane 전환 후 duplicate key·item loss 없음 |
| GF-S04 | native posture module | subscription 해제 후 event 없음, resume 후 재구독 정상 |

## 환경 우선순위

1. 기존 CI/unit/component/E2E
2. Android Studio resizable 또는 foldable emulator
3. 실제 Galaxy Z Fold
4. 실기기가 없으면 Samsung Remote Test Lab

emulator 통과를 Samsung 실기기 통과로 표기하지 않는다. Remote Test Lab이나 실기기에서 실행하지 못한 Samsung 고유 app continuity와 posture는 `BLOCKED` 또는 `NOT_RUN`으로 남긴다.
