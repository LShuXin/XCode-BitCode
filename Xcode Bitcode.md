# Xcode Bitcode

> 最近项目中接入某第三方SDK后，打包的时候发现有如下报错：xxx.o was build without full bitcode error ：`Linker command failed with exit code 1`。 然后经过搜索，设置`Enable Bitcode 为 NO`，就没有这个报错了。笔者简单了解了一下Bitcode，今天给大家介绍一下。

#### Xcode之Bitcode

- **`Bitcode`是`Xcode7`的新特性。**
- 查看Bitcode：TARGETS -> Build Settings -> 搜索Enable Bitcode ：

#### Bitcode的[官方说明](https://links.jianshu.com/go?to=https%3A%2F%2Fhelp.apple.com%2Fxcode%2Fmac%2Fcurrent%2F%23%2Fdevbbdc5ce4f)：

> 官方：
>  Bitcode is an intermediate representation of a compiled program. apps you upload to App Store Connect that contain bitcode will be compiled and linked on the App Store. Including bitcode will allow Apple to re-optimize your app binary in the future without the need to submit a new version of your app to the App Store.
>  For iOS apps, bitcode is the default, but optional. For watchOS and tvOS apps, bitcode is required. If you provide bitcode, all apps and frameworks in the app bundle (all targets in the project) need to include bitcode.
>
> 翻译：
>  `Bitcode`是编译后的程序的中间表现，包含`Bitcode`并上传到`App Store Connect`的Apps会在`App Store`上编译和链接。包含`Bitcode`可以在不提交新版本App的情况下，允许Apple在将来的时候再次优化你的App 二进制文件。
>  对于iOS Apps，`Enable bitcode` 默认为`YES`，是可选的（可以改为NO）。对于WatchOS和tvOS，bitcode是强制的。如果你的App支持bitcode，App Bundle（项目中所有的target）中的所有的Apps和frameworks都需要包含`Bitcode`。

看了以上内容，我们就可以对Bitcode有一个简单的了解了。那么如果我们项目中在使用某些 Framework 或 .a 的时候，遇到了类似笔者遇到的错误的时候，我们就需要[查看所用的 Framework 或 .a 是否支持 bitcode ](https://links.jianshu.com/go?to=https%3A%2F%2Fforums.developer.apple.com%2Fthread%2F19775)。

#### 查看 framework 或者 .a 文件是否支持 bitcode，支持哪些架构：

- **首先给大家介绍一个工具：[otool](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.manpagez.com%2Fman%2F1%2Fotool)**
- **说明：**
   [otool](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.manpagez.com%2Fman%2F1%2Fotool)：`object file display tool.`
   用于查看object file的工具。

我们可以使用[otool](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.manpagez.com%2Fman%2F1%2Fotool)查看`framework`或者`.a` 是否支持设置`Enable Bitcode`为`YES`，在终端中使用如下命令查看：

- **otool -l framwork路径下的实体文件 | grep __LLVM**

说明：  使用otool 工具 查看`framework`文件的`load commands`内容，然后搜索`load commands`中的`__LLVM`。

如果上述命令的输出结果有`__LLVM`，那么就说明，所用的`framework`或`.a`支持设置`Enable bitcode`为`YES`，否则不支持。

- **示例：**



```undefined
 otool -l /Users/wangyongwang/Documents/QiBitcode/QiBitcode.framework/QiBitcode | grep __LLVM
```

1. 如果上述命令没有输出结果，那么说明所用的`framework`或`.a`**不支持**设置`Enable bitcode`为`YES`；
2. 如果有如下的输出结果，那么说明所用的`framework`或`.a`**支持**设置`Enable bitcode`为`YES`；



```undefined
   segname __LLVM
   segname __LLVM
   segname __LLVM
   segname __LLVM
```

- **App支持Enable Bitcode的必要条件：**

1. 使用的framework或者.a 文件支持设置 Enable bitcode为YES；
2. 使用的framework或者.a 文件支持的架构是齐全的；

- **那么为什么有些framework没有做成支持Enable bitcode的方式呢？**

我查到了如下[资料](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bbsmax.com%2FA%2Fxl56E6axJr%2F)：可能笔者自己理解上还有些问题，笔者就不解读了，大家自行解读。

> 1. **Build static library or framework via Xcode 7, while user build application using Xcode 6.**
>     "Framework and library providers need to include bitcode for Xcode 7 development, and Xcode 7 generates bitcode by default.

However, bitcode-enabled framework and library products do not work well with Xcode 6. If you still need to support Xcode 6 development,
 you must produce an additional version of your products without bitcode.
 To build a library without bitcode, either use Xcode 7 with the build setting Enable Bitcode disabled (ENABLE_BITCODE=NO)
 or use Xcode 6."

- **查看framework支持的架构有哪些：**

先给大家介绍下[`lipo`](https://links.jianshu.com/go?to=https%3A%2F%2Fss64.com%2Fosx%2Flipo.html)

> **lipo** : Create or operate on a universal file: convert a universal binary to a single architecture file, or vice versa.
>  创建或者是操作一个通用文件，转变通用文件为单独的架构文件或者反过来转变单独架构文件为通用文件。

给大家介绍一下查看Framework支持的架构，这里我们会用到`lipo info`。

**`lipo info`解读**

> -info Briefly list the architecture types in the input universal file.
>  Lists the names of each archive.
>  简单地列举出来输入的通用文件的架构类型，列举出来每个架构的名字：

- **使用方式：lipo -info framework或者.a实体文件路径**
- **使用示例：**



```undefined
 lipo -info /Users/wangyongwang/Documents/QiBitcode/QiBitcode.framework/QiBitcode
```

> 结果示例：Architectures in the fat file:/Users/wangyongwang/Documents/QiBitcode/QiBitcode.framework/QiBitcode are: **armv7 i386 x86_64 arm64**

#### 关于Architectures:

截止到2018年Apple新发布了iPhone XS, iPhone XS Max, iPhone XR后，iPhone及CPU对应情况：

| CPU    | iPhone                                                       |
| ------ | ------------------------------------------------------------ |
| armv6  | iPhone, iPhone 3G                                            |
| armv7  | iPhone 3GS, iPhone4(GSM),iPhone 4(CDMA),iPhone 4S            |
| armv7s | iPhone 5, iPhone 5C                                          |
| arm64  | iPhone 5S, iPhone SE, iPhone 6, iPhone 6 Plus, iPhone 6s, iPhone 6s Plus, iPhone 7, iPhone 7 Plus, iPhone 8, iPhone 8 Plus, iPhone X |
| arm64e | iPhone XS, iPhone XS Max, iPhone XR                          |

对于iPhone而言：iPhone 5S之前使用的是32位微处理器，iPhone 5S及之后都是64位的微处理器

模拟器上使用的CPU情况由所用的电脑确定

| CPU    | iPhone        |
| ------ | ------------- |
| i386   | 32 位微处理器 |
| x86_64 | 64 位微处理器 |

> 推荐文章：
>  [iOS 重绘之drawRect](https://www.jianshu.com/p/f9a731af56cf)
>  [iOS 编写高质量Objective-C代码（八）](https://www.jianshu.com/p/9ef6467ed06f)
>  [iOS KVC与KVO简介](https://www.jianshu.com/p/a8fdebe91eb5)
>  [iOS 本地化（IB篇）](https://www.jianshu.com/p/67a2f54d7498)
>  [iOS 本地化（非IB篇）](https://www.jianshu.com/p/ce3f2bdcfa6a)
>  [奇舞周刊](https://links.jianshu.com/go?to=https%3A%2F%2Fweekly.75team.com)