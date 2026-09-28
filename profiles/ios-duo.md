# Profile: iOS Adaptive Layout and Evidence-Gated Duo

## 적용 조건

- SwiftUI, UIKit 또는 혼합 iOS 앱
- compact/regular size class, 회전, iPad·멀티윈도우, 넓은 화면 대응
- 사용자가 “iPhone Duo” 또는 유사한 듀얼 패널 기기를 요구함

`Duo`는 요구사항 라벨로 취급한다. 설치된 Xcode SDK나 제공된 공식 문서에서 전용 API·기기 동작이 확인되지 않으면 일반 iOS adaptive layout만 구현한다.

## Project facts

- SwiftUI/UIKit/mixed와 deployment target
- app, extension, widget, test target
- `WindowGroup`, scene, multi-window, state restoration
- `NavigationStack`, `NavigationSplitView`, `UISplitViewController`, custom navigation
- supported orientation과 iPad/Mac Catalyst 설정
- design system, layout helper, snapshot/UI test
- 실행 가능한 최소 build/test 명령

## Risk scan

- `UIScreen.main.bounds` 또는 screen-global geometry
- `UIDevice.current.orientation` 중심 layout
- 고정 width/height, magic breakpoint, 절대 위치
- safe area 무시와 overlay·sheet·keyboard 충돌
- compact/regular size class 고정 가정
- resize·trait 변경 후 갱신되지 않는 값
- 목록/상세, 편집/미리보기처럼 2열에 자연스러운 구조
- custom navigation과 selection state 결합
- Dynamic Type, RTL, VoiceOver에서 잘림
- 카메라·지도·미디어의 aspect ratio·sensor orientation 의존

문자열 존재만으로 결함을 확정하지 않는다. 고정 크기는 아이콘·터치 타깃처럼 의도된 경우 `NOT_A_PROBLEM`일 수 있다.

## Platform evidence

- 전용 Duo API가 로컬 SDK에 있는지 symbol과 SDK 버전으로 확인한다.
- 없으면 `NO_VERIFIED_DUO_SPECIFIC_API`를 기록한다.
- 공식 문서가 제공된 경우 정확한 링크·제목과 SDK 일치 여부를 기록한다.
- 근거가 없으면 window/container geometry, size class, safe area, 기존 scene API만 허용한다.

## 구현 층위

### A. Compatibility — 기본 SAFE

- container-relative layout
- compact/regular 전환에서 잘림·겹침 수정
- 기존 navigation과 상태를 유지하는 spacing·max-width 수정
- 회전·resize·Dynamic Type 대응

### B. Adaptive UX — 기본 RISKY

- `NavigationStack`과 `NavigationSplitView` 구조 변경
- 목록/상세 또는 편집/미리보기 2열
- selection, deep link, compact fallback 재설계

### C. Device-specific enhancement — 검증 전 AMBIGUOUS

- 힌지·자세·물리 패널 전용 UI
- 전용 scene나 display 동작
- 존재가 확인되지 않은 API

## 추가 금지

- 기기명 문자열로 layout 분기
- 앱 전체에서 `UIScreen.main`을 새 breakpoint source로 사용
- 승인 없는 scene, entitlement, deployment target 변경
- wide layout 때문에 compact navigation 제거

## QA matrix

- compact portrait/landscape
- regular portrait/landscape
- split view 또는 resizable window가 지원될 때 크기 전환
- navigation selection과 deep link
- sheet, popover, keyboard, safe area
- Dynamic Type 기본/큰 크기, RTL, VoiceOver 영향 범위
- 실제 Duo 환경이 없으면 일반 adaptive 결과와 Duo 실기기 미검증을 분리

## 역할별 추가 출력

- analyzer: `Duo-specific API status`
- architect: `General adaptive fallback`
- reviewer: compact fallback과 scene/navigation 정합성
- QA: `Verified on actual Duo hardware/simulator: Yes | No`
