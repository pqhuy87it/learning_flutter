## StateNotifierProvider vs StateProvider trong Riverpod

### 1. Khác biệt cốt lõi

**StateProvider** — chứa **một giá trị đơn giản**, bên ngoài ghi trực tiếp vào state bằng `.notifier.state = ...`. Không có logic kiểm soát cách state thay đổi.

**StateNotifierProvider** — chứa **một class StateNotifier** có methods riêng để thay đổi state. Bên ngoài chỉ thay đổi state thông qua các methods mà class đó cho phép.

```dart
// StateProvider: ai cũng ghi trực tiếp được
final counterProvider = StateProvider<int>((ref) => 0);
ref.read(counterProvider.notifier).state = 999;   // ✅ ghi thẳng, không ai kiểm soát

// StateNotifierProvider: ghi qua method, có kiểm soát
final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(),
);

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() {
    if (state >= 100) return;  // ← validation, giới hạn max
    state++;
  }
}

ref.read(counterProvider.notifier).increment();     // ✅ qua method
// ref.read(counterProvider.notifier).state = 999;  // ❌ state là protected
```

---

### 2. StateProvider — Chi tiết

#### Cấu trúc

```dart
// Khai báo: chỉ cần giá trị khởi tạo
final nameProvider = StateProvider<String>((ref) => '');
final isDarkModeProvider = StateProvider<bool>((ref) => false);
final selectedTabProvider = StateProvider<int>((ref) => 0);
final sortTypeProvider = StateProvider<PointExchangeSortType>((ref) {
  return PointExchangeSortType.lowPoints;
});
```

#### Đọc và ghi

```dart
class FilterBar extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Đọc
    final sortType = ref.watch(sortTypeProvider);
    final isDark = ref.watch(isDarkModeProvider);

    return Column(
      children: [
        // Ghi trực tiếp — không validation
        Switch(
          value: isDark,
          onChanged: (v) {
            ref.read(isDarkModeProvider.notifier).state = v;
          },
        ),

        // Ghi bằng update — dựa trên giá trị cũ
        ElevatedButton(
          onPressed: () {
            ref.read(selectedTabProvider.notifier).update((current) => current + 1);
          },
          child: const Text('Next tab'),
        ),

        // Gán enum mới
        DropdownButton<PointExchangeSortType>(
          value: sortType,
          items: PointExchangeSortType.values.map((t) {
            return DropdownMenuItem(value: t, child: Text(t.label));
          }).toList(),
          onChanged: (v) {
            ref.read(sortTypeProvider.notifier).state = v!;
          },
        ),
      ],
    );
  }
}
```

#### Vấn đề của StateProvider

```dart
final amountProvider = StateProvider<int>((ref) => 1000);

// Bất kỳ widget nào cũng ghi được bất kỳ giá trị gì
ref.read(amountProvider.notifier).state = -500;    // ✅ cho phép — nhưng sai logic
ref.read(amountProvider.notifier).state = 9999999; // ✅ cho phép — vượt max
ref.read(amountProvider.notifier).state = 0;       // ✅ cho phép — nhưng min là 100

// Không có chỗ nào đặt validation
// Mỗi widget phải tự validate trước khi ghi — dễ quên, dễ sai
```

---

### 3. StateNotifierProvider — Chi tiết

#### Cấu trúc

```dart
// Khai báo: cần class StateNotifier riêng
final amountProvider = StateNotifierProvider<AmountNotifier, int>(
  //                                        ↑ class         ↑ state type
  (ref) => AmountNotifier(),
);

class AmountNotifier extends StateNotifier<int> {
  AmountNotifier() : super(1000);  // giá trị khởi tạo

  // state là protected — chỉ thay đổi được bên trong class
  // Bên ngoài gọi methods

  void setAmount(int value) {
    if (value < 100) return;          // min 100
    if (value > 10000) {
      state = 10000;                   // cap max
      return;
    }
    state = value;
  }

  void increment(int step) {
    setAmount(state + step);           // reuse validation
  }

  void decrement(int step) {
    setAmount(state - step);
  }

  void reset() {
    state = 1000;
  }
}
```

#### Đọc và ghi

```dart
class AmountSelector extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Đọc state
    final amount = ref.watch(amountProvider);

    // Đọc notifier để gọi methods
    final notifier = ref.read(amountProvider.notifier);

    return Column(
      children: [
        Text('Amount: $amount'),

        Row(
          children: [
            // Ghi qua method — validation tự động
            ElevatedButton(
              onPressed: () => notifier.decrement(100),
              child: const Text('-100'),
            ),
            ElevatedButton(
              onPressed: () => notifier.increment(100),
              child: const Text('+100'),
            ),
          ],
        ),

        ElevatedButton(
          onPressed: () => notifier.reset(),
          child: const Text('Reset'),
        ),

        // notifier.state = -500;  // ❌ state là protected, không ghi trực tiếp được
      ],
    );
  }
}
```

---

### 4. So sánh cùng bài toán

#### Bài toán: Quản lý trạng thái search/filter cho Point Exchange

##### Dùng StateProvider

```dart
// 3 StateProvider riêng biệt
final searchQueryProvider = StateProvider<String>((ref) => '');
final favoriteOnlyProvider = StateProvider<bool>((ref) => false);
final sortTypeProvider = StateProvider<PointExchangeSortType>(
  (ref) => PointExchangeSortType.lowPoints,
);

// Widget
class FilterBar extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final query = ref.watch(searchQueryProvider);
    final favOnly = ref.watch(favoriteOnlyProvider);
    final sort = ref.watch(sortTypeProvider);

    return Column(
      children: [
        TextField(
          onChanged: (v) => ref.read(searchQueryProvider.notifier).state = v,
        ),
        Switch(
          value: favOnly,
          onChanged: (v) => ref.read(favoriteOnlyProvider.notifier).state = v,
        ),
        DropdownButton<PointExchangeSortType>(
          value: sort,
          onChanged: (v) => ref.read(sortTypeProvider.notifier).state = v!,
          items: PointExchangeSortType.values
              .map((t) => DropdownMenuItem(value: t, child: Text(t.label)))
              .toList(),
        ),

        // Reset — phải reset từng provider riêng lẻ
        ElevatedButton(
          onPressed: () {
            ref.read(searchQueryProvider.notifier).state = '';
            ref.read(favoriteOnlyProvider.notifier).state = false;
            ref.read(sortTypeProvider.notifier).state =
                PointExchangeSortType.lowPoints;
            // Dễ quên reset 1 provider → state không đồng bộ
          },
          child: const Text('Reset'),
        ),
      ],
    );
  }
}
```

##### Dùng StateNotifierProvider

```dart
// 1 StateNotifier quản lý toàn bộ search state
@freezed
class PointExchangeSearchState with _$PointExchangeSearchState {
  const factory PointExchangeSearchState({
    @Default('') String query,
    @Default(false) bool favoriteOnly,
    @Default(PointExchangeSortType.lowPoints) PointExchangeSortType sort,
  }) = _PointExchangeSearchState;
}

final searchStateProvider = StateNotifierProvider
    SearchStateNotifier, PointExchangeSearchState>(
  (ref) => SearchStateNotifier(),
);

class SearchStateNotifier extends StateNotifier<PointExchangeSearchState> {
  SearchStateNotifier() : super(const PointExchangeSearchState());

  void updateQuery(String query) {
    state = state.copyWith(query: query);
  }

  void toggleFavorite() {
    state = state.copyWith(favoriteOnly: !state.favoriteOnly);
  }

  void updateSort(PointExchangeSortType sort) {
    state = state.copyWith(sort: sort);
  }

  // Reset tất cả cùng lúc — không bao giờ quên field nào
  void reset() {
    state = const PointExchangeSearchState();
  }

  // Logic phức tạp: clear query khi đổi sort
  void updateSortAndClearQuery(PointExchangeSortType sort) {
    state = state.copyWith(sort: sort, query: '');
  }
}

// Widget
class FilterBar extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final searchState = ref.watch(searchStateProvider);
    final notifier = ref.read(searchStateProvider.notifier);

    return Column(
      children: [
        TextField(onChanged: notifier.updateQuery),
        Switch(
          value: searchState.favoriteOnly,
          onChanged: (_) => notifier.toggleFavorite(),
        ),
        DropdownButton<PointExchangeSortType>(
          value: searchState.sort,
          onChanged: (v) => notifier.updateSort(v!),
          items: PointExchangeSortType.values
              .map((t) => DropdownMenuItem(value: t, child: Text(t.label)))
              .toList(),
        ),
        ElevatedButton(
          onPressed: notifier.reset,  // reset toàn bộ 1 lần
          child: const Text('Reset'),
        ),
      ],
    );
  }
}
```

---

### 5. Ví dụ thực tế từ project — ServiceOperatorsNotifier

Code hiện tại đang dùng StateNotifierProvider:

```dart
final serviceOperatorsStateNotifierProvider = StateNotifierProvider
    ServiceOperatorsNotifier, ServiceOperatorsDocumentData?>((ref) {
  return ServiceOperatorsNotifier();
});

class ServiceOperatorsNotifier
    extends StateNotifier<ServiceOperatorsDocumentData?> {
  StreamSubscription<DocumentSnapshot>? serviceOperatorsListener;

  ServiceOperatorsNotifier() : super(null) {
    // Setup Firestore listener
    CollectionReference collection = FirebaseFirestore.instance
        .collection(FirestoreCollectionNames.serviceOperators);

    serviceOperatorsListener = collection
        .doc(ApiClient.instance.serviceOperatorId)
        .snapshots(includeMetadataChanges: false)
        .listen(
      (snapshot) {
        if (snapshot.exists) {
          state = ServiceOperatorsDocumentData.fromJson(
            snapshot.data() as Map<String, dynamic>,
          );
          if (state?.enableApigee != null) {
            ApigeeClient.instance.useApigee = state!.enableApigee!;
          }
        } else {
          state = null;
        }
      },
      onError: (error) =>
          DebugUtil.logger.e('serviceOperators listen failed: $error'),
    );
  }

  @override
  void dispose() {
    serviceOperatorsListener?.cancel();
    super.dispose();
  }
}
```

Nếu dùng StateProvider thì **không thể**:

```dart
// ❌ StateProvider không có chỗ đặt StreamSubscription
// ❌ Không có constructor để setup listener
// ❌ Không có dispose để cancel listener
// ❌ Không có side effect logic (update ApigeeClient.useApigee)

final serviceOperatorsProvider = StateProvider<ServiceOperatorsDocumentData?>(
  (ref) => null,  // chỉ có giá trị khởi tạo, không có gì khác
);
```

---

### 6. StateNotifier có thể dùng Ref

```dart
final exchangeHistoryProvider = StateNotifierProvider
    ExchangeHistoryNotifier, List<TransactionsDocumentData>>((ref) {
  return ExchangeHistoryNotifier(ref);
});

class ExchangeHistoryNotifier
    extends StateNotifier<List<TransactionsDocumentData>> {
  final Ref ref;

  ExchangeHistoryNotifier(this.ref) : super([]);

  Future<void> loadHistory() async {
    // Truy cập provider khác thông qua ref
    final user = ref.read(usersStateNotifierProvider);
    if (user == null) return;

    final uid = FirebaseAuth.instance.currentUser?.uid;
    if (uid == null) return;

    final snapshot = await FirebaseFirestore.instance
        .collection(FirestoreCollectionNames.transactions)
        .where('uid', isEqualTo: uid)
        .orderBy('createdAt', descending: true)
        .limit(50)
        .get();

    state = snapshot.docs
        .map((doc) => TransactionsDocumentData.fromJson(doc.data()))
        .toList();
  }

  // Watch provider khác — react khi dependency thay đổi
  Future<void> loadFilteredHistory() async {
    final sortType = ref.read(sortTypeProvider);
    // ... load và sort theo sortType
  }

  void clearHistory() {
    state = [];
  }
}
```

StateProvider **không** truy cập được provider khác từ bên trong logic thay đổi state. Mọi logic phải nằm ở widget layer:

```dart
// StateProvider: logic phải ở widget
ElevatedButton(
  onPressed: () {
    final user = ref.read(usersStateNotifierProvider);
    if (user == null) return;  // ← logic ở widget
    // load data...
    ref.read(historyProvider.notifier).state = loadedData;  // ← ghi thẳng
  },
  child: Text('Load'),
)

// StateNotifierProvider: logic ở notifier
ElevatedButton(
  onPressed: () {
    ref.read(exchangeHistoryProvider.notifier).loadHistory();  // ← gọi method
    // Notifier tự xử lý validation, ref.read, error handling
  },
  child: Text('Load'),
)
```

---

### 7. So sánh xử lý state phức tạp

#### Bài toán: Point Exchange Form

```dart
// ════════════════════════════════════════
// StateProvider — state rải rác, logic ở widget
// ════════════════════════════════════════

final selectedProviderProvider = StateProvider<PointExchangeProviderType?>(
  (ref) => null,
);
final exchangeAmountProvider = StateProvider<int>((ref) => 0);
final isConfirmedProvider = StateProvider<bool>((ref) => false);
final errorMessageProvider = StateProvider<String?>((ref) => null);

// Widget phải chứa toàn bộ business logic
class ExchangeFormWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        // Validation ở widget — dài, khó test, dễ quên
        final provider = ref.read(selectedProviderProvider);
        final amount = ref.read(exchangeAmountProvider);

        if (provider == null) {
          ref.read(errorMessageProvider.notifier).state = 'Provider required';
          return;
        }
        if (amount <= 0) {
          ref.read(errorMessageProvider.notifier).state = 'Invalid amount';
          return;
        }
        if (amount > 10000) {
          ref.read(errorMessageProvider.notifier).state = 'Exceeds maximum';
          return;
        }

        ref.read(errorMessageProvider.notifier).state = null;

        // Duplicate check ở widget
        final isDuplicate = await ref
            .read(pointExchangeAsyncNotifierProvider.notifier)
            .checkDuplicate(
              amounts: [amount],
              providerType: provider,
            );

        if (isDuplicate) {
          ref.read(errorMessageProvider.notifier).state = 'Duplicate exchange';
          return;
        }

        ref.read(isConfirmedProvider.notifier).state = true;
      },
      child: const Text('Confirm'),
    );
  }
}
```

```dart
// ════════════════════════════════════════
// StateNotifierProvider — state tập trung, logic ở notifier
// ════════════════════════════════════════

@freezed
class ExchangeFormState with _$ExchangeFormState {
  const factory ExchangeFormState({
    PointExchangeProviderType? selectedProvider,
    @Default(0) int amount,
    @Default(false) bool isConfirmed,
    @Default(false) bool isLoading,
    String? errorMessage,
  }) = _ExchangeFormState;
}

final exchangeFormProvider =
    StateNotifierProvider<ExchangeFormNotifier, ExchangeFormState>(
  (ref) => ExchangeFormNotifier(ref),
);

class ExchangeFormNotifier extends StateNotifier<ExchangeFormState> {
  final Ref ref;

  ExchangeFormNotifier(this.ref) : super(const ExchangeFormState());

  void selectProvider(PointExchangeProviderType provider) {
    state = state.copyWith(
      selectedProvider: provider,
      errorMessage: null,  // clear error khi chọn provider mới
    );
  }

  void setAmount(int amount) {
    if (amount < 0) amount = 0;
    if (amount > 10000) amount = 10000;
    state = state.copyWith(
      amount: amount,
      errorMessage: null,
    );
  }

  Future<void> confirm() async {
    // Validation tập trung
    if (state.selectedProvider == null) {
      state = state.copyWith(errorMessage: 'Provider required');
      return;
    }
    if (state.amount <= 0) {
      state = state.copyWith(errorMessage: 'Invalid amount');
      return;
    }

    state = state.copyWith(isLoading: true, errorMessage: null);

    // Duplicate check
    try {
      final isDuplicate = await ref
          .read(pointExchangeAsyncNotifierProvider.notifier)
          .checkDuplicate(
            amounts: [state.amount],
            providerType: state.selectedProvider!,
          );

      if (isDuplicate) {
        state = state.copyWith(
          isLoading: false,
          errorMessage: 'Duplicate exchange',
        );
        return;
      }

      state = state.copyWith(isLoading: false, isConfirmed: true);
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        errorMessage: 'Error: $e',
      );
    }
  }

  void reset() {
    state = const ExchangeFormState();
  }
}

// Widget rất gọn — chỉ gọi method
class ExchangeFormWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(exchangeFormProvider);
    final notifier = ref.read(exchangeFormProvider.notifier);

    return Column(
      children: [
        DropdownButton<PointExchangeProviderType>(
          value: formState.selectedProvider,
          onChanged: (v) => notifier.selectProvider(v!),
          items: PointExchangeProviderType.values
              .map((t) => DropdownMenuItem(value: t, child: Text(t.name)))
              .toList(),
        ),
        TextField(
          onChanged: (v) => notifier.setAmount(int.tryParse(v) ?? 0),
        ),
        if (formState.errorMessage != null)
          Text(formState.errorMessage!, style: TextStyle(color: Colors.red)),
        if (formState.isLoading)
          const CircularProgressIndicator()
        else
          ElevatedButton(
            onPressed: notifier.confirm,
            child: const Text('Confirm'),
          ),
      ],
    );
  }
}
```

---

### 8. Testability

#### StateProvider — khó test logic vì logic nằm ở widget

```dart
// Muốn test validation logic → phải test widget → cần pumpWidget, tap, verify
// Chậm, phức tạp, brittle

testWidgets('shows error when amount is 0', (tester) async {
  await tester.pumpWidget(
    ProviderScope(child: MaterialApp(home: ExchangeFormWidget())),
  );
  await tester.tap(find.text('Confirm'));
  await tester.pumpAndSettle();
  expect(find.text('Invalid amount'), findsOneWidget);
});
```

#### StateNotifierProvider — dễ test vì logic tách riêng

```dart
// Test pure logic — không cần Widget, không cần pumpWidget
test('confirm shows error when provider is null', () async {
  final container = ProviderContainer();
  final notifier = container.read(exchangeFormProvider.notifier);

  await notifier.confirm();

  expect(
    container.read(exchangeFormProvider).errorMessage,
    'Provider required',
  );
});

test('setAmount caps at 10000', () {
  final container = ProviderContainer();
  final notifier = container.read(exchangeFormProvider.notifier);

  notifier.setAmount(99999);

  expect(container.read(exchangeFormProvider).amount, 10000);
});

test('confirm detects duplicate', () async {
  final container = ProviderContainer(
    overrides: [
      pointExchangeAsyncNotifierProvider.overrideWith(
        () => MockNotifier(duplicateResult: true),
      ),
    ],
  );
  final notifier = container.read(exchangeFormProvider.notifier);
  notifier.selectProvider(PointExchangeProviderType.docomo);
  notifier.setAmount(500);

  await notifier.confirm();

  expect(
    container.read(exchangeFormProvider).errorMessage,
    'Duplicate exchange',
  );
  expect(container.read(exchangeFormProvider).isConfirmed, false);
});

test('reset clears all state', () {
  final container = ProviderContainer();
  final notifier = container.read(exchangeFormProvider.notifier);

  notifier.selectProvider(PointExchangeProviderType.docomo);
  notifier.setAmount(500);
  notifier.reset();

  final state = container.read(exchangeFormProvider);
  expect(state.selectedProvider, isNull);
  expect(state.amount, 0);
  expect(state.errorMessage, isNull);
});
```

---

### 9. So sánh tổng hợp

```
                        StateProvider              StateNotifierProvider
─────────────────────────────────────────────────────────────────────────
State type              Giá trị đơn (int,          Object phức tạp
                        String, bool, enum)        (class với nhiều field)

Cách ghi state          .notifier.state = x        Qua methods trong
                        (trực tiếp, ai cũng ghi)   StateNotifier class

Validation              ❌ Không có                 ✅ Trong method

Side effects            ❌ Không (không có class)   ✅ Gọi API, read ref,
                                                    setup listener

Lifecycle               ❌ Không có constructor     ✅ Constructor + dispose
                        hay dispose

Nhiều state liên quan   Nhiều provider rời rạc,     1 class quản lý tất cả,
                        dễ mất đồng bộ              đảm bảo consistency

Business logic          Ở widget layer             Ở notifier class
                        (khó test, khó reuse)       (dễ test, dễ reuse)

Test                    Cần widget test             Unit test thuần
                        (pumpWidget, tap...)        (nhanh, đơn giản)

Code lượng              Ít (1 dòng khai báo)       Nhiều hơn (class riêng)

Ví dụ phù hợp          Theme toggle,               Form state, search filter,
                        tab selection,              Firestore listener,
                        simple counter              exchange flow, cart
```

---

### 10. Lưu ý: StateNotifier đã deprecated

Từ Riverpod 2.0+, Riverpod team khuyến khích dùng **Notifier** / **AsyncNotifier** thay cho **StateNotifier**:

```dart
// ❌ StateNotifier (legacy, vẫn hoạt động nhưng deprecated)
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() => state++;
}
final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(),
);

// ✅ Notifier (mới, khuyên dùng)
class CounterNotifier extends AutoDisposeNotifier<int> {
  @override
  int build() => 0;  // giá trị khởi tạo trong build() thay vì super()
  void increment() => state++;
}
final counterProvider = NotifierProvider.autoDispose<CounterNotifier, int>(
  CounterNotifier.new,
);

// ✅ Hoặc dùng codegen
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  void increment() => state++;
}
```

Khác biệt chính: Notifier mới có `ref` sẵn trong class (không cần truyền qua constructor), có `build()` method thay vì constructor, và tích hợp tốt hơn với autoDispose/family. Tuy nhiên concept so sánh với StateProvider vẫn giống hệt — Notifier kiểm soát cách state thay đổi, StateProvider thì không.
