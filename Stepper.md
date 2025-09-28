Chào bạn, rất vui được giải thích chi tiết về widget `Stepper` trong Flutter. Đây là một widget rất hữu ích của Material Design, giúp hướng dẫn người dùng qua một quy trình gồm nhiều bước một cách tuần tự.

### 1. Stepper là gì?

`Stepper` là một widget hiển thị tiến trình thông qua một chuỗi các bước (steps). Nó thường được sử dụng trong các tác vụ như:

*   **Quy trình đăng ký tài khoản nhiều bước.**
*   **Quá trình thanh toán trong ứng dụng thương mại điện tử.**
*   **Điền một biểu mẫu phức tạp được chia thành nhiều phần.**
*   **Hướng dẫn cài đặt hoặc onboarding cho người dùng mới.**

`Stepper` cung cấp cho người dùng một cái nhìn tổng quan về tiến trình của họ, cho họ biết họ đang ở bước nào, đã hoàn thành những bước nào, và còn bao nhiêu bước nữa.

### 2. Các thành phần chính

Một `Stepper` được cấu thành từ một danh sách các widget `Step`. Mỗi `Step` lại có các thuộc tính quan trọng sau:

*   `title`: (Bắt buộc) Một widget (thường là `Text`) để hiển thị tiêu đề của bước.
*   `subtitle`: (Tùy chọn) Một widget (thường là `Text`) để hiển thị thông tin phụ bên dưới tiêu đề.
*   `content`: (Bắt buộc) Widget chứa nội dung chính của bước đó, ví dụ như các `TextField`, `Checkbox`, v.v. Nội dung này chỉ hiển thị khi bước đó đang hoạt động (`isActive`).
*   `state`: Trạng thái của bước, quyết định cách nó được hiển thị. Có các loại `StepState`:
    *   `StepState.indexed`: Trạng thái mặc định, hiển thị số thứ tự của bước.
    *   `StepState.editing`: Thường hiển thị một icon bút chì, cho biết người dùng đang ở bước này. Trạng thái này được ngầm định khi `isActive` là true.
    *   `StepState.complete`: Hiển thị một icon checkmark (dấu tích), cho biết bước đã hoàn thành.
    *   `StepState.disabled`: Hiển thị bước bị mờ đi, cho biết người dùng chưa thể tương tác.
    *   `StepState.error`: Hiển thị một icon lỗi (dấu chấm than), cho biết có lỗi xảy ra ở bước này.
*   `isActive`: Một giá trị `bool` xác định xem bước này có đang là bước hiện tại hay không. Nếu `true`, `content` của nó sẽ được hiển thị.

### 3. Các thuộc tính quan trọng của `Stepper`

*   `steps`: (Bắt buộc) Một danh sách các đối tượng `Step` (`List<Step>`).
*   `currentStep`: (Bắt buộc) Một số nguyên (`int`) chỉ định chỉ số (index) của bước hiện tại. **Đây là thuộc tính cốt lõi để quản lý trạng thái của Stepper.**
*   `type`: Loại `Stepper`. Có hai loại:
    *   `StepperType.vertical`: Các bước được sắp xếp theo chiều dọc (mặc định).
    *   `StepperType.horizontal`: Các bước được sắp xếp theo chiều ngang.
*   `onStepTapped`: Một hàm callback được gọi khi người dùng nhấn vào tiêu đề của một bước. Thường dùng để cho phép người dùng quay lại các bước trước đó.
*   `onStepContinue`: Một hàm callback được gọi khi người dùng nhấn nút "Continue". Đây là nơi bạn sẽ xử lý logic để chuyển sang bước tiếp theo (thường là tăng giá trị `currentStep`).
*   `onStepCancel`: Một hàm callback được gọi khi người dùng nhấn nút "Cancel". Đây là nơi bạn xử lý logic để quay lại bước trước đó (thường là giảm giá trị `currentStep`).
*   `controlsBuilder`: Một hàm builder cho phép bạn tùy chỉnh hoàn toàn các nút điều khiển (Continue, Cancel). Rất hữu ích khi bạn muốn thay đổi văn bản, icon, hoặc layout của các nút này.

### 4. Quản lý trạng thái (State Management)

`Stepper` là một widget "không trạng thái" (stateless) theo nghĩa là nó không tự quản lý bước hiện tại. **Bạn phải quản lý `currentStep` trong một `StatefulWidget`**.

Luồng hoạt động cơ bản như sau:
1.  Khai báo một biến `int _currentStep = 0;` trong `State` của bạn.
2.  Truyền biến này vào thuộc tính `currentStep` của `Stepper`.
3.  Trong `onStepContinue`, gọi `setState` để tăng `_currentStep`.
4.  Trong `onStepCancel`, gọi `setState` để giảm `_currentStep`.
5.  Khi `setState` được gọi, widget sẽ được xây dựng lại với giá trị `currentStep` mới, và `Stepper` sẽ cập nhật giao diện tương ứng.

### 5. Ví dụ chi tiết: Form đăng ký 3 bước

Đây là một ví dụ hoàn chỉnh về một form đăng ký sử dụng `Stepper` dọc.

```dart
import 'package:flutter/material.dart';

class StepperDemo extends StatefulWidget {
  @override
  _StepperDemoState createState() => _StepperDemoState();
}

class _StepperDemoState extends State<StepperDemo> {
  // Biến quản lý bước hiện tại
  int _currentStep = 0;

  // GlobalKeys để validate các Form
  final _formKeys = [
    GlobalKey<FormState>(),
    GlobalKey<FormState>(),
    GlobalKey<FormState>()
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Flutter Stepper Demo'),
      ),
      body: Stepper(
        // Loại Stepper: vertical hoặc horizontal
        type: StepperType.vertical,
        // Bước hiện tại
        currentStep: _currentStep,
        // Callback khi nhấn vào một bước
        onStepTapped: (step) => setState(() => _currentStep = step),
        // Callback khi nhấn nút Continue
        onStepContinue: () {
          // Validate form của bước hiện tại
          if (_formKeys[_currentStep].currentState!.validate()) {
            // Nếu chưa phải bước cuối cùng
            if (_currentStep < _getSteps().length - 1) {
              setState(() {
                _currentStep += 1;
              });
            } else {
              // Đây là bước cuối cùng, xử lý hoàn tất
              print('Hoàn thành tất cả các bước!');
              // Ví dụ: gửi dữ liệu lên server
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Đăng ký thành công!')),
              );
            }
          }
        },
        // Callback khi nhấn nút Cancel
        onStepCancel: () {
          if (_currentStep > 0) {
            setState(() {
              _currentStep -= 1;
            });
          }
        },
        // Danh sách các bước
        steps: _getSteps(),
        // Tùy chỉnh các nút điều khiển
        controlsBuilder: (BuildContext context, ControlsDetails details) {
          return Padding(
            padding: const EdgeInsets.only(top: 16.0),
            child: Row(
              children: <Widget>[
                ElevatedButton(
                  onPressed: details.onStepContinue,
                  child: Text(_currentStep == _getSteps().length - 1 ? 'XÁC NHẬN' : 'TIẾP TỤC'),
                ),
                // Chỉ hiển thị nút 'QUAY LẠI' nếu không phải là bước đầu tiên
                if (_currentStep != 0)
                  TextButton(
                    onPressed: details.onStepCancel,
                    child: const Text('QUAY LẠI'),
                  ),
              ],
            ),
          );
        },
      ),
    );
  }

  // Hàm trả về danh sách các Step
  List<Step> _getSteps() {
    return [
      // Bước 1: Thông tin tài khoản
      Step(
        title: const Text('Tài khoản'),
        content: Form(
          key: _formKeys[0],
          child: Column(
            children: [
              TextFormField(
                decoration: InputDecoration(labelText: 'Email'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Vui lòng nhập email';
                  }
                  return null;
                },
              ),
              TextFormField(
                decoration: InputDecoration(labelText: 'Mật khẩu'),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Vui lòng nhập mật khẩu';
                  }
                  return null;
                },
              ),
            ],
          ),
        ),
        // Bước này đang active nếu _currentStep là 0
        isActive: _currentStep >= 0,
        // Trạng thái của bước
        state: _currentStep > 0 ? StepState.complete : StepState.indexed,
      ),
      // Bước 2: Thông tin cá nhân
      Step(
        title: const Text('Thông tin cá nhân'),
        content: Form(
          key: _formKeys[1],
          child: Column(
            children: [
              TextFormField(
                decoration: InputDecoration(labelText: 'Họ và tên'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Vui lòng nhập họ và tên';
                  }
                  return null;
                },
              ),
              TextFormField(
                decoration: InputDecoration(labelText: 'Địa chỉ'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Vui lòng nhập địa chỉ';
                  }
                  return null;
                },
              ),
            ],
          ),
        ),
        isActive: _currentStep >= 1,
        state: _currentStep > 1 ? StepState.complete : StepState.indexed,
      ),
      // Bước 3: Xác nhận
      Step(
        title: const Text('Xác nhận'),
        content: Form(
          key: _formKeys[2],
          child: Center(
            child: Text('Vui lòng kiểm tra lại thông tin và xác nhận.'),
          ),
        ),
        isActive: _currentStep >= 2,
        state: _currentStep > 2 ? StepState.complete : StepState.indexed,
      ),
    ];
  }
}
```

**Giải thích ví dụ trên:**

1.  **`_currentStep`**: Biến trạng thái để theo dõi bước hiện tại.
2.  **`_getSteps()`**: Một hàm được tạo ra để giữ cho `build` method gọn gàng. Hàm này trả về một `List<Step>`.
3.  **`isActive` và `state`**: Logic `_currentStep >= index` và `_currentStep > index` được sử dụng để tự động cập nhật trạng thái của các bước. Khi bạn ở bước 1 (`_currentStep = 1`), bước 0 sẽ có `state` là `StepState.complete`.
4.  **`onStepContinue`**: Trước khi chuyển bước, nó kiểm tra `_formKeys[_currentStep].currentState!.validate()` để đảm bảo người dùng đã điền đúng thông tin.
5.  **`controlsBuilder`**: Được sử dụng để tùy chỉnh nút. Ở bước cuối cùng, nút "TIẾP TỤC" được đổi thành "XÁC NHẬN". Nút "QUAY LẠI" cũng được ẩn đi ở bước đầu tiên.

### 6. StepperType.horizontal

Để chuyển sang giao diện ngang, bạn chỉ cần thay đổi một dòng:

```dart
Stepper(
  type: StepperType.horizontal,
  // ... các thuộc tính khác
)
```

Giao diện ngang thường phù hợp hơn trên các màn hình lớn như máy tính bảng hoặc web. Trên điện thoại, `StepperType.vertical` thường mang lại trải nghiệm tốt hơn vì không gian hiển thị `content` rộng rãi hơn.

### Kết luận

`Stepper` là một widget mạnh mẽ và trực quan để tạo các luồng công việc nhiều bước. Chìa khóa để sử dụng nó hiệu quả là hiểu rõ cách quản lý trạng thái thông qua `currentStep` trong một `StatefulWidget` và cách sử dụng các thuộc tính `isActive` và `state` để cung cấp phản hồi trực quan cho người dùng.
