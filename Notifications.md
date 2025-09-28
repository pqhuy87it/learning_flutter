Chào bạn, rất vui được giải thích chi tiết về cách sử dụng **Notifications** (Thông báo) trong Flutter. Đây là một chủ đề lớn và cực kỳ quan trọng, vì nó là cách chính để ứng dụng của bạn tương tác với người dùng ngay cả khi họ không mở ứng dụng.

Chúng ta sẽ chia chủ đề này thành 2 phần chính và rõ ràng:

1.  **Local Notifications (Thông báo cục bộ):** Được kích hoạt và quản lý **hoàn toàn bên trong ứng dụng của bạn**. Rất lý tưởng cho các tính năng như báo thức, nhắc nhở, thông báo khi một tác vụ nền hoàn thành.
2.  **Push Notifications (Thông báo đẩy):** Được gửi từ một **máy chủ từ xa (server)** đến ứng dụng của bạn. Dùng cho tin nhắn mới, thông báo khuyến mãi, cập nhật từ mạng xã hội, v.v.

---

### Phần 1: Local Notifications (Thông báo cục bộ)

Đây là điểm khởi đầu tốt nhất. Chúng ta sẽ sử dụng package phổ biến và mạnh mẽ nhất là `flutter_local_notifications`.

#### Bước 1: Cài đặt và Cấu hình

1.  **Thêm dependency:**
    Mở file `pubspec.yaml` và thêm vào:
    ```yaml
    dependencies:
      flutter_local_notifications: ^17.0.0 # Luôn kiểm tra phiên bản mới nhất
      timezone: ^0.9.3
      flutter_native_timezone_updated_gradle: ^1.0.1
    ```
    Chạy `flutter pub get`.

2.  **Cấu hình cho Android:**
    *   **Icon:** Đặt một icon thông báo mặc định tên là `app_icon.png` (hoặc tên khác) vào thư mục `android/app/src/main/res/drawable/`. Icon này phải là màu trắng trên nền trong suốt.
    *   **Permissions (Android 13+):** Mở `android/app/src/main/AndroidManifest.xml` và thêm quyền này vào bên trong thẻ `<manifest>`:
        ```xml
        <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
        ```

3.  **Cấu hình cho iOS:**
    Mở `ios/Runner/AppDelegate.swift` và thêm đoạn code sau vào bên trong class `AppDelegate` để cho phép thông báo hiển thị khi ứng dụng đang chạy ở foreground:
    ```swift
    import UIKit
    import Flutter

    @UIApplicationMain
    @objc class AppDelegate: FlutterAppDelegate {
      override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
      ) -> Bool {
        // Thêm dòng này
        if #available(iOS 10.0, *) {
          UNUserNotificationCenter.current().delegate = self as? UNUserNotificationCenterDelegate
        }
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
      }
    }
    ```

#### Bước 2: Khởi tạo Dịch vụ Thông báo

Tạo một class riêng để quản lý tất cả logic về thông báo là một good practice.

**File: `lib/notification_service.dart`**
```dart
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/data/latest.dart' as tz;
import 'package:timezone/timezone.dart' as tz;

class NotificationService {
  // Singleton pattern
  static final NotificationService _notificationService = NotificationService._internal();
  factory NotificationService() {
    return _notificationService;
  }
  NotificationService._internal();

  final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
      FlutterLocalNotificationsPlugin();

  Future<void> init() async {
    // 1. Cài đặt cho Android
    const AndroidInitializationSettings initializationSettingsAndroid =
        AndroidInitializationSettings('app_icon'); // Tên file icon đã thêm ở bước 1

    // 2. Cài đặt cho iOS
    final DarwinInitializationSettings initializationSettingsIOS =
        DarwinInitializationSettings(
      onDidReceiveLocalNotification: onDidReceiveLocalNotification,
    );

    // 3. Gói cài đặt chung
    final InitializationSettings initializationSettings = InitializationSettings(
      android: initializationSettingsAndroid,
      iOS: initializationSettingsIOS,
    );

    // Khởi tạo timezone
    tz.initializeTimeZones();

    // 4. Khởi tạo plugin
    await flutterLocalNotificationsPlugin.initialize(
      initializationSettings,
      // Xử lý khi người dùng nhấn vào thông báo (khi app không bị kill)
      onDidReceiveNotificationResponse: onDidReceiveNotificationResponse,
    );

    // Yêu cầu quyền trên iOS
    await flutterLocalNotificationsPlugin
        .resolvePlatformSpecificImplementation<IOSFlutterLocalNotificationsPlugin>()
        ?.requestPermissions(
          alert: true,
          badge: true,
          sound: true,
        );
    
    // Yêu cầu quyền trên Android 13+
     await flutterLocalNotificationsPlugin
        .resolvePlatformSpecificImplementation<AndroidFlutterLocalNotificationsPlugin>()
        ?.requestNotificationsPermission();
  }

  // Callback khi nhận notification trên iOS đời cũ
  void onDidReceiveLocalNotification(
      int id, String? title, String? body, String? payload) async {
    // Hiển thị dialog hoặc làm gì đó
  }

  // Callback khi người dùng nhấn vào notification
  void onDidReceiveNotificationResponse(NotificationResponse response) async {
    final String? payload = response.payload;
    if (payload != null) {
      print('NOTIFICATION PAYLOAD: $payload');
      // Ở đây, bạn có thể điều hướng đến một màn hình cụ thể
      // Dùng một global navigator key hoặc một state management solution
    }
  }

  // Hàm để hiển thị một thông báo đơn giản
  Future<void> showNotification(int id, String title, String body, String payload) async {
    const AndroidNotificationDetails androidPlatformChannelSpecifics =
        AndroidNotificationDetails(
      'main_channel', // id của channel
      'Main Channel', // tên của channel
      channelDescription: 'Kênh thông báo chính',
      importance: Importance.max,
      priority: Priority.high,
      ticker: 'ticker',
    );
    const DarwinNotificationDetails iOSPlatformChannelSpecifics =
        DarwinNotificationDetails();
    const NotificationDetails platformChannelSpecifics = NotificationDetails(
        android: androidPlatformChannelSpecifics, iOS: iOSPlatformChannelSpecifics);

    await flutterLocalNotificationsPlugin.show(
      id,
      title,
      body,
      platformChannelSpecifics,
      payload: payload,
    );
  }

  // Hàm để đặt lịch thông báo
  Future<void> scheduleNotification(int id, String title, String body, DateTime scheduledTime) async {
    await flutterLocalNotificationsPlugin.zonedSchedule(
      id,
      title,
      body,
      tz.TZDateTime.from(scheduledTime, tz.local),
      const NotificationDetails(
        android: AndroidNotificationDetails(
          'scheduled_channel',
          'Scheduled Channel',
          channelDescription: 'Kênh thông báo hẹn giờ',
        ),
      ),
      androidAllowWhileIdle: true,
      uiLocalNotificationDateInterpretation: UILocalNotificationDateInterpretation.absoluteTime,
      payload: 'Scheduled Payload'
    );
  }

   Future<void> cancelNotification(int id) async {
    await flutterLocalNotificationsPlugin.cancel(id);
  }
}
```

#### Bước 3: Sử dụng Dịch vụ

1.  **Khởi tạo trong `main.dart`:**
    ```dart
    Future<void> main() async {
      WidgetsFlutterBinding.ensureInitialized();
      // Khởi tạo notification service
      await NotificationService().init(); 
      runApp(MyApp());
    }
    ```

2.  **Gọi từ giao diện người dùng:**
    ```dart
    class HomePage extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: Text('Notification Demo')),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    NotificationService().showNotification(
                      0,
                      'Thông báo tức thì',
                      'Đây là nội dung của thông báo.',
                      'instant_payload', // Dữ liệu đính kèm
                    );
                  },
                  child: Text('Hiển thị thông báo ngay'),
                ),
                ElevatedButton(
                  onPressed: () {
                    NotificationService().scheduleNotification(
                      1,
                      'Thông báo hẹn giờ',
                      'Thông báo này sẽ xuất hiện sau 5 giây.',
                      DateTime.now().add(const Duration(seconds: 5)),
                    );
                  },
                  child: Text('Hẹn giờ thông báo sau 5s'),
                ),
              ],
            ),
          ),
        );
      }
    }
    ```

---

### Phần 2: Push Notifications (Thông báo đẩy)

Push Notifications phức tạp hơn nhiều vì nó liên quan đến một dịch vụ bên ngoài. Giải pháp phổ biến và được khuyến nghị nhất là **Firebase Cloud Messaging (FCM)**.

#### Luồng hoạt động của Push Notification:

1.  **Cài đặt:** Ứng dụng của bạn tích hợp SDK của Firebase.
2.  **Đăng ký:** Khi người dùng cài đặt và mở ứng dụng, ứng dụng sẽ yêu cầu một "FCM token" từ máy chủ Firebase. Token này là một địa chỉ duy nhất cho cặp (ứng dụng + thiết bị).
3.  **Gửi Token lên Server:** Ứng dụng của bạn gửi FCM token này lên **máy chủ backend của bạn** và lưu nó lại, thường là liên kết với một tài khoản người dùng.
4.  **Kích hoạt:** Một sự kiện xảy ra trên server của bạn (ví dụ: người dùng A gửi tin nhắn cho người dùng B).
5.  **Gửi yêu cầu đến FCM:** Server của bạn lấy FCM token của người dùng B từ database, sau đó tạo một yêu cầu chứa nội dung thông báo và gửi nó đến máy chủ FCM của Google/Firebase.
6.  **Đẩy về thiết bị:** Máy chủ FCM sử dụng token để tìm thiết bị của người dùng B và đẩy thông báo xuống thiết bị đó thông qua các dịch vụ của hệ điều hành (APNs cho iOS, dịch vụ của Google cho Android).
7.  **Hiển thị:** Ứng dụng của bạn nhận được thông báo và hiển thị nó cho người dùng.

#### Các bước triển khai cơ bản với FCM:

1.  **Tạo dự án Firebase:** Lên trang web Firebase, tạo một dự án mới, và đăng ký ứng dụng Flutter của bạn cho cả Android và iOS.
2.  **Thêm Firebase vào Flutter:** Sử dụng FlutterFire CLI để tự động cấu hình. Chạy lệnh: `flutterfire configure`.
3.  **Thêm Dependencies:**
    ```yaml
    dependencies:
      firebase_core: ^2.27.1
      firebase_messaging: ^14.7.20
    ```
4.  **Viết Code trong Flutter:**
    *   Khởi tạo Firebase trong `main.dart`: `await Firebase.initializeApp(...)`.
    *   Yêu cầu quyền nhận thông báo từ người dùng.
    *   Lấy FCM token: `final fcmToken = await FirebaseMessaging.instance.getToken();`. Gửi token này lên server của bạn.
    *   Lắng nghe các thông báo đến:
        *   `FirebaseMessaging.onMessage.listen(...)`: Xử lý khi nhận thông báo lúc ứng dụng đang ở **foreground**.
        *   `FirebaseMessaging.onMessageOpenedApp.listen(...)`: Xử lý khi người dùng nhấn vào thông báo để mở ứng dụng từ trạng thái **background**.
        *   `FirebaseMessaging.instance.getInitialMessage()`: Xử lý khi người dùng nhấn vào thông báo để mở ứng dụng từ trạng thái **terminated** (bị kill).

**Lưu ý:** Để hiển thị Push Notification, bạn vẫn cần sử dụng `flutter_local_notifications` khi ứng dụng đang ở foreground trên Android, vì FCM mặc định không tự hiển thị.

### Tóm tắt: Chọn loại nào?

| Tính năng | Local Notifications | Push Notifications |
| :--- | :--- | :--- |
| **Nguồn gốc** | Từ bên trong ứng dụng | Từ một server bên ngoài |
| **Kết nối mạng** | Không cần | Bắt buộc |
| **Mục đích** | Báo thức, nhắc nhở, countdown, tác vụ nền | Tin nhắn, tin tức, cập nhật real-time, marketing |
| **Độ phức tạp** | Tương đối đơn giản | Phức tạp (cần backend, Firebase) |
| **Package chính** | `flutter_local_notifications` | `firebase_messaging` + `flutter_local_notifications` |

Hy vọng giải thích chi tiết này sẽ giúp bạn nắm vững cách triển khai thông báo trong ứng dụng Flutter của mình
