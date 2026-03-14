Huy, đây là danh sách toàn diện các class có keyword `Sliver` trong Flutter, phân loại theo chức năng.

---

# Toàn bộ Sliver Classes trong Flutter

## 1. Widget Layer — Sliver dùng trực tiếp trong CustomScrollView

### 1.1 — List & Grid (Hiển thị danh sách/lưới)

| Class | Mô tả |
|---|---|
| `SliverList` | Danh sách lazy-loading, mỗi item có thể khác kích thước. Có các named constructor: `.builder()`, `.list()`, `.separated()` |
| `SliverFixedExtentList` | Giống SliverList nhưng mọi item **bắt buộc cùng một height cố định** (truyền qua `itemExtent`). Hiệu quả hơn vì không cần layout từng child để biết size |
| `SliverPrototypeExtentList` | Tương tự SliverFixedExtentList nhưng thay vì truyền pixel value, truyền một **prototype widget** — framework layout widget đó một lần rồi dùng size của nó cho toàn bộ list |
| `SliverGrid` | Lưới lazy-loading 2D. Có `.builder()`, `.list()`, `.count()`, `.extent()` |
| `SliverAnimatedList` | Phiên bản animated của SliverList — hỗ trợ animation khi insert/remove item |
| `SliverAnimatedGrid` | Phiên bản animated của SliverGrid |
| `SliverReorderableList` | Cho phép user kéo thả để sắp xếp lại thứ tự item |

### 1.2 — App Bar & Header (Thanh điều hướng)

| Class | Mô tả |
|---|---|
| `SliverAppBar` | App bar co giãn khi scroll, có `pinned`, `floating`, `snap`. Có named constructor `.medium()` và `.large()` theo Material 3 |
| `SliverPersistentHeader` | Header co giãn tổng quát — là nền tảng bên dưới mà SliverAppBar sử dụng. Cần cung cấp `SliverPersistentHeaderDelegate` |

### 1.3 — Fill & Spacing (Lấp đầy không gian)

| Class | Mô tả |
|---|---|
| `SliverFillRemaining` | Lấp đầy **toàn bộ không gian còn lại** trong viewport. Thường dùng cho item cuối cùng cần chiếm hết phần trống |
| `SliverFillViewport` | Mỗi child chiếm **đúng toàn bộ viewport** — giống PageView nhưng trong sliver context |
| `SliverToBoxAdapter` | Chuyển đổi một **box widget thông thường** (Text, Container...) thành sliver. Cầu nối giữa box world và sliver world |
| `SliverPadding` | Phiên bản sliver của `Padding` — thêm padding quanh một sliver child |

### 1.4 — Grouping & Layout (Nhóm và bố cục)

| Class | Mô tả |
|---|---|
| `SliverMainAxisGroup` | Nhóm nhiều sliver theo **trục chính** (vertical nếu scroll dọc). Khi group scroll ra khỏi viewport, tất cả sliver bên trong cũng biến mất — kể cả pinned header |
| `SliverCrossAxisGroup` | Nhóm nhiều sliver theo **trục ngang** — giống `Row` nhưng cho sliver. Các sliver con nằm cạnh nhau |
| `SliverCrossAxisExpanded` | Dùng bên trong `SliverCrossAxisGroup`, tương tự `Expanded` — chiếm phần còn lại theo `flex` |
| `SliverConstrainedCrossAxis` | Giới hạn `maxExtent` trên trục ngang cho sliver con bên trong `SliverCrossAxisGroup` |

### 1.5 — Visual Effects (Hiệu ứng thị giác)

| Class | Mô tả |
|---|---|
| `SliverOpacity` | Phiên bản sliver của `Opacity` |
| `SliverAnimatedOpacity` | Phiên bản sliver của `AnimatedOpacity` — transition opacity có animation |
| `SliverFadeTransition` | Phiên bản sliver của `FadeTransition` — dùng với `Animation<double>` |
| `SliverVisibility` | Phiên bản sliver của `Visibility` — show/hide sliver |
| `SliverOffstage` | Phiên bản sliver của `Offstage` — ẩn sliver nhưng vẫn giữ state |
| `SliverIgnorePointer` | Phiên bản sliver của `IgnorePointer` — vô hiệu hóa touch trên sliver |
| `DecoratedSliver` | Phiên bản sliver của `DecoratedBox` — áp decoration (border, background, shadow...) lên sliver |

### 1.6 — Utility (Tiện ích)

| Class | Mô tả |
|---|---|
| `SliverLayoutBuilder` | Phiên bản sliver của `LayoutBuilder` — cho biết `SliverConstraints` hiện tại để build widget tùy theo constraints |
| `SliverSafeArea` | Phiên bản sliver của `SafeArea` — tránh notch, status bar, home indicator |
| `SliverOverlapAbsorber` | Hấp thụ phần overlap của sliver trước đó (dùng trong `NestedScrollView`) |
| `SliverOverlapInjector` | Inject lại phần overlap đã hấp thụ — đi kèm với `SliverOverlapAbsorber` |

### 1.7 — Cupertino (iOS-style)

| Class | Mô tả |
|---|---|
| `CupertinoSliverNavigationBar` | Navigation bar kiểu iOS với large title co lại khi scroll |
| `CupertinoSliverRefreshControl` | Pull-to-refresh kiểu iOS trong sliver context |

---

## 2. Delegate Layer — Cung cấp children cho Sliver

| Class | Mô tả |
|---|---|
| `SliverChildDelegate` | Abstract base class cho tất cả delegate |
| `SliverChildBuilderDelegate` | Cung cấp children qua **builder callback** — lazy, tạo khi cần |
| `SliverChildListDelegate` | Cung cấp children qua **List\<Widget\> có sẵn** — eager, tạo tất cả ngay |
| `SliverPersistentHeaderDelegate` | Abstract delegate cho `SliverPersistentHeader` — define `minExtent`, `maxExtent`, `build()` |
| `SliverGridDelegate` | Abstract base class cho grid layout strategy |
| `SliverGridDelegateWithFixedCrossAxisCount` | Grid với **số cột cố định** |
| `SliverGridDelegateWithMaxCrossAxisExtent` | Grid với **chiều rộng tối đa mỗi ô cố định**, số cột tự tính |

---

## 3. Rendering Layer — RenderObject cho Sliver

Đây là layer thấp nhất, nơi layout và paint thực sự xảy ra. Mỗi sliver widget ở trên đều tạo ra một `RenderSliver` tương ứng:

### 3.1 — Base class & Protocol

| Class | Mô tả |
|---|---|
| `RenderSliver` | **Abstract base class** cho tất cả sliver render object. Define sliver protocol: nhận `SliverConstraints`, trả `SliverGeometry` |
| `SliverConstraints` | Input của sliver protocol — chứa thông tin scroll state: `scrollOffset`, `remainingPaintExtent`, `cacheExtent`, `crossAxisExtent`, `overlap`... |
| `SliverGeometry` | Output của sliver protocol — sliver báo lại: `scrollExtent`, `paintExtent`, `layoutExtent`, `maxPaintExtent`, `hitTestExtent`... |
| `SliverPhysicalParentData` | Parent data chứa `paintOffset` — vị trí paint của child sliver |
| `SliverPhysicalContainerParentData` | Mở rộng `SliverPhysicalParentData` với danh sách children (dùng cho multi-child sliver) |
| `SliverLogicalParentData` | Parent data với `layoutOffset` — vị trí logic trong scroll extent |
| `SliverLogicalContainerParentData` | Tương tự nhưng cho multi-child |
| `SliverHitTestResult` | Kết quả hit test cho sliver |
| `SliverHitTestEntry` | Entry trong hit test result |

### 3.2 — Render Object cụ thể

| Class | Mô tả |
|---|---|
| `RenderSliverList` | Render cho `SliverList` — layout children theo main axis, variable extent |
| `RenderSliverFixedExtentList` | Render cho `SliverFixedExtentList` — tối ưu vì biết trước extent |
| `RenderSliverFixedExtentBoxAdaptor` | Base class cho sliver có fixed extent children |
| `RenderSliverGrid` | Render cho `SliverGrid` |
| `RenderSliverFillViewport` | Render cho `SliverFillViewport` |
| `RenderSliverFillRemaining` | Render cho `SliverFillRemaining` |
| `RenderSliverFillRemainingWithScrollable` | Variant cho phép child bên trong scrollable |
| `RenderSliverFillRemainingAndOverscroll` | Variant xử lý overscroll |
| `RenderSliverToBoxAdapter` | Render cho `SliverToBoxAdapter` — adapt box child vào sliver protocol |
| `RenderSliverPadding` | Render cho `SliverPadding` |
| `RenderSliverEdgeInsetsPadding` | Base class cho sliver padding render |
| `RenderSliverPersistentHeader` | Base render cho persistent header |
| `RenderSliverFloatingPersistentHeader` | Variant floating |
| `RenderSliverPinnedPersistentHeader` | Variant pinned |
| `RenderSliverFloatingPinnedPersistentHeader` | Variant floating + pinned |
| `RenderSliverScrollingPersistentHeader` | Variant scrolling (không pin, không float) |
| `RenderSliverOpacity` | Render cho `SliverOpacity` |
| `RenderSliverAnimatedOpacity` | Render cho `SliverAnimatedOpacity` |
| `RenderSliverOffstage` | Render cho `SliverOffstage` |
| `RenderSliverIgnorePointer` | Render cho `SliverIgnorePointer` |
| `RenderSliverMainAxisGroup` | Render cho `SliverMainAxisGroup` |
| `RenderSliverCrossAxisGroup` | Render cho `SliverCrossAxisGroup` |
| `RenderSliverOverlapAbsorber` | Render cho `SliverOverlapAbsorber` |
| `RenderSliverOverlapInjector` | Render cho `SliverOverlapInjector` |
| `RenderSliverHelpers` | **Mixin** cung cấp helper methods cho sliver có box children (hit test, paint transform) |
| `RenderSliverMultiBoxAdaptor` | **Base class** cho sliver render nhiều box children (SliverList, SliverGrid kế thừa từ đây) |
| `RenderSliverBoxChildManager` | Interface để `RenderSliverMultiBoxAdaptor` giao tiếp với element layer để tạo/hủy children |

---

## 4. Các class hỗ trợ khác

| Class | Mô tả |
|---|---|
| `SliverOverlapAbsorberHandle` | Handle chia sẻ giữa `SliverOverlapAbsorber` và `SliverOverlapInjector` để truyền overlap extent |
| `SliverGridLayout` | Abstract — layout strategy cho grid |
| `SliverGridRegularTileLayout` | Implementation cụ thể cho grid tile đều nhau |
| `SliverGridParentData` | Parent data cho grid child, chứa `crossAxisOffset` |
| `SliverMultiBoxAdaptorParentData` | Parent data cho multi-box sliver child, chứa `keepAlive` flag và `index` |

---

## 5. Tổng kết bức tranh kiến trúc

```
                    ┌──────────────────────┐
                    │   CustomScrollView   │
                    │    (slivers: [...])   │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
     ┌────────▼──────┐ ┌──────▼───────┐ ┌──────▼──────┐
     │  SliverAppBar  │ │  SliverList   │ │ SliverGrid  │  ← Widget Layer
     └────────┬──────┘ └──────┬───────┘ └──────┬──────┘
              │               │                │
              │        ┌──────▼───────┐        │
              │        │SliverChild   │        │
              │        │BuilderDelegate│       │           ← Delegate Layer
              │        └──────┬───────┘        │
              │               │                │
     ┌────────▼──────┐ ┌──────▼───────┐ ┌──────▼──────┐
     │RenderSliver   │ │RenderSliver  │ │RenderSliver │
     │PinnedPersist..│ │List          │ │Grid         │  ← Render Layer
     └────────┬──────┘ └──────┬───────┘ └──────┬──────┘
              │               │                │
              └───────────────┼────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │ SliverConstraints  │ ← Input
                    │ SliverGeometry     │ ← Output
                    │ (Sliver Protocol)  │
                    └───────────────────┘
```

Tổng cộng có khoảng **60-70 class** liên quan đến Sliver trong Flutter framework, trải dài trên 3 layer: Widget → Delegate → RenderObject. Với Senior, layer Widget là nơi dùng hàng ngày, layer Delegate là nơi customize hành vi, và layer RenderObject là nơi cần hiểu để debug performance hoặc tạo custom sliver riêng.
