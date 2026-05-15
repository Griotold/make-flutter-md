# iOS Build

## 왜 release-only인가

iOS 26+ 보안 정책 강화로 Flutter 디버그 모드가 실기기와 시뮬레이터에서 동작하지 않는다. 모든 iOS 빌드는 `--release` 플래그를 사용한다.

## 실행 주체 — Claude Code가 직접

빌드와 설치는 **Claude Code가 Bash 도구로 직접 호출**하는 게 default. 사용자한테 "터미널에서 이 명령 실행하세요" 떠넘기지 않는다. 빌드 5~10분이 걸리므로 `run_in_background: true`로 띄우고 완료 알림을 받는다.

```
Bash(command="flutter build ios --release && flutter install --release --device-timeout 15",
     run_in_background=true)
```

## 표준 실기기 설치 명령

```bash
flutter build ios --release && flutter install --release --device-timeout 15
```

## 빌드 전 사전 체크리스트 (Claude가 자동 수행)

빌드 명령 전에 두 가지를 항상 확인:

1. **연결된 디바이스 확인**:

   ```bash
   flutter devices
   ```

   타겟 iPhone이 목록에 보이지 않으면 사용자에게 "iPhone 연결/잠금 해제 확인 필요" 알린 뒤 대기.

2. **디바이스 잠금 해제 상태 안내**: USB든 Wi-Fi든 **잠금 해제 필수**. `devicectl`이 빌드 도중 developer disk image를 마운트할 때 잠금 해제가 필요하기 때문 — Wi-Fi만의 문제가 아님. 빌드 시작 직전 사용자에게 "iPhone 잠금 해제하고 화면 켜두세요" 한 줄 알림.

## 왜 `flutter run --release`는 쓰면 안 되는가

`flutter run`은 hot-restart 입력을 위해 터미널을 점유한 채 대기한다. Claude Code 워크플로우에서 다음 명령으로 진행할 수 없으므로 사용 금지. **반드시 `flutter build ios --release && flutter install --release`**.

## Wi-Fi 무선 빌드

같은 Wi-Fi + 아이폰 잠금 해제면 USB 없이 배포 가능. 무선 기기가 감지되지 않으면 `--device-timeout` 값을 늘린다 (기본 15초 → 30~60초).

## 흔한 실패 분기 + 복구

| 실패 메시지 키워드 | 원인 | 복구 |
|---|---|---|
| `Device is locked` / `developer disk image` | 잠금 상태에서 마운트 시도 | 사용자에게 잠금 해제 요청 → 재실행 |
| `No signing certificate` / `provisioning profile` | Xcode 사인 미설정 | `open ios/Runner.xcworkspace` 안내, Xcode에서 Team 선택 후 재실행 |
| `Device not found` / `--device-timeout` 타임아웃 | 무선 기기 늦게 광고 | `--device-timeout 60`으로 재시도, 그래도 안 되면 USB 연결 안내 |
| `Pods` 관련 에러 (CocoaPods sandbox sync 등) | Pod 의존성 변경 미반영 | `cd ios && pod install --repo-update && cd ..` 후 재빌드 |
| `Provisioning profile doesn't include the currently selected device` | 새 기기 UDID 미등록 | Apple Developer 콘솔에서 device 등록 + profile 재생성, 사용자 안내 |

각 케이스에서 Claude는 (a) 즉시 시도할 수 있는 복구는 시도, (b) 사용자 액션 필요한 건 명확히 한 줄 안내 후 대기.

## 백그라운드 실행 권장

iOS 빌드는 보통 5~10분(첫 빌드 + Pod install 시 더 길게). 항상 `run_in_background: true`로 띄워 다른 작업과 병렬화. 빌드 끝나면 자동 알림 받고 다음 진행.

장시간 빌드 중에 사용자가 다른 질문하면 **빌드 상태 묻지 않고** 그냥 답하면서 진행. 빌드 완료 알림이 자체적으로 옴.

## CI / 빌드 검증

서명 없이 빌드만 검증할 때:

```bash
flutter build ios --release --no-codesign
```

## 첫 빌드 시 추가 확인 (오디오/광고 통합 직후)

`learnings/audio-policy.md`의 첫 빌드 검수(무음 모드 + 유튜브 동시 재생)는 실기기 install 후 반드시 수행. AdMob/IAP 통합 직후에도 오디오 카테고리가 덮어쓰일 수 있어 같은 두 시나리오 재확인.
