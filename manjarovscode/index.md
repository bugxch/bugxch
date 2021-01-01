# 使用linux vs code调试C++程序


最近练习leetcode编程，我希望在manjaro系统的vs code上可以单步调试C++代码，找了很多资料都不能正常运行，所以参考[官方资料](https://code.visualstudio.com/docs/cpp/cpp-debug)自己整好了，记录一下。

<!--more-->

## 预置条件

首先完成如下工作

1. 安装vs code;

2. 安装插件，如下图所示

   ![](https://code.visualstudio.com/assets/docs/cpp/cpp/cpp-extension.png)

3. 确认linux系统已经正确安装gcc，输入如下命令

   ```shell
   gcc -v # 查看gcc版本
   sudo pacman -S build-essential gdb # 安装必要工具軟件
   ```

## 创建Hello world工程

在本地创建目录，我在本地创建了leetcode的目录，所以有如下的步骤

```shell
mkdir project
cd project
mkdir 
cd helloworld
code . # 在当前目录下打开vs code，当前打开的文件夹就是这个“工作空间”
```

阅读完本博客之后，你会在当前的目录下面创建下面的3个文件

- `tasks.json` (compiler build settings)
- `launch.json` (debugger settings)

### 添加源文件

如图所示，添加新的源文件`helloworld.cpp`

![](https://code.visualstudio.com/assets/docs/cpp/msvc/new-file-button.png)

在该文件中粘贴如下的源代码

```c++
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{
    vector<string> msg {"Hello", "C++", "World", "from", "VS Code", "and the C++ extension!"};

    for (const string& word : msg)
    {
        cout << word << " ";
    }
    cout << endl;
}
```

然后保存该文件。

![](https://code.visualstudio.com/assets/docs/cpp/msvc/file-explorer.png)

### 构建helloworld.cpp

接下来，你将创建一个`task.json`文件告诉VS code如何构建(编译)当前的程序。这将触发g++编译器按照源代码创建一个可执行程序。在主菜单选择**终端->配置默认生成任务**，然后在下拉菜单选择g++ build active file，如下图所示

![](https://code.visualstudio.com/assets/docs/cpp/wsl/build-active-file.png)

你将在`.vscode`文件夹下面看到`tasks.json`文件，我们进一步编辑这个文件

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "shell",
			"label": "C/C++: g++ build active file",
			"command": "/usr/bin/g++",
			"args": [
				"-g",
				"${file}",
				"-o",
				"${fileDirname}/${fileBasenameNoExtension}"
			],
			"options": {
				"cwd": "${workspaceFolder}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			}
		}
	]
}
```

> 关于json文件中变量的具体含义可以进一步参考[Visual Studio Code Variables Reference](https://code.visualstudio.com/docs/editor/variables-reference)

文件中的指令指定了程序如何运行，当前文件中的`args`参数指定了传输给gcc的参数，这些参数必须按照编译器期望的顺序排列。

这个任务告诉g++将源文件`${file}`编译，在当前文件夹下面创建可执行文件`helloword`，注意可执行文件的名称和源文件相同，但是去掉了扩展后缀名。`label`字段表示你能看到的任务列表，你可以写成任何你想写的东西。`group`中的`"isDefault": true`表示你可以使用`Ctrl+Shift+B`运行该任务，这个仅仅是为了使用上的方便，你依然可以通过菜单中的选项运行该任务。

### 运行编译程序

回到原来的`helloworld.cpp`程序，按下`Ctrl+Shift+B`运行该任务，请留意编辑器下方的终端的打印，在任务运行结束之后一般会提示成功或者失败，如果运行顺利，你可以看到如下的提示

![](https://code.visualstudio.com/assets/docs/cpp/wsl/wsl-task-in-terminal.png)

如果留意可以看到当前的文件夹中已经生成了可执行程序`helloworld`文件，打开新的终端，即可运行该程序

```shell
./helloworld # 运行可执行程序
```

![](https://code.visualstudio.com/assets/docs/cpp/wsl/wsl-bash-terminal.png)

### 修改tasks.json

你可以修改这个文件中的参数，比如将`${workspaceFolder}/*.cpp`替换`${file}`，或者也可以将`${fileDirname}/${fileBasenameNoExtension}`替换成一个硬编码的程序名称`helloworld.out`。

### 调试源程序

接下来你将创建`launch.json`文件，当按下`F5`的时候VS Code调用GDB的调试器用于调试程序。找到菜单中的**运行 > 添加配置**，然后选择**C++ (GDB/LLDB)**，如下图所示

![](https://code.visualstudio.com/assets/docs/cpp/wsl/build-and-debug-active-file.png)

我们选择**g++ build and debug active file**，你可以看到此处VS Code自动创建了文件`launch.json`文件，文件的内容如下

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++ build active file",
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

很明显在这个文件中`program`指定了需要debug的程序，和之前的文件一样，此处就是没有后缀名的与源文件一样的程序，这个例子中就是`helloworld`。默认情况下，C++插件不会在代码中插入任何断点，所以`stopAtEntry`是`false`。如果将该项改为`true`，那么就可以让调试器停在主函数的断点处。

### 开始调试

回到源文件，按下`F5`就可以开始调试了，在代码编辑器的上访可以看到调试的控制条，包括了单步调试，跳过调试，重启调试和停止调试的功能。稍微探索一下就可以发现，`F9`是添加断点，其他的调试方法鼠标悬停在上面都会显示快捷键，你看到的编辑器应该是这样的

![](https://code.visualstudio.com/assets/docs/cpp/wsl/debug-view-variables.png)

调试中可以看到每个变量的值，以及监视窗口。

> 需要注意的是，当前的版本（2019年3月份之后）不会在单步调试模式下将cout的结果打印出来，只有程序运行完成之后才会统一打印出来。

