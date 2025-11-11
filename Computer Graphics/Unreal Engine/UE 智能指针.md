Unreal Engine 提供了多种智能指针类型，其中四种主要的智能指针有：`TObjectPtr`、`TSharedPtr`、`TWeakPtr` 和 `TUniquePtr`。

1. **`TObjectPtr`**：Unreal Engine 5 中引入的一种现代化指针类型，专门用于指向 `UObject` 及其派生类。
2. **`TSharedPtr`**：引用计数智能指针，允许多个指针共享同一个对象的所有权。
3. **`TSharedRef`**：非空共享所有权智能指针，是 `TSharedPtr` 的非空变体，它总是指向一个有效的对象，永远不会是 `nullptr`。
4. **`TWeakPtr`**：弱引用智能指针，提供对由 `TSharedPtr` 或 `TSharedRef` 管理的对象的非拥有引用，它不会增加对象的引用计数，允许对象在适当的时候被销毁。
5. **`TUniquePtr`**：独占所有权智能指针，确保在任何时刻只有一个指针拥有对象，当 `TUniquePtr` 被销毁时，它所拥有的对象也会被自动删除。

## 最佳实践建议

1. **优先使用 `TObjectPtr`**：对于 `UObject` 指针，优先使用 `TObjectPtr` 替代原始指针
2. **选择合适的共享指针**：当需要共享所有权时，优先使用 `TSharedRef`（如果确定不为空）或`TSharedPtr`
3. **避免循环引用**：在可能形成循环引用的场景中使用 `TWeakPtr`
4. **独占所有权使用 `TUniquePtr`**：当需要独占所有权时使用 `TUniquePtr`
5. **合理使用 `Make` 函数**：优先使用 `MakeShared` 和 `MakeUnique` 而非直接 `new`
