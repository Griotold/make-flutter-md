# hive_ce 패턴

## 왜 hive_ce (원본 hive 금지)

원본 `hive` / `hive_flutter` 패키지는 2년 이상 업데이트가 끊긴 유지보수 중단 상태다. `hive_ce` (Community Edition) + `hive_ce_flutter`가 Hive v2의 공식 계승 포크이며 활발히 관리된다.

추가 기능: IsolatedHive, WASM 지원, @GenerateAdapters 자동 어댑터 생성, DevTools Inspector.

원본 hive는 제안조차 금지. Flutter 프로젝트에서 로컬 저장소가 필요하면 항상 `hive_ce` + `hive_ce_flutter`를 사용한다.

## pubspec 추가

```yaml
dependencies:
  hive_ce: ^<latest>
  hive_ce_flutter: ^<latest>

dev_dependencies:
  hive_ce_generator: ^<latest>
  build_runner: ^<latest>
```

(버전은 `flutter pub add hive_ce hive_ce_flutter` 로 최신 채움)

## 어댑터 생성 패턴

모든 저장 모델 클래스를 `@GenerateAdapters` 리스트에 등록한다.

```dart
import 'package:hive_ce/hive.dart';

part 'hive_registrar.g.dart';

@GenerateAdapters([
  AdapterSpec<Score>(),
  AdapterSpec<UserSettings>(),
  AdapterSpec<Medal>(),
])
void _generated() {}
```

실행: `dart run build_runner build --delete-conflicting-outputs`

## boxName 컨벤션

Box 이름(저장소 식별자)은 단수 형태 + 소문자로 통일한다.

예시:
- `'settings'` (사용자 설정 단일 박스)
- `'scores'` (점수 기록 리스트)
- `'app_settings'` (앱 전체 상태)
- `'user_profile'`

복수형 사용도 가능하지만, 한 프로젝트 내에서는 일관되게 유지한다.

## 초기화 순서

hive는 반드시 다음 순서로 초기화한다:

1. `Hive.initFlutter()` — 플랫폼별 저장소 경로 설정
2. `Hive.registerAdapters()` — hive_ce_generator 자동 생성 어댑터 등록
3. `Hive.openBox<T>(boxName)` — 실제 박스 열기

```dart
// 예: main() 또는 app startup 로직
await Hive.initFlutter();
Hive.registerAdapters(); // hive_ce_generator에서 자동 생성
final settingsBox = await Hive.openBox<UserSettings>('settings');
final scoresBox = await Hive.openBox<Score>('scores');
```

어댑터를 등록하지 않고 박스를 열려고 하면 런타임 에러가 발생한다. 초기화 순서를 절대 바꾸지 않는다.

## 스키마 변경 시

**typeId 재사용 금지** — 같은 ID를 다른 클래스에 주면 역직렬화 실패.

**필드 추가 OK** — null 또는 default 처리.

```dart
class Score {
  final int points;
  final DateTime playedAt;
  final String? playerName; // 추가: null 처리 가능
}
```

**필드 삭제 금지, deprecated 처리** — 새 필드 추가가 안전.

```dart
class ScoreV2 {
  final int points;
  final DateTime playedAt;
  // oldField 제거, 새로운 필드만 추가
}
```

## IsolatedHive 언제 쓸지

별도 Dart isolate에서 hive 접근 시에만 사용 (백그라운드 작업). 일반 UI 스레드 작업엔 불필요.

```dart
final isolateHive = await IsolatedHive.initialize();
final box = await isolateHive.openBox<Score>('scores');
```
