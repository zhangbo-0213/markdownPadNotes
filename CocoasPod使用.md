## 前言
iOS开发会经常用到 cocoapods 管理第三方库，简单、方便、高效。如何集成 cocoapods 在 cocoapods 官网和 Podfile 语法说明会有详细介绍。

## 什么是Podfile
集成cocoapods时会用到的一个文件 Podfile,  CocoaPods是用ruby实现的，因此Podfile文件的语法就是ruby的语法。
podfile是一个说明文件，用以描述管理一个或者多个Xcode project的target的依赖库。这个文件应该且必须被命名为Podfile。

Podfile是一个规范，描述了一个或多个一套工程目标的依赖项

一个简单写法:
``` Ruby
target 'MyApp' do
  pod 'AFNetworking', '~> 3.0'
end
```
这是最简单最普遍的写法，针对MyApp这个target引入AFNetworking这个依赖库，也是大家平时用的最多的一种方式。   

复杂的例子：
``` Ruby
# 下面两行是指明依赖库的来源地址
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/Artsy/Specs.git'
 
# 说明平台是ios，版本是9.0
platform :ios, '9.0'
 
# 忽略引入库的所有警告
inhibit_all_warnings!
 
# 针对MyApp target引入AFNetworking
# 针对MyAppTests target引入OCMock，并继承引入AFNetworking
target 'MyApp' do 
    pod 'AFNetworking', '~> 3.0' 
    target 'MyAppTests' do
       inherit! :search_paths 
       pod 'OCMock', '~> 2.0.1' 
    end
end
# 这个是cocoapods的一些配置,官网并没有太详细的说明,一般采取默认就好了,也就是不写.
post_install do |installer|       
   installer.pods_project.targets.each do |target| 
     puts target.name 
   end
end
```   
## 主配置
install! 这个命令是 cocoapods 声明的一个安装命令，用于安装引入 Podfile 里面的依赖库。

install! 这个命令还有一些个人设置选项，例如： 
``` Ruby
install! 'cocoapods', 
  :deterministic_uuids => false, 
  # 禁用确定性 UUID（Universally Unique Identifier）生成，当将 "deterministic_uuids" 设置为 false 时，它指示系统禁用确定性UUID生成，即使相同的输入也会产生不同的UUID。这样可以增加随机性，并减少冲突的可能性，以确保生成的UUID更加唯一。
  :integrate_targets => false
  # 禁用目标集成。目标集成是指将不同的目标或任务合并到一个系统或框架中，以便它们可以共同工作和互相影响。在某些应用程序或系统中，可能需要将多个目标集成到一个整体系统中，以实现更复杂的功能或业务需求。
  # 通过将 "integrate_targets" 设置为 false，表示禁止目标集成，即不允许不同的目标或任务共同作用于系统或框架。这可能意味着每个目标或任务需要独立处理，没有相互之间的影响或交互。
```  
其他支持设置的选项：
``` Ruby
Supported Keys:
:clean
:deduplicate_targets
:deterministic_uuids
:integrate_targets
:lock_pod_sources
:share_schemes_for_development_pods
```   
## Dependencies（依赖项）
Podfile指定每个target的依赖项          
    
    pod 指定特定的依赖库
    podspec 可以提供一个API来创建podspecs
    target 通过target指定依赖范围 

### pod - 指定项目的依赖项
依赖项规范是由Pod的名称和一个可选版本组合在一起： 
- 如果后面不指定依赖项的版本，cocoapods 会默认选取最新的版本 ：  
``` Ruby 
pod 'SSZipArchive'
```  
- 如果想要指定依赖库的版本，需要加上具体版本号 ： 
``` Ruby
pod 'Objection', '0.9'
``` 
- 也可以指定版本范围 ： 
``` Ruby
'> 0.1' #高于0.1版本（不包含0.1版本）的任意一个版本

'>= 0.1' #高于0.1版本（包含0.1版本）的任意一个版本

'< 0.1' #低于0.1版本（不包含0.1版本）的任意一个

'<= 0.1' #低于0.1版本（包含0.1版本）的任意一个

'~> 0.1.2' #版本 0.1.2的版本到0.2 ，不包括0.2。这个基于你指定的版本号的最后一个部分。可以把最后一位看出x, 变成0.1.x, 这个例子等效于>= 0.1.2并且 <0.2.0，,并且始终是你指定范围内的最新版本, 比如会使用0.1.9。

'~> 0.1' #版本0.1和版本1.0之间的任意版本,不包括1.0和比1.0更高的版本,相当于0.x, 会在(0.1,1.0)之间取最新的版本号

'~> 0' #版本0或比版本0更高的版本,这基本上和不指定版本号的效果是一样的。
 ```  
 关于版本形式规范详情请参考链接：[语义化版本](https://semver.org/lang/zh-CN/)  
 ### Build configurations （编译配置）  
 默认情况下，依赖项会被安装在所有 target 的 build configurations 中。为了调试或者其他原因，依赖项只能在给定的 build configurations 中被使用：  
 ``` Ruby 
 pod 'PonyDebugger', :configurations => ['Debug', 'Beta']  
 # 只有在 Debug 和 Beta 模式下才启用依赖项 
 ``` 
 或者弄白名单只指定一个 build configurations : 
 ``` Ruby
 pod 'PonyDebugger', :configurations => 'Debug'   
 ``` 
 如果没有指定具体的配置，依赖项会包含在所有的配置中。   
 ### Subpecs  
 一般情况通过依赖库名称引入，cocoaspod 会默认安装依赖库所有内容。  
 也可以指定安装具体依赖库的某个子模块：
 ``` Ruby 
 pod 'QueryKit/Attribute' #仅安装 QueryKit库下的 Attribute 模块   
 pod 'QueryKit', :subspecs => ['Attribute', 'QuerySet'] #仅安装 QueryKit 库下的 Attribute 和 QuerySet 模块 
 ``` 
 ### Using the files from a local path(使用本地文件)  
 可以指定依赖库的来源地址，如果想引入本地一个库，可以这样写： 
 ``` Ruby 
 pod 'AFNetworking', :path => '~/Documents/AFNetworking'  
 ```  
 使用这个选项之后，Cocoapods 会将给定的文件夹作为 Pod 的源，并在工程中直接引用这些文件。这意味着编辑的部分可以保证留在 CocoaPods 安装中，如果更新本地 AFNetworking 中的代码，cocoaspod 也会自动更新。制作 pod 模版工程也是采用这种方式。 
 注意 ： Pod 的 podspec 文件也应该被放在这个文件夹中。   

 **有时需要使用依赖库指定的分支或节点**  
 ``` Ruby
# 引入 master 分支（默认） 
 pod 'AFNetworking', :git => 'https://github.com/repo/AFNetworking.git' 
# 引入指定分支 
 pod 'AFNetworking', :git => 'https://github.com/repo/AFNetworking.git', :branch => 'dev'  
# 引入某个节点代码  
 pod 'AFNetworking', :git => 'https://github.com/repo/AFNetworking.git', :tag => '0.7.0'  
# 引入某个特殊的提交节点 
 pod 'AFNetworking', :git => 'https://github.com/repo/AFNetworking.git', :commit => '134g8256cj'  
 ```  
 需要注意，虽然这样可以满足任何在 Pod 中的依赖项通过其他 Pods 但是 podspec 必须存在于仓库的根目录中。  
 ### 从外部引入 podspec 
 **podspec 可以从另一个源库的地址引入**  
 ``` Ruby
 pod 'JSONKIT', :podspec => 'https://example.com/JSONKit.podspec'  
 ```  
 cocoaspod 使用给定的 podspec 文件中定义的代码库的依赖关系。如果没有传入任何参数，podspec 优先使用根目录，如果是其他情况，必须在后面指定(一般使用默认设置)：
 ``` Ruby 
 # 不指定表示使用根目录下的 podspec ，默认一般都会放在根目录下   
 podspec  
 # 如果 podspec 的名字与库名不一样，可以指定podspec Name 
 podspec :name => 'HelpKit' 
 # 如果 podspec 不在根目录下，可以指定路径
 podspec :path => '/Documents/HelpKit/HelpKit.podspec'
 ```  

 ### target  
 在给定的块内定义 pod 的 target(Xcode 工程中的taregt) 和指定依赖范围。一个 target 应该与 Xcode 工程的 target 关联。默认情况下，target 会包含定义在 target 块外的依赖，除非块外指定不使用 inhert! 进行继承。 
 - 定义一个 target ZipApp 引入 SSZipArchive 库    
 ``` Ruby 
 target 'ZipApp' do
    pod 'SSZipArchive'
end
 ``` 
 - 定义一个 ZipAPP 的 target 仅引入 SSZipArchive 库，在定义 ZipAPPTests 的target 引入 Nimble 的同时也会继承 ZipApp target 里的 SSZipArchive 库  
 ``` Ruby  
 # ZipApp 中仅引入 SSZipArchive 库
 target 'ZipApp' do 
    pod 'SSZipArchive'
    # ZipAppTests 中引入 SSZipArchive + Nimble
    target 'ZipAppTests' do
        inherit! :search_paths
        pod 'Nimble' 
    end 
 end
 ```  
 - target 块中多嵌套  
 ``` Ruby
 # ShowApp 仅引入 ShowKit 库
 target 'ShowApp' do
    pod 'ShowKit'
    # ShowTV 引入 ShowKit 和 ShowTVKit 库
    target 'ShowTV' do 
        pod 'ShowTVKit'
    end
    # ShowWatch 引入 ShowKit 和 WatchKit TestKit 库
    target 'ShowWatch' do
        inherit! :search_paths
        pod 'WatchKit'
        pod 'TestKit'
    end
end
 ``` 
### 抽象 target 
定义一个新的抽象 target, 可以方便的用于目标依赖继承 
- 简单写法 
``` Ruby 
abstract_target 'NetWorking' do
    pod 'NetWorkingKit'
    # APP1 和 APP2 都会默认引入 NetWorkingKit 库
    target 'App1' 
    target 'App2'
end
``` 
- 定义一种 abstract_target 包含多个target 
``` Ruby
# 这是一个抽象的 target, 也就是工程中并没有这样一个 target 引入 ShowsKit 
abstract_target 'Shows' do
    pod 'ShowsKit'
    # target ShowiOS 引入 ShowsKit 和 ShowiOSKit
    target 'ShowiOS' do
        pod 'ShowiOSKit'
    end
    #tartget ShowTV 引入 ShowsKit 和 ShowTVKit 
    target 'ShowTV' do 
        pod 'ShowTVKit'
    end
    #target ShowTest 引入指明继承的 ShowsKit 和 TestKit, Setting 库
    target 'ShowTest' do
        inherit! :search_paths
        pod 'TestKit'
        pod 'SettingKit'
end
```  
Podfile 中自带一个隐藏的，默认的 abstract_target, 也可以用这种方式达到上述例子效果： 
``` Ruby
pod 'ShowKit'  
# ShowiOS 会引入 ShowKit 和 ShowiOSKit
target 'ShowiOS' do
    pod 'ShowiOSKit'
end
# ShowTV 会引入 ShowKit 和 ShowTVKit
target 'ShowTV' do
    pod 'ShowTVKit'
end
``` 
**abstract! 和 inherit!**  
abstract! 指示当前 target 是抽象的，不会与Xcode target 相关联  
inherit! 指示当前的 target 的继承模式： 
``` Ruby 
target 'App' do
    target 'AppTests' do
    inherit! :search_paths
    end
end
``` 
### Target configuration(目标配置) 
使用 target 配置来控制 cocoaspod 生成 project   
说明使用的平台，工程文件中项目链接配置  
**platform**  
platform 用于指定建立静态库的平台，CocoaPods 提供默认的平台版本配置： 
``` Ruby 
iOS -> 4.3 
OS X -> 10.6 
tvOS -> 9.0
watchOS -> 2.0
```  
如果部署目标需要 iOS < 4.3, armv6体系结构将被添加到 ARCHS, 如： 
``` Ruby 
# 指定具体平台和版本  
platform :ios, '4.0'  
platform :ios 
``` 
**project** 
如果没有显式的 project 被指定，那么会默认使用 target 的 父target 指定的 project 作为目标，如果没有任何一个target指定目标，那么就会使用和 Podfile 在同一目录下的 project 。同时也可以指定是否这些设置在 release 或 debug 模式下生效（必须指定一个名称和 :release/:debug 关联起来）  
确定 project ：
``` Ruby 
# MyGPSApp 这个 target 引入的库 只能在 Fast GPS 这个工程中使用  
target 'MyGPSApp' do
    project 'FastGPS'
    ...
end
```  
使用自定义编译配置  
``` Ruby 
# 名为 ‘TestProject’ 的项目配置，该项目有两个构建配置，分别名为 ‘Mac App Store’ 和 'Test'
project 'TestProject', 'Mac App Store' => :release, 'Test' => :debug   
# 将 ‘Mac App Store’ 构建配置设置为 Release 模式  
# 将 ‘Test’ 构建配置设置为 Debug 模式 

# 构建配置指在编译和构建应用程序时使用的设置和选项。通过为不同的构建配置指定不同的模式，可以在不同的场景下使用不同的设置。如，发布(Release)模式包括启用优化，禁用调试信息等，而调试模式可能允许更多的调试功能和详细信息。
``` 
**inhibit_all_warnings!**  
inhibit_all_warnings! 屏蔽所有来自 cocoapods 依赖库的警告，可以全局定义，也可以在子target里面定义，也可以指定某一个库：  
``` Ruby 
# 隐藏 SSZipArchive 库警告 
pod 'SSZipArchive', :inhibit_warnings => true 
# 不隐藏 ShowTVKit 库警告 
pod 'ShowTVKit', :inhibit_warnings => true 
```  
**use_frameworks!**  
通过指定 use_frameworks! 要求生成的是 framework 而不是静态库  
如果使用 use_frameworks! 命令会在Pods工程下的Frameworks 目录下生成依赖库的 framework 
如果不使用 use_frameworks! 命令会在 Pods工程下的Products目录下生成 .a 的静态库  

### Workapce 
默认情况下，我们不需要指定，直接使用与 Podfile 所在目录的工程名就可以，如果需要指定另外的名称，而不是使用工程的名称，可以这样指定： 
``` Ruby 
workspace 'MyWorkspace'  
``` 
### Source  
source 是指定 pod 的来源。如果不指定 source ，默认使用 CocoaPods 官方的 source（建议使用官方设置）  :  
``` Ruby  
CocoasPods Master Repository 
# 使用其他来源地址 
source 'https://github.com/artsy/Specs.git' 
# 使用官方默认地址  
source 'https://github.com/CocoaPods/Specs.git'
```   
### Hooks 
Podfile 提供 hook 机制，会在安装过程中调用。hook 是全局性的，不存在在每个 target 中。 

### Plugin 
指定应在安装期间使用的插件，使用此方法指定在安装期间使用的插件，以及当他被调用时，应传递给插件的选项，例如：
``` Ruby 
# 指定在安装期间使用 cocoapods-keys 和 slather 插件
plugin 'cocoapods-keys', :keyring => 'Eidolon'  
plugin 'slather'
```  

### pre_install 
当完成下载，在进入安装之前，可以使用 hook 机制通过 pre_install 指定需要的更改，更改完进入安装阶段。  
``` Ruby  
pre_install do |installer|
    # 做安装前的更改 
end 
```  
### post_install  
当安装完成，但是生成的工程还没有写入磁盘时，可以指定要执行的操作。
可以在写入磁盘前修改工程配置 : 
``` Ruby 
post_install do |installer| 
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config| 
            config.build_settings['GCC_ENABLE_OBJC_GC'] = 'supported'
        end
    end
end
```  
### def 
可以通过 def 命令来声明一个 pod 集: 
``` Ruby 
def 'CustomPods'
    pod 'ShowKit'
end 
``` 
在需要引入的 target 处引入: 
``` Ruby 
target 'MyTarget' do 
    CustomPods 
end 
``` 
这样写的好处在于：如果有多个 target, 而不同的target之间并不是全部包含，可以通过这种方式分开引入   

**具体实例**  
``` Ruby
# 定义函数 返回 地址源 的字符串
def private_source; return 'http://git.lotuscars.com.cn/LotusCustomeriOS/cocoapods.git'; end
def mirror_source; return 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'; end

# 声明 cocoaspod 的安装源
source mirror_source
source private_source

# Uncomment the next line to define a global platform for your project
platform :ios, '13.0'

# 生成 framework 而不是 静态库
use_frameworks!
# 启用 模块化 头文件
use_modular_headers!

# 定义一个标志位
use_local_3D_flag = true

# 定义在安装过程中的函数
install! 'cocoapods',
       :deterministic_uuids => false

def L3D_Pod(use_local_3D)
  # L3DKit内部会依赖指定版本的filament
  # 使用本地源码, 安装的子模块以及本地源码路径
if use_local_3D
  pod 'L3DKit', :subspecs => ['ios', 'implemention', 'bullet', 'json', 'cn_bundle'], :path => '../L3DKit'
else
  # 使用远程源码
  pod 'L3DKit', :git => 'https://git.lotuscars.com.cn/L3DGroup/L3DKit.git'
end

end

target 'L3dModelView' do
  # Comment the next line if you don't want to use dynamic frameworks

  # L3DKit
  L3D_Pod(use_local_3D_flag)
  #其他库
  pod 'libextobjc'
  pod 'YYModel'
end

#!!! 是否需要源码调试filament
need_source_debug_filament = true

post_install do |installer|
  # 添加宏FILAMENT_APP_USE_METAL
  installer.pods_project.targets.each do |target|
    if target.name == 'L3DKit'
        target.build_configurations.each do |config|
            config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= ['$(inherited)']
            config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] << 'L3D_TARGET_IOS=1'
            config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] << 'FILAMENT_APP_USE_METAL=1'
        end
      puts "L3DKit L3D_TARGET_IOS=1"
    end
  end
  
  for index in 0 ... installer.pod_targets.size
    if installer.pod_targets[index].name == 'Filament'
      filaPod = installer.pod_targets[index]
      puts filaPod.root_spec
        if need_source_debug_filament
          puts "build filament debug libs...\n"
          value = %x(python3 ./build_filament_lib.py #{filaPod.root_spec.version})
          puts "build_filament_lib result: #{value}"
#      filaPod.instance_variables.each { |ivar| puts "11 #{ivar}: #{filaPod.instance_variable_get(ivar)}" }
        end
        %x(python3 set_debug_flag.py -need_source_debug_filament #{need_source_debug_filament})
    end
  end
  # 为M1适配，排除模拟器的arm64
  installer.pods_project.build_configurations.each do |config|
    config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
  end
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      # 获取变量
      target_name = target.name
      build_settings = config.build_settings
      # Xcode 14 (bug ⚠️)
      config.build_settings["DEVELOPMENT_TEAM"] = "Z8XM7947MA"
    end
  end
  # 设置每个工程的最低支持系统为13，不然 XCode 14.3会报错
  installer.generated_projects.each do |project|
        project.targets.each do |target|
            target.build_configurations.each do |config|
                config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
             end
        end
 end
end
``` 

