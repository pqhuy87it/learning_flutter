```dart
class AsyncValueNotifier<T> implements ValueListenable<T> {
  /// Default comparison function for [AsyncValueNotifier].
  static bool defaultIsEqual<T>(T a, T b) {
    if (a == b) {
      return true;
    } else if (a is double && b is double) {
      return a.isNaN && b.isNaN;
    }
    return false;
  }

  final _toAdd = <VoidCallback>[];
  final _toRemove = <VoidCallback, int>{};
  var _list = <VoidCallback>[];
  var _weakList = <_ListenerRef>[];
  var _expando = Expando<_ListenerRef>();
  var _pending = false;
  var _dispatching = false;
  var _disposed = false;
  T _value;

  /// Creates a new [AsyncValueNotifier] with the given [value].
  ///
  /// [cancelable] determines whether the notifier should suppress unchanged notifications.
  ///
  /// [distinct] determines whether the notifier should ignore duplicate listeners while triggering.
  AsyncValueNotifier(
    T value, {
    this.isEqual,
    this.distinct = false,
    this.cancelable = false,
    this.weakListener = false,
  }) : _value = value;

  /// Disposes the notifier and (eagerly) clears listeners.
  ///
  /// Safe to call multiple times; subsequent calls are no‑ops. Pending microtasks will observe `_disposed` and avoid notifying.
  void dispose() {
    if (!_disposed) {
      if (!_dispatching) {
        _clearListeners();
      }
      _toAdd.clear();
      _toRemove.clear();
      _disposed = true;
    }
  }

  /// Whether the notifier should use weak references for listeners. If true, listeners that are no longer strongly referenced elsewhere will be garbage collected and automatically removed.
  final bool weakListener;

  /// The comparison function used to determine if two values are equal.
  bool Function(T, T)? isEqual;

  /// Whether the notifier should ignore duplicate listeners while triggering.
  bool distinct;

  /// Whether the notifier should ignore unchanged notifications.
  bool cancelable;

  /// Whether the notifier is currently pending notification.
  bool get pending => _pending;

  /// Whether the notifier is currently dispatching notifications.
  bool get dispatching => _dispatching;

  /// Whether the notifier has been disposed.
  bool get disposed => _disposed;

  /// Returns a list of active listener callbacks.
  List<VoidCallback> get listeners {
    if (weakListener) {
      final it = _weakList.where((ref) => ref.target != null);
      final result =
          List<VoidCallback>.unmodifiable(it.map((ref) => ref.target!));
      if (!_dispatching && result.length != _weakList.length) {
        _weakList = it.toList();
      }
      return result;
    } else {
      return List.unmodifiable(_list);
    }
  }

  /// Assigns a new value.
  ///
  /// Behavior:
  /// * The first *distinct* assignment in a turn schedules a microtask; later assignments in the same turn simply update [value] (only the final value is observed by listeners).
  /// * On microtask execution we re‑check disposal and (if [cancelable]) whether the value reverted; if reverted, we skip notifying.
  /// * [value] is updated *before* scheduling completes, so synchronous reads after the setter see the new value even though listeners have not run.
  set value(T newValue) {
    if (!_disposed && !_isEqual(newValue)) {
      if (!_pending) {
        _pending = true;
        final oldValue = _value;
        // Future.microtask() is not suitable here cause it has own error handling logic.
        scheduleMicrotask(() {
          _pending = false;
          if (!_disposed && (!cancelable || !_isEqual(oldValue))) {
            _dispatching = true;
            final distinctListeners = <VoidCallback>{};
            if (weakListener) {
              var gced = false;
              for (final ref in _weakList) {
                if (_disposed) {
                  break;
                }
                final listener = ref.target;
                if (listener == null) {
                  gced = true;
                } else if (distinctListeners.add(listener) || !distinct) {
                  _runListener(listener);
                }
              }
              _dispatching = false;
              if (_disposed) {
                _clearListeners();
              } else if (!gced && _toRemove.isEmpty) {
                for (final listener in _toAdd) {
                  _addRef(listener, _expando, _weakList);
                }
              } else {
                final newList = <_ListenerRef>[];
                final newExpando = Expando<_ListenerRef>();
                for (final ref in _weakList) {
                  final listener = ref.target;
                  if (listener != null && _needAdd(listener)) {
                    if (newExpando[listener] == null) {
                      newExpando[listener] = ref;
                      ref.count = 1;
                    } else {
                      ref.count++;
                    }
                    newList.add(ref);
                  }
                }
                for (final listener in _toAdd) {
                  if (_needAdd(listener)) {
                    _addRef(listener, newExpando, newList);
                  }
                }
                _toAdd.clear();
                _toRemove.clear();
                _weakList = newList;
                _expando = newExpando;
              }
            } else {
              for (final listener in _list) {
                if (_disposed) {
                  break;
                }
                if (distinctListeners.add(listener) || !distinct) {
                  _runListener(listener);
                }
              }
              _dispatching = false;
              if (_disposed) {
                _clearListeners();
              } else if (_toRemove.isEmpty) {
                for (final listener in _toAdd) {
                  _list.add(listener);
                }
              } else {
                final newList = <VoidCallback>[];
                for (final listener in _list) {
                  if (_needAdd(listener)) {
                    newList.add(listener);
                  }
                }
                for (final listener in _toAdd) {
                  if (_needAdd(listener)) {
                    newList.add(listener);
                  }
                }
                _toAdd.clear();
                _toRemove.clear();
                _list = newList;
              }
            }
          }
        });
      }
      _value = newValue;
    }
  }

  @override
  T get value => _value;

  /// Register a closure to be called when the object notifies its listeners. This operation takes effect while [dispatching] is false.
  @override
  void addListener(VoidCallback listener) => _setListener(listener, true);

  /// Unregisters a closure so it will no longer be called when the object notifies its listeners. This operation takes effect while [dispatching] is false.
  @override
  void removeListener(VoidCallback listener) => _setListener(listener, false);

  @override
  String toString() => '${describeIdentity(this)}($_value)';

  bool _isEqual(T other) =>
      isEqual?.call(_value, other) ?? defaultIsEqual(_value, other);

  void _setListener(VoidCallback listener, bool add) {
    if (!_disposed) {
      if (_dispatching) {
        if (add) {
          _toAdd.add(listener);
        } else {
          _toRemove[listener] = (_toRemove[listener] ?? 0) + 1;
        }
      }
      if (weakListener) {
        if (add) {
          _addRef(listener, _expando, _weakList);
        } else {
          var ref = _expando[listener];
          if (ref != null) {
            _weakList.remove(ref);
            if (--ref.count == 0) {
              _expando[listener] = null;
            }
          }
        }
      } else if (add) {
        _list.add(listener);
      } else {
        _list.remove(listener);
      }
    }
  }

  void _clearListeners() {
    if (weakListener) {
      for (final ref in _weakList) {
        if (ref.target != null) {
          _expando[ref.target!] = null;
        }
      }
      _weakList.clear();
    } else {
      _list.clear();
    }
  }

  void _addRef(
    VoidCallback listener,
    Expando<_ListenerRef> expando,
    List<_ListenerRef> list,
  ) {
    var ref = expando[listener];
    if (ref == null) {
      ref = _ListenerRef(listener);
      expando[listener] = ref;
    }
    ref.count++;
    list.add(ref);
  }

  bool _needAdd(VoidCallback listener) {
    final del = _toRemove[listener];
    if (del != null && del > 0) {
      _toRemove[listener] = del - 1;
      return false;
    }
    return true;
  }

  // Allow subsequent listeners to run even if one throws while preserving uncaught exception propagation for debugging.
  @pragma('vm:notify-debugger-on-exception')
  void _runListener(VoidCallback listener) {
    try {
      listener();
    } catch (e, s) {
      if (!kDebugMode) {
        FlutterError.reportError(FlutterErrorDetails(
          exception: e,
          stack: s,
          library: 'async_value_notifier',
          informationCollector: () => <DiagnosticsNode>[
            DiagnosticsProperty<VoidCallback>('listener', listener),
            DiagnosticsProperty<AsyncValueNotifier>('AsyncValueNotifier', this),
          ],
        ));
      }
    }
  }
}

class _ListenerRef {
  final WeakReference<VoidCallback> _value;
  var count = 0;
  _ListenerRef(VoidCallback listener) : _value = WeakReference(listener);

  VoidCallback? get target => _value.target;
}
```

Tuyệt vời! Đây là một đoạn code rất thú vị và được viết tốt. AsyncValueNotifier là một phiên bản nâng cao, mạnh mẽ hơn của lớp ValueNotifier có sẵn trong Flutter. Hãy cùng phân tích chi tiết.
Tổng quan
AsyncValueNotifier là một lớp quản lý trạng thái triển khai ValueListenable, giống như ValueNotifier. Tuy nhiên, nó được thiết kế để giải quyết một số vấn đề và cung cấp các tính năng nâng cao mà ValueNotifier không có, bao gồm:

Thông báo bất đồng bộ (Asynchronous Notification): Các listener không được gọi ngay lập tức mà được lên lịch để thực thi trong một microtask.
Gộp các cập nhật (Update Batching): Nhiều lần thay đổi giá trị trong cùng một "luồng" (event loop turn) sẽ chỉ kích hoạt một lần thông báo duy nhất.
Hủy thông báo (Notification Cancellation): Có thể cấu hình để hủy thông báo nếu giá trị thay đổi rồi lại quay về giá trị ban đầu trước khi thông báo được gửi đi.
Listener yếu (Weak Listeners): Tùy chọn sử dụng tham chiếu yếu (weak references) để tự động dọn dẹp các listener đã bị thu gom rác (garbage collected), giúp ngăn chặn rò rỉ bộ nhớ.
Listener duy nhất (Distinct Listeners): Đảm bảo mỗi listener chỉ được gọi một lần ngay cả khi nó được thêm vào nhiều lần.
Hàm so sánh tùy chỉnh (Custom Equality Check): Cho phép cung cấp hàm so sánh riêng để quyết định khi nào một giá trị được coi là "mới".

Đây là một công cụ mạnh mẽ dành cho các kịch bản quản lý trạng thái phức tạp hơn.

Phân tích chi tiết các thành phần
1. Cơ chế Bất đồng bộ và Gộp cập nhật
Đây là trái tim của AsyncValueNotifier, nằm trong setter của value:
set value(T newValue) {
  if (!_disposed && !_isEqual(newValue)) {
    if (!_pending) { // Chỉ thực hiện nếu chưa có microtask nào được lên lịch
      _pending = true;
      final oldValue = _value;
      scheduleMicrotask(() { // Lên lịch thông báo
        // ... logic thông báo ...
      });
    }
    _value = newValue; // Cập nhật giá trị ngay lập tức
  }
}


_pending flag: Khi bạn gán giá trị mới lần đầu, _pending sẽ được đặt thành true và một microtask được lên lịch. Nếu bạn tiếp tục gán các giá trị mới khác ngay sau đó (trong cùng một hàm chẳng hạn), điều kiện !_pending sẽ là false, do đó không có microtask mới nào được tạo.
scheduleMicrotask: Đảm bảo rằng việc thông báo cho các listener sẽ chỉ xảy ra sau khi tất cả các đoạn code đồng bộ hiện tại đã chạy xong.
Cập nhật đồng bộ: Điều quan trọng là _value = newValue; được thực thi ngay lập tức. Điều này có nghĩa là nếu bạn đọc notifier.value ngay sau khi gán, bạn sẽ nhận được giá trị mới, mặc dù các listener chưa được thông báo.

Lợi ích: Điều này giúp tránh việc rebuild UI không cần thiết. Ví dụ:
notifier.value = 1; // Lên lịch thông báo với giá trị cuối cùng sẽ là 1
notifier.value = 2; // Không lên lịch mới, chỉ cập nhật _value
notifier.value = 3; // Không lên lịch mới, chỉ cập nhật _value
// Kết thúc hàm. Microtask sẽ chạy và thông báo cho listener với giá trị là 3 (chỉ 1 lần).

2. Quản lý Listener Nâng cao
AsyncValueNotifier xử lý việc thêm/xóa listener một cách rất an toàn, ngay cả khi điều đó xảy ra trong lúc đang thông báo.

_dispatching flag: Cờ này được bật lên true khi notifier bắt đầu vòng lặp để gọi các listener.
_toAdd và _toRemove: Nếu addListener hoặc removeListener được gọi trong khi _dispatching là true (ví dụ, một listener tự hủy chính nó), thao tác sẽ không được thực hiện ngay lập tức trên danh sách listener. Thay vào đó, nó được đưa vào hàng đợi trong _toAdd hoặc _toRemove.
Xử lý hàng đợi: Sau khi vòng lặp thông báo kết thúc, _dispatching được đặt lại thành false, và code sẽ xử lý các listener trong _toAdd và _toRemove để cập nhật lại danh sách listener chính.

Lợi ích: Cách tiếp cận này ngăn ngừa lỗi "concurrent modification" (sửa đổi tập hợp trong khi đang duyệt qua nó), một lỗi phổ biến trong các hệ thống callback.
3. Tùy chọn Cấu hình (cancelable, distinct, weakListener)

cancelable: true: Bên trong microtask, trước khi thông báo, nó sẽ kiểm tra lại: !_isEqual(oldValue). Nếu giá trị hiện tại lại bằng với giá trị cũ (trước khi setter được gọi), thông báo sẽ bị hủy.
// Giả sử giá trị ban đầu là 0 và cancelable = true
notifier.value = 1;
notifier.value = 0; // Quay về giá trị ban đầu
// Microtask chạy, thấy giá trị hiện tại (0) bằng giá trị cũ (0) -> không thông báo.


distinct: true: Khi thông báo, nó sử dụng một Set để theo dõi các listener đã được gọi, đảm bảo mỗi hàm listener chỉ được thực thi một lần.

weakListener: true: Đây là một tính năng tối ưu hóa bộ nhớ rất mạnh mẽ.

Thay vì lưu trực tiếp VoidCallback, nó lưu _ListenerRef, một lớp bao bọc chứa WeakReference<VoidCallback>.
WeakReference: Là một tham chiếu đến một đối tượng nhưng không ngăn trình thu gom rác (Garbage Collector - GC) xóa đối tượng đó.
Expando: Được sử dụng để tạo một "bản đồ" hiệu quả từ một đối tượng listener đến _ListenerRef tương ứng của nó, giúp việc tìm và xóa nhanh hơn.
Lợi ích: Nếu một listener (ví dụ một hàm callback trong một State của StatefulWidget) không còn được tham chiếu ở bất kỳ đâu khác ngoài AsyncValueNotifier, GC có thể thu dọn nó. Lần thông báo tiếp theo, AsyncValueNotifier sẽ thấy tham chiếu này là null và tự động loại bỏ nó khỏi danh sách. Điều này giúp ngăn ngừa memory leak một cách tự động khi bạn quên gọi removeListener.



4. An toàn và Xử lý Lỗi

_disposed flag: Hầu hết mọi hoạt động đều kiểm tra !_disposed trước tiên, đảm bảo notifier không làm gì sau khi đã được dispose().
_runListener: Mỗi listener được gọi bên trong một khối try-catch. Nếu một listener ném ra một ngoại lệ, nó sẽ không làm sập toàn bộ quá trình thông báo. Các listener khác vẫn sẽ được gọi. Trong chế độ không phải debug, lỗi sẽ được báo cáo tới FlutterError.reportError.

Kết luận
AsyncValueNotifier không phải là một sự thay thế đơn giản cho ValueNotifier mà là một công cụ chuyên dụng, hiệu suất cao và an toàn cho các kịch bản phức tạp.

Khi nào nên sử dụng?
Khi bạn thực hiện nhiều thay đổi trạng thái liên tiếp và chỉ muốn UI cập nhật một lần duy nhất.
Khi bạn làm việc với các đối tượng tồn tại lâu dài (long-lived objects) và lo lắng về rò rỉ bộ nhớ từ các listener không được hủy đăng ký.
Khi bạn cần sự linh hoạt trong việc so sánh các giá trị trạng thái (ví dụ với các đối tượng phức tạp).
Khi bạn cần một hệ thống thông báo trạng thái mạnh mẽ và có khả năng phục hồi sau lỗi.



Tóm lại, đây là một lớp được thiết kế kỹ lưỡng, giải quyết nhiều vấn đề thực tế trong quản lý trạng thái của Flutter với hiệu suất và sự an toàn được đặt lên hàng đầu.
