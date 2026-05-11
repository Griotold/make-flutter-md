# 수익화 (광고 + IAP)

## 광고 — Google Mobile Ads (AdMob)

패키지: `google_mobile_ads` (공식).

테스트 ID와 프로덕션 ID를 반드시 분리한다. 테스트 ID를 사용해야 AdMob 계정이 정지되지 않는다.

```dart
// dev/test
const testInterstitial = 'ca-app-pub-3940256099942544/4411468910'; // Google 공식 테스트 ID
// prod (Remote Config 또는 .env에서)
final adUnitId = kDebugMode ? testInterstitial : prodId;
```

전면광고(Interstitial)는 사용자 동작 후에만 띄운다 (게임 판 완료 직후 등). 배너 광고는 화면 하단에 고정하되, 게임 조작을 방해하지 않도록 SafeArea 위에 배치한다.

초기화는 `main()`에서 `MobileAds.instance.initialize()` 호출. 사용자 동의(GDPR/CCPA) 처리 후 초기화해야 한다.

## 한국 IDFA (App Tracking Transparency)

iOS 14.5+ 부터 IDFA 접근에 사용자 동의 필수. `app_tracking_transparency` 패키지로 시스템 다이얼로그를 띄운다.

앱 첫 실행 시 **광고 초기화 전에** 다음을 호출한다:

```dart
final status = await AppTrackingTransparency.requestTrackingAuthorization();
// status: denied, restricted, provisional, authorized 중 하나
// authorized인 경우에만 IDFA 사용 가능
```

`Info.plist`에 한국어 설명 문구를 필수로 넣는다:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>더 나은 광고 경험을 위해 기기 추적 권한이 필요합니다.</string>
```

권한 거부 후에도 광고는 정상 표시되지만, 성능(타게팅 정확도)이 낮아진다.

## IAP — `in_app_purchase` 공식 패키지

서드파티 IAP 라이브러리(예: RevenueCat)는 금지. App Store 정책상 공식 `in_app_purchase` 만 사용해야 한다.

상품 ID(Product ID)는 App Store Connect와 1:1 매핑된다. 테스트 환경에서는 Sandbox 테스트 계정으로 검증한다.

트랜잭션 검증(Receipt validation)은 앱 로컬이 아니라 서버(Cloud Functions)에서 수행한다. 클라이언트에서 `InAppPurchase.instance.queryPastPurchases()`로 구매 이력을 조회한 후, 서버로 전달해서 Apple 백엔드에 검증해야 한다.

구매 흐름:
1. `InAppPurchase.instance.queryProductDetails([productIds])`로 상품 정보 로드
2. `productDetails.purchaseDetails.buy()`로 구매 시작
3. `InAppPurchase.instance.purchaseStream`에서 구매 상태 수신
4. 완료(PurchaseStatus.purchased) → 서버 검증 → 프로덕션 반영

## Restore Purchase 플로우 (App Store 필수)

사용자가 새 기기로 변경했거나 앱을 재설치했을 때, 이전 구매 기록을 복원해야 한다. App Store 심사 가이드라인에서 설정 메뉴에 "구매 복원" 버튼을 반드시 요구한다.

```dart
// 설정 화면에서
await InAppPurchase.instance.restorePurchases();
```

`restorePurchases()` 호출 후 `purchaseStream`에서 복원된 구매 목록이 흐른다. 각 복원 구매에 대해 서버 검증을 다시 수행하고, 프로덕션 데이터(Remove Ads 여부 등)를 업데이트한다.

누락 시 App Store 심사 리젝트. "사용자가 구매를 복원할 수 없습니다" 피드백으로 거절된다.

## 자주 만드는 하드 룰 (프로젝트별 CLAUDE.md에 명시)

프로젝트 특성에 맞게 다음 규칙을 선택해서 프로젝트 `CLAUDE.md` 상단에 명시한다:

- **"Daily/핵심 플로우엔 전면광고 절대 없음"** — 공유/바이럴을 보호하기 위해, 핵심 게임플레이(Daily 완료, 공유 버튼 근처)에는 전면광고를 삽입하지 않는다. 배너 광고만 허용.

- **"진행도 IAP 금지"** — 점수, 꽃잎, 배지, 스트릭 같은 게임 진행도를 결코 IAP로 판매하지 않는다. 오직 비진행 기능(Remove Ads, 스킨, 테마)만 판매.

- **"Remove Ads 구매 시 광고가 진짜 사라짐"** — 구매 후에도 광고를 계속 띄우는 사기 행위는 금지. `purchaseStream`에서 구매 완료를 감지하면, 그 즉시 UI 상태를 업데이트해서 광고 위젯 자체를 제거한다.

이 규칙들은 앱 심사 통과 및 사용자 신뢰 유지에 필수. 프로젝트 시작 시 규칙을 선택해서 명시해 두면, 개발 중 팀 전체가 같은 기준을 따를 수 있다.
