# 오디오 정책 (BGM + 효과음)

## 왜 이 learning이 따로 있는가

Flutter `audioplayers` / `just_audio`의 기본 동작은 보통 iOS의 `playback` 카테고리 — **무음 모드 무시 + 다른 앱 오디오 끊음**. 그대로 출시하면 "지하철에서 효과음 터짐", "유튜브 듣다가 게임 켰더니 끊김" 같은 매너 사고가 즉시 발생한다. 게임/유틸 앱은 거의 항상 `ambient + mixWithOthers`가 정답인데, 명시적으로 박아두지 않으면 잊는다.

## iOS AVAudioSession 카테고리 비교

| 카테고리 | 무음 스위치 | 다른 앱 오디오 | 게임/유틸 적합 |
|---|---|---|---|
| `ambient` | 존중 → 무음이면 무음 | mix 가능 | ✅ |
| `soloAmbient` (Flutter 기본 케이스) | 존중 | 끊음 | ❌ 유튜브 끊김 |
| `playback` (음악 앱용) | 무시 → 항상 소리 남 | 끊음(기본) | ❌ 매너 위반 |

## 표준 정책 (모든 신규 앱 공통)

- **모든 사운드(BGM + 효과음) = `ambient` 카테고리 + `mixWithOthers: true`**
- 무음 모드 → 게임 무음
- 다른 앱 음악 재생 중 → 게임 BGM 자동 음소거(또는 옵션), 효과음은 mix되어 작게 들림
- 설정 화면에서 **"BGM 끄기 / 효과음 끄기" 별도 토글**(배터리/취향 대응)
- 햅틱은 별도 — 무음 스위치와 무관, 사용자가 설정에서 끌 수 있음
- 옵션: "다른 앱 음악 재생 중 감지 → 게임 효과음 볼륨 0.3배 자동 다운"(매너 점수)

## Flutter 구현 패턴

1. **`audio_session` 패키지로 카테고리 설정** — 앱 시작 시 1회 등록:

   ```dart
   final session = await AudioSession.instance;
   await session.configure(const AudioSessionConfiguration(
     avAudioSessionCategory: AVAudioSessionCategory.ambient,
     avAudioSessionCategoryOptions: AVAudioSessionCategoryOptions.mixWithOthers,
   ));
   ```

2. **사운드 재생**: `audioplayers` (간단) 또는 `just_audio` (스트림/플레이리스트 필요 시).
3. **다른 앱 음악 재생 감지**: `session.otherAudioActive` 또는 `interruptionEventStream` 구독 → BGM 일시정지 + 효과음 볼륨 다운.
4. **설정 저장**: `hive_ce` `app_settings` 박스에 `bgm_enabled`, `sfx_enabled`, `haptic_enabled` bool로 저장 (기본 `true`).

## Hive 키 컨벤션

| 박스 | 키 | 기본값 |
|---|---|---|
| `app_settings` | `bgm_enabled` | `true` |
| `app_settings` | `sfx_enabled` | `true` |
| `app_settings` | `haptic_enabled` | `true` |

## 하드 룰

- `playback` 카테고리 사용 금지 (음악 재생 앱 외).
- 첫 빌드 전에 반드시 실기기에서 (a) 무음 모드 (b) 유튜브 동시 재생 두 시나리오 확인.
- BGM/효과음/햅틱 각각 별도 토글. "사운드 끄기" 하나로 묶지 말 것.
- 광고 SDK가 자체 오디오 카테고리를 덮어쓸 수 있음 — 광고 통합 후 다시 두 시나리오 재확인.
- 햅틱은 `haptic_enabled = false`면 절대 발생 안 하도록 한 곳에서 게이트.

## Android (cross-platform 갈 때)

- Android는 `AudioFocus`/`AudioAttributes`로 대응. `audio_session`이 양쪽 추상화 제공.
- `usage: AndroidAudioUsage.game` + `contentType: AndroidAudioContentType.sonification` + `audioFocusGainTransientMayDuck` 조합이 iOS `ambient + mixWithOthers`와 가장 가까움.
- 본 레포는 iOS only가 기본이라 Android 케이스는 사용자가 명시적으로 cross-platform 요청할 때만 적용.
