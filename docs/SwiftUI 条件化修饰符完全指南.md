# SwiftUI 条件化修饰符完全指南

在 SwiftUI 开发中，处理条件化视图的传统方式是使用 `if-else` 语句，但这往往会破坏视图的声明式结构和链式调用。本文将深入探讨如何通过扩展 `View` 协议实现 `.if` 和 `.else` 修饰符，构建更优雅的条件化视图逻辑。

---

## 一、核心实现

### 基础实现代码
```swift
extension View {
    /// 条件化应用修饰符
    @ViewBuilder
    func `if`<Content: View>(
        _ condition: Bool,
        transform: (Self) -> Content
    ) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
    
    /// 提供 else 分支支持
    @ViewBuilder
    func `else`<Content: View>(
        _ condition: Bool,
        transform: (Self) -> Content
    ) -> some View {
        if !condition {
            transform(self)
        } else {
            self
        }
    }
}

// 可选：链式 else 扩展
extension View {
    @ViewBuilder
    func `else`<Content: View>(
        @ViewBuilder transform: (Self) -> Content
    ) -> some View {
        transform(self)
    }
}
```

### 代码解析
1. **`@ViewBuilder` 作用**：支持返回多种视图类型
2. **泛型 `<Content: View>`**：保证转换后的内容仍然是 View
3. **条件反转**：`.else` 实质是 `!condition` 的语法糖
4. **链式调用设计**：每个方法都返回 `some View` 保持链式

---

## 二、实际应用案例

### 案例 1：权限控制按钮
```swift
struct AdminButton: View {
    let isAdmin: Bool
    
    var body: some View {
        Button("删除用户") {
            // 管理员操作
        }
        .if(isAdmin) {
            $0.buttonStyle(.borderedProminent)
               .tint(.red)
        }
        .else {
            $0.hidden()
        }
    }
}
```
**解析**：
- 当 `isAdmin` 为 true 时显示红色警示按钮
- 否则完全隐藏按钮（不占位）
- 避免传统方案中重复写两个 Button

### 案例 2：响应式布局
```swift
struct AdaptiveView: View {
    @Environment(\.horizontalSizeClass) var sizeClass
    
    var body: some View {
        Text("内容区域")
            .if(sizeClass == .compact) {
                $0.padding(8)
                   .font(.body)
            }
            .else {
                $0.padding(20)
                   .font(.title)
            }
    }
}
```
**优势**：
- 保持单一视图声明
- 清晰表达两种状态的差异

### 案例 3：表单验证提示
```swift
struct EmailField: View {
    @Binding var email: String
    @State private var showError = false
    
    var body: some View {
        VStack(alignment: .leading) {
            TextField("邮箱", text: $email)
                .textFieldStyle(.roundedBorder)
                .onChange(of: email) { _ in
                    validateEmail()
                }
                .if(showError) {
                    $0.overlay(
                        Image(systemName: "exclamationmark.triangle")
                            .foregroundColor(.red)
                            .padding(.trailing, 8),
                        alignment: .trailing
                    )
                }
            
            Text("请输入有效邮箱地址")
                .font(.caption)
                .foregroundColor(.red)
                .if(!showError) { $0.hidden() }
        }
    }
    
    private func validateEmail() {
        showError = !email.isEmpty && !email.contains("@")
    }
}
```

---

## 三、进阶用法

### 1. 链式条件判断
```swift
Text("智能提示")
    .if(priority == .high) { $0.foregroundColor(.red) }
    .else(priority == .medium) { $0.foregroundColor(.orange) }
    .else { $0.foregroundColor(.green) }
```

### 2. 与动画结合
```swift
Circle()
    .fill(Color.blue)
    .frame(width: 100)
    .if(shouldAnimate) {
        $0.animation(.easeInOut.repeatForever(), value: scale)
    }
    .scaleEffect(scale)
```

### 3. 调试辅助工具
```swift
extension View {
    func debugBorder(_ color: Color = .red) -> some View {
        self.border(color)
    }
}

MyView()
    .if(showDebugViews) { $0.debugBorder() }
```

---

## 四、与传统方案对比

### 传统方案
```swift
if condition {
    Text("Hello")
        .font(.title)
        .padding()
} else {
    Text("Hello")
        .padding()
}
```

### 问题：
1. 代码重复（`Text("Hello")` 写两次）
2. 破坏修饰符链
3. 难以维护多个条件分支

### `.if` 方案优势：
1. **DRY 原则**：避免重复视图声明
2. **链式友好**：保持修饰符调用流
3. **可读性强**：线性表达条件逻辑

---

## 五、性能考量

### 最佳实践：
1. **避免深层嵌套**：
   ```swift
   // 不推荐
   .if(a) { $0.if(b) { $0.if(c) { ... } } }
   
   // 推荐
   .if(a && b && c) { $0... }
   ```

2. **预计算复杂条件**：
   ```swift
   var shouldShowBadge: Bool {
       user.isActive && unreadCount > 0
   }
   
   .if(shouldShowBadge) { $0.badge(...) }
   ```

3. **谨慎用于高频更新**：在 `ObservableObject` 更新时，条件化修饰符会触发视图重建

---

## 六、完整示例：智能标签系统

```swift
struct SmartTag: View {
    enum Status { case new, hot, expired }
    
    let text: String
    let status: Status
    var onTap: (() -> Void)? = nil
    
    var body: some View {
        Text(text)
            .font(.caption)
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .if(status == .new) {
                $0.background(Color.blue.opacity(0.2))
                   .foregroundColor(.blue)
            }
            .else(status == .hot) {
                $0.background(Color.red.opacity(0.2))
                   .foregroundColor(.red)
            }
            .else {
                $0.background(Color.gray.opacity(0.2))
                   .foregroundColor(.gray)
            }
            .if(onTap != nil) {
                $0.onTapGesture { onTap?() }
                   .contentShape(Rectangle())
            }
    }
}
```

**使用场景**：
```swift
HStack {
    SmartTag(text: "限时", status: .hot)
    SmartTag(text: "已结束", status: .expired)
    SmartTag(text: "新上架", status: .new, onTap: {
        print("点击新标签")
    })
}
```

---

## 七、总结

### 适用场景
1. 需要保持修饰符链的条件化样式
2. 简单的二元状态切换
3. 构建可配置的组件库

### 不适用场景
1. 完全不同的视图层级结构
2. 需要 `switch-case` 的多分支场景
3. 性能敏感的列表项

通过 `.if` 和 `.else` 修饰符，我们可以写出更符合 SwiftUI 声明式理念的代码，让视图的逻辑分支像 CSS 媒体查询一样优雅。这种模式特别适合需要维护多种状态但又希望保持代码整洁的项目。