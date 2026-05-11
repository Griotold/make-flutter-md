# Firebase 셋업

## 셋업 순서

```bash
# 1. CLI 로그인
firebase login

# 2. flutterfire CLI로 프로젝트 연결
dart pub global activate flutterfire_cli
flutterfire configure

# 3. 필요한 패키지 추가
flutter pub add firebase_core firebase_auth cloud_firestore
# 필요 시 추가
flutter pub add firebase_storage cloud_functions firebase_remote_config
flutter pub add firebase_messaging firebase_analytics firebase_crashlytics
```

## main.dart 초기화

```dart
await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);
```

`flutterfire configure`가 자동 생성한 `firebase_options.dart`를 import 하면 플랫폼별 설정이 자동으로 로드된다.

## Anonymous Auth → Google upgrade 패턴

앱 시작 시 anonymous 로그인을 자동으로 수행한다. 사용자가 계정 업그레이드를 원할 때만 Google 로그인 credential과 `linkWithCredential()`을 호출하면 uid가 유지된 채로 계정이 업그레이드된다. 데이터 마이그레이션은 불필요하다.

## firestore.rules 안전 기본값

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 기본: 전부 거부
    match /{document=**} {
      allow read, write: if false;
    }

    // 사용자 본인 소유 데이터
    match /users/{uid}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // 공용 리더보드 (읽기만)
    match /leaderboards/{doc} {
      allow read: if true;
      allow write: if false; // Cloud Function에서만 쓰기
    }
  }
}
```

기본적으로 모든 접근을 거부하고, 필요한 경로만 명시적으로 허용한다.

## Cloud Functions onCall 패턴

```ts
export const myFn = onCall({region: 'asia-northeast3'}, async (request) => {
  // request.auth.uid 자동 검증 (auth 없으면 unauthenticated 에러)
  const uid = request.auth?.uid;
  if (!uid) throw new HttpsError('unauthenticated', '');
  // 본 로직
});
```

`onCall()` 함수는 Firebase Auth를 자동으로 검증한다. request.auth를 확인하지 않으면 비인증 사용자도 함수를 호출할 수 있으므로, uid 기반 필터링은 필수다.

## firestore.indexes.json

복합 쿼리(Firestore에서 두 개 이상의 조건으로 필터/정렬)를 추가할 때마다 인덱스를 명시해야 한다. `firebase deploy --only firestore:indexes`로 배포한다. 인덱스가 누락되면 Firestore 에러 메시지에 콘솔 링크가 나온다. 해당 링크를 클릭하면 인덱스를 자동 생성할 수 있다.

## 주요 패키지 조합 (자주 같이 쓰임)

`firebase_core` (필수), `firebase_auth` + `cloud_firestore` (대부분의 앱), `firebase_storage` (이미지/파일 업로드 및 다운로드), `cloud_functions` (서버 로직, onCall 함수), `firebase_remote_config` (피처 토글, A/B 테스트), `firebase_messaging` (푸시 알림), `firebase_analytics` + `firebase_crashlytics` (관측성). 앱의 요구사항에 따라 필요한 패키지만 선택적으로 추가한다.
