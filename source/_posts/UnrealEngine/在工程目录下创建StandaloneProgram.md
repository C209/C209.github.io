---
title: 在工程目录下创建StandaloneProgram
categories: 
    - Unreal Engine
    - Program
---
# 一. 创建基础文件
## 1. 创建.Target.cs
在项目的Source目录下直接创建Program的Target.cs文件，建议直接复制某个现成的Program。

注意，要么直接在Source里面这一级放所有的Target.cs，要么都放进目录层级里面去，不能有的在里面有的在外面。UBT的搜索逻辑是如果搜索到了其中一个，则不会再往下进行搜索了。所以有的项目的做法是把它们整理进不同的文件目录，如Runtime和Program，这样也清晰一些。这里因为项目仅有这么一个特殊的Program，挪其他的代码位置的话，git提交记录会有一堆rename，不太方便，所以就和项目的Game.Target.cs这些直接放在一块了。

## 2. 启动模块
独立Program必然有至少一个模块，该模块提供启动的入口，负责各平台的main函数入口。建议是各平台的main函数调用自己实现的一个Main，真正的启动逻辑在这个里面，这样可以实现平台无关性。

我们可以将我们所需的模块直接放进项目Source中的某一级目录，不必和Target.cs在一个层级下。它们可以多个模块在同一个目录下，无需彼此相互独立。

## 3. 配置.Target.cs
### 指定生成的project的目录
```
SolutionDirectory = "MyPrograms";
```
将一会生成的project在sln中显示在MyPrograms下，否则统一在Programs中很难找到。

# 二. 对项目目录下的uproject执行generate sln
这将在项目工程的sln里的MyPrograms中新增一个project，后续的改动就可以直接在sln中修改了。

# 三. 修改编译相关属性
建议将Target.cs中的LinkType这样修改：```LinkType = TargetLinkType.Monolithic;```

如果是Modular的话，就会将各个依赖的模块编译为独立的dll进来，然后在运行起来的时候链接起来。虽然这和Editor的启动方式一致，似乎也并不会出现编译问题，但是在项目目录的Binaries下出现一堆dll其实也挺麻烦的。下次想找项目build出来的exe和dll的时候也不太方便，干脆编进一个整体比较简单。

# 四. 修改运行时的项目路径
在你的Main函数入口处，覆盖ProjectDir:
```
FGenericPlatformMisc::SetOverrideProjectDir(FString::Printf(TEXT("../../Programs/%s/"), FApp::GetProjectName()));
```
因为默认的路径是Engine/Programs/，这样找起来不太方便。调整之后就会出现在项目目录下的Programs目录里了。

# 五. 配置模块
### 1. 指定Program需要的模块
在刚刚创建的Target.cs中，配置LaunchModuleName为提供入口的那个模块，并将其他依赖的模块添加至ExtraModuleNames。这些额外依赖的模块后续仍然需要手动加载，暂时不表。

### 2. 手动加载其他依赖的模块
在启动的时候，适当的地方主动调用FModuleManager::Get().LoadModuleChecked()进行加载。

# 六. 编译
设置刚刚生成出来的project为start up project，编译并运行它即可。

# 参考资料
- [Standalone Application](https://zhuanlan.zhihu.com/p/147598518)
- [Create A Standalone Application](https://imzlp.com/posts/31962/)
- [Program 类型工程的限制和解决方法](https://zhuanlan.zhihu.com/p/145633340)