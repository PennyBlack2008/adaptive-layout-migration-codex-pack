# Profile: React Native Android Foldable and Galaxy Fold

## 적용 조건

- React Native 앱의 Android target
- Galaxy Z Fold 계열, Android foldable, large screen, multi-window 대응
- fold/unfold 중 app continuity 또는 posture-aware UX 요구

물리 기기 모델이 아니라 현재 앱 window를 기본 입력으로 사용한다. Galaxy Fold 전용 동작은 Samsung·Android 공식 근거와 현재 dependency에서 확인될 때만 구현한다.

QA 단계에서는 `qa/galaxy-fold-matrix.md`도 읽는다.

## Project facts

- React Native와 React 버전
- Expo managed, prebuild, bare 중 하나
- New Architecture와 Hermes 활성화 여부
- package manager와 lockfile
- React Navigation 또는 다른 navigation 구조
- styling/layout 체계와 design token
- Android `minSdk`, `compileSdk`, `targetSdk`, Gradle·AGP·Kotlin 버전
- `MainActivity`/`MainApplication` 언어와 lifecycle 처리
- `AndroidManifest.xml`의 orientation, `resizeableActivity`, `configChanges`
- 기존 Jetpack WindowManager 또는 foldable 관련 dependency
- 기존 Native Module/TurboModule과 native event 패턴
- state persistence, form/navigation/scroll restoration
- unit, component, E2E, screenshot test와 emulator/device 명령

Expo managed 앱에서 native 변경이 필요하면 prebuild/config plugin/dev client 영향까지 분석하며 자동 전환하지 않는다.

## 공식 동작 기준

1. React 컴포넌트의 창 크기 반응은 `useWindowDimensions()`를 우선한다.
2. module scope나 영구 cache의 `Dimensions.get()` 값은 회전·fold·window resize 후 오래될 수 있다.
3. breakpoint는 물리 화면이 아니라 현재 앱 window의 density-independent width/height에 적용한다.
4. 기본 폭 구간은 기존 design system이 없을 때만 후보로 사용한다.
   - compact: `< 600`
   - medium: `600..<840`
   - expanded: `>= 840`
5. height도 확인한다. medium width라도 compact height이면 2열이 부적절할 수 있다.
6. hinge·fold state·occlusion은 width로 추정하지 않는다. Jetpack WindowManager `FoldingFeature` 또는 이미 검증된 동등 API가 필요하다.

## Risk scan

- module scope 또는 `StyleSheet.create`에서 캐시된 `Dimensions.get('window'|'screen')`
- `screen` 크기를 app window처럼 사용
- `DeviceInfo`나 Galaxy 모델명으로 layout 분기
- orientation만으로 row/column 또는 pane 결정
- 고정 width/height, absolute position, magic aspect ratio
- cover/unfolded 전환 후 state·selection·scroll·form 유실
- breakpoint 변경 때 navigation tree가 불필요하게 remount
- split-screen/pop-up/DeX resize에 반응하지 않음
- expanded window에서 콘텐츠가 무제한 늘어남
- keyboard, safe area/insets, status/navigation bar 겹침
- FlatList/SectionList의 column 전환과 key 안정성
- native fold event listener 등록·해제 또는 lifecycle 누수
- manifest의 resizability/orientation/configChanges와 RN template 불일치
- Android 16+ large-screen 동작과 target SDK 영향

검색 결과를 버그로 확정하기 전에 실제 render path와 lifecycle을 읽는다.

## 구현 층위

### A. Responsive compatibility — 기본 SAFE

- 컴포넌트 안에서 `useWindowDimensions()`로 현재 window 변화 구독
- flex, percentage, min/max width로 reflow
- 기존 navigation과 상태를 유지하는 compact/medium/expanded 표현 전환
- 콘텐츠 최대 너비와 spacing 조정
- cover screen, split-screen 같은 좁은 window fallback 보존

새 공용 `useWindowSizeClass` helper는 여러 화면이 같은 규칙을 실제로 공유하고 승인 계획에 있을 때만 만든다.

### B. Adaptive multi-pane UX — 기본 RISKY

- 목록/상세, 편집/미리보기 2열
- breakpoint에 따른 navigation tree·selection 변경
- pane 간 state ownership, deep link, back behavior 변경
- expanded 전용 기능 또는 정보 밀도 변경

compact와 expanded 사이 전환에서 사용자 context가 유지되는 완료 기준이 필요하다.

### C. Posture and hinge — RISKY

- Jetpack WindowManager `FoldingFeature` 사용
- Kotlin Native Module/TurboModule로 bounds, orientation, occlusion, state 전달
- tabletop/book posture 전용 배치
- native dependency와 Gradle·Codegen 변경

먼저 현재 dependency가 필요한 정보를 이미 제공하는지 확인한다. 새 dependency나 native module은 사용자 승인 없이 추가하지 않는다. New Architecture 앱은 현재 RN 버전에 맞는 typed spec·Codegen 패턴을 사용하고, legacy 앱을 이 작업 때문에 자동 마이그레이션하지 않는다.

native event를 쓰면 시작·중지 lifecycle, 초기 snapshot, event ordering, listener cleanup을 계획과 리뷰에 포함한다. JS width만으로 hinge 위치나 `HALF_OPENED` 상태를 추정하지 않는다.

### D. Manifest and continuity — 조건부 RISKY

- `resizeableActivity`, orientation, aspect ratio, `configChanges`
- activity recreation 또는 configuration update 중 state 보존
- target SDK에 따른 large-screen behavior

현재 RN template과 앱의 설정을 먼저 비교한다. Samsung 문서의 snippet을 무조건 복사하지 않는다. manifest 변경은 플랫폼 동작을 바꾸므로 별도 승인 단계로 둔다.

## 추가 금지

- package·RN·Expo·AGP·Kotlin 버전 임의 업그레이드
- Expo workflow 자동 전환
- width로 fold posture 추정
- Galaxy 모델명 allowlist
- JS와 native 양쪽에 서로 다른 breakpoint source 생성
- expanded UI 때문에 compact navigation 삭제
- 실기기 없이 Samsung 전용 동작 통과 주장

## 역할별 추가 출력

### Analyzer

- `React Native architecture: New | Legacy | Mixed | Unknown`
- `Workflow: Expo managed | Expo prebuild | Bare`
- `Fold posture source: Existing verified API | Requires native module | Not requested`
- `Manifest continuity status`

### Architect

- 각 단계에 `JS-only | Android native | Manifest/build` 경계
- dependency·native 변경을 독립 RISKY 단계로 분리
- compact fallback과 fold/unfold state continuity 완료 기준

### Reviewer

- stale dimension cache
- navigation remount와 state loss
- native listener lifecycle
- JS/native breakpoint 불일치
- 승인 없는 package·manifest 변경

### QA

- `qa/galaxy-fold-matrix.md`의 해당 시나리오 ID 사용
- Android Foldable Emulator와 실제 Galaxy/Remote Test Lab 결과 분리
- general adaptive, fold transition, posture-aware 결과를 별도 상태로 기록

## 공식 참고 자료

- React Native Dimensions: https://reactnative.dev/docs/dimensions
- React Native height and width: https://reactnative.dev/docs/height-and-width
- React Native Turbo Native Modules: https://reactnative.dev/docs/turbo-native-modules-introduction
- Android window size classes: https://developer.android.com/develop/adaptive-apps/guides/use-window-size-classes
- Android FoldingFeature: https://developer.android.com/reference/androidx/window/layout/FoldingFeature
- Android orientation/aspect/resizability: https://developer.android.com/develop/adaptive-apps/guides/app-orientation-aspect-ratio-resizability
- Samsung App Continuity: https://developer.samsung.com/galaxy-z/app-continuity.html
- Samsung foldable codelab and Remote Test Lab path: https://developer.samsung.com/codelab/galaxy-z/app-continuity.html
