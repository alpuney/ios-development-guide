# SwiftUI View Order

A style guide for organizing SwiftUI view files consistently across the project.

---

## Structure Overview

```
Main View {
    // MARK: - Dependencies
    // MARK: - State
    // MARK: - Properties
    // MARK: - Initialization
    // MARK: - Body
}

// MARK: - Computed Properties (extension)
// MARK: - Subviews (extension)
// MARK: - Actions (extension)
// MARK: - Helpers (extension)
// MARK: - Constants (extension)
// MARK: - Protocol Conformances (extension)

#Preview
```

---

## Sections

### Dependencies

Externally injected values that the view reads from its environment.

```swift
@Environment(\.dismiss) private var dismiss
@Environment(\.modelContext) private var modelContext
@Query(sort: \Item.date, order: .reverse) private var items: [Item]
```

### State

View-owned mutable state, including persisted and parent-driven bindings.

```swift
@State private var title = ""
@AppStorage("isDarkMode") private var isDarkMode = false
@FocusState private var isTitleFocused: Bool
@Binding var isPresented: Bool
```

### Properties

Stored properties only. These are non-reactive values passed in or defined at init time.

```swift
let category: Category
private let maxItemCount = 50
```

### Initialization

Custom initializers when default memberwise init is not sufficient.

```swift
init(category: Category) {
    self.category = category
    _title = State(initialValue: category.name)
}
```

### Body

The single `var body: some View` declaration. Keep it high-level and delegate complexity to subviews.

```swift
var body: some View {
    NavigationStack {
        contentView
            .navigationTitle("Items")
            .toolbar { toolbarContent }
    }
    .onAppear { loadInitialData() }
    .onChange(of: scenePhase) { handleSceneChange($1) }
}
```

---

### Computed Properties (extension)

Derived values consumed by `body` or subviews. Placed here to keep the main struct lean.

```swift
extension ItemListView {
    private var filteredItems: [Item] {
        items.filter { !$0.isArchived }
    }

    private var isFormValid: Bool {
        !title.trimmingCharacters(in: .whitespaces).isEmpty
    }
}
```

### Subviews (extension)

Private view compositions. Use computed properties for simple views, `@ViewBuilder` methods when parameters are needed.

```swift
extension ItemListView {
    private var contentView: some View {
        List(filteredItems) { item in
            rowView(for: item)
        }
    }

    @ViewBuilder
    private func rowView(for item: Item) -> some View {
        HStack {
            Text(item.title)
            Spacer()
            if item.isPinned {
                Image(systemName: "pin.fill")
            }
        }
    }
}
```

### Actions (extension)

User-triggered methods — button taps, swipes, submissions, and other gesture responses.

```swift
extension ItemListView {
    private func addItem() {
        let item = Item(title: title)
        modelContext.insert(item)
        title = ""
    }

    private func deleteItem(_ item: Item) {
        modelContext.delete(item)
    }
}
```

### Helpers (extension)

Internal logic that is not directly triggered by the user — formatting, validation, data transformation.

```swift
extension ItemListView {
    private func formattedDate(_ date: Date) -> String {
        date.formatted(date: .abbreviated, time: .shortened)
    }

    private func loadInitialData() {
        // fetch or prepare data
    }
}
```

### Constants (extension)

Static values scoped to the view — layout dimensions, string literals, configuration values.

```swift
extension ItemListView {
    private enum Constants {
        static let horizontalPadding: CGFloat = 16
        static let rowSpacing: CGFloat = 8
        static let maxVisibleItems = 20
    }
}
```

### Protocol Conformances (extension)

When the view conforms to additional protocols like `DropDelegate` or custom protocols.

```swift
extension ItemListView: DropDelegate {
    func performDrop(info: DropInfo) -> Bool {
        // handle drop
        return true
    }
}
```

### Preview

```swift
#Preview {
    ItemListView(category: .sample)
        .modelContainer(for: Item.self, inMemory: true)
}
```

---

## Quick Reference

| Order | Section                | Location        | Purpose                                  |
|-------|------------------------|-----------------|------------------------------------------|
| 1     | Dependencies           | Main struct     | `@Environment`, `@Query`                 |
| 2     | State                  | Main struct     | `@State`, `@AppStorage`, `@FocusState`, `@Binding` |
| 3     | Properties             | Main struct     | Stored properties (public, then private) |
| 4     | Initialization         | Main struct     | `init()` when needed                     |
| 5     | Body                   | Main struct     | `var body: some View`                    |
| 6     | Computed Properties    | Extension       | Derived values for body/subviews         |
| 7     | Subviews               | Extension       | Private views and `@ViewBuilder` methods |
| 8     | Actions                | Extension       | User-triggered methods                   |
| 9     | Helpers                | Extension       | Internal logic and utilities             |
| 10    | Constants              | Extension       | `private enum Constants`                 |
| 11    | Protocol Conformances  | Extension       | `DropDelegate`, custom protocols         |
| 12    | Preview                | File bottom     | `#Preview`                               |
