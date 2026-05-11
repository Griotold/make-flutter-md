# hive_ce 패턴

## 왜 hive_ce (원본 hive 금지)

원본 `hive` / `hive_flutter` 패키지는 2년 이상 업데이트가 끊긴 유지보수 중단 상태다. `hive_ce` (Community Edition) + `hive_ce_flutter`가 Hive v2의 공식 계승 포크이며 활발히 관리된다.

추가 기능:
- **IsolatedHive**: 백그라운드 isolate에서 hive 접근 가능
- **WASM 지원**: 웹 타겟 빌드 가능
- **`@GenerateAdapters` 자동 생성**: build_runner 통합으로 어댑터 코드 자동화
- **DevTools Inspector**: 로컬 저장소 시각화 및 디버깅

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

실제 버전은 작성 시점에 `flutter pub add hive_ce hive_ce_flutter` 로 최신을 자동 확인하고, dev_dependencies는 `flutter pub add -d hive_ce_generator build_runner` 로 추가한다.

## 어댑터 생성 패턴

hive_ce_generator는 Dart 클래스를 hive 저장소 형식으로 자동 변환하는 어댑터를 생성한다. 모든 저장할 모델 클래스를 `@GenerateAdapters` 리스트에 등록한다.

```dart
// lib/data/hive_registrar.g.dart 자동 생성 대상 모음
import 'package:hive_ce/hive.dart';

part 'hive_registrar.g.dart';

@GenerateAdapters([
  AdapterSpec<Score>(),
  AdapterSpec<UserSettings>(),
  AdapterSpec<Medal>(),
])
void _generated() {}
```

실행:

```bash
dart run build_runner build --delete-conflicting-outputs
```

`--delete-conflicting-outputs` 플래그는 기존 `*.g.dart` 파일을 덮어쓴다. 실제 앱 부팅 시 `Hive.registerAdapters()` 호출로 자동 생성된 어댑터를 등록한다.

## boxName 컨벤션

Box 이름(저장소 식별자)은 단수 형태 + 소문자로 통일한다.

예시:
- `'settings'` (사용자 설정 단일 박스)
- `'scores'` (점수 기록 리스트)
- `'app_settings'` (앱 전체 상태)
- `'user_profile'`

복수형 사용도 가능하지만, 한 프로젝트 내에서는 일관되게 유지한다. 혼용하면 코드 가독성이 떨어진다.

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

모델 클래스에 필드를 추가하거나 삭제할 때 주의해야 할 점:

**typeId는 절대 재사용 금지**: 어댑터 스펙에 `typeId` 를 명시했다면, 그 ID는 이미 저장된 데이터에 매핑되어 있다. 같은 typeId를 다른 클래스에 재할당하면 역직렬화 실패.

**필드 추가는 안전**: 새 필드는 null 또는 기본값으로 처리 가능. 기존 저장 데이터와 호환성 유지.

```dart
class Score {
  final int points;
  final DateTime playedAt;
  final String? playerName; // 추가: null 처리 가능
}
```

**필드 삭제 시 위험**: 저장된 데이터에 그 필드가 있으면 역직렬화 중 오류. 해결책:
1. 삭제 대신 deprecated 필드로 두고 읽기만 허용
2. 또는 새 클래스를 만들고 마이그레이션 로직 추가

```dart
// 비추천: 직접 삭제
// final String oldField;  // 위험!

// 권장: deprecated로 유지하거나 새 클래스 도입
class ScoreV2 {
  final int points;
  final DateTime playedAt;
  // oldField 제거, 새로운 필드만 추가
}
```

## IsolatedHive 언제 쓸지

`IsolatedHive`는 별도의 Dart isolate (스레드)에서 hive를 접근할 때 사용한다. 백그라운드 작업, 무거운 계산, 또는 메인 UI 스레드를 차단하지 않아야 하는 경우에만 필요.

**일반 UI 스레드에서 hive 접근**: `Hive.openBox()` + 동기 읽기/쓰기로 충분. 별도 설정 없음.

**백그라운드 isolate에서 hive 접근** (예: 앱 시작 시 대량 로드, 또는 Workmanager와 연동):

```dart
final isolateHive = await IsolatedHive.initialize();
final box = await isolateHive.openBox<Score>('scores');
// isolate 내에서 데이터 접근 및 수정
```

대부분의 Flutter 앱은 메인 스레드에서만 hive를 접근하므로 `IsolatedHive`는 불필요. 성능 문제가 관찰될 때만 도입한다.
