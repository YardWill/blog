> 说到c++编译运行，大家第一个想到的应该是VS2015这种微软出的大型IDE，对于一些大型项目也确实应该使用VS这种大型的IDE，但是作为一个业余的爱好者，只是想使用c++来运行一些东西，比如一些算法问题，那么VS这种大型的IDE就显得鸡肋，还会消耗不必要的内存，这个时候VSCode这种可安装插件的编辑器就显得非常高效。

主要步骤
===
- 安装VSCode
- 在VSCode内安装c++插件
- 安装g++编译、调试环境
- 修改VSCode调试配置文件

安装VSCode
===
[VSCode下载地址](https://code.visualstudio.com/?utm_expid=101350005-25.TcgI322oRoCwQD7KJ5t8zQ.0)
按照步骤安装

在VSCode内安装c++插件
===
Ctrl+P之后输入
```
ext install c++
```

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-771cca4e1307dcd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装左边第一个c/c++的插件（微软的官方插件）。
安装完成之后重启VSCode生效。

安装g++编译、调试环境
===
目前windows下调试仅支持 Cygwin 和 MinGW。这里使用的是MinGW. 
**[Download mingw-get-setup.exe (86.5 kB)](https://sourceforge.net/projects/mingw/files/latest/download?source=files)**

按照流程安装，安装完之后打开界面：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-3b225e8185a546ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
安装此处选中的模块。全选中之后按左上角Installationt->Apply Changes进行安装（最好翻墙）。
***然后配置环境变量***
别忘了这步就好（不懂配置的可以自己搜索，配环境变量应该是对程序员而言最见到那的事了）
有时候修改环境变量需要重启计算机

修改VSCode调试配置文件
===
用VSCode打开一个文件夹（因为VSCode会生成一个配置文件，所以必须在一个文件夹内运行）
新建一个a.cpp
写一个最简单的程序

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-daca50e608f0ba0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击右边的蜘蛛，再点击左边调试栏上方的设置按钮，选择c++编译环境，将launch.json的文件内容替换成如下：

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "C++ Launch (GDB)",                 // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg",                           // 配置类型，这里只能为cppdbg
            "request": "launch",                        // 请求配置类型，可以为launch（启动）或attach（附加）
            "launchOptionType": "Local",                // 调试器启动类型，这里只能为Local
            "targetArchitecture": "x86",                // 生成目标架构，一般为x86或x64，可以为x86, arm, arm64, mips, x64, amd64, x86_64
            "program": "${file}.exe",                   // 将要进行调试的程序的路径
            "miDebuggerPath":"c:\\MinGW\\bin\\gdb.exe", // miDebugger的路径，注意这里要与MinGw的路径对应
            "args": ["blackkitty",  "1221", "# #"],     // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false,                       // 设为true时程序将暂停在程序入口处，一般设置为false
            "cwd": "${workspaceRoot}",                  // 调试程序时的工作目录，一般为${workspaceRoot}即代码所在目录
            "externalConsole": true,                    // 调试时是否显示控制台窗口，一般设置为true显示控制台
            "preLaunchTask": "g++"　　                  // 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc
        }
    ]
}
```
**注意miDebuggerPath要与MinGw的路径对应** 
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-16fcf06e168f065d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
替换后保存，然后切换至a.cpp，按F5进行调试，此时会弹出一个信息框要求你配置任务运行程序，点击它~
选择任务运行程序，点击Others，跳出tasks.json的配置文件。
替换成如下代码
```
{
    "version": "0.1.0",
    "command": "g++",
    "args": ["-g","${file}","-o","${file}.exe"],    // 编译命令参数
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": ["relative", "${workspaceRoot}"],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    }
}
```
保存一下，然后切换至a.cpp，再次按F5启动调试

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-ddfaedfc814c5548.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后就可以设置断点调试了