---
title: App热修复详解
date: 2017-01-05
tags: [iOS]
description: App Hotfix(热修复)详解
---

# App Hotfix(热修复)详解

定义：

　　从广义的角度理解，大家都比较认同 Hotfix 是在移动端不需要重新发版，通过在线更新对版本 Bug 的修复。

## 现在比较流行的热修复技术分为三种：

## 一、使用JSPatch进行热修复：

>　　Objective-C 是动态语言，OC上所有方法的调用/类的生成都通过 Objective-C Runtime 在运行时进行，我们可以通过类名和方法名反射得到相应的类和方法，也可以替换某个类的方法为新的实现，还可以新注册一个类，为类添加方法替换方法，通过这些即可实现动态修复 APP 技术。

>　　[JSPatch][1]是一个在Github上的开源项目，[JSPatch下载地址][1]。[JSPatch][1] 的实现主要是通过 Objective-C的 runtime 原理,即利用JS传递字符串给OC，OC通过 Runtime 接口调用和替换OC方法。具体实现原理请参考作者的帖子：[JSPatch 实现原理详解 (整改版)][4]。

>　　[OC转JS工具][2],具体实现参考该[博客][3]

----

### 大体实现思路如下：

 1. [首先，开发者提供热修复脚本](#1)；
 2. [其次，要将脚本上传到后台，后台需要提供上传的操作页面](#2)；
 3. [然后，终端设备每次运行后请求获取最新的脚本文件](#3)；
 4. [最后，解析脚本文件，调用 JSPatch 引擎，执行脚本文件并修复](#4)；


#### <span id = "1">1、开发者提供热修复脚本:</span>

脚本的书写：

 　　脚本书写也很简单，先用Objective-C将要需要更改的代码改好，然后根据需要修改的代码更改成js代码即可，具体书写方法请参照 [JSPatch][1]使用说明，或者使用上面提供的[OC转JS工具][2]。

 例如：

　　OBjective-C代码，这里是需要修改的内容。

```Objective-C
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

      /** 该代码在上线后的项目里面并没有
        *  即在 APP 上线后又临时修改的。
        *  添加或者修改我们需要改动代码，如无需改动，该方法不变，此处拿修改标题做测试，可以做很堵其他操作；
        */
        self.title = @"Welcome to use JSPatchDemo";
      }
      /**
      * 省略其他代码
      */
@end
```

　　javascript代码：JS 属于链式语法，相信很多人都会，可以查看 JSPatch 的语法规则自己写，也可以通过[OC转JS工具][2]将上面的代码转换成JS 代码


```javascript
defineClass('ViewController', {
    viewDidLoad: function() {
        self.super().viewDidLoad();

        /** 该代码在上线后的项目里面并没有
         *  即在 APP 上线后又临时修改的。
         *  添加或者修改我们需要改动代码，如无需改动，该方法不变，此处拿修改标题做测试，可以做很堵其他操作；
         */
        self.setTitle("Welcome to use JSPatchDemo");
    },
});
```

　　至此，热修复脚本已经编写完成。

----

#### <span id = "2">2、要将脚本上传到后台，后台需要提供上传的操作页面:</span>

　　此处需要后台配置，将上面已经写好的脚本放到服务器上面，并开发接口由APP 请求接口获取到脚本文件

 ----

#### <span id = "3">3、终端设备每次运行后请求获取最新的脚本文件:</span>

　　终端获取请求后台开发的接口，获取到脚本内容，可以根据当前版本与脚本编号判断当前脚本是否是最新脚本，如果是最新脚本，则执行脚本内容。

 ----

#### <span id = "4">4、解析脚本文件，调用 JSPatch 引擎，执行脚本文件并修复:</span>

 首先将JSPatch添加到项目中
 ```
    pod 'JSPatch'
 ```

 然后导入头文件`#import "JSPatch.h"`, 在程序启动的时候注册 JSPatch

```OBjective-C
@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //此处作为演示，加载本地的 js 文件
    //热修复功能一般是从服务器段请求下来，然后再经过版本判断等操作，再加载等。
    NSString * jsPath = [[NSBundle mainBundle] pathForResource:@"JSFileName" ofType:@"js"];
    //在 js 文件存在的情况下，调用 JSPatch，如果js 文件不存在，程序会崩溃。
    if ([[NSFileManager defaultManager] fileExistsAtPath:jsPath] ) {
    NSString *scriptPatch = [NSString stringWithContentsOfFile:jsPath encoding:NSUTF8StringEncoding error:nil];
        [JPEngine startEngine];
        [JPEngine evaluateScript:scriptPatch];
    }
}
```

　　至此，利用JSPatch引擎进行 APP 热修复功能差不多就如此了，剩下的就是调试优化上诉各个环节。


## 二、使用ReactNative实现热修复：

>　　[ReactNative][5]是facebook提供的一种开源框架，使用JS语法进行跨平台开发，深受开发者青睐。直接使用ReactNative实现热修复的好像不是很多，比较流行的是使用微软的CodePush。

>　　这个是[CodePush][6]的官方网站地址,这个是CodePush[GitHut][7]地址。具体实现参考该[博客][8]，讲的特别详细，我就不具体讲解了。


## 三、使用Lua+Wax打补丁的方案实现热修复：

>　　由于 lua热修复框架多年不更新，此处省略。


[1]:https://github.com/bang590/JSPatch
[2]:http://bang590.github.io/JSPatchConvertor
[3]:http://www.jianshu.com/p/8cec322531ae
[4]:https://segmentfault.com/a/1190000003870981
[5]:https://github.com/facebook/react-native
[6]:http://microsoft.github.io/code-push/docs/getting-started.html
[7]:https://github.com/Microsoft/react-native-code-push#getting-started
[8]:http://blog.csdn.net/u011151353/article/details/50688681
