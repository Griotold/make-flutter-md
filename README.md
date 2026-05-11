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

> 📚 이 프로젝트는 https://github.com/hemille/make-flutter-md 의 규칙을 따른다.
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
- 자세한 패턴: [onboarding-tutorial.md](https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/onboarding-tutorial.md)

## 5. Learnings 참조 (필요 시 WebFetch)

| 주제 | URL |
|---|---|
| iOS 빌드 | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/ios-build.md |
| integration_test 우회 | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/integration-test.md |
| hive_ce 패턴 | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/hive-ce-patterns.md |
| Firebase 셋업 | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/firebase-setup.md |
| 수익화 (광고/IAP) | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/monetization.md |
| 인앱 튜토리얼 | https://raw.githubusercontent.com/hemille/make-flutter-md/main/learnings/onboarding-tutorial.md |

## 6. 이 프로젝트 고유

- **패키지명:** {{PACKAGE_NAME}}
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
