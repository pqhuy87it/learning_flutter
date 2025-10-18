Generics (hay kiểu dữ liệu tổng quát) là một trong những tính năng mạnh mẽ nhất của Dart, và bạn chắc chắn sẽ dùng nó hàng ngày trong Flutter.

Nói một cách đơn giản, **Generics cho phép bạn viết code có thể hoạt động với nhiều kiểu dữ liệu khác nhau, nhưng vẫn giữ được sự an toàn về kiểu (type-safety)**.

Ký hiệu của Generics là cặp dấu ngoặc nhọn: `<T>`. Chữ `T` ở đây là một "biến giữ chỗ" (placeholder) cho một kiểu dữ liệu sẽ được xác định sau.

-----

## 1\. Vấn đề mà Generics giải quyết

Để hiểu tại sao Generics quan trọng, hãy xem điều gì xảy ra khi *không* có nó.

Giả sử bạn muốn tạo một lớp `Cache` (bộ đệm) đơn giản để lưu trữ *một* giá trị.

**Cách 1: Dùng `dynamic` (Không an toàn)**
Bạn có thể dùng `dynamic` để chấp nhận mọi kiểu:

```dart
class DynamicCache {
  dynamic value;
  
  void setValue(dynamic val) {
    value = val;
  }
}

// Cách dùng
var cache = DynamicCache();
cache.setValue('Hello'); // Gán String

// LỖI: Trình biên dịch không biết 'value' là String
// Lỗi này chỉ bị phát hiện khi chạy (runtime error)
int myValue = cache.value; // Crash!
```

Vấn đề: Bạn mất hoàn toàn *type safety*. Trình biên dịch không thể giúp bạn, và ứng dụng sẽ bị crash lúc chạy.

**Cách 2: Viết nhiều lớp (Không tái sử dụng)**
Bạn có thể viết một lớp riêng cho mỗi kiểu:

```dart
class StringCache {
  String value = '';
  void setValue(String val) { value = val; }
}

class IntCache {
  int value = 0;
  void setValue(int val) { value = val; }
}
// Phải viết thêm UserCache, ProductCache...
```

Vấn đề: Code này an toàn, nhưng bạn phải lặp lại code (boilerplate) rất nhiều.

-----

## 2\. Giải pháp: Dùng Generics

Với Generics, bạn tạo một "khuôn mẫu". Bạn nói: "Tôi muốn một lớp `Cache` hoạt động với một kiểu `T` nào đó. `T` là gì thì lúc sử dụng tôi sẽ báo sau."

```dart
// 1. ĐỊNH NGHĨA lớp với Generics <T>
class Cache<T> {
  T? value; // Dùng T như một kiểu dữ liệu

  void setValue(T val) {
    value = val;
  }
}

// 2. SỬ DỤNG lớp và chỉ định kiểu cụ thể
void main() {
  // Tạo một Cache chỉ dành cho String
  var stringCache = Cache<String>();
  stringCache.setValue('Hello Flutter');

  // Lấy giá trị ra, Dart biết chắc chắn nó là String
  String myString = stringCache.value!;
  print(myString.toUpperCase()); // An toàn!

  // LỖI: Trình biên dịch sẽ báo lỗi ngay lập tức
  // stringCache.setValue(123); // Lỗi: The argument type 'int' can't be assigned to 'String'.

  // ---

  // Tái sử dụng cùng lớp Cache đó cho kiểu int
  var intCache = Cache<int>();
  intCache.setValue(100);

  // An toàn!
  int myInt = intCache.value!;
  print(myInt + 50); 
}
```

Đây chính là sức mạnh của Generics: **An toàn kiểu (Type Safety) + Tái sử dụng code (Reusability)**.

-----

## 3\. Các cách sử dụng Generics trong Flutter

### a. Trên Lớp (Classes) - Như ví dụ trên

Đây là cách phổ biến nhất. Bạn thấy nó ở khắp mọi nơi trong Flutter:

  * `StatefulWidget` và `State<T>`: `class MyScreen extends StatefulWidget { ... }` và `class _MyScreenState extends State<MyScreen> { ... }`.
  * `FutureBuilder<T>`: Cho Flutter biết kiểu dữ liệu mà `future` sẽ trả về (ví dụ: `FutureBuilder<String>`).
  * `StreamBuilder<T>`: Tương tự, cho biết kiểu dữ liệu của `stream`.

### b. Trên Bộ sưu tập (Collections) - Rất quan trọng\!

Bạn đã dùng cái này rồi, ngay cả khi bạn không nhận ra:

  * `List<E>`: (E là viết tắt của Element).
      * `List<String> names = ['Alice', 'Bob'];` (Một danh sách chỉ chứa `String`).
      * `List<int> numbers = [1, 2, 3];` (Một danh sách chỉ chứa `int`).
  * `Map<K, V>`: (K là Key, V là Value).
      * `Map<String, int> scores = {'Alice': 100, 'Bob': 80};` (Map có key là `String`, value là `int`).
  * `Set<E>`:
      * `Set<String> tags = {'flutter', 'dart'};` (Một tập hợp chỉ chứa `String`).

### c. Trên Phương thức (Methods/Functions)

Bạn cũng có thể tạo các hàm "tổng quát".

```dart
// Một hàm nhận vào một List kiểu <T>
// và trả về phần tử đầu tiên cũng có kiểu <T>
T? getFirstElement<T>(List<T> list) {
  if (list.isEmpty) {
    return null;
  }
  return list[0];
}

void testMethod() {
  List<String> names = ['Alice', 'Bob'];
  List<int> numbers = [10, 20];

  // Trình biên dịch tự động suy luận T là String
  String? first_name = getFirstElement(names); // Kiểu trả về là String?

  // Trình biên dịch tự động suy luận T là int
  int? first_number = getFirstElement(numbers); // Kiểu trả về là int?
}
```

-----

## 4\. Giới hạn Kiểu (Type Constraints)

Đôi khi, bạn muốn `T` không phải là *bất kỳ* kiểu nào, mà phải là một kiểu *cụ thể* (ví dụ: một kiểu phải là `num`, hoặc một kiểu phải có hàm `.compareTo()`).

Chúng ta dùng từ khóa `extends` để giới hạn `T`.

**Ví dụ:** Bạn muốn viết một hàm tính tổng một danh sách, nhưng hàm này chỉ nên áp dụng cho `int` hoặc `double` (kiểu `num`).

```dart
// T phải là một kiểu kế thừa từ 'num'
T sum<T extends num>(List<T> list) {
  num total = 0; // Bắt đầu bằng 0
  for (var item in list) {
    total = total + item; // An toàn, vì T chắc chắn là 'num'
  }
  return total as T;
}

void main() {
  var intTotal = sum([1, 2, 3]);       // Hoạt động
  var doubleTotal = sum([1.5, 2.5]); // Hoạt động

  // LỖI: Trình biên dịch sẽ báo lỗi
  // vì 'String' không 'extends num'
  // var stringTotal = sum(['a', 'b']); 
}
```

## Tổng kết lợi ích

1.  🔒 **An toàn kiểu (Type Safety):** Phát hiện lỗi kiểu dữ liệu ngay lúc viết code (compile-time), chứ không phải lúc ứng dụng đang chạy (runtime).
2.  ♻️ **Tái sử dụng code (Code Reusability):** Viết một lớp hoặc phương thức (`Cache<T>`) và dùng nó cho nhiều kiểu (`Cache<String>`, `Cache<int>`, `Cache<User>`).
3.  ✨ **Rõ ràng (Clarity):** Code dễ đọc hơn nhiều. `List<Product>` ngay lập tức cho bạn biết danh sách này chứa gì, thay vì `List` (thực chất là `List<dynamic>`).
