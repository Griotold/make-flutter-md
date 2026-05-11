# make-flutter-md — Design Spec

**작성일:** 2026-05-11
**상태:** 디자인 합의 완료, 구현 대기

## 1. 목적

hemille가 Flutter로 게임/유틸 앱을 여러 개 만들면서 반복되는 규칙과 깨달음을 한 곳에 모아, 신규 프로젝트 시작 시 Claude Code가 자동으로 끌어다 쓰는 컨벤션 레포.

해결하려는 세 가지 문제 (모두 우선순위 동일):

1. **반복 작성 제거** — 매 프로젝트마다 CLAUDE.md에 같은 Flutter/iOS/`hive_ce`/Riverpod 규칙을 다시 쓰는 비효율 제거.
2. **축적된 학습 박제** — 여러 프로젝트를 거치며 얻은 검증된 패턴(iOS 26 release-only, `integration_test` 우회법, AdMob 한국 IDFA 등)을 한곳에 모아 잊지 않고 재사용.
3. **신규 프로젝트 부트스트랩 가속** — 새 Flutter 앱 시작할 때 폴더 구조/CLAUDE.md/디렉터리 셋업까지 손으로 안 하고 Claude가 자동으로.

### 1.1 사용자 배경 (워크플로우 가정)

앞으로 만드는 앱은 **본인이 0에서 아이디어를 내지 않는다**. Claude Code가 제안하거나, 마켓에 출시된 앱을 살짝 비트는 방향으로 진행한다. 이 워크플로우에는 함정이 있다 — **본인(hemille)이 본인 앱의 사용법을 모를 수 있다**. 이 상황을 정상화하기 위해 모든 신규 앱에 **인앱 튜토리얼**을 표준 기능으로 강제한다 (§4.2, §5.6 참조).

## 2. 사용 흐름

사람이 하는 일은 두 줄 + 한 마디뿐:

```bash
mkdir ~/Desktop/side/tetris-clone && cd ~/Desktop/side/tetris-clone
claude
```

```
> 테트리스 게임 만들자
```

Claude Code가 자동으로 하는 일:

1. 사용자 글로벌 `~/.claude/CLAUDE.md`가 세션 시작 시 자동 로드됨 — 거기에 "빈 디렉터리에서 Flutter 앱 만들자는 요청 → make-flutter-md README WebFetch" 트리거가 박혀있음.
2. 트리거 매치 → `https://raw.githubusercontent.com/hemille/make-flutter-md/main/README.md` WebFetch.
3. README의 "Claude Code 부트스트랩 절차" 섹션대로 실행 (flutter create → 디렉터리 생성 → CLAUDE.md 작성 → git init).
4. 끝나면 사용자에게 "셋업 완료" 알리고 `superpowers:brainstorming` skill 진입.
5. 개발 도중 특정 주제 필요 시 (예: `hive_ce` 패턴) 해당 `learnings/*.md`를 WebFetch.

## 3. 레포 구조

```
make-flutter-md/                          (public GitHub repo)
├── README.md                              # 사람용 안내 + Claude 부트스트랩 절차서
│                                          # + CLAUDE.md 템플릿 인라인 + learnings 인덱스
│
├── learnings/                             # 깊은 주제별 메모 (필요 시 WebFetch)
│   ├── ios-build.md
│   ├── integration-test.md
│   ├── hive-ce-patterns.md
│   ├── firebase-setup.md
│   ├── monetization.md
│   └── onboarding-tutorial.md             # 표준 기능: Gemini 영상 인앱 튜토리얼
│
└── docs/                                  # 이 레포 자체의 메타 문서
    └── superpowers/
        ├── specs/2026-05-11-make-flutter-md-design.md   (이 파일)
        └── plans/                                       (구현 플랜)
```

## 4. README.md 구성

README.md는 두 청중을 동시에 대응:

- **상단 (사람용):** 이 레포가 뭔지, 사람이 직접 읽을 필요 없다는 안내.
- **중단 (Claude 실행용):** "Claude Code 부트스트랩 절차" 섹션. 명령형 단계서.
- **하단:** CLAUDE.md 템플릿 인라인 + learnings 인덱스 표.

### 4.1 부트스트랩 절차 (Step 1~6)

| Step | 내용 |
|---|---|
| 1 | 사용자에게 앱 이름 확인 (디렉터리명 그대로 vs 변경, Dart 패키지명은 snake_case) |
| 2 | `flutter create <pkg> --platforms=ios --org com.hemille` 실행 |
| 3 | `docs/superpowers/specs/`, `plans/`, `docs/solutions/`, `docs/ideas/` 디렉터리 생성 (각 `.gitkeep`) |
| 4 | 루트에 CLAUDE.md 작성 (인라인 템플릿에서 `{{PROJECT_NAME}}`, `{{PACKAGE_NAME}}`, `{{CONCEPT_PLACEHOLDER}}` 치환) |
| 5 | `git init` + 초기 커밋 (`git add .` 전체 스테이징, 사용자 컨벤션 그대로). GitHub remote 추가는 자동 금지, 사용자에게 확인 |
| 6 | `superpowers:brainstorming` skill invoke |

### 4.2 CLAUDE.md 템플릿 (Step 4에서 복사할 인라인 내용)

다음 6개 섹션 포함:

1. **플랫폼 / 빌드** — iOS only, `--release` 강제, 실기기 설치 명령
2. **기술 스택** — Riverpod, `hive_ce`, GoRouter(조건부), Firebase(조건부)
3. **작업 컨벤션** — main 직접 작업, `git add .` 전체 스테이징, `docs/superpowers/` 워크플로우
4. **표준 기능 (모든 앱 공통)** — 이 레포가 정의하는 신규 앱 필수 기능. v1에는 다음 1개:
   - **인앱 튜토리얼** (무성 영상, 실제 앱 스크린샷 + Nano Banana 오버레이 + ffmpeg 스티치, 빌드 타임 생성, `assets/tutorial/`에 번들)
   - 첫 실행 시 자동 재생 + 설정/메인의 ⓘ 버튼에서 언제든 재생
   - `hive_ce`에 `tutorial_seen` 플래그 저장, 스킵 가능하되 "다시 보기" 진입점 필수
   - 가상 화면 금지 — 실제 앱 스크린샷만 사용
   - 자세한 패턴은 `learnings/onboarding-tutorial.md` 참조
5. **Learnings 참조 표** — 각 learning의 raw GitHub URL (Claude가 필요 시 WebFetch)
6. **프로젝트 고유** — 컨셉/하드 룰/활성 플랜 (brainstorming 후 채워질 빈칸)

## 5. learnings/ 6개 파일 스코프

각 파일은 1–3KB 목표. "왜"와 "정확한 명령" 위주, 일반 Flutter 튜토리얼 내용은 제외.

### 5.1 `learnings/ios-build.md`

- iOS 26+ release-only 정책 배경
- 실기기 설치 명령 (`flutter build ios --release && flutter install --release --device-timeout 15`)
- `flutter run --release`가 터미널 블로킹돼서 쓰면 안 되는 이유
- Wi-Fi 무선 빌드 조건 + `--device-timeout` 튜닝
- CI에서 `--no-codesign` 빌드 검증 패턴

### 5.2 `learnings/integration-test.md`

- `flutter test integration_test/...`가 iOS 26 실기기에서 안 되는 이유 (debug 모드 강제)
- **권장 경로:** `flutter drive --driver=test_driver/integration_test.dart --target=... --release` + `test_driver/integration_test.dart` 러너 파일 내용 인라인
- **대안 경로:** iOS 16–25 시뮬레이터 + `flutter test -d <sim_id>`
- CI에서 `flutter test`와 통합 테스트 분리 패턴

### 5.3 `learnings/hive-ce-patterns.md`

- `hive_ce`를 쓰는 이유 (원본 hive 유지보수 중단)
- pubspec 추가 패턴 (`hive_ce`, `hive_ce_flutter`, `hive_ce_generator`)
- `@GenerateAdapters` 어노테이션 + `build_runner` 명령
- boxName 컨벤션 (단수 + 소문자, 예: `'scores'`, `'settings'`)
- 마이그레이션 패턴 (스키마 변경 시)
- IsolatedHive 언제 쓸지

### 5.4 `learnings/firebase-setup.md`

- flutterfire CLI 셋업 순서 (firebase login → flutterfire configure → 패키지 추가)
- Anonymous Auth → Google upgrade 패턴
- `firestore.rules` 안전 기본값 + 자주 쓰는 패턴 (사용자 소유 데이터, 공용 리더보드)
- Cloud Functions `onCall` 패턴 (auth uid 자동 검증, region 설정)
- 자주 쓰는 패키지 조합
- `firestore.indexes.json` 관리

### 5.5 `learnings/monetization.md`

- AdMob 테스트 ID vs 프로덕션 ID 분리 패턴
- 한국 IDFA 동의 다이얼로그 (App Tracking Transparency) 요건
- `in_app_purchase` 공식 패키지 사용 (서드파티 금지)
- 자주 만드는 하드 룰 패턴:
  - "Daily/핵심 플로우엔 전면광고 절대 없음"
  - "진행도 IAP 금지"
  - "Remove Ads 구매 시 진짜 광고 사라짐"
- Restore Purchase 플로우 (App Store 필수)

### 5.6 `learnings/onboarding-tutorial.md`

- **배경:** §1.1의 워크플로우 가정 — 본인이 본인 앱 사용법을 모를 수 있음. 본인 + 일반 사용자 모두를 위한 표준 기능.
- **표시 시점:**
  - 첫 실행 시 자동 (`hive_ce`의 `app_settings` 박스에 `tutorial_seen: bool`)
  - 설정 또는 메인 화면의 ⓘ 버튼에서 언제든 재생 (재시청 진입점 *필수*)
- **포맷:** 짧은 영상 (20~60초), **음성 없음**. 실제 앱 스크린샷 기반 + 텍스트 오버레이 + 크로스페이드. 무음이라 재생 환경 무관(음소거/공공장소).
- **콘텐츠 원칙:** **반드시 실제 앱 화면을 보여줄 것.** 프롬프트로 AI가 상상한 가상 화면 사용 금지 — 사용자가 보는 화면과 튜토리얼이 일치해야 의미 있음.
- **생성 파이프라인 (빌드 타임, 런타임 생성 금지):**
  1. 개발 도중 실기기/시뮬레이터에서 핵심 플로우의 **실제 스크린샷 5~8장** 수집 → `tool/tutorial-source/raw/` 에 저장
  2. **Nano Banana** (Gemini 2.5 image edit API) 호출 — 각 스크린샷에 화살표/하이라이트/탭 인디케이터 오버레이 추가. 원본 화면 구조는 보존, 시선 유도 요소만 더하기.
  3. **ffmpeg**로 스티치 — 크로스페이드 전환 + 텍스트 자막 오버레이 (각 프레임 3~5초)
  4. 출력: `assets/tutorial/{lang}.mp4`
  5. 전체 빌드 스크립트: `dart run tool/build_tutorial.dart` (Gemini API 키는 `.env`에서 읽고, 절대 커밋 금지)
- **재생 패키지:** `video_player` (chewie 불필요 — 컨트롤 최소화, 자동 재생 + 탭으로 스킵)
- **언어:** 디바이스 로케일에 따라 `ko.mp4` / `en.mp4` 분기. 영어 없으면 한국어 폴백. 텍스트 오버레이가 언어별이라 ffmpeg 합성 단계가 언어별로 한 번씩 실행됨 (Nano Banana 단계는 1회로 충분).
- **하드 룰:**
  - 스킵 가능 (Skip 버튼 항상 노출).
  - "다시 보기" 진입점이 1~2탭 내 접근 가능해야 함.
  - 튜토리얼 자체에 광고 절대 없음.
  - 음성 없음 — 무성 영상 + 텍스트 오버레이로 충분.
  - 가상 화면 절대 사용 금지 — 실제 앱 스크린샷만.
- **의존성:** Nano Banana API 키 (`GEMINI_API_KEY`), 로컬 `ffmpeg` 설치, `image` Dart 패키지(스크린샷 전처리용, 선택적).

## 6. 사용자 글로벌 `~/.claude/CLAUDE.md` 추가 내용

이 디자인이 동작하려면 사용자 글로벌 CLAUDE.md에 다음 섹션이 추가되어야 함 (평생 한 번 셋업):

```markdown
## Flutter 신규 프로젝트 부트스트랩

빈/거의 빈 디렉터리에서 Flutter 앱 새로 만들자는 요청을 받으면:

1. 먼저 https://raw.githubusercontent.com/hemille/make-flutter-md/main/README.md
   를 WebFetch로 읽는다.
2. 거기 적힌 부트스트랩 절차를 그대로 따른다.
3. 절차에는 CLAUDE.md 템플릿 + 디렉터리 구조 + brainstorming 진입까지 포함.
```

> 구현 플랜에서 이 추가도 Step으로 포함 — 사용자가 직접 손으로 글로벌 CLAUDE.md 수정해야 하므로, 정확한 추가 위치/문구를 사용자에게 보여주고 확인.

## 7. 유지보수 규칙

이 레포 자체의 운영 규칙 — README 하단에도 인라인됨.

- **main 직접 작업, force-push 금지.** WebFetch 캐시 무결성을 위해 히스토리 갈아엎지 않음.
- **새 learning 추가 시 3곳 동시 업데이트:**
  1. `learnings/<topic>.md` 파일 추가
  2. `README.md`의 learnings 인덱스 표
  3. `README.md` 안 CLAUDE.md 템플릿의 learnings 참조 표
- **부트스트랩 절차(Step 1~6) 또는 CLAUDE.md 템플릿 변경 시:** 기존 프로젝트의 local CLAUDE.md엔 영향 없음 (스냅샷이라). 다만 새 프로젝트는 새 절차/템플릿 적용.
- **이 레포 자체도 `docs/superpowers/` 컨벤션 따름:** 새 큰 변경은 spec → plan → 실행.

## 8. 의도적으로 안 하는 것 (YAGNI)

- ❌ **Bootstrap 스크립트(`new-flutter-app.sh`).** 사용자가 손으로 안 돌리는 게 원칙. Claude가 Step 1~6 직접 실행.
- ❌ **버전 태깅 / 릴리즈.** main이 항상 최신. 옛 버전을 다시 끌어다 쓸 일 없음.
- ❌ **Android 빌드 가이드.** iOS only가 기본. 프로젝트별 예외는 local CLAUDE.md에서 처리.
- ❌ **flutter create 자동화 옵션 확장(앱 아이콘, 스플래시 등).** 너무 프로젝트별이라 부트스트랩에 안 넣음. 필요 시 brainstorming 결과로.
- ❌ **CI/GitHub Actions 템플릿.** 너 솔로 워크플로우라 CI 필요한 프로젝트가 많지 않음. 필요한 프로젝트에서만 도입.

## 9. 검증 기준 (구현 완료 정의)

다음이 다 되면 v1 완성:

1. README.md가 GitHub에서 raw URL로 fetch 가능.
2. 빈 디렉터리에서 (시뮬레이션 또는 실제) `mkdir foo && cd foo && claude` → "테스트용 앱 만들자"라고 했을 때 Step 1~6이 의도대로 실행됨.
3. 생성된 새 프로젝트의 CLAUDE.md가 6개 섹션 모두 갖춤 + learnings URL 표 정확.
4. 6개 learning 파일 모두 작성됨 + 인덱스/템플릿 표에 모두 연결.
5. 사용자 글로벌 `~/.claude/CLAUDE.md`에 트리거 추가됨.
6. CLAUDE.md 템플릿의 §4 "표준 기능"이 인앱 튜토리얼을 명시하고, `learnings/onboarding-tutorial.md`로 정확히 연결됨.
