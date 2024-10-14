# 什么是是vcpkg
vcpkg是c++的一个跨平台的包管理器，可以很方便的管理第三方库。
团队协作：项目开发成员之间的库版本统一（使用 vcpkg.json配置文件 控制第三方库的版本统一）
库管理：方便的安装和卸载第三方库文件（使用vcpkg指令可以快速安装和删除）
管理同个库的不同版本（在项目中，由vcpkg.json决定具体使用的版本）
项目构建时，vcpkg 会自动处理依赖关系，确保所有需要的库都已正确下载和链接
# 第三方库结构
## 第三方库的一般内容
第三方库一般包含库的头文件(.h)，源文件(.cpp)，链接文件(动态dll 或 静态llb)
minizip/
├── include/
│   ├── minizip.h          # 头文件
│   ├── minizip_utils.h    # 其他头文件

├── lib/
│   ├── minizip.lib        # Windows 静态库
│   ├── minizip.a          # Linux 静态库
│   └── ...                # 其他库文件（如动态库 .dll 或 .so 文件）
├── src/                   # 源代码（可选）
│   ├── minizip.cpp        # 源文件
│   └── ...                # 其他源文件
├── examples/              # 示例代码（可选）
│   ├── example.cpp        # 示例源文件
│   └── ...
└── README.md              # 使用说明文件

## 在vcpkg中的路径
注意 lib目录默认是release版
debug的 lib dll需要去debug目录
├── <vcpkg_DIR>/        # vcpkg 安装目录 (你可以根据需要命名，比如 vcpkg)
│   ├── include/       # 包含库的头文件
│   │   └── MyLib/     # MyLib 库的头文件目录
│   │       └── mylib.h
│   │
│   ├── lib/           # 库文件目录
│   │   ├── libmylib.a      # 静态库文件
│   │   ├── libmylib.so     # 动态库文件 (Linux)
│   │   └── mylib.dll       # 动态库文件 (Windows)
│   │
│   ├── share/         # 额外的 CMake 配置文件等
│   │   └── cmake/
│   │       └── MyLib/
│   │           └── MyLibConfig.cmake   # MyLib 库的 CMake 配置文件
│   │
│   └── bin/           # 可执行文件 (如果库提供了工具)
│       └── mylib_tool  # 假设库提供的命令行工具
# 安装vcpkg
## 安装位置
vcpkg 安装位置随意，跨平台时也是
库文件安装在 vcpkg的安装文件夹内，使用相对路径（vcpkg执行文件）进行链接
## windows下安装
安装需要使用git
请自行解决
### 创建一个文件夹用于安装vcpkg
### 打开cmd，进入此文件夹
在文件管理中打开新建的文件夹，地址栏输入cmd即可
### clone git上的vcpkg源码
git clone https://github.com/Microsoft/vcpkg
下载完成后，会自动建立一个vcpkg文件夹
### 进入vcpkg的源码文件夹，进行安装
执行下面的指令即可完成安装
bootstrap-vcpkg.bat
### 链接系统内所有的c++编译器
vcpkg integrate install
指令执行结果：![image](https://github.com/user-attachments/assets/eadef2a6-9a86-4d10-a48a-ee7118b3b347)

### 安装完成后
需要将 vcpkg 加入系统环境变量中，方便在终端中使用 vcpkg 指令

### 卸载 vcpkg
直接删除安装目录即可

## ubuntu下安装


# 使用vcpkg
终端打开项目文件夹

初始化
项目已有vcpkg.json文件时，跳过这一步，直接安装库文件
vcpkg new --application # 初始化
这会生成一个 vcpkg.json 和 vcpkg-configuration.json文件，包含库文件的清单
vcpkg.json是vcpkg 的配置文件，包含项目中使用的第三方库的列表，库名，版本等
vcpkg-configuration.json 引入了基线约束，指定项目应使用的依赖项的最低版本
团队开发时推荐将 两个文件 上传到 git进行同步，以同步团队中第三方库的版本
安装第三方库
在项目中已有 vcpkg.json时会优先安装 vcpkg.json内指定的库版本
当设备中已有相同库的不同版本时，会安装并使用 vcpkg.json内指定的库版本
使用一次 install指令，会自动安装 vcpkg.json中所有缺失的库
vcpkg search 库名 # 查看商店中是否存在指定库

vcpkg install 库名 # 安装第三方库文件 
vcpkg install 库名:x64-windows    # 安装指定版本
这个库文件实际会安装在 vcpkg 的安装目录下
在使用（编译）时，通过vcpkg寻找库文件进行链接和编译
在 vcpkg.json中，库文件的地址是相对于 vcpkg执行文件的相对路径
卸载第三方库
vcpkg remove 库名 # 卸载第三方库
在 cmake文件中链接 vcpkg和第三方库
链接 vcpkg
在cmake文件中加入下面这句话即可
-DCMAKE_TOOLCHAIN_FILE=<vcpkg_dir>/scripts/buildsystems/vcpkg.cmake"

# 设置vcpkg路径
set(vcpkg_DIR "${CMAKE_SOURCE_DIR}/path_to_vcpkg")  # 替换为实际vcpkg路径
list(APPEND CMAKE_PREFIX_PATH "${vcpkg_DIR}")

# 找到库
find_package(<PackageName> REQUIRED)  # 替换为实际的库名称

# 添加包含目录
include_directories(${<PackageName}_INCLUDE_DIRS})

# 链接库
target_link_libraries(your_target_name ${<PackageName}_LIBRARIES})  # 替换为你的目标名称
链接第三方库
使用cmake管理c++项目时，除了需要在cmake文件中添加 vakg的路径，还需要添加所有第三方库的路径
引入库文件地址时，建议使用相对路径（相对于vcpkg的可执行文件），在团队开发时，只需要将第三方库安装在vcpkg中即可
# 设置vcpkg路径
set(vcpkg_DIR "${CMAKE_SOURCE_DIR}/path_to_vcpkg")  # 替换为vcpkg的实际路径

# 添加库文件包含目录
include_directories("${vcpkg_DIR}/include") # 引入第三方库的头文件的包含目录

# 链接库文件
target_link_libraries(your_target_name "${vcpkg_DIR}/lib/libname.lib")  # 引入第三方库的链接文件
vcpkg常用指令
集成到全局：vcpkg integrate install
移除全局：vcpkg integrate remove

集成到工程：vcpkg integrate project（在“\scripts\buildsystems”目录下，生成nuget配置文件）

查看商店中是否存在指定库：vcpkg search 库名
查看支持的架构：vcpkg help triplet

指定编译某种架构的程序库：vcpkg install xxxx:x64-windows（x86-windows）

安装库：vcpkg install xxxx
卸载已安装库：vcpkg remove xxxx

指定卸载平台：vcpkg remove xxxx:x64-windows
移除所有旧版本库：vcpkg remove --outdated

查看已经安装的库：vcpkg list
更新已经安装的库：vcpkg update xxx
导出已经安装的库：vcpkg export xxxx --7zip（–7zip –raw –nuget –ifw –zip）
