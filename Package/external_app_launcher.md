`external_app_launcher` là một package Flutter giúp bạn thực hiện hai việc chính:

1.  **Kiểm tra** xem một ứng dụng khác đã được cài đặt trên thiết bị hay chưa.
2.  **Mở** ứng dụng đó. Nếu ứng dụng chưa được cài đặt, nó có thể tự động chuyển hướng người dùng đến App Store (trên iOS) hoặc Play Store (trên Android).

Đây là hướng dẫn chi tiết cách cài đặt, cấu hình và sử dụng package này.

-----

## 1\. Cài đặt

Thêm `external_app_launcher` vào file `pubspec.yaml` của bạn:

```yaml
dependencies:
  flutter:
    sdk: flutter
  external_app_launcher: ^4.0.3 # Kiểm tra pub.dev để có phiên bản mới nhất
```

Sau đó, chạy lệnh `flutter pub get` trong terminal của bạn.

-----

## 2\. Cấu hình (Rất quan trọng)

Bạn **bắt buộc** phải cấu hình cho cả Android và iOS để package này hoạt động.

### a. Cấu hình cho Android (Package Name)

Kể từ Android 11 (API 30), bạn phải khai báo các ứng dụng mà bạn muốn "nhìn thấy" (query). Nếu không, Android sẽ mặc định là không tìm thấy ứng dụng nào, ngay cả khi nó đã được cài đặt.

Mở file: `android/app/src/main/AndroidManifest.xml`

Thêm khối `<queries>` vào bên trong thẻ `<manifest>` (ngang hàng với thẻ `<application>`):

```xml
<manifest ...>
    <application ...>
        </application>
    
    <queries>
        <package android:name="com.facebook.katana" /> <package android:name="com.instagram.android" /> <package android:name="com.twitter.android" /> </queries>
</manifest>
```

Bạn cần tìm **Package Name** (ID ứng dụng) của ứng dụng Android mà bạn muốn mở.

### b. Cấu hình cho iOS (URL Scheme)

iOS sử dụng "URL Schemes" để các ứng dụng gọi lẫn nhau. Bạn cũng phải khai báo các scheme này trong file `Info.plist`.

Mở file: `ios/Runner/Info.plist`

Thêm khối `LSApplicationQueriesSchemes` vào bên trong `<dict>` chính:

```xml
<dict>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>fb</string> <string>instagram</string> <string>twitter</string> </array>
    
    </dict>
```

Bạn cần tìm **URL Scheme** của ứng dụng iOS mà bạn muốn mở.

-----

## 3\. Cách sử dụng

Tất cả các hàm đều nằm trong class `LaunchApp`. Bạn chỉ cần import package:

```dart
import 'package:external_app_launcher/external_app_launcher.dart';
```

### a. Hàm `isAppInstalled` (Kiểm tra cài đặt)

Hàm này trả về `true` nếu ứng dụng được tìm thấy, và `false` nếu không (hoặc nếu bạn cấu hình sai).

```dart
// Ví dụ kiểm tra Instagram
bool isInstalled = await LaunchApp.isAppInstalled(
    androidPackageName: 'com.instagram.android',
    iosUrlScheme: 'instagram://', // Thường là tên ứng dụng + ://
);

if (isInstalled) {
    print('Instagram đã được cài đặt.');
} else {
    print('Instagram CHƯA được cài đặt.');
}
```

### b. Hàm `launchApp` (Mở ứng dụng)

Đây là hàm chính để mở ứng dụng. Nếu ứng dụng không được cài đặt, nó sẽ mở App Store/Play Store.

Hàm này nhận các tham số:

  * `androidPackageName`: (Bắt buộc) ID package của ứng dụng Android.
  * `iosUrlScheme`: (Bắt buộc) URL Scheme của ứng dụng iOS.
  * `appStoreLink`: (Bắt buộc) Link App Store của ứng dụng (lấy từ trình duyệt).
  * `openStore`: (Tùy chọn, mặc định là `true`)
      * Nếu `true`: Tự động mở store nếu không tìm thấy ứng dụng.
      * Nếu `false`: Không làm gì cả nếu không tìm thấy ứng dụng.

<!-- end list -->

```dart
// Ví dụ mở ứng dụng Instagram
await LaunchApp.launchApp(
    androidPackageName: 'com.instagram.android',
    iosUrlScheme: 'instagram://',
    appStoreLink: 'itms-apps://itunes.apple.com/app/instagram/id389801252',
    // (Bạn có thể bỏ qua openStore nếu muốn giữ mặc định là true)
    // openStore: true 
);
```

### c. Hàm `openAppStore` (Chỉ mở Store)

Nếu bạn chỉ muốn mở trang của ứng dụng trên store mà không cần kiểm tra, hãy dùng hàm này.

```dart
// Mở trang Instagram trên store tương ứng
await LaunchApp.openAppStore(
    androidPackageName: 'com.instagram.android',
    appStoreLink: 'itms-apps://itunes.apple.com/app/instagram/id389801252',
);
```

-----

## Ví dụ đầy đủ trong một `ElevatedButton`

Đây là một ví dụ phổ biến: Nhấn nút để mở Facebook. Nếu chưa cài, nó sẽ tự động mở store.

```dart
import 'package:flutter/material.dart';
import 'package:external_app_launcher/external_app_launcher.dart';

class MyAppLauncher extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Mở Ứng Dụng Khác'),
      ),
      body: Center(
        child: ElevatedButton(
          child: Text('Mở Facebook'),
          onPressed: () async {
            // Mở Facebook, nếu không có thì mở store
            await LaunchApp.launchApp(
              androidPackageName: 'com.facebook.katana',
              iosUrlScheme: 'fb://',
              appStoreLink: 'itms-apps://itunes.apple.com/app/facebook/id284882215',
            );
          },
        ),
      ),
    );
  }
}
```

### 💡 Mẹo tìm `Package Name` và `URL Scheme`:

  * **Android (Package Name):** Mở trang của ứng dụng trên Google Play Store trên trình duyệt. URL sẽ có dạng:
    `https://play.google.com/store/apps/details?id=`**`com.facebook.katana`**
    Phần `id=` chính là package name.
  * **iOS (URL Scheme):** Cái này khó tìm hơn. Bạn thường phải Google "tên ứng dụng + url scheme" (ví dụ: "instagram url scheme").
  * **iOS (App Store Link):** Tìm ứng dụng trên App Store, nhấn "Share" và "Copy Link". Sửa phần `https://` ở đầu link thành `itms-apps://`.
