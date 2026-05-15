# 디자인 도구 (Stitch + Nano Banana)

## 왜 디자인을 분리해 다루는가

AI 생성 Flutter 앱이 Material 3 디폴트 그대로 출시되면 마켓에서 즉시 "generic AI app"으로 인식돼 다운로드/유지율이 무너진다. 디자인 시스템과 비주얼 에셋을 brainstorm 직후~코드 시작 전에 못 박는 것이 표준.

## 도구 분담

| 작업 | 도구 |
|---|---|
| Design system (컬러/타이포/스페이싱) | Stitch |
| 핵심 화면 시안 (메인/세컨더리 2~3개) | Stitch (text-to-screen) |
| 화면 일관성 적용 | Stitch (apply design system) |
| 앱 아이콘 (마켓 등록용) | Nano Banana |
| 스플래시 이미지 | Nano Banana |
| 인앱 일러스트 / 빈 상태 그래픽 | Nano Banana |
| 마케팅 스크린샷 합성 배경 | Nano Banana |
| 튜토리얼 카드 이미지 | Nano Banana (`onboarding-tutorial.md` 별도) |

**Stitch**: Google의 AI UI 디자인 도구. `stitch-mcp` MCP 서버로 호출. 핵심 흐름:
- `upload_design_md` → `create_design_system_from_design_md` — `DESIGN.md`로부터 design system 생성
- `generate_screen_from_text` — 텍스트로 화면 시안 생성
- `apply_design_system` — 기존 화면에 design system 적용
- `generate_variants` — 변주 시안
- `fetch_screen_image` / `fetch_screen_code` — 결과 받기

**Nano Banana**: Gemini 2.5 image generation. 텍스트→이미지, 이미지 편집, 다중 참조 합성 지원. **빌드 타임에만 호출** (런타임 생성 금지 — 네트워크/비용/대기 문제).

## 단계별 흐름

### Phase 0 — brainstorming 직후: `docs/design/DESIGN.md` 작성

디자인 시스템 source of truth.

- 컨셉 한 줄 (예: "고양이 + 테트리스 → 따뜻한 파스텔 + 8bit 픽셀 강조")
- 컬러 팔레트 (3~5색, hex)
- 타이포 (제목/본문, Google Fonts 우선)
- 분위기 키워드 (예: warm / cozy / playful)
- 참고 레퍼런스 (마켓에 있는 비슷한 앱 캡처 링크)

### Phase 1 — Stitch로 design system + 핵심 화면

1. `DESIGN.md`를 Stitch에 업로드 → design system 생성
2. 메인 화면 + 세컨더리 1~2개 `generate_screen_from_text`
3. 사용자에게 시안 보여주고 승인 받기. 미승인 시 `generate_variants`로 변주
4. 승인된 시안은 Flutter 코드로 손번역 (Stitch→Flutter 자동 변환은 없다고 가정)

### Phase 2 — Flutter 골격에 적용

- 컬러/타이포는 `ThemeData`에 한 번 박아 일관성 확보
- Stitch 시안은 reference. Flutter 위젯은 직접 작성

### Phase 3 — 이미지 에셋 (Nano Banana, 빌드 타임)

빌드 타임 스크립트(`tool/build_assets.dart` 권장)로 생성:
- `assets/icon/icon_*.png` — 마켓 등록용 아이콘 (1024x1024)
- `assets/splash/splash.png` — 스플래시
- `assets/illustration/empty_*.png` — 빈 상태 일러스트
- `assets/tutorial/card_N.{png|mp4}` — 튜토리얼 (`onboarding-tutorial.md` 별도 처리)

`GEMINI_API_KEY`는 `.env`에서 읽고 `.gitignore` 포함. **절대 커밋 금지.**

## 하드 룰

- Material 3 디폴트 `ThemeData` 그대로 출시 금지.
- 모든 이미지 에셋은 같은 design system(컬러/스타일 키워드) 안에서 생성해 톤 일관성 확보.
- 아이콘은 마켓 등록 전 별도 검수 — 32x32 thumbnail 가독성 확인.
- 일러스트는 **실제 이 앱 UI를 반영** — 가상 다른 앱 화면 그리기 금지.
- 런타임 이미지 생성 금지. 빌드 타임에만.
- 최종 `assets/`만 커밋. raw 시안(`docs/design/raw/` 등)은 `.gitignore`.

## 결정 보류 / 미확정

- Stitch 출력 형식(HTML/CSS vs 그 외)과 Flutter 손번역 비용은 시점별로 다름. v1 적용 시 실측 후 본 learning 갱신.
- image-to-video 엔드포인트(Veo 2/3 등)는 `onboarding-tutorial.md` 참조.
