
SwiftUI 2 在 WWDC 2020上的发布，苹果引入了一个新的应用程序生命周期(`App`)。替代了 `AppDelegate`。

## 应用入口
当创建一个新的 SwiftUI 应用程序时，应用程序的主类是这样的:
```swift
import SwiftUI

@main
struct SwiftUIApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## 初始化配置与第三方库
大多数情况下，应用启动时都需要执行初始化：默认数据配置、SDK 和第三方库等。

使用 `AppDelegate` 时，会在 `application(_ application:didFinishLaunchingWithOptions launchOptions:)` 操作:
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // configs...
        print("Application is starting...")
        return true
    }
```

而在 `App` 的生命周期里：
```swift
import SwiftUI

@main
struct SwiftUIApp: App {

  init() {
        // configs...
        print("Application is starting...")
    }

  var body: some Scene {
    WindowGroup {
      ContentView()
    }
  }
}
```

## 处理应用的生命周期
有时候，我们需要根据应用的生命周期来处理操作。比如，我们打算让应用程序进入活动状态时刷新数据，或应用进入后台状态时缓存数据。

在 `AppDelegate` 里，实现
`applicationWillResignActive`，
`applicationDidBecomeActive`，
`applicationWillEnterForeground`，
`applicationDidEnterBackground`，
`applicationWillTerminate` 方法

而在 `App` 里，苹果提供了一个新的 API：`ScenePhase`

SwiftUI 中跟踪应用程序的状态，可以用 `@Environment` 属性包装器来获取当前值，用 `.onChange(of:perform:)` 监听更改。
```swift
import SwiftUI

@main
struct SwiftUIAppApp: App {
    
    @Environment(\.scenePhase) var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { phase in
            switch phase {
            case .active:
                print("Application is active")
            case .inactive:
                print("Application is inactive")
            case .background:
                print("Application is background")
            default:
                print("unexpected value.")
            }
        }
    }
}
```
值得一提的是，应用里的其他位置也可以使用这种监听方式：
```swift
import SwiftUI

struct ContentView: View {
    
    @Environment(\.scenePhase) private var scenePhase
    
    var body: some View {
        Text("Hello, world!")
            .padding()
            .onChange(of: scenePhase, perform: { value in
                if value == .active {
                    print("Application view is active...")
                }
            })
    }
}
```
是不是很 nice，脑补一下写通知的过程...
> 通知：我不要面子啊

## 处理 Deep Link
在 `AppDelegate` 里，实现 `application(_:open:options:)` 获取到 url 后处理

而在 `App` 里，使用 `.onOpenURL` modifier 即可。
```swift
import SwiftUI

@main
struct SwiftUIAppApp: App {

    var body: some Scene {
        WindowGroup {
            ContentView()
                
                .onOpenURL(perform: { url in
                    print("Received URL: \(url)")
                })
        }
    }
}

```
同样的，你也可以在应用的其他位置实现这个 modifier。这就意味着你在任何地方都可以处理深度链接。当然，前提是启动应用的时候，实现的位置必须是 active 的状态
```swift
import SwiftUI

struct ContentView: View {

    var body: some View {
        Text("Hello, world!")
            .onOpenURL(perform: { url in
                print("Received URL: \(url)")
            })
    }
}
```

尽管在 iOS 9 后应该使用 `universal links`，毕竟这个更安全。
但这并不能否认 `onOpenURL` 的方便性。我们仍然可以定义并使用。
> 终端通过 deep link 调起 App:
> 
> xcrun simctl openurl booted `your url`


## 我不管，我就是要用 AppDelegate
> ~~那你就在创建项目的时候，选择 `AppDelegate` 咯。略略略略...~~


`App` 的生命周期并没有完全的可以代替 `AppDelegate`，有些情况下我们还是的要用到 `AppDelegate` 或者其方法。比如：SDK 的回调，第三方库的 Extension，method swizzling 等等。

综合各种原因，Swift 提供了一种 `AppDelegate` 与 `App` 连接的方法：`@UIApplicationDelegateAdaptor`

```swift
import SwiftUI

class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        print("Application didFinishLaunchingWithOptions")
        return true
    }

    func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        print("willFinishLaunchingWithOptions")
        return true
    }
}

@main
struct SwiftUIAppApp: App {
    
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
    
    init() {
        print("Application is starting...")
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## 总结
SwiftUI 2 新增的程序入口，目的之一就是为了简化应用程序的启动。一眼望去，很简洁，也方便后续别人接手的时候更容易一目了然，易于维护。总的来说是不错的。

短处也是有的，目前来说，相对于 `AppDelegate`，`App` 还不够完善，一些功能还是得依赖与 `AppDelegate` 交互实现。并且官方的 Demo 少得可怜，多数还是得靠着自己琢磨。期待后续官网补充与更新吧！而且这毕竟只是刚开始。Swift 历经五个版本才趋于稳定。相比较来说，SwiftUI 还年轻。

> 参考：
> 
> - [The Ultimate Guide to the SwiftUI 2 Application Life Cycle](https://peterfriese.dev/ultimate-guide-to-swiftui2-application-lifecycle/)
