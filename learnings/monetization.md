# 수익화 (광고 + IAP)

## 광고 — Google Mobile Ads (AdMob)

패키지: `google_mobile_ads` (공식). 테스트 ID와 프로덕션 ID 분리 필수 (계정 정지 방지).

```dart
// dev/test
const testInterstitial = 'ca-app-pub-3940256099942544/4411468910'; // Google 공식 테스트 ID
// prod (Remote Config 또는 .env에서)
final adUnitId = kDebugMode ? testInterstitial : prodId;
```

전면광고(Interstitial)는 사용자 동작 후만 띄운다. 배너 광고는 하단 고정, SafeArea 위 배치로 조작 방해 방지.

초기화: `main()`에서 `MobileAds.instance.initialize()` 호출 (사용자 동의 처리 후).

## 한국 IDFA (App Tracking Transparency)

iOS 14.5+ IDFA 접근 시 사용자 동의 필수. `app_tracking_transparency` 패키지로 다이얼로그 표시.

앱 첫 실행 시 **광고 초기화 전에** 호출:

```dart
final status = await AppTrackingTransparency.requestTrackingAuthorization();
// status: denied, restricted, provisional, authorized 중 하나
// authorized인 경우에만 IDFA 사용 가능
```

`Info.plist` 필수:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>더 나은 광고 경험을 위해 기기 추적 권한이 필요합니다.</string>
```

권한 거부 시에도 광고는 표시되지만 타게팅 정확도 감소.

## IAP — `in_app_purchase` 공식 패키지

공식 `in_app_purchase`만 사용 (RevenueCat 등 서드파티 금지).

상품 ID는 App Store Connect 1:1 매핑. Sandbox 테스트 계정으로 검증.

트랜잭션 검증은 서버(Cloud Functions)에서 수행. 클라이언트에서 `queryPastPurchases()` → 서버에서 Apple 백엔드 검증.

구매 흐름:
1. `InAppPurchase.instance.queryProductDetails([productIds])`로 상품 정보 로드
2. `productDetails.purchaseDetails.buy()`로 구매 시작
3. `InAppPurchase.instance.purchaseStream`에서 구매 상태 수신
4. 완료(PurchaseStatus.purchased) → 서버 검증 → 프로덕션 반영

## Restore Purchase 플로우 (App Store 필수)

새 기기 또는 재설치 시 이전 구매 복원 필수. App Store 심사 요구사항.

```dart
// 설정 화면에서
await InAppPurchase.instance.restorePurchases();
```

`purchaseStream`에서 복원된 구매 수신 → 서버 검증 → 프로덕션 데이터 업데이트.

누락 시 App Store 심사 거절.

## 자주 만드는 하드 룰 (프로젝트별 CLAUDE.md에 명시)

프로젝트 특성에 맞게 규칙 선택 후 `CLAUDE.md` 상단에 명시:

- **"Daily/핵심 플로우엔 전면광고 절대 없음"** — 핵심 게임플레이(Daily 완료, 공유 버튼)에 전면광고 금지. 배너 광고만 허용.

- **"진행도 IAP 금지"** — 점수, 배지, 스트릭 등 진행도는 IAP 판매 금지. 오직 Remove Ads, 스킨, 테마만 판매.

- **"Remove Ads 구매 시 광고가 진짜 사라짐"** — 구매 후 광고 계속 표시 금지. `purchaseStream`에서 완료 감지 후 즉시 광고 위젯 제거.
