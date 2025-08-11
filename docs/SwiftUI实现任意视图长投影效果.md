# SwiftUI实现任意视图长投影效果

长投影（Long Shadow）是一种流行的设计风格，它通过创建一系列渐变的阴影层来营造出立体感和深度感。本文将介绍如何在SwiftUI中实现这种效果，从基础到进阶逐步深入，最终形成一个可复用的组件。



## 单投影效果

最简单的长投影实现方式是使用两层视图：一层作为阴影，另一层作为主体内容。通过`ZStack`将两者叠加，并为阴影层设置偏移量，就能快速创建出基本的投影效果。这种方法虽然简单，但已经能够呈现出明显的立体感。

```swift
import SwiftUI

struct LongShadowText: View {
    var body: some View {
        ZStack {
            // 阴影层
            Text("投影")
                .font(.system(size: 80, weight: .bold))
                .foregroundColor(.black)
                .offset(x: CGFloat(5), y: CGFloat(5))
            
            // 文字层
            Text("投影")
                .font(.system(size: 80, weight: .bold))
                .foregroundColor(.white)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(.cyan)
    }
}

// 预览
#Preview {
    LongShadowText()
}
```



## 投影叠加

为了创建更加真实的长投影效果，我们可以使用多个阴影层叠加的方式。通过`ForEach`循环生成一系列逐渐偏移的阴影层，每层阴影都略微偏移前一层，从而形成连续的投影效果。这种方法的优势在于可以精确控制投影的长度和密度。

```swift
import SwiftUI

struct LongShadowText: View {
    var body: some View {
        ZStack {
            // 阴影层
            ZStack {
                ForEach(0..<20, id: \.self) { i in
                    Text("长投影")
                        .font(.system(size: 80, weight: .bold))
                        .offset(x: CGFloat(i), y: CGFloat(i))
                }
            }
            
            // 文字层
            Text("长投影")
                .font(.system(size: 80, weight: .bold))
                .foregroundColor(.white)
        }
    }
}

#Preview {
    LongShadowText()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(.cyan)
}
```



## 文本长投影组件

将长投影效果封装成可复用的组件可以大大提高开发效率。我们创建一个`LongShadowText`视图，它接受文本内容、字体样式、颜色和投影偏移量等参数，使得这个组件可以在不同场景下灵活使用。这种封装方式遵循了SwiftUI的组合式设计理念。

```swift
import SwiftUI

struct LongShadowText: View {
    let text: String
    let font: Font
    let textColor: Color
    let shadowColor: Color
    let offset: Int
    
    init(
        text: String,
        font: Font = .system(size: 20, weight: .bold),
        textColor: Color = .white,
        shadowColor: Color = .black,
        offset: Int = 5
    ) {
        self.text = text
        self.font = font
        self.textColor = textColor
        self.shadowColor = shadowColor
        self.offset = offset
    }
    
    var body: some View {
        ZStack {
            // 阴影层
            ZStack {
                ForEach(0..<offset, id: \.self) { i in
                    Text(text)
                        .font(font)
                        .foregroundStyle(shadowColor)
                        .offset(x: CGFloat(i), y: CGFloat(i))
                }
            }
            
            // 文字层
            Text(text)
                .font(font)
                .foregroundStyle(textColor)
        }
    }
}

#Preview {
    LongShadowText(
        text: "以梦为马",
        font: .system(size: 80),
        shadowColor: .black,
        offset: 20
    )
    .frame(maxWidth: .infinity, maxHeight: .infinity)
    .background(.cyan)
}
```



## 支持任意视图

更进一步，我们可以创建一个支持任意视图的长投影组件`LongShadowView`。它使用泛型和`@ViewBuilder`来接受任何SwiftUI视图作为内容，不仅限于文本。这种实现方式极大地扩展了组件的适用范围，可以用于图标、形状或其他自定义视图。

```swift
import SwiftUI

struct LongShadowView<Content: View>: View {
    let color: Color
    let offset: Int
    let content: () -> Content
    
    init(color: Color = .gray, offset: Int = 5, @ViewBuilder content: @escaping () -> Content) {
        self.color = color
        self.offset = offset
        self.content = content
    }
    
    var body: some View {
        ZStack {
            // 阴影层
            color.mask(
                ZStack {
                    ForEach(0..<offset, id: \.self) { index in
                        content()
                            .offset(x: CGFloat(index + 1), y: CGFloat(index + 1))
                    }
                }
            )
            
            // 原始内容
            content()
        }
    }
}

// 使用示例
struct LongShadowViewDemo: View {
    var body: some View {
        VStack(spacing: 0) {
            LongShadowView {
                Image(systemName: "apple.logo")
                    .resizable()
                    .scaledToFit()
                    .frame(height: 120)
                    .foregroundStyle(.black)
            }
            .background(.yellow)
            
            LongShadowView(color: .brown, offset: 20) {
                Text("SwiftUI")
                    .font(.system(size: 80, weight: .bold))
                    .foregroundStyle(.white)
            }
            .background(.green)
            
            LongShadowView(color: .purple, offset: 20) {
                VStack {
                    LongShadowView(color: .purple) {
                        Text("APPLE")
                            .foregroundStyle(.white)
                            .font(.largeTitle)
                            .fontWeight(.black)
                    }
                }
                .frame(width: 260, height: 80)
                .background(.cyan)
            }
            .background(.brown)
        }
    }
}

// 预览
#Preview {
    LongShadowViewDemo()
}
```



## 支持视图修饰符

为了与SwiftUI的设计风格更加契合，我们可以将长投影效果实现为视图修饰符。这种方式更加符合SwiftUI的声明式语法，使用起来也更加直观。修饰符版本还添加了模糊效果选项，可以通过逐渐降低阴影层的不透明度来创建更加柔和的投影效果。

```swift
import SwiftUI

struct LongShadow: ViewModifier {
    let color: Color
    let offset: Int
    let isBlur: Bool
    
    func body(content: Content) -> some View {
        // 缓存原始内容，避免重复构建
        let baseContent = content
        
        ZStack {
            // 阴影层
            color.mask(
                ZStack {
                    ForEach(0..<offset, id: \.self) { index in
                        baseContent
                            .offset(x: CGFloat(index + 1), y: CGFloat(index + 1))
                            .opacity(isBlur ? (1 - Double(index) / Double(offset)) : 1)
                    }
                }
            )
            
            // 原始内容
            baseContent
        }
    }
}

extension View {
    /// 添加长阴影效果
    func longShadow(
        color: Color = .gray,
        offset: Int = 5,
        isBlur: Bool = false
    ) -> some View {
        self.modifier(LongShadow(color: color, offset: offset, isBlur: isBlur))
    }
}

// 使用示例
struct LongShadowDemo: View {
    var body: some View {
        VStack(spacing: 0) {
            Image(systemName: "apple.logo")
                .resizable()
                .scaledToFit()
                .frame(height: 120)
                .foregroundStyle(.black)
                .longShadow(color: .gray.opacity(0.5), offset: 10)
                .background(.yellow)
            
            Text("SwiftUI")
                .font(.system(size: 80, weight: .bold))
                .foregroundStyle(.white)
                .longShadow(color: .brown, offset: 20, isBlur: true)
                .background(.green)
            
            VStack {
                Text("APPLE")
                    .foregroundStyle(.white)
                    .font(.largeTitle)
                    .fontWeight(.black)
                    .longShadow(color: .purple)
            }
            .frame(width: 260, height: 80)
            .background(.cyan)
            .longShadow(color: .purple, offset: 20)
            .background(.brown)
        }
    }
}

// 预览
#Preview {
    LongShadowDemo()
}
```

通过这些实现方式，我们展示了SwiftUI强大的组合能力和灵活性。从简单的文本投影到支持任意视图的通用解决方案，再到符合SwiftUI风格的修饰符实现，开发者可以根据具体需求选择最适合的方式来实现长投影效果。这些技术不仅可以用于创建视觉上吸引人的界面元素，也体现了SwiftUI声明式UI的强大之处。



## 开源项目支持

本文介绍的长投影效果已经封装成开源项目**LongShadow**，欢迎在项目中直接使用：

GitHub仓库：[https://github.com/swiftuihome/LongShadow](https://github.com/swiftuihome/LongShadow)
