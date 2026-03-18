## AsyncNotifierProvider vs StateNotifierProvider trong Riverpod

### 1. Khác biệt cốt lõi

**StateNotifierProvider** — state luôn có giá trị ngay lập tức. Khi tạo provider, constructor chạy đồng bộ, state sẵn sàng từ đầu. Nếu cần load data async, phải tự quản lý loading/error bên trong state.

**AsyncNotifierProvider** — state là `AsyncValue<T>`, tự động có 3 trạng thái: `AsyncLoading`, `AsyncData`, `AsyncError`. Method `build()` là `Future`, Riverpod tự quản lý loading/error/data lifecycle.

```dart
// StateNotifierProvider: state = int (luôn có giá trị)
final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(),  // constructor đồng bộ
);

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);  // state = 0 ngay lập tức
}

// AsyncNotifierProvider: state = AsyncValue<List<Todo>>
final todoProvider = AsyncNotifierProvider.autoDispose<TodoNotifier, List<Todo>>(
  TodoNotifier.new,
);

class TodoNotifier extends AutoDisposeAsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    // Riverpod tự set AsyncLoading trước khi build() chạy
    return await fetchTodos();  // Riverpod tự set AsyncData hoặc AsyncError
  }
}
```

---

### 2. StateNotifierProvider xử lý async — Phải tự quản lý

#### Cách 1: State chứa loading/error thủ công

```dart
@freezed
class ExchangeState with _$ExchangeState {
  const factory ExchangeState({
    @Default([]) List<PointExchangeModel> exchanges,
    @Default(false) bool isLoading,
    String? errorMessage,
  }) = _ExchangeState;
}

final exchangeProvider =
    StateNotifierProvider<ExchangeNotifier, ExchangeState>(
  (ref) => ExchangeNotifier(ref),
);

class ExchangeNotifier extends StateNotifier<ExchangeState> {
  final Ref ref;

  ExchangeNotifier(this.ref) : super(const ExchangeState()) {
    loadExchanges();  // gọi async trong constructor
  }

  Future<void> loadExchanges() async {
    // Phải tự set loading
    state = state.copyWith(isLoading: true, errorMessage: null);

    try {
      final data = await FirebaseFunctionsClient.instance
          .getPointExchangeSettingInfo();
      // Phải tự set data
      state = state.copyWith(isLoading: false, exchanges: _transform(data));
    } catch (e) {
      // Phải tự set error
      state = state.copyWith(isLoading: false, errorMessage: e.toString());
    }
  }

  Future<void> refresh() async {
    // Lặp lại pattern loading/try/catch/error
    state = state.copyWith(isLoading: true, errorMessage: null);
    try {
      final data = await FirebaseFunctionsClient.instance
          .getPointExchangeSettingInfo();
      state = state.copyWith(isLoading: false, exchanges: _transform(data));
    } catch (e) {
      state = state.copyWith(isLoading: false, errorMessage: e.toString());
    }
  }

  List<PointExchangeModel> _transform(List<PointExchangeSettingsDocumentData> data) {
    // transform logic...
    return [];
  }
}
```

Widget phải tự check loading/error:

```dart
class ExchangeListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final exchangeState = ref.watch(exchangeProvider);

    // Phải tự kiểm tra từng trạng thái
    if (exchangeState.isLoading) {
      return const CircularProgressIndicator();
    }
    if (exchangeState.errorMessage != null) {
      return Text('Error: ${exchangeState.errorMessage}');
    }
    if (exchangeState.exchanges.isEmpty) {
      return const Text('No exchanges available');
    }

    return ListView.builder(
      itemCount: exchangeState.exchanges.length,
      itemBuilder: (context, index) {
        return ExchangeCard(model: exchangeState.exchanges[index]);
      },
    );
  }
}
```

---

### 3. AsyncNotifierProvider xử lý async — Tự động quản lý

```dart
final exchangeProvider = AsyncNotifierProvider.autoDispose
    ExchangeNotifier, List<PointExchangeModel>>(
  ExchangeNotifier.new,
);

class ExchangeNotifier
    extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {
  @override
  Future<List<PointExchangeModel>> build() async {
    // Riverpod tự set AsyncLoading trước khi build() chạy
    final data = await FirebaseFunctionsClient.instance
        .getPointExchangeSettingInfo();
    return _transform(data);
    // Thành công → Riverpod tự set AsyncData
    // Exception → Riverpod tự set AsyncError
  }

  Future<void> refresh() async {
    // ref.invalidateSelf() trigger build() lại
    // Riverpod tự quản lý loading/error/data
    ref.invalidateSelf();
  }

  List<PointExchangeModel> _transform(
    List<PointExchangeSettingsDocumentData> data,
  ) {
    return [];
  }
}
```

Widget dùng `.when()` — pattern matching gọn gàng:

```dart
class ExchangeListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final exchangeAsync = ref.watch(exchangeProvider);

    // .when() bắt buộc handle cả 3 trạng thái — không quên được
    return exchangeAsync.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
      data: (exchanges) {
        if (exchanges.isEmpty) {
          return const Text('No exchanges available');
        }
        return ListView.builder(
          itemCount: exchanges.length,
          itemBuilder: (context, index) {
            return ExchangeCard(model: exchanges[index]);
          },
        );
      },
    );
  }
}
```

---

### 4. So sánh cùng bài toán — Point Exchange Flow

#### StateNotifierProvider

```dart
@freezed
class PointExchangeState with _$PointExchangeState {
  const factory PointExchangeState({
    @Default([]) List<PointExchangeModel> exchanges,
    @Default(false) bool isLoading,
    @Default(false) bool isExchanging,
    String? errorMessage,
    String? exchangeError,
    String? successTransactionId,
  }) = _PointExchangeState;
}

class PointExchangeNotifier extends StateNotifier<PointExchangeState> {
  final Ref ref;

  PointExchangeNotifier(this.ref) : super(const PointExchangeState()) {
    loadExchanges();
  }

  Future<void> loadExchanges() async {
    state = state.copyWith(isLoading: true, errorMessage: null);
    try {
      final services = await ref.read(servicesProvider.future);
      final mainService = services.firstWhereOrNull(
        (e) => e.type == ServiceType.point && e.subType == ServiceSubType.main,
      );
      if (mainService == null) {
        state = state.copyWith(
          isLoading: false,
          errorMessage: '事業者情報の取得に失敗しました。',
        );
        return;
      }

      final settings = await FirebaseFunctionsClient.instance
          .getPointExchangeSettingInfo();

      final models = _buildModels(mainService, settings);
      state = state.copyWith(isLoading: false, exchanges: models);
    } catch (e) {
      state = state.copyWith(isLoading: false, errorMessage: e.toString());
    }
  }

  Future<void> exchangeForDocomo(ExchangePointForDocomoRequest request) async {
    // Phải track loading riêng cho exchange (khác loading của list)
    state = state.copyWith(
      isExchanging: true,
      exchangeError: null,
      successTransactionId: null,
    );
    try {
      final result = await FirebaseFunctionsClient.instance
          .exchangePointForDocomo(request);
      result.when(
        success: (response) {
          state = state.copyWith(
            isExchanging: false,
            successTransactionId: response.transactionId,
          );
        },
        failure: (error) {
          state = state.copyWith(
            isExchanging: false,
            exchangeError: error.message,
          );
        },
      );
    } catch (e) {
      state = state.copyWith(
        isExchanging: false,
        exchangeError: e.toString(),
      );
    }
  }

  List<PointExchangeModel> _buildModels(
    ServicesDocumentData mainService,
    List<PointExchangeSettingsDocumentData> settings,
  ) {
    return [];
  }
}

// Widget — phải check nhiều boolean flags
class ExchangeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(pointExchangeProvider);

    // Check loading list
    if (state.isLoading) return const CircularProgressIndicator();

    // Check error list
    if (state.errorMessage != null) return Text(state.errorMessage!);

    // Check loading exchange
    if (state.isExchanging) return const Text('Exchanging...');

    // Check error exchange
    if (state.exchangeError != null) {
      return Column(
        children: [
          Text('Exchange failed: ${state.exchangeError}'),
          ElevatedButton(
            onPressed: () => ref.read(pointExchangeProvider.notifier)
                .exchangeForDocomo(request),
            child: const Text('Retry'),
          ),
        ],
      );
    }

    // Check success
    if (state.successTransactionId != null) {
      return Text('Success: ${state.successTransactionId}');
    }

    // Finally, show list
    return ListView.builder(
      itemCount: state.exchanges.length,
      itemBuilder: (_, i) => ExchangeCard(model: state.exchanges[i]),
    );
  }
}
```

#### AsyncNotifierProvider

```dart
class PointExchangeNotifier
    extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {
  @override
  Future<List<PointExchangeModel>> build() async {
    // Riverpod quản lý loading/error/data cho list
    final services = await ref.watch(servicesProvider.future);
    final mainService = services.firstWhereOrNull(
      (e) => e.type == ServiceType.point && e.subType == ServiceSubType.main,
    );
    if (mainService == null) {
      throw '事業者情報の取得に失敗しました。';
    }

    final settings = await FirebaseFunctionsClient.instance
        .getPointExchangeSettingInfo();

    return _buildModels(mainService, settings);
  }

  Future<ExchangePointForDocomoResponse> exchangeForDocomo(
    ExchangePointForDocomoRequest request,
  ) async {
    // Không cần quản lý isExchanging trong state
    // Caller (widget hoặc controller) tự quản lý UI loading
    final result = await FirebaseFunctionsClient.instance
        .exchangePointForDocomo(request);

    return result.when(
      success: (response) {
        // Refresh list sau khi exchange thành công
        ref.invalidateSelf();
        return response;
      },
      failure: (error) {
        if (error.isAlreadyExists) throw ApiIsAlreadyExistsException();
        throw ApiExceptionHandler.getError(error).getMessageDetails();
      },
    );
  }

  // Mutate state trực tiếp mà giữ previous data
  Future<void> exchangeForDocomoWithStateUpdate(
    ExchangePointForDocomoRequest request,
  ) async {
    // Giữ data cũ, chỉ set loading
    state = const AsyncLoading<List<PointExchangeModel>>()
        .copyWithPrevious(state);
    //   ↑ AsyncLoading nhưng vẫn giữ data cũ
    //   UI có thể show loading indicator + list cũ cùng lúc

    state = await AsyncValue.guard(() async {
      final result = await FirebaseFunctionsClient.instance
          .exchangePointForDocomo(request);
      return result.when(
        success: (_) => build(),  // reload list
        failure: (error) => throw error,
      );
    });
  }

  List<PointExchangeModel> _buildModels(
    ServicesDocumentData mainService,
    List<PointExchangeSettingsDocumentData> settings,
  ) {
    return [];
  }
}

// Widget — gọn gàng với .when()
class ExchangeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final exchangeAsync = ref.watch(pointExchangeAsyncNotifierProvider);

    return exchangeAsync.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Column(
        children: [
          Text('Error: $error'),
          ElevatedButton(
            onPressed: () => ref.invalidate(pointExchangeAsyncNotifierProvider),
            child: const Text('Retry'),
          ),
        ],
      ),
      data: (exchanges) => ListView.builder(
        itemCount: exchanges.length,
        itemBuilder: (_, i) => ExchangeCard(model: exchanges[i]),
      ),
    );
  }
}
```

---

### 5. AsyncValue — Sức mạnh của AsyncNotifierProvider

#### 5.1 Pattern matching đầy đủ

```dart
final asyncState = ref.watch(exchangeProvider);

// .when() — bắt buộc handle cả 3 trạng thái
asyncState.when(
  data: (data) => ShowList(data),
  loading: () => LoadingSpinner(),
  error: (e, st) => ErrorWidget(e),
);

// .whenOrNull() — chỉ handle trạng thái quan tâm
asyncState.whenOrNull(
  data: (data) => ShowList(data),
);

// .whenData() — chỉ transform data, giữ nguyên loading/error
asyncState.whenData((data) {
  return data.where((e) => e.exchangeRate > 100).toList();
});

// .hasValue — check có data không (kể cả đang loading với previous data)
if (asyncState.hasValue) {
  print(asyncState.value!.length);
}

// .isLoading, .hasError — check trạng thái
if (asyncState.isLoading) print('loading...');
if (asyncState.hasError) print(asyncState.error);
```

#### 5.2 Loading với previous data

```dart
class ExchangeNotifier
    extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {

  Future<void> refreshKeepingData() async {
    // Cách 1: copyWithPrevious — giữ data cũ khi loading
    state = const AsyncLoading<List<PointExchangeModel>>()
        .copyWithPrevious(state);
    // Lúc này: state.isLoading = true, state.hasValue = true
    // UI có thể show spinner overlay trên list cũ

    state = await AsyncValue.guard(() async {
      return await _fetchExchanges();
    });
  }

  Future<void> refreshClearData() async {
    // Cách 2: AsyncLoading thuần — xoá data cũ
    state = const AsyncLoading();
    // Lúc này: state.isLoading = true, state.hasValue = false
    // UI show full loading screen

    state = await AsyncValue.guard(() async {
      return await _fetchExchanges();
    });
  }
}

// Widget xử lý cả 2 case
class ExchangeListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncState = ref.watch(exchangeProvider);

    return Stack(
      children: [
        // Hiển thị list nếu có data (kể cả đang loading)
        if (asyncState.hasValue)
          ListView.builder(
            itemCount: asyncState.value!.length,
            itemBuilder: (_, i) => ExchangeCard(model: asyncState.value![i]),
          )
        else if (asyncState.hasError)
          Center(child: Text('Error: ${asyncState.error}'))
        else
          const SizedBox.shrink(),

        // Overlay loading indicator nếu đang refresh
        if (asyncState.isLoading)
          const Positioned.fill(
            child: ColoredBox(
              color: Color(0x40000000),
              child: Center(child: CircularProgressIndicator()),
            ),
          ),
      ],
    );
  }
}
```

#### 5.3 AsyncValue.guard — Thay thế try/catch

```dart
// StateNotifier: try/catch thủ công mỗi lần
class OldNotifier extends StateNotifier<ExchangeState> {
  Future<void> doSomething() async {
    state = state.copyWith(isLoading: true, error: null);
    try {
      final data = await api.fetch();
      state = state.copyWith(isLoading: false, data: data);
    } catch (e) {
      state = state.copyWith(isLoading: false, error: e.toString());
    }
  }

  Future<void> doAnotherThing() async {
    state = state.copyWith(isLoading: true, error: null);
    try {
      final data = await api.fetchOther();
      state = state.copyWith(isLoading: false, otherData: data);
    } catch (e) {
      state = state.copyWith(isLoading: false, error: e.toString());
    }
  }
  // ↑ Lặp đi lặp lại cùng pattern loading/try/catch
}

// AsyncNotifier: AsyncValue.guard gói gọn
class NewNotifier extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {
  Future<void> doSomething() async {
    state = const AsyncLoading<List<PointExchangeModel>>()
        .copyWithPrevious(state);
    state = await AsyncValue.guard(() async {
      return await api.fetch();
    });
    // Thành công → AsyncData
    // Exception → AsyncError
    // 3 dòng thay vì 8 dòng
  }

  Future<void> doAnotherThing() async {
    state = await AsyncValue.guard(() async {
      return await api.fetchOther();
    });
  }
}
```

---

### 6. ref trong Notifier — Khác biệt quan trọng

#### StateNotifier: phải truyền ref qua constructor

```dart
final exchangeProvider =
    StateNotifierProvider<ExchangeNotifier, ExchangeState>(
  (ref) => ExchangeNotifier(ref),  // ← truyền ref
);

class ExchangeNotifier extends StateNotifier<ExchangeState> {
  final Ref ref;  // ← phải giữ ref riêng

  ExchangeNotifier(this.ref) : super(const ExchangeState());

  Future<void> load() async {
    final services = await ref.read(servicesProvider.future);
    // ref.watch() KHÔNG dùng được trong StateNotifier
    // Chỉ dùng ref.read() — không tự rebuild khi dependency thay đổi
  }
}
```

#### AsyncNotifier: ref có sẵn, dùng watch được trong build()

```dart
class ExchangeNotifier
    extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {
  // Không cần constructor nhận ref
  // ref có sẵn trong class

  @override
  Future<List<PointExchangeModel>> build() async {
    // ✅ ref.watch() TRONG build() — tự rebuild khi dependency thay đổi
    final services = await ref.watch(servicesProvider.future);
    final operator = ref.watch(serviceOperatorsProvider);

    // Khi servicesProvider hoặc serviceOperatorsProvider thay đổi
    // → build() tự động chạy lại
    // → state chuyển AsyncLoading → AsyncData mới
    return _buildModels(services, operator);
  }

  Future<void> exchange(ExchangePointForDocomoRequest request) async {
    // ref.read() trong methods khác build()
    final client = ref.read(firebaseFunctionsProvider);
    await client.exchangePointForDocomo(request);
    ref.invalidateSelf();  // trigger build() lại
  }
}
```

Sự khác biệt này rất quan trọng. Code hiện tại trong project có vấn đề:

```dart
// Code hiện tại
class PointExchangeAsyncNotifier
    extends AutoDisposeAsyncNotifier<List<PointExchangeModel>> {

  @override
  Future<List<PointExchangeModel>> build() async {
    final services = await ref.watch(servicesProvider.future);  // ✅ watch
    serviceOperator = ref.read(serviceOperatorsStateNotifierProvider);  // ⚠️ read
    //                     ↑
    // Dùng ref.read → KHÔNG rebuild khi serviceOperator thay đổi
    // Nếu đổi thành ref.watch → tự rebuild khi operator cập nhật từ Firestore
  }
}
```

---

### 7. Lifecycle và autoDispose

#### StateNotifier: quản lý lifecycle thủ công

```dart
class ServiceOperatorsNotifier
    extends StateNotifier<ServiceOperatorsDocumentData?> {
  StreamSubscription? _listener;

  ServiceOperatorsNotifier() : super(null) {
    // init trong constructor
    _listener = FirebaseFirestore.instance
        .collection('serviceOperators')
        .doc(operatorId)
        .snapshots()
        .listen((snapshot) {
      state = ServiceOperatorsDocumentData.fromJson(snapshot.data()!);
    });
  }

  @override
  void dispose() {
    // Phải override dispose, dễ quên
    _listener?.cancel();
    super.dispose();
  }
}

// autoDispose phải khai báo riêng
final provider = StateNotifierProvider.autoDispose
    ServiceOperatorsNotifier, ServiceOperatorsDocumentData?>(
  (ref) => ServiceOperatorsNotifier(),
);
```

#### AsyncNotifier: dùng ref.onDispose, autoDispose mặc định

```dart
class ServiceOperatorsNotifier
    extends AutoDisposeAsyncNotifier<ServiceOperatorsDocumentData?> {

  @override
  Future<ServiceOperatorsDocumentData?> build() async {
    StreamSubscription? listener;

    listener = FirebaseFirestore.instance
        .collection('serviceOperators')
        .doc(operatorId)
        .snapshots()
        .listen((snapshot) {
      state = AsyncData(
        ServiceOperatorsDocumentData.fromJson(snapshot.data()!),
      );
    });

    // Cleanup đăng ký ngay cạnh setup — không bao giờ quên
    ref.onDispose(() {
      listener?.cancel();
    });

    // Lần đầu fetch đồng bộ
    final snapshot = await FirebaseFirestore.instance
        .collection('serviceOperators')
        .doc(operatorId)
        .get();

    return snapshot.exists
        ? ServiceOperatorsDocumentData.fromJson(snapshot.data()!)
        : null;
  }
}
```

---

### 8. Xử lý multiple async operations

#### StateNotifier: nhiều loading flags lộn xộn

```dart
@freezed
class ExchangeState with _$ExchangeState {
  const factory ExchangeState({
    @Default([]) List<PointExchangeModel> exchanges,
    @Default(false) bool isLoadingList,        // loading list
    @Default(false) bool isCheckingDuplicate,   // loading check
    @Default(false) bool isExchanging,          // loading exchange
    @Default(false) bool isDeletingAuth,        // loading delete
    String? listError,
    String? duplicateError,
    String? exchangeError,
  }) = _ExchangeState;
}

// Widget: if/else lồng nhau rối
if (state.isLoadingList) return LoadingList();
if (state.listError != null) return ErrorList(state.listError!);
if (state.isCheckingDuplicate) return CheckingDuplicate();
if (state.isExchanging) return Exchanging();
// ...
```

#### AsyncNotifier: tách provider cho từng concern

```dart
// Provider 1: Danh sách exchanges
@riverpod
class ExchangeList extends _$ExchangeList {
  @override
  Future<List<PointExchangeModel>> build() async {
    return await _fetchExchanges();
  }
}

// Provider 2: Thao tác exchange (tách riêng)
@riverpod
class ExchangeAction extends _$ExchangeAction {
  @override
  FutureOr<void> build() {
    // Không làm gì khi init — chờ user trigger
  }

  Future<ExchangePointForDocomoResponse> exchangeDocomo(
    ExchangePointForDocomoRequest request,
  ) async {
    state = const AsyncLoading();

    final result = await AsyncValue.guard(() async {
      // Check duplicate
      final isDuplicate = await _checkDuplicate(request);
      if (isDuplicate) throw const DuplicateExchangeException();

      // Execute exchange
      return await _executeExchange(request);
    });

    // Cập nhật state
    if (result is AsyncData) {
      state = result;
      // Refresh list
      ref.invalidate(exchangeListProvider);
    } else {
      state = result as AsyncError;
    }

    return (result as AsyncData).value;
  }
}

// Widget: mỗi provider handle riêng
class ExchangeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final listAsync = ref.watch(exchangeListProvider);
    final actionAsync = ref.watch(exchangeActionProvider);

    return Stack(
      children: [
        // List
        listAsync.when(
          data: (data) => ExchangeListView(exchanges: data),
          loading: () => const ListSkeleton(),
          error: (e, _) => ListError(e),
        ),

        // Action overlay
        if (actionAsync.isLoading)
          const FullScreenLoading(message: 'Exchanging...'),
      ],
    );
  }
}
```

---

### 9. Testing

#### StateNotifier: test trực tiếp nhưng phải check nhiều fields

```dart
test('loadExchanges sets loading then data', () async {
  final container = ProviderContainer(
    overrides: [
      firebaseFunctionsProvider.overrideWithValue(mockClient),
    ],
  );

  when(() => mockClient.getPointExchangeSettingInfo())
      .thenAnswer((_) async => mockSettings);

  final notifier = container.read(exchangeProvider.notifier);

  // Check initial state
  expect(container.read(exchangeProvider).isLoading, false);
  expect(container.read(exchangeProvider).exchanges, isEmpty);

  // Trigger load
  await notifier.loadExchanges();

  // Check final state — phải verify nhiều fields
  final state = container.read(exchangeProvider);
  expect(state.isLoading, false);
  expect(state.errorMessage, isNull);
  expect(state.exchanges, isNotEmpty);
  expect(state.exchanges.first.provider, PointExchangeProviderType.docomo);
});

test('loadExchanges sets error on failure', () async {
  when(() => mockClient.getPointExchangeSettingInfo())
      .thenThrow(Exception('Network error'));

  final notifier = container.read(exchangeProvider.notifier);
  await notifier.loadExchanges();

  final state = container.read(exchangeProvider);
  expect(state.isLoading, false);           // phải verify loading = false
  expect(state.errorMessage, isNotNull);    // phải verify error set
  expect(state.exchanges, isEmpty);          // phải verify data không đổi
});
```

#### AsyncNotifier: test bằng AsyncValue pattern matching

```dart
test('build returns exchanges on success', () async {
  final container = ProviderContainer(
    overrides: [
      servicesProvider.overrideWith((ref) async => mockServices),
      firebaseFunctionsProvider.overrideWithValue(mockClient),
    ],
  );

  when(() => mockClient.getPointExchangeSettingInfo())
      .thenAnswer((_) async => mockSettings);

  // Chờ provider resolve
  final result = await container.read(exchangeProvider.future);

  // Check data trực tiếp — không cần check isLoading, errorMessage
  expect(result, isNotEmpty);
  expect(result.first.provider, PointExchangeProviderType.docomo);
});

test('build throws on failure', () async {
  final container = ProviderContainer(
    overrides: [
      servicesProvider.overrideWith((ref) async => []),
    ],
  );

  // Verify error bằng .future — tự throw
  expect(
    () async => await container.read(exchangeProvider.future),
    throwsA(isA<String>()),
  );

  // Hoặc check AsyncValue trực tiếp
  // Cần đợi provider settle
  await container.read(exchangeProvider.future).catchError((_) => []);
  final state = container.read(exchangeProvider);
  expect(state, isA<AsyncError<List<PointExchangeModel>>>());
});

test('exchangeDocomo refreshes list on success', () async {
  // ... setup mocks

  final notifier = container.read(exchangeActionProvider.notifier);
  await notifier.exchangeDocomo(request);

  // Verify list provider was invalidated (will refetch)
  // AsyncNotifier tự handle state transitions
  final actionState = container.read(exchangeActionProvider);
  expect(actionState.hasError, false);
});
```

---

### 10. So sánh tổng hợp

```
                        StateNotifierProvider          AsyncNotifierProvider
──────────────────────────────────────────────────────────────────────────────
State type              T (giá trị trực tiếp)          AsyncValue<T>
                                                       (Loading/Data/Error)

Init data               Constructor (đồng bộ)          build() (async)
                        Phải gọi async riêng           Riverpod tự quản lý

Loading/Error           Tự quản lý bằng                Tự động có
                        boolean flags trong state       AsyncLoading, AsyncError

ref access              Truyền qua constructor          Có sẵn trong class
                        Chỉ ref.read()                  ref.watch() trong build()

Auto rebuild khi        ❌ Không (ref.read)             ✅ Có (ref.watch trong build)
dependency thay đổi

Cleanup                 Override dispose()              ref.onDispose() cạnh setup

Widget pattern          if/else kiểm tra               .when() pattern matching
                        isLoading, error, data          bắt buộc handle cả 3

AsyncValue.guard        ❌ Không có                     ✅ Thay try/catch

copyWithPrevious        ❌ Tự implement                 ✅ Có sẵn

Refresh data            Gọi method + set loading        ref.invalidateSelf()
                        + try/catch lại                 hoặc AsyncValue.guard

Phù hợp cho            State đồng bộ phức tạp         Bất kỳ state nào cần
                        (form, local UI state)          fetch async data
                        Không có async init

Deprecated?             ⚠️ Legacy (vẫn hoạt động)     ✅ Recommended
```

---

### 11. Lưu ý

StateNotifierProvider vẫn hoạt động tốt và sẽ không bị xoá trong tương lai gần. Tuy nhiên Riverpod team khuyến khích dùng **Notifier** (đồng bộ) và **AsyncNotifier** (async) cho code mới. Với project hiện tại đang dùng cả hai, không cần migrate ngay — nhưng khi viết feature mới, ưu tiên AsyncNotifier cho bất kỳ state nào liên quan đến API call hoặc Firestore.
