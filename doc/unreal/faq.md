# FAQ

## “new (std::nothrow) int[0] return nullptr, try fix it!”告警

Unreal重载了new，而且处理不符合C++规范：用no-throw方式new一个长度为0的数组，返回了nullptr，标准应该是返回有效值，只有OOM时才返回nullptr，这让遵从规范的V8误以为是OOM了，进而abort。目前只发现Window有该问题，而且该问题也经epic官方确认了。
Puerts如果发现引擎有该bug，会通过覆盖内存分配行为修复该问题，并打印“new (std::nothrow) int[0] return nullptr, try fix it!”告警。该告警只是提示下当前UE版本存在该Bug，没什么影响。

## 自动绑定模式有些扩展函数用不了

这是由于puerts模块启动得比较早，它启动的时候遍历Class遍历不了比它后启动的模块里的扩展函数。
解决办法：你所有模块启动完，调用下如下api就可以让它重新查找扩展函数。

~~~c++
IPuertsModule::Get().InitExtensionMethodsMap();
~~~

## 勾选Wait Debugger选项启动卡住

这个选项就是卡住进程来等待调试器连接，连上了就往下走了。

如果你还没配置好调试器，又不小心选了这个选项，也就没办法进入把这个选项去掉，此时可以关闭进程，打开Config\DefaultPuerts.ini把WaitDebugger改为False。


## ts生成蓝图的StaticClass调用，返回UClass使用不符合预期

ts类是没有StaticClass方法的，所以StaticClass调用其实是继承链上第一个有StaticClass方法的类，返回的也是该StaticClass方法所在类的UClass。

没理解这点可能会导致一些误解：比如我创建的对象为啥没子类方法，必然CreateDefaultSubobject报错说类是abstract的，无法创建等。

正确的做法应该是通过UE.Class.Load("path/to/your/blueprint/file")去加载。

## mac下提示“无法打开libv8.dylib，因为无法验证开发者”

进入该dylib文件所在目录（通常是：youproject/Plugins/Puerts/ThirdParty/v8/Lib/macOSdylib），执行如下命令：

~~~bash
sudo xattr -r -d com.apple.quarantine *.dylib
~~~

## 纯蓝图工程下加入puerts插件，提示"XXXProject counld not compiled. Try rebuilding from source manually"

对于纯蓝图工程，双击uproject文件，UE可能不会自动编译第三方的C++ Plugins。需要手动生成vs工程（mac下xcode工程），然后在ide编译。

## 打包后运行时报一些字段找不到

这大多数是由UE对FName编辑器和运行时处理不一致导致的，默认编辑器下大小写敏感，运行时大小写不敏感。

打比方，你在蓝图定义了一个count字段，在编辑器下生成代码，这个字段生成为count，运行也正常。

而打包后，如果在你访问这个蓝图前，已经有另外一个地方初始化了一个“Count”字段，那么你访问这个蓝图时，该字段会是"Count"，因为FName.ToString返回的是第一次构造该FName输入的字符串，后面只要转成小写后和第一次一样的FName，都是重用第一次的。

所以你在脚本访问的count字段会不存在（变成了Count字段）。

## UE5 EA生成按钮不显示

因为UE5 EA的toolbar extension失效了，要等epic自己修复或提供新API 。临时可以用控制台命令生成：Puerts.Gen。

## UE5下报 Construct TypeScript Object TestActor_C_1(000001E5057BD300) on illegal thread!

关闭AsyncLoadingThreadEnabled选项（该选项ue4默认关闭, ue5默认打开了）


