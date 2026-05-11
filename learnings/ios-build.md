# iOS Build

## 왜 release-only인가

iOS 26+ 보안 정책 강화로 Flutter 디버그 모드가 실기기와 시뮬레이터에서 동작하지 않는다. 따라서 모든 iOS 빌드는 `--release` 플래그를 사용해야 한다.

## 실기기 설치 명령

```bash
flutter build ios --release && flutter install --release --device-timeout 15
```

## 왜 `flutter run --release`는 쓰면 안 되는가

`flutter run --release`를 사용하면 터미널이 대기 상태로 멈춘다. Claude Code 워크플로우에서 다음 명령을 진행할 수 없으므로 사용 금지. 대신 `flutter build ios --release && flutter install --release`를 사용한다.

## Wi-Fi 무선 빌드

같은 Wi-Fi 네트워크에 연결되고 아이폰이 잠금 해제된 상태면 USB 없이 배포 가능하다. 무선 기기가 감지되지 않으면 `--device-timeout` 값을 늘린다.

## CI / 빌드 검증

빌드 검증 및 CI 환경에서는 `--no-codesign` 플래그를 사용한다:

```bash
flutter build ios --release --no-codesign
```
