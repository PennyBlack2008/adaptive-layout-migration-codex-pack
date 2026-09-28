# Adaptive Layout Migration Team for Codex

Codex가 모바일 프로젝트를 조사하고, 폴더블·대화면·동적 창 크기 대응을 계획하고, 최소 범위로 구현한 뒤 계획 정합성 리뷰와 QA까지 수행하도록 만드는 역할 분리 프롬프트 팩입니다.

지원 프로필:

- SwiftUI/UIKit 기반 iOS adaptive layout 및 근거 기반 Duo 대응
- React Native 기반 Android adaptive layout 및 Galaxy Fold 대응

프롬프트는 기기 이름만으로 화면 크기나 기능을 가정하지 않습니다. 현재 창 크기와 설치된 SDK를 먼저 확인하며, 검증되지 않은 전용 API는 구현하지 않습니다.

## 구조

```text
adaptive-layout-migration-codex-pack/
├── AGENTS.md
├── prompts/
│   ├── supervisor.md
│   ├── project-analyzer.md
│   ├── migration-architect.md
│   ├── implementer.md
│   ├── migration-reviewer.md
│   └── qa-engineer.md
├── profiles/
│   ├── ios-duo.md
│   └── react-native-android-foldable.md
└── qa/
    └── galaxy-fold-matrix.md
```

## 설치

저장소 내용을 대상 앱의 루트에 복사합니다. 대상 저장소에 `AGENTS.md`가 이미 있으면 덮어쓰지 말고 이 팩의 라우팅과 워크플로 규칙을 병합하십시오. 기존 저장소의 더 엄격한 안전·빌드·배포 규칙이 우선합니다.

## 실행

### 자동 프로필 선택

```text
prompts/supervisor.md를 읽고 adaptive layout migration workflow를 SAFE_AUTO 모드로 실행해줘.
프로젝트 스택에 맞는 profile 하나만 선택하고, 현재 사용자 변경은 보존해.
push·PR·패키지 업그레이드는 하지 마.
```

### React Native + Galaxy Fold

```text
prompts/supervisor.md와 profiles/react-native-android-foldable.md를 사용해
이 React Native 앱을 Galaxy Fold와 동적 Android 창 크기에 대응시켜줘.
SAFE_AUTO 범위는 끝까지 진행하고 RISKY 변경만 승인 요청해.
```

### iOS adaptive layout

```text
prompts/supervisor.md와 profiles/ios-duo.md를 사용해
이 iOS 앱의 compact/regular layout을 점검하고 안전한 호환성 수정을 적용해줘.
확인되지 않은 기기 전용 API는 사용하지 마.
```

## 동작 방식

```text
프로필 선택 → 읽기 전용 분석 → 최소 계획 → 위험 게이트
→ 구현 → 계획 정합성·감산 리뷰 → QA → 회귀 시 새 컨텍스트 재조사
```

- supervisor는 코드 대신 역할과 승인 상태를 관리합니다.
- implementer만 승인 계획 안에서 코드를 수정합니다.
- reviewer는 계획 밖 변경을 품질과 무관하게 이탈로 판정합니다.
- QA는 실제 실행한 범위와 실기기 미검증 범위를 분리합니다.
- 같은 실패가 반복되면 임시 패치를 중단하고 새 분석가가 원인부터 다시 조사합니다.

## 산출물

```text
.codex/adaptive-layout-migration/
├── progress.md
├── 01-project-analysis.md
├── 02-migration-plan.md
├── 03-implementation-report.md
├── 04-review-report.md
├── 05-qa-report.md
└── regression-01.md
```

## 설계 기준

- 역할 분리, 고정 입출력, 금지 행동
- 승인된 계획에 대한 구현 정합성
- 완료 기준을 만족하는 최소 diff
- 삭제 가능한 코드도 찾는 감산 리뷰
- SDK·공식 문서·컴파일러 근거 우선
- 물리 기기명이 아닌 현재 앱 창과 컨테이너 기준

## 공식 참고 자료

- [React Native Dimensions](https://reactnative.dev/docs/dimensions)
- [React Native Native Modules](https://reactnative.dev/docs/turbo-native-modules-introduction)
- [Android window size classes](https://developer.android.com/develop/adaptive-apps/guides/use-window-size-classes)
- [Android FoldingFeature](https://developer.android.com/reference/androidx/window/layout/FoldingFeature)
- [Samsung App Continuity](https://developer.samsung.com/galaxy-z/app-continuity.html)
