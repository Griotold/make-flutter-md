# Integration Test (iOS 26 우회)

## 문제

`flutter test integration_test/...` 는 디버그 모드를 강제하는데, iOS 26+ 실기기와 시뮬레이터에서 디버그 모드가 동작하지 않는다. 따라서 표준 통합 테스트 명령은 사용할 수 없다.

## 권장 경로 — flutter drive (release)

릴리즈 모드로 통합 테스트를 실행한다. iOS 26 에서도 동작한다.

```bash
flutter drive --driver=test_driver/integration_test.dart --target=integration_test/<file>.dart --release
```

러너 파일 `test_driver/integration_test.dart` 를 다음과 같이 작성한다. 첫 통합 테스트 작성 시 추가한다:

```dart
import 'package:integration_test/integration_test_driver.dart';

Future<void> main() => integrationDriver();
```

## 대안 경로 — iOS 16~25 시뮬레이터

iOS 16 이상 25 이하 버전 시뮬레이터를 부팅한 뒤 다음 명령으로 실행한다:

```bash
flutter test integration_test/<file>.dart -d <sim_id>
```

단점: 해당 버전 시뮬레이터를 맥에 미리 설치해 두어야 한다.

## CI에서 분리

`flutter test` (단위 테스트)와 `flutter drive` (통합 테스트)를 별도 단계로 실행한다. 한 명령으로 묶지 않는다.
