## StateProvider vs Provider trong Riverpod

### 1. Khác biệt cốt lõi

**Provider** — tạo giá trị **read-only**. Bên ngoài không thể thay đổi giá trị, chỉ provider tự quyết định giá trị dựa trên logic bên trong và dependencies.

**StateProvider** — tạo giá trị **read-write**. Bên ngoài có thể thay đổi giá trị trực tiếp thông qua `.notifier.state`.

```dart
// Provider: read-only — không ai thay đổi được từ bên ngoài
final greetingProvider = Provider<String>((ref) {
  return 'Hello World';
});

// StateProvider: read-write — ai cũng thay đổi được
final counterProvider = StateProvider<int>((ref) {
  return 0;  // giá trị khởi tạo
});
```

```dart
void main() {
  final container = ProviderContainer();

  // Provider: chỉ đọc
  final greeting = container.read(greetingProvider);  // "Hello World"
  // container.read(greetingProvider.notifier).state = 'Hi';  // ❌ KHÔNG CÓ notifier

  // StateProvider: đọc + ghi
  final count = container.read(counterProvider);              // 0
  container.read(counterProvider.notifier).state = 5;         // ✅ ghi được
  container.read(counterProvider.notifier).state++;           // ✅ tăng lên 6
}
```

---

### 2. Provider — Chi tiết

#### Giá trị phụ thuộc logic bên trong

```dart
// Giá trị cố định
final apiBaseUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});

// Giá trị computed từ provider khác
final exchangeRateTextProvider = Provider<String>((ref) {
  final exchanges = ref.watch(pointExchangeAsyncNotifierProvider);
  return exchanges.when(
    data: (data) => '${data.length} providers available',
    loading: () => 'Loading...',
    error: (e, _) => 'Error: $e',
  );
});

// Giá trị derived — tự cập nhật khi dependency thay đổi
final activeExchangesProvider = Provider<List<PointExchangeModel>>((ref) {
  final allExchanges = ref.watch(pointExchangeAsyncNotifierProvider).value ?? [];
  final now = DateTime.now();
  return allExchanges.where((e) {
    if (e.periodStart == null || e.periodEnd == null) return true;
    return now.isAfter(e.periodStart!) && now.isBefore(e.periodEnd!);
  }).toList();
});

// Instance object — dependency injection
final firebaseFunctionsProvider = Provider<FirebaseFunctionsClient>((ref) {
  return FirebaseFunctionsClient.instance;
});

final pointExchangeRepoProvider = Provider<PointExchangeRepository>((ref) {
  final client = ref.watch(firebaseFunctionsProvider);
  return PointExchangeRepositoryImpl(client);
});
```

#### Cách dùng trong Widget

```dart
class ExchangeHeader extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Chỉ đọc, không thay đổi
    final rateText = ref.watch(exchangeRateTextProvider);
    final baseUrl = ref.read(apiBaseUrlProvider);

    return Text(rateText);
  }
}
```

#### Giá trị có thể thay đổi — nhưng chỉ gián tiếp

Provider tự cập nhật khi **dependency** thay đổi:

```dart
final filterProvider = StateProvider<String>((ref) => '');

// Provider này tự rebuild khi filterProvider thay đổi
final filteredListProvider = Provider<List<PointExchangeModel>>((ref) {
  final filter = ref.watch(filterProvider);    // dependency
  final all = ref.watch(pointExchangeAsyncNotifierProvider).value ?? [];

  if (filter.isEmpty) return all;
  return all.where((e) =>
    e.paymentServiceProviderName.toLowerCase().contains(filter.toLowerCase())
  ).toList();
});

// Bên ngoài:
// - KHÔNG thể thay đổi filteredListProvider trực tiếp
// - CHỈ thay đổi filterProvider → filteredListProvider tự tính lại
ref.read(filterProvider.notifier).state = 'docomo';
// → filteredListProvider tự rebuild với filter mới
```

---

### 3. StateProvider — Chi tiết

#### Dùng cho state đơn giản

```dart
// Boolean toggle
final isFavoriteOnlyProvider = StateProvider<bool>((ref) => false);

// Enum selection
final sortTypeProvider = StateProvider<PointExchangeSortType>((ref) {
  return PointExchangeSortType.lowPoints;
});

// String input
final searchQueryProvider = StateProvider<String>((ref) => '');

// Number
final selectedAmountProvider = StateProvider<int>((ref) => 1000);

// Nullable selection
final selectedProviderTypeProvider = StateProvider<PointExchangeProviderType?>((ref) {
  return null;
});
```

#### Cách đọc và ghi

```dart
class ExchangeFilterBar extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ĐỌC giá trị hiện tại — rebuild khi thay đổi
    final isFavoriteOnly = ref.watch(isFavoriteOnlyProvider);
    final sortType = ref.watch(sortTypeProvider);
    final searchQuery = ref.watch(searchQueryProvider);

    return Column(
      children: [
        // GHI — thay đổi state
        TextField(
          onChanged: (value) {
            ref.read(searchQueryProvider.notifier).state = value;
          },
        ),

        // Toggle boolean
        Switch(
          value: isFavoriteOnly,
          onChanged: (value) {
            ref.read(isFavoriteOnlyProvider.notifier).state = value;
            // Hoặc toggle:
            // ref.read(isFavoriteOnlyProvider.notifier).update((state) => !state);
          },
        ),

        // Chọn enum
        DropdownButton<PointExchangeSortType>(
          value: sortType,
          items: PointExchangeSortType.values.map((type) {
            return DropdownMenuItem(value: type, child: Text(type.label));
          }).toList(),
          onChanged: (value) {
            if (value != null) {
              ref.read(sortTypeProvider.notifier).state = value;
            }
          },
        ),
      ],
    );
  }
}
```

#### `.notifier.state` vs `.notifier.update()`

```dart
// Gán trực tiếp
ref.read(counterProvider.notifier).state = 10;

// Update dựa trên giá trị cũ — an toàn hơn
ref.read(counterProvider.notifier).update((current) => current + 1);

// Ví dụ thực tế: toggle favorite
ref.read(isFavoriteOnlyProvider.notifier).update((current) => !current);

// Ví dụ: giới hạn giá trị
ref.read(selectedAmountProvider.notifier).update((current) {
  final newValue = current + 100;
  return newValue > 10000 ? 10000 : newValue;  // max 10000
});
```

---

### 4. So sánh song song cùng một bài toán

#### Bài toán: Quản lý trạng thái filter cho danh sách point exchange

```dart
// ════════════════════════════════════════
// Dùng StateProvider — state đơn giản, ghi trực tiếp
// ════════════════════════════════════════

final searchQueryProvider = StateProvider<String>((ref) => '');
final favoriteOnlyProvider = StateProvider<bool>((ref) => false);
final sortTypeProvider = StateProvider<PointExchangeSortType>((ref) {
  return PointExchangeSortType.lowPoints;
});

// Provider derived từ StateProvider — read-only, tự tính
final filteredExchangesProvider = Provider<List<PointExchangeModel>>((ref) {
  final query = ref.watch(searchQueryProvider);
  final favoriteOnly = ref.watch(favoriteOnlyProvider);
  final sortType = ref.watch(sortTypeProvider);
  final exchanges = ref.watch(pointExchangeAsyncNotifierProvider).value ?? [];

  var result = exchanges.where((e) {
    final matchQuery = query.isEmpty ||
        e.paymentServiceProviderName.toLowerCase().contains(query.toLowerCase());
    final matchFavorite = !favoriteOnly || e.isFavorite;
    return matchQuery && matchFavorite;
  }).toList();

  result.sort((a, b) {
    if (sortType == PointExchangeSortType.lowPoints) {
      return a.tokyoPointRate.compareTo(b.tokyoPointRate);
    } else if (sortType == PointExchangeSortType.highExchangeRate) {
      return b.exchangeRate.compareTo(a.exchangeRate);
    }
    return 0;
  });

  return result;
});
```

```dart
// Widget sử dụng
class ExchangeListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final filtered = ref.watch(filteredExchangesProvider);
    final query = ref.watch(searchQueryProvider);
    final favOnly = ref.watch(favoriteOnlyProvider);

    return Column(
      children: [
        // Ghi vào StateProvider
        TextField(
          onChanged: (v) => ref.read(searchQueryProvider.notifier).state = v,
        ),
        Switch(
          value: favOnly,
          onChanged: (v) => ref.read(favoriteOnlyProvider.notifier).state = v,
        ),

        // Đọc từ Provider (read-only, derived)
        Text('${filtered.length} results'),
        Expanded(
          child: ListView.builder(
            itemCount: filtered.length,
            itemBuilder: (_, i) => ExchangeCard(model: filtered[i]),
          ),
        ),
      ],
    );
  }
}
```

Luồng data:

```
User gõ "docomo"
    ↓
searchQueryProvider.state = "docomo"     (StateProvider — ghi)
    ↓
filteredExchangesProvider rebuild         (Provider — tự tính lại)
    ↓
Widget rebuild với danh sách đã filter   (hiển thị)
```

---

### 5. Khi nào dùng cái nào

#### Dùng Provider khi

Giá trị **chỉ đọc** hoặc **tự tính** từ nguồn khác:

```dart
// Config cố định
final appConfigProvider = Provider<AppConfig>((ref) => AppConfig());

// Dependency injection
final repositoryProvider = Provider<PointExchangeRepository>((ref) {
  return PointExchangeRepositoryImpl(ref.watch(clientProvider));
});

// Derived/computed value
final totalExchangeablePointProvider = Provider<int>((ref) {
  final exchanges = ref.watch(pointExchangeAsyncNotifierProvider).value ?? [];
  return exchanges.fold(0, (sum, e) => sum + e.maxExchangePointPerUse);
});

// Format/transform data
final exchangeSummaryProvider = Provider<String>((ref) {
  final count = ref.watch(filteredExchangesProvider).length;
  final total = ref.watch(pointExchangeAsyncNotifierProvider).value?.length ?? 0;
  return '$count / $total providers';
});
```

#### Dùng StateProvider khi

Giá trị **đơn giản** mà user hoặc UI cần **thay đổi trực tiếp**:

```dart
// UI toggle
final isDarkModeProvider = StateProvider<bool>((ref) => false);

// Form input
final selectedTabProvider = StateProvider<int>((ref) => 0);

// Filter/sort selection
final sortProvider = StateProvider<SortType>((ref) => SortType.newest);

// Temporary UI state
final isExpandedProvider = StateProvider<bool>((ref) => false);
```

#### KHÔNG dùng StateProvider khi

State phức tạp hoặc cần validation — dùng Notifier thay thế:

```dart
// ❌ StateProvider — không có chỗ đặt validation logic
final amountProvider = StateProvider<int>((ref) => 0);
// Ai cũng có thể gán: ref.read(amountProvider.notifier).state = -999;
// Không có guard nào ngăn giá trị sai

// ✅ Notifier — kiểm soát được cách state thay đổi
class AmountNotifier extends AutoDisposeNotifier<int> {
  @override
  int build() => 0;

  void setAmount(int value) {
    if (value < 0) return;                    // validation
    if (value > 10000) {
      state = 10000;                          // cap max
      return;
    }
    state = value;
  }

  void increment(int step) {
    setAmount(state + step);                  // reuse validation
  }
}
```

```dart
// ❌ StateProvider — state là object phức tạp
final searchStateProvider = StateProvider<PointExchangeSearchState>((ref) {
  return PointExchangeSearchState();
});
// Thay đổi 1 field phải copy toàn bộ object:
ref.read(searchStateProvider.notifier).update((s) =>
    s.copyWith(freeWord: 'docomo'));
// Không rõ ràng, dễ sai

// ✅ Notifier — method rõ ràng cho từng action
class SearchNotifier extends AutoDisposeNotifier<PointExchangeSearchState> {
  @override
  PointExchangeSearchState build() => const PointExchangeSearchState();

  void updateQuery(String query) {
    state = state.copyWith(freeWord: query);
  }

  void toggleFavorite() {
    state = state.copyWith(favorite: !state.favorite);
  }

  void updateSort(PointExchangeSortType sort) {
    state = state.copyWith(sort: sort);
  }

  void reset() {
    state = const PointExchangeSearchState();
  }
}
```

---

### 6. So sánh tổng hợp

```
                     Provider                    StateProvider
────────────────────────────────────────────────────────────────────
Đọc giá trị          ✅ ref.watch/read           ✅ ref.watch/read
Ghi giá trị          ❌ Không                     ✅ .notifier.state =
Có notifier           ❌ Không                     ✅ Có
Tự rebuild khi        dependency thay đổi         .state bị gán hoặc
                      (ref.watch bên trong)        dependency thay đổi
Validation            Không cần (read-only)       ❌ Không có chỗ đặt
Use case              Config, DI, computed         Simple UI state
                      value, derived data          (toggle, selection,
                                                    input text)
Tương đương React     useMemo, derived state       useState cho primitive
```

Cách nhớ: **Provider** giống biến `final` — gán một lần, đọc nhiều lần. **StateProvider** giống biến `var` — gán đi gán lại thoải mái, nhưng chỉ phù hợp với giá trị đơn giản.
