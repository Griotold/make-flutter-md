# make-flutter-md Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a public GitHub conventions repo that Claude Code auto-fetches when bootstrapping new Flutter projects.

**Architecture:** Single `README.md` at root (bootstrap procedure + CLAUDE.md template inline + learnings index) + `learnings/` subfolder with 6 topic docs. Public GitHub repo. New projects pull via WebFetch on raw URLs. Global `~/.claude/CLAUDE.md` carries a one-line trigger.

**Tech Stack:** Markdown only. GitHub (public repo). No build, no test framework, no CI.

**Spec reference:** `docs/superpowers/specs/2026-05-11-make-flutter-md-design.md`

---

## File Structure

Files this plan creates (relative to repo root `/Users/hemille/Desktop/side/make-flutter-md/`):

| Path | Responsibility |
|---|---|
| `README.md` | Entry point. Human intro + Claude bootstrap procedure (Steps 1~6) + inline CLAUDE.md template + learnings index + maintenance rules. |
| `learnings/ios-build.md` | iOS 26+ release-only policy, exact install command, Wi-Fi deploy, CI no-codesign. |
| `learnings/integration-test.md` | Why `flutter test integration_test` fails on iOS 26 + `flutter drive` workaround + `test_driver/integration_test.dart` content. |
| `learnings/hive-ce-patterns.md` | Why `hive_ce`, pubspec deps, `@GenerateAdapters`, boxName convention, migrations. |
| `learnings/firebase-setup.md` | flutterfire CLI flow, anonymous→Google upgrade, firestore.rules patterns, Cloud Functions onCall. |
| `learnings/monetization.md` | AdMob test/prod IDs, Korean IDFA (ATT), `in_app_purchase`, hard-rule patterns, Restore Purchase. |
| `learnings/onboarding-tutorial.md` | Card carousel UX (3~4 PageView cards), image-mandatory video-optional policy, Nano Banana → image-to-video pipeline, Claude-proposes-user-approves motion decision, build-time generation, file naming convention. |

All files use Korean prose (user's working language) consistent with the spec and existing CLAUDE.md files.

**Source material for learning content** — read the user's existing Flutter project CLAUDE.md files to extract verified patterns (do not invent):
- `/Users/hemille/Desktop/side/bongjoha/CLAUDE.md` — iOS only, integration_test workaround, main branch policy
- `/Users/hemille/Desktop/side/flutter-2048/CLAUDE.md` — Riverpod, hive_ce, Firebase, ads/IAP, hard-rule examples
- `/Users/hemille/Desktop/side/brick-boom/CLAUDE.md` — docs/superpowers structure, commit policy
- `/Users/hemille/Desktop/side/cal-tracker-flutter/CLAUDE.md` — Firebase Auth anonymous→Google, Cloud Functions onCall
- `/Users/hemille/.claude/CLAUDE.md` — global iOS release-only command, hive_ce mandate

---

### Task 1: Create `learnings/ios-build.md`

**Files:**
- Create: `learnings/ios-build.md`

**Required content (must include verbatim):**
- Heading: `# iOS Build`
- Section "왜 release-only인가": iOS 26+ 보안 정책상 Flutter debug 모드가 실기기/시뮬레이터에서 동작 안 함 (글로벌 `~/.claude/CLAUDE.md`에 명시).
- Section "실기기 설치 명령" with this exact command block:
  ```bash
  flutter build ios --release && flutter install --release --device-timeout 15
  ```
- Section "왜 `flutter run --release`는 쓰면 안 되는가": 터미널이 대기 상태로 멈춤 (Claude Code 워크플로우에서 다음 명령 진행 불가).
- Section "Wi-Fi 무선 빌드": 같은 Wi-Fi + 아이폰 잠금 해제 상태면 USB 없이 배포 가능. 무선 기기 안 잡히면 `--device-timeout` 값 늘릴 것.
- Section "CI / 빌드 검증": `flutter build ios --release --no-codesign` 패턴.

**Length target:** 1~2KB.

**Source:** `/Users/hemille/.claude/CLAUDE.md` (Flutter / iOS 섹션) + `/Users/hemille/Desktop/side/bongjoha/CLAUDE.md` Section 1.

- [ ] **Step 1:** Read `/Users/hemille/.claude/CLAUDE.md` "Flutter / iOS" 섹션과 `/Users/hemille/Desktop/side/bongjoha/CLAUDE.md` Section 1 to extract verified facts.

- [ ] **Step 2:** Write `learnings/ios-build.md` with the 5 required sections above. Pure 사실 + 정확한 명령 위주. 다른 설명 추가하지 말 것.

- [ ] **Step 3:** Verify file size with `wc -c learnings/ios-build.md`. Expected: 800~2200 bytes.

- [ ] **Step 4:** Verify required headings present:
  ```bash
  grep -E '^#{1,2} ' learnings/ios-build.md
  ```
  Expected: `# iOS Build` and 5 H2 sections.

- [ ] **Step 5:** Commit
  ```bash
  git add . && git commit -m "learnings: iOS build 패턴 문서화"
  ```

---

### Task 2: Create `learnings/integration-test.md`

**Files:**
- Create: `learnings/integration-test.md`

**Required content:**
- Heading: `# Integration Test (iOS 26 우회)`
- Section "문제": `flutter test integration_test/...` 는 debug 모드를 강제하는데 iOS 26 실기기/시뮬레이터에서 debug 모드 동작 안 함.
- Section "권장 경로 — flutter drive (release)":
  - 명령 블록:
    ```bash
    flutter drive --driver=test_driver/integration_test.dart --target=integration_test/<file>.dart --release
    ```
  - **러너 파일 인라인** (정확한 내용 — 다음 4줄):
    ```dart
    import 'package:integration_test/integration_test_driver.dart';

    Future<void> main() => integrationDriver();
    ```
  - 위치: `test_driver/integration_test.dart` (없으면 첫 통합 테스트 작성 시 추가).
- Section "대안 경로 — iOS 16~25 시뮬레이터":
  ```bash
  flutter test integration_test/<file>.dart -d <sim_id>
  ```
  - 단점: 해당 버전 시뮬레이터를 맥에 미리 설치해 두어야 함.
- Section "CI에서 분리": `flutter test` (단위)와 `flutter drive` (통합)을 별도 단계로 실행. 한 명령으로 묶지 않음.

**Length target:** 1~2KB.

**Source:** `/Users/hemille/Desktop/side/bongjoha/CLAUDE.md` Section 1.1.

- [ ] **Step 1:** Read `/Users/hemille/Desktop/side/bongjoha/CLAUDE.md` Section 1.1.

- [ ] **Step 2:** Write `learnings/integration-test.md`. 러너 파일 내용은 위 블록 4줄 그대로 포함.

- [ ] **Step 3:** Verify with:
  ```bash
  wc -c learnings/integration-test.md
  grep 'integrationDriver' learnings/integration-test.md
  grep '\-\-driver=test_driver/integration_test.dart' learnings/integration-test.md
  ```
  Expected: 800~2200 bytes, 2 grep matches (driver mention + integrationDriver function).

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "learnings: integration_test iOS 26 우회 패턴"
  ```

---

### Task 3: Create `learnings/hive-ce-patterns.md`

**Files:**
- Create: `learnings/hive-ce-patterns.md`

**Required content:**
- Heading: `# hive_ce 패턴`
- Section "왜 hive_ce (원본 hive 금지)": 원본 hive/hive_flutter 2년+ 업데이트 끊김. `hive_ce` + `hive_ce_flutter`가 공식 계승 포크. 추가 기능: IsolatedHive, WASM 지원, `@GenerateAdapters` 자동 어댑터 생성, DevTools Inspector. **원본 hive는 제안조차 금지.**
- Section "pubspec 추가":
  ```yaml
  dependencies:
    hive_ce: ^<latest>
    hive_ce_flutter: ^<latest>

  dev_dependencies:
    hive_ce_generator: ^<latest>
    build_runner: ^<latest>
  ```
  (실제 버전은 사용 시점에 `flutter pub add hive_ce hive_ce_flutter` 로 최신 채움)
- Section "어댑터 생성 패턴":
  ```dart
  // lib/data/hive_registrar.g.dart 자동 생성 대상 모음
  @GenerateAdapters([
    AdapterSpec<Score>(),
    AdapterSpec<UserSettings>(),
  ])
  void _generated() {}
  ```
  실행: `dart run build_runner build --delete-conflicting-outputs`
- Section "boxName 컨벤션": 단수 + 소문자, 예: `'scores'`, `'settings'`, `'app_settings'`. 복수형 사용도 OK지만 한 프로젝트 내에서는 일관되게.
- Section "초기화 순서":
  ```dart
  await Hive.initFlutter();
  Hive.registerAdapters(); // hive_ce_generator 자동 생성
  final settings = await Hive.openBox<UserSettings>('settings');
  ```
- Section "스키마 변경 시": typeId는 절대 재사용 금지. 필드 추가는 OK (null/default 처리). 필드 삭제는 deprecated로 두고 새 필드 추가하는 게 안전.
- Section "IsolatedHive 언제 쓸지": 백그라운드 isolate에서 hive 접근 필요할 때만. 일반 UI 스레드 작업엔 불필요.

**Length target:** 2~3KB.

**Source:** `/Users/hemille/.claude/CLAUDE.md` (Flutter 패키지 선택 섹션) + `/Users/hemille/Desktop/side/flutter-2048/CLAUDE.md`.

- [ ] **Step 1:** Read the source files above.

- [ ] **Step 2:** Write `learnings/hive-ce-patterns.md` with all 7 sections.

- [ ] **Step 3:** Verify:
  ```bash
  wc -c learnings/hive-ce-patterns.md
  grep -c '^## ' learnings/hive-ce-patterns.md
  grep 'GenerateAdapters' learnings/hive-ce-patterns.md
  ```
  Expected: 1500~3500 bytes, ≥6 H2 sections, GenerateAdapters mention.

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "learnings: hive_ce 패턴 (어댑터, boxName, 마이그레이션)"
  ```

---

### Task 4: Create `learnings/firebase-setup.md`

**Files:**
- Create: `learnings/firebase-setup.md`

**Required content:**
- Heading: `# Firebase 셋업`
- Section "셋업 순서":
  ```bash
  # 1. CLI 로그인
  firebase login

  # 2. flutterfire CLI로 프로젝트 연결
  dart pub global activate flutterfire_cli
  flutterfire configure

  # 3. 필요한 패키지 추가
  flutter pub add firebase_core firebase_auth cloud_firestore
  # 필요 시 추가
  flutter pub add firebase_storage cloud_functions firebase_remote_config
  flutter pub add firebase_messaging firebase_analytics firebase_crashlytics
  ```
- Section "main.dart 초기화":
  ```dart
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  ```
- Section "Anonymous Auth → Google upgrade 패턴": 앱 시작 시 anonymous 로그인 자동 → 사용자가 원할 때만 Google credential과 `linkWithCredential` 호출 → uid 유지된 채로 계정 업그레이드. 데이터 마이그레이션 불필요.
- Section "firestore.rules 안전 기본값":
  ```js
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {
      // 기본: 전부 거부
      match /{document=**} {
        allow read, write: if false;
      }

      // 사용자 본인 소유 데이터
      match /users/{uid}/{document=**} {
        allow read, write: if request.auth != null && request.auth.uid == uid;
      }

      // 공용 리더보드 (읽기만)
      match /leaderboards/{doc} {
        allow read: if true;
        allow write: if false; // Cloud Function에서만 쓰기
      }
    }
  }
  ```
- Section "Cloud Functions onCall 패턴":
  ```ts
  export const myFn = onCall({region: 'asia-northeast3'}, async (request) => {
    // request.auth.uid 자동 검증 (auth 없으면 unauthenticated 에러)
    const uid = request.auth?.uid;
    if (!uid) throw new HttpsError('unauthenticated', '');
    // 본 로직
  });
  ```
- Section "firestore.indexes.json": 복합 쿼리 추가할 때마다 인덱스 명시. `firebase deploy --only firestore:indexes`로 배포. 인덱스 누락 시 Firestore 에러 메시지에 콘솔 링크 나옴.
- Section "주요 패키지 조합 (자주 같이 쓰임)": `firebase_core` (필수), `firebase_auth` + `cloud_firestore` (대부분 앱), `firebase_storage` (이미지/파일), `cloud_functions` (서버 로직), `firebase_remote_config` (피처 토글), `firebase_messaging` (푸시), `firebase_analytics` + `firebase_crashlytics` (관측).

**Length target:** 2~3KB.

**Source:** `/Users/hemille/Desktop/side/flutter-2048/CLAUDE.md` + `/Users/hemille/Desktop/side/cal-tracker-flutter/CLAUDE.md` (특히 Cloud Functions 패턴).

- [ ] **Step 1:** Read both source files.

- [ ] **Step 2:** Write `learnings/firebase-setup.md` with all 7 sections.

- [ ] **Step 3:** Verify:
  ```bash
  wc -c learnings/firebase-setup.md
  grep 'flutterfire configure' learnings/firebase-setup.md
  grep 'request.auth.uid' learnings/firebase-setup.md
  ```
  Expected: 2000~3500 bytes, both grep matches.

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "learnings: Firebase 셋업 + Auth/Firestore/Functions 패턴"
  ```

---

### Task 5: Create `learnings/monetization.md`

**Files:**
- Create: `learnings/monetization.md`

**Required content:**
- Heading: `# 수익화 (광고 + IAP)`
- Section "광고 — Google Mobile Ads (AdMob)":
  - 패키지: `google_mobile_ads` (공식).
  - 테스트 ID vs 프로덕션 ID 분리:
    ```dart
    // dev/test
    const testInterstitial = 'ca-app-pub-3940256099942544/4411468910'; // Google 공식 테스트 ID
    // prod (Remote Config 또는 .env에서)
    final adUnitId = kDebugMode ? testInterstitial : prodId;
    ```
- Section "한국 IDFA (App Tracking Transparency)":
  - iOS 14.5+ 부터 IDFA 접근에 사용자 동의 필요.
  - `app_tracking_transparency` 패키지로 다이얼로그 띄우기.
  - 패턴: 앱 첫 실행 시 *광고 초기화 전에* `AppTrackingTransparency.requestTrackingAuthorization()` 호출.
  - `Info.plist`에 `NSUserTrackingUsageDescription` 한국어 문구 필수.
- Section "IAP — `in_app_purchase` 공식 패키지":
  - 서드파티 IAP 라이브러리 금지 (App Store 정책 준수 위해).
  - 상품 ID는 App Store Connect와 1:1 매핑.
  - 트랜잭션 검증은 서버(Cloud Functions) 권장.
- Section "Restore Purchase 플로우 (App Store 필수)":
  - 설정 메뉴에 "구매 복원" 버튼 필수.
  - `InAppPurchase.instance.restorePurchases()` 호출 → `purchaseStream`에서 복원된 구매 처리.
  - 누락 시 App Store 심사 리젝트.
- Section "자주 만드는 하드 룰 (프로젝트별 CLAUDE.md에 명시)":
  - **"Daily/핵심 플로우엔 전면광고 절대 없음"** — 공유/바이럴 보호.
  - **"진행도 IAP 금지"** — 점수/꽃잎/배지 등 게임 진행도를 돈으로 사지 못함.
  - **"Remove Ads 구매 시 광고가 진짜 사라짐"** — 속임수 금지.
  - 이 룰들은 프로젝트 특성에 맞게 골라서 CLAUDE.md 상단에 박아둘 것.

**Length target:** 2~3KB.

**Source:** `/Users/hemille/Desktop/side/flutter-2048/CLAUDE.md` (특히 "핵심 하드 룰" 섹션).

- [ ] **Step 1:** Read flutter-2048 CLAUDE.md.

- [ ] **Step 2:** Write `learnings/monetization.md` with all 5 sections.

- [ ] **Step 3:** Verify:
  ```bash
  wc -c learnings/monetization.md
  grep 'ca-app-pub-3940256099942544' learnings/monetization.md
  grep 'NSUserTrackingUsageDescription' learnings/monetization.md
  grep 'Remove Ads' learnings/monetization.md
  ```
  Expected: 1800~3500 bytes, all 3 grep matches.

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "learnings: 수익화 (AdMob, IDFA, IAP, Restore, 하드 룰)"
  ```

---

### Task 6: Create `learnings/onboarding-tutorial.md`

**Files:**
- Create: `learnings/onboarding-tutorial.md`

**Required content (this is the longest learning, has many design decisions from §5.6):**

- Heading: `# 인앱 튜토리얼 (표준 기능)`
- Section "왜 이 기능이 모든 신규 앱에 표준인가":
  - 아이디어가 Claude/마켓 기반이라 본인도 사용법을 모를 수 있음.
  - 본인 + 일반 사용자 모두를 위한 표준화된 onboarding.
- Section "UX 패턴 — Sudoku 힌트 스타일":
  - 3~4장 카드, Flutter `PageView`로 좌우 스와이프.
  - 각 카드 = 비주얼(이미지 or 영상) + 텍스트 1~2줄.
  - 페이지 인디케이터(점) 하단.
  - 마지막 카드에 "시작하기" 버튼.
- Section "표시 시점":
  - 첫 실행 시 자동 (조건: `app_settings` 박스의 `tutorial_seen` 플래그 false).
  - 설정 화면 또는 메인 화면 ⓘ 버튼에서 언제든 재생 (재시청 진입점 *필수*, 1~2탭 내).
- Section "카드 비주얼 정책 (이미지 필수, 영상 선택)":
  - 모든 카드 이미지 필수 (Nano Banana 생성).
  - 영상은 모션이 정보일 때만 (제스처/슬라이드/애니메이션 설명). 단순 "여기를 탭" 같은 건 이미지로 충분.
  - 기본은 이미지(default-cheap). 빌드 설정에서 `motion: true`로 영상 명시.
- Section "모션 여부 결정 권한 (Claude 제안 + 사용자 승인)":
  - 빌드 직전, Claude가 카드별로 모션 필요성 판단 → 표로 제안:
    ```
    | # | 텍스트 요약              | 제안   | 이유                          |
    |---|--------------------------|--------|-------------------------------|
    | 1 | "빈 셀을 탭하세요"        | image  | 정적 인디케이터로 충분         |
    | 2 | "숫자를 드래그해 옮기기"  | video  | 드래그 모션이 핵심 정보        |
    ```
  - 사용자가 카드별 OK/변경 → 설정 파일 갱신 → 빌드 실행.
  - 한 카드라도 미승인이면 빌드 거부.
  - 다음 빌드부터 같은 결정 재사용 (재확인 안 함).
- Section "빌드 타임 생성 파이프라인":
  1. `tool/tutorial.yaml` (또는 동급 설정 파일)에 카드별 정의:
     ```yaml
     cards:
       - text_key: tutorial.card1
         image_prompt: "..."
         motion: false
       - text_key: tutorial.card2
         image_prompt: "..."
         motion: true
         motion_prompt: "셀이 하이라이트되며 들어옴"
     ```
  2. 빌드 스크립트 (`dart run tool/build_tutorial.dart`) 실행:
     - 모션 결정 확인 (미승인 있으면 중단)
     - 카드별로 Nano Banana(Gemini 2.5 Pro image generation) API 호출 → 이미지 받음
     - `motion: true` 카드는 추가로 image-to-video API 호출 → 짧은 mp4 받음 (3~5초)
     - 출력 위치:
       - 정적: `assets/tutorial/card_N.png`
       - 모션: `assets/tutorial/card_N.mp4`
  3. `GEMINI_API_KEY`는 `.env`에서 읽음. `.env`는 `.gitignore`에 포함 (절대 커밋 금지).
  4. 런타임 생성 금지 (사용자 대기/네트워크/비용 문제).
- Section "재생 위젯 패턴":
  - Flutter `PageView` 사용.
  - 각 페이지가 파일 확장자 보고 분기:
    - `.png` → `Image.asset(...)`
    - `.mp4` → `video_player` 패키지 (autoplay, looping, 컨트롤 숨김)
  - 보이지 않는 페이지의 영상은 일시정지 (배터리/메모리).
  - `chewie` 불필요 (컨트롤 UI 안 보임).
- Section "다국어":
  - 비주얼(이미지/영상)은 **언어 무관** — 한 번만 생성.
  - 텍스트는 Flutter UI 레이어에서 ARB/intl로 렌더 (`text_key`로 참조).
  - 빌드 비용 절감 효과 큼.
- Section "Hive 플래그":
  - 박스 이름: `app_settings`
  - 키: `tutorial_seen` (bool)
  - 첫 실행 시 false → 튜토리얼 종료 후 true.
  - "다시 보기"는 플래그 안 건드림 (재진입 자유).
- Section "하드 룰":
  - 이미지 필수, 영상은 모션이 정보일 때만.
  - 스킵 가능 (Skip 버튼 항상 노출).
  - "다시 보기" 1~2탭 내 접근.
  - 튜토리얼 자체에 광고 절대 없음.
  - 음성 없음.
  - 일러스트가 실제 앱 UI를 충실히 반영 (가상 다른 앱 화면 금지).
- Section "결정 보류 (v1 빌드 시점에 정함)":
  - image-to-video에 어느 API 정확히 쓸지 (Veo 2, Veo 3, Gemini 2.5 비디오 모달 등 시점별 옵션 변함).
  - v1엔 가장 안정적/비용 합리적인 것 선택. 결정 시 본 learning 갱신.

**Length target:** 3~5KB (가장 긴 learning).

**Source:** Spec §5.6 (가장 풍부한 출처) — 모든 결정 사항이 거기에 있음.

- [ ] **Step 1:** Read spec §5.6 thoroughly (lines 148~191 of design doc).

- [ ] **Step 2:** Write `learnings/onboarding-tutorial.md` with all 12 sections above.

- [ ] **Step 3:** Verify:
  ```bash
  wc -c learnings/onboarding-tutorial.md
  grep -c '^## ' learnings/onboarding-tutorial.md
  grep 'motion: true' learnings/onboarding-tutorial.md
  grep 'tutorial_seen' learnings/onboarding-tutorial.md
  grep 'Nano Banana' learnings/onboarding-tutorial.md
  ```
  Expected: 3000~5500 bytes, ≥10 H2 sections, all grep matches.

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "learnings: 인앱 튜토리얼 표준 (카드 캐러셀 + Nano Banana + 모션 옵트인)"
  ```

---

### Task 7: Create `README.md`

**Files:**
- Create: `README.md`

This is the entry point. Three audiences (human reader on GitHub / Claude bootstrap procedure / Claude reading CLAUDE.md template). Structure must be precise.

**Required structure (top to bottom):**

1. **Header** — 레포 한 줄 설명.
2. **사람용 안내 박스** — "이 레포는 Claude Code 자동 사용용. 직접 읽고 따라할 필요 없음. 새 프로젝트 시작은 `mkdir foo && cd foo && claude` 후 '(앱 이름) 만들자'." 라고 명시.
3. **🤖 Claude Code 부트스트랩 절차** — 명령형 Step 1~6. (스펙 §4.1의 6단계 그대로)
4. **📄 CLAUDE.md 템플릿** — 인라인 코드 펜스. Step 4에서 복사할 내용. Placeholder는 `{{PROJECT_NAME}}`, `{{PACKAGE_NAME}}`, `{{CONCEPT_PLACEHOLDER}}`만.
5. **📚 Learnings 인덱스** — 표 형태 (6개 파일 모두 링크).
6. **🔧 이 레포 유지보수 규칙** — main 직접, force-push 금지, 새 learning 3곳 동시 업데이트 등.

**README.md 전문 (이대로 작성):**

````markdown
# make-flutter-md

hemille가 Flutter로 게임/유틸 앱 만들 때 쓰는 규칙과 학습을 모아둔 레포.
새 프로젝트 시작 시 Claude Code가 이 문서를 WebFetch로 읽고 자동 부트스트랩한다.

> **사람용 안내:** 이 레포는 Claude Code가 자동으로 사용하도록 설계됨.
> 본인이 직접 읽고 따라할 필요는 없지만, 어떤 규칙이 있는지 알고 싶으면 아래 참고.
> 새 프로젝트 시작 방법: `mkdir foo && cd foo && claude` 후 "(앱 이름) 만들자"라고만 하면 됨.

---

## 🤖 Claude Code 부트스트랩 절차

> 이 섹션부터는 Claude Code가 실행하는 절차서다.
> 빈/거의 빈 디렉터리에서 Flutter 앱 신규 생성 요청이 오면 아래 순서대로 따른다.

### Step 1: 사용자에게 앱 이름 확인

- 디렉터리 이름(예: `tetris-clone`)을 그대로 쓸지, 다른 이름이 좋을지 사용자에게 확인.
- Dart 패키지 이름은 snake_case 필수 (예: `tetris_clone`).
- `{{PROJECT_NAME}}`은 사람 친화 이름(디렉터리명 또는 한글 가능), `{{PACKAGE_NAME}}`은 snake_case.

### Step 2: Flutter 프로젝트 생성

```bash
flutter create <package_name> --platforms=ios --org com.hemille
```

- iOS only가 기본. 사용자가 명시적으로 Android 원하면 예외(`--platforms=ios,android`).
- `--org`는 com.hemille 기본 (예외 시 사용자 확인).

### Step 3: 컨벤션 디렉터리 구조 생성

```bash
mkdir -p docs/superpowers/specs docs/superpowers/plans docs/solutions docs/ideas
touch docs/superpowers/specs/.gitkeep \
      docs/superpowers/plans/.gitkeep \
      docs/solutions/.gitkeep \
      docs/ideas/.gitkeep
```

### Step 4: 루트에 CLAUDE.md 작성

아래 "📄 CLAUDE.md 템플릿" 섹션의 내용을 그대로 복사해서 새 프로젝트 루트의 `CLAUDE.md`에 쓴다.
`{{PROJECT_NAME}}`, `{{PACKAGE_NAME}}`, `{{CONCEPT_PLACEHOLDER}}`만 치환.

### Step 5: git 초기화

```bash
git init
git add .
git commit -m "초기 부트스트랩 (make-flutter-md 따름)"
```

> ⚠️ GitHub 리모트 추가는 자동으로 하지 말 것. 사용자에게 확인 후 진행.

### Step 6: brainstorming 진입

부트스트랩 끝나면 사용자에게 "셋업 완료. 이제 어떤 앱인지 같이 brainstorm 시작할게"라고 알리고
`superpowers:brainstorming` skill을 invoke한다.

---

## 📄 CLAUDE.md 템플릿

> Step 4에서 이 코드 블록의 내용을 새 프로젝트의 CLAUDE.md로 복사한다.
> Placeholder 토큰만 치환하고 나머지는 그대로.

```markdown
# {{PROJECT_NAME}}

{{CONCEPT_PLACEHOLDER}}

> 📚 이 프로젝트는 https://github.com/Griotold/make-flutter-md 의 규칙을 따른다.
> 핵심 규칙은 아래 인라인. 깊은 주제는 learnings 표 참조.

## 1. 플랫폼 / 빌드

- iOS only, `--release` 강제 (iOS 26+ 정책)
- 실기기 설치: `flutter build ios --release && flutter install --release --device-timeout 15`
- `--debug` / `flutter run --release` 사용 금지 (터미널 블로킹)

## 2. 기술 스택

- 상태관리: **Riverpod 2.x**
- 로컬 저장: **`hive_ce`** + `hive_ce_flutter` (원본 hive 금지)
- 라우팅: **GoRouter** (Firebase 쓸 때만 도입, 단순 앱은 Navigator)
- 백엔드: 필요 시 **Firebase** (Auth / Firestore / Functions / Remote Config)

## 3. 작업 컨벤션

- 브랜치: **`main` 직접 작업**, feature 브랜치 금지
- 커밋: **`git add .` 전체 스테이징**, 작게 자주
- 작업 문서:
  - Spec: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
  - Plan: `docs/superpowers/plans/YYYY-MM-DD-<topic>-plan.md`
  - Solution: `docs/solutions/`
- 워크플로우: `brainstorming` → `writing-plans` → 실행 → `ce:review`

## 4. 표준 기능 (모든 앱 공통)

- **인앱 튜토리얼** — 3~4장 카드 캐러셀, 각 카드 = 비주얼 + 설명 텍스트
- **이미지 필수, 영상 선택** (모션이 정보일 때만 image-to-video)
- 모션 여부: Claude가 카드별 제안 → 사용자 승인 → 빌드. 미승인 시 빌드 거부
- 비주얼은 빌드 타임 생성 (`assets/tutorial/card_N.{png|mp4}`)
- 텍스트는 Flutter UI 레이어(다국어 분리, 비주얼은 언어 무관)
- 첫 실행 자동 재생 + ⓘ 버튼에서 언제든 재생
- `hive_ce` `app_settings` 박스의 `tutorial_seen` 플래그로 첫 실행 감지
- 자세한 패턴: [onboarding-tutorial.md](https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/onboarding-tutorial.md)

## 5. Learnings 참조 (필요 시 WebFetch)

| 주제 | URL |
|---|---|
| iOS 빌드 | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/ios-build.md |
| integration_test 우회 | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/integration-test.md |
| hive_ce 패턴 | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/hive-ce-patterns.md |
| Firebase 셋업 | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/firebase-setup.md |
| 수익화 (광고/IAP) | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/monetization.md |
| 인앱 튜토리얼 | https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/onboarding-tutorial.md |

## 6. 이 프로젝트 고유

- **컨셉:** {{CONCEPT_PLACEHOLDER}}
- **타겟 플랫폼 예외:** 없음 (iOS only)
- **하드 룰:** (brainstorming 후 채워짐)
- **활성 플랜:** 없음 (brainstorming 후 생성)
```

---

## 📚 Learnings 인덱스

| 파일 | 내용 |
|---|---|
| [ios-build.md](./learnings/ios-build.md) | iOS 26 release-only, 실기기 무선 빌드, `--device-timeout` |
| [integration-test.md](./learnings/integration-test.md) | `flutter test integration_test`가 안 되는 이유 + `flutter drive` 우회 |
| [hive-ce-patterns.md](./learnings/hive-ce-patterns.md) | `@GenerateAdapters`, boxName 컨벤션, 마이그레이션 패턴 |
| [firebase-setup.md](./learnings/firebase-setup.md) | flutterfire 셋업, firestore.rules 패턴, Auth anonymous |
| [monetization.md](./learnings/monetization.md) | AdMob 한국 IDFA, in_app_purchase 패턴, "성역" 룰 |
| [onboarding-tutorial.md](./learnings/onboarding-tutorial.md) | 카드 캐러셀 + Nano Banana + 모션 옵트인 |

---

## 🔧 이 레포 유지보수 규칙

- **main 직접 작업, force-push 금지** (cached WebFetch 무결성을 위해 히스토리 갈아엎지 않음)
- **새 learning 추가 시 3곳 동시 업데이트:**
  1. `learnings/<topic>.md` 파일 추가
  2. README.md "📚 Learnings 인덱스" 표
  3. README.md "📄 CLAUDE.md 템플릿" 안 §5 Learnings 참조 표
- **부트스트랩 절차(Step 1~6) 또는 CLAUDE.md 템플릿 변경 시:** 기존 프로젝트의 local CLAUDE.md엔 영향 없음 (스냅샷). 새 프로젝트부터 새 절차/템플릿 적용.
- **이 레포 자체도 `docs/superpowers/` 컨벤션 따름:** 큰 변경은 spec → plan → 실행.
````

**Length target:** README 자체는 6~10KB.

- [ ] **Step 1:** Write the entire `README.md` exactly as shown above. **Do not modify, paraphrase, or shorten** — this content is the spec-approved template.

- [ ] **Step 2:** Verify:
  ```bash
  wc -c README.md
  grep -c '^## ' README.md
  grep '{{PROJECT_NAME}}' README.md
  grep '{{PACKAGE_NAME}}' README.md
  grep '{{CONCEPT_PLACEHOLDER}}' README.md
  grep -c 'raw.githubusercontent.com/Griotold/make-flutter-md' README.md
  ```
  Expected: 5500~10000 bytes, ≥5 H2 sections, all 3 placeholders present, ≥7 raw URL references (6 learnings + 1 template repo link).

- [ ] **Step 3:** Verify all 6 learning files are linked in BOTH the human-facing index AND the CLAUDE.md template's §5 table:
  ```bash
  for f in ios-build integration-test hive-ce-patterns firebase-setup monetization onboarding-tutorial; do
    cnt=$(grep -c "learnings/$f.md" README.md)
    echo "$f: $cnt references"
  done
  ```
  Expected: each file referenced ≥2 times (relative path in index + raw URL in template).

- [ ] **Step 4:** Commit
  ```bash
  git add . && git commit -m "README: 부트스트랩 절차 + CLAUDE.md 템플릿 + learnings 인덱스"
  ```

---

### Task 8: Cross-reference verification

**Files:**
- No new files. Verifies prior tasks.

**Purpose:** Catch any mismatch between README's CLAUDE.md template, README's learnings index, and the actual learnings/ files. Spec §9.3, §9.4, §9.6 require all to agree.

- [ ] **Step 1:** Verify all 6 learning files exist:
  ```bash
  ls -la learnings/
  ```
  Expected: 6 .md files (ios-build, integration-test, hive-ce-patterns, firebase-setup, monetization, onboarding-tutorial).

- [ ] **Step 2:** Verify each learning file is linked in README's index (relative path):
  ```bash
  for f in learnings/*.md; do
    base=$(basename "$f")
    if ! grep -q "./learnings/$base" README.md; then
      echo "MISSING in index: $base"
    fi
  done
  ```
  Expected: no "MISSING" output.

- [ ] **Step 3:** Verify each learning file is linked in CLAUDE.md template's §5 table (raw URL):
  ```bash
  for f in learnings/*.md; do
    base=$(basename "$f")
    if ! grep -q "raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/$base" README.md; then
      echo "MISSING in template §5: $base"
    fi
  done
  ```
  Expected: no "MISSING" output.

- [ ] **Step 4:** Verify CLAUDE.md template has all 6 numbered sections:
  ```bash
  for n in 1 2 3 4 5 6; do
    grep "^## $n\." README.md || echo "MISSING template section $n"
  done
  ```
  Expected: 6 matches printed, no MISSING.

- [ ] **Step 5:** Verify the §4 표준 기능 section in template explicitly references onboarding-tutorial.md:
  ```bash
  grep -A 10 '^## 4\.' README.md | grep 'onboarding-tutorial'
  ```
  Expected: 1 match.

- [ ] **Step 6:** No commit needed for verification, but if any step found mismatch, fix the offending file inline and commit:
  ```bash
  git add . && git commit -m "fix: 크로스 레퍼런스 정합성 수정"
  ```

---

### Task 9: Push to GitHub

**Files:**
- No file changes. Sets up remote and pushes.

**⚠️ This task requires user confirmation before pushing** — creating a public repo on user's GitHub account is a hard-to-reverse action.

- [ ] **Step 1:** Confirm GitHub username with user (spec assumes `hemille`):
  - Ask: "GitHub username이 `hemille` 맞아? 아니면 어떤 이름이야?"
  - 다르면 README.md의 raw URL과 CLAUDE.md 템플릿의 URL을 모두 새 username으로 교체 후 커밋.

- [ ] **Step 2:** Confirm repo creation method with user. Offer two options:
  - (a) Claude가 `gh repo create Griotold/make-flutter-md --public --source=. --push` 실행
  - (b) 사용자가 직접 GitHub 웹에서 빈 repo 만든 다음, Claude가 `git remote add origin ...` + `git push -u origin main`

- [ ] **Step 3:** Execute chosen method.

- [ ] **Step 4:** Verify raw URL actually resolves (이게 디자인 검증의 핵심):
  ```bash
  curl -sf -o /dev/null -w '%{http_code}\n' \
    https://raw.githubusercontent.com/Griotold/make-flutter-md/main/README.md
  ```
  Expected: `200`.

  같은 검증을 6개 learning에도:
  ```bash
  for f in ios-build integration-test hive-ce-patterns firebase-setup monetization onboarding-tutorial; do
    code=$(curl -sf -o /dev/null -w '%{http_code}' \
      https://raw.githubusercontent.com/Griotold/make-flutter-md/main/learnings/$f.md)
    echo "$f.md: $code"
  done
  ```
  Expected: all `200`.

- [ ] **Step 5:** Report repo URL to user.

---

### Task 10: Provide global `~/.claude/CLAUDE.md` trigger text

**Files:**
- No automated modification. **User must manually add the trigger section** — automated edits to global config files are not appropriate without explicit confirmation.

- [ ] **Step 1:** Show the user this exact text to append to `~/.claude/CLAUDE.md`:

  ```markdown
  ## Flutter 신규 프로젝트 부트스트랩

  빈/거의 빈 디렉터리에서 Flutter 앱 새로 만들자는 요청을 받으면:

  1. 먼저 https://raw.githubusercontent.com/Griotold/make-flutter-md/main/README.md
     를 WebFetch로 읽는다.
  2. 거기 적힌 부트스트랩 절차(Step 1~6)를 그대로 따른다.
  3. 절차에는 CLAUDE.md 템플릿 + 디렉터리 구조 + brainstorming 진입까지 포함.
  ```

- [ ] **Step 2:** Tell user where to add it: 글로벌 CLAUDE.md 파일 끝에 추가 (다른 섹션과 헤딩 레벨 동일하게 `## `).

- [ ] **Step 3:** Wait for user to confirm "추가했어".

- [ ] **Step 4:** No commit (이 변경은 이 레포 밖이라서).

---

### Task 11: End-to-end smoke test

**Files:**
- Creates a throwaway test directory (사용자 사이드에서). 결과 확인 후 삭제.

**Purpose:** §9.2 검증 — 빈 디렉터리에서 실제 흐름이 의도대로 동작하는지 확인.

- [ ] **Step 1:** 사용자에게 안내: 새 터미널 창에서 다음 실행 권유
  ```bash
  mkdir ~/Desktop/side/_smoke_test && cd ~/Desktop/side/_smoke_test
  claude
  ```

- [ ] **Step 2:** 새 Claude Code 세션에서 사용자가: "스모크 테스트용 앱 만들자"

- [ ] **Step 3:** 다음이 자동으로 일어나야 함:
  - 글로벌 CLAUDE.md의 트리거가 매치되어 README WebFetch 발생
  - Step 1~6이 실행 (이름 확인 → flutter create → 디렉터리 생성 → CLAUDE.md 작성 → git init → brainstorming 진입)
  - 생성된 `_smoke_test/CLAUDE.md`가 6개 섹션 모두 갖추고 raw URL 표 정확

- [ ] **Step 4:** 사용자가 결과 확인 후 보고. 문제 발견 시 해당 task로 돌아가서 수정.

- [ ] **Step 5:** 스모크 테스트 디렉터리 정리:
  ```bash
  rm -rf ~/Desktop/side/_smoke_test
  ```

- [ ] **Step 6:** 최종 완료 보고.

---

## 완료 정의 (spec §9 매핑)

| Spec 검증 기준 | 충족하는 Task |
|---|---|
| §9.1 README가 raw URL fetch 가능 | Task 9 Step 4 |
| §9.2 빈 디렉터리에서 Step 1~6 정상 실행 | Task 11 |
| §9.3 새 프로젝트 CLAUDE.md가 6개 섹션 + URL 표 정확 | Task 7 + Task 8 + Task 11 |
| §9.4 6개 learning 모두 작성 + 인덱스/템플릿 연결 | Task 1~6 + Task 8 |
| §9.5 글로벌 CLAUDE.md에 트리거 추가 | Task 10 |
| §9.6 §4 표준 기능이 onboarding-tutorial.md로 정확 연결 | Task 7 Step 3 + Task 8 Step 5 |
