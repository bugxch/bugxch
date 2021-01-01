# Source Insight 基础配置


我们提交的代码，要求不能有多余的空格与 TAB 键，而且代码是在 linux 环境中编译和运行的，而我们经常使用 source insight 编辑代码，因此有必要针对性地配置以省去反复去掉空格和 TAB 键的麻烦。

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201812/sourceinsight.png?raw=true)


我们的配置基本上都在 Preference 的菜单里，下面逐个介绍如何设置。

### 处理 TAB 和空格

- **去掉每行末尾的空格和 TAB 键**：设置`Options->Perferences->Files-> Remove extra white space when saving`。
- **TAB 键可见**：设置`Options->Document Options->Editing Options->Visible tabs`，就可以在代码里面看到 TAB 键。

### 处理换行键

*nix 系统和 Windows 系统文件中的换行符不同。

1. *nix 系统中的换行符只有一个字符，`\n`；
2. Windows 系统中的换行由两个字符组成，`\r\n`；

这种区别的影响包括

1. Unix/Mac 系统下的文件在 Windows 里打开的话，所有文字会变成一行；
2. Windows 里的文件在 Unix/Mac 下打开的话，在每行的结尾可能会多出一个 ^M 符号。
3. Linux 保存的文件在 windows 上用记事本看的话会出现黑点。

在 linux 下，可以使用命令`unix2dos`把 linux 文件格式转换成 windows 文件格式，命令`dos2unix`把 windows 格式转换成 linux 文件格式。

为了保证在 Windows 环境下打开文件之后仍然保持 linxu 的文件的显示正确，设置`Preference->Other->Default file format`为 Unix(LF)。

### 标题栏显示文件完整路径

这个功能在查看文档路径是非常有用，去掉`Preference->Display->Options->Trim long path names while elipses`。

### 文件名首字母不要大写

勾选`Preference->Display->Options->Show exact case of file names`。

### 其他设置

将 Preference 下面的所有 tab 页都过一遍。

#### General

- `Project File Synchronization->Remove missing file from project`选上可以避免因文件找不到而弹出错误对话框；
- 把`Misc->Use stricter confirmation dialog`选项去掉可以使确认时不输入”yes”。

#### Typing

- `ource Editing->Indent commands affect #-preprocessor statements`去掉后（默认值），进行多行缩进时不会影响预处理语句（如 #if…#endif）。

- `Auto Completion->Use detailed completion window`，选上后，联想时可以出现该函数的详细信息 。

- `Auto Completion->Insert paremeters for functions`，去掉后，自动联想不会把整个参数都输出到当前行。

- `Browsing in Lists->Match syllables while typing(slower)`在 symbol list 框检索符号时是否采用音节匹配方式，如对于函数 FindNext，输入 find 或者 next 都可以找到该函数。该功能可能导致反应缓慢（视工程和机器配置而定），建议关闭，因为即使在关闭状态下也可以通过先输入空格再输入单词来动态启用该功能。

  注意 Browsing in Lists 里其实有两个功能，但一般我们只能看到 Match syllables while typing 这一条，应该是 si 的菜单设计没有做好，导致在中文 windows 下不能显示全，另一个功能是 Match members while typing，用于打开 / 关闭按成员变量名来检索类 / 结构体的功能。

#### Files

- `Opening Files->Sharing: Let other programs modify files`，以共享方式打开文件，这个很重要，保证可以在其它编辑中同时编辑该文件。典型的场景就是用 ide 环境去动态编译调试，而用 si 静态阅读；
- `Customize 'Open' Command...`，用于设置 Ctrl+O 打开的页面，默认选项是 Project File list view in Project Window，建议保持默认。
- `Saving Files->Preserve Undo data and revision marks after saving`，如果发现保存后就不能 undo 了，请检查该选项是否选中。
- `Remove extra white space when saving`。保存时自动去除每行尾部的空格和 tab。建议选中。

####  Languages

自定义其它编程语言的语法解析，这个… 还是另写一篇来讲吧。

- `Conditional Parsing`不要错过了，这里的 Conditions 功能实在让人喜欢。Conditions 是什么意思呢？我们的代码中一般都会有一些开关宏，通过在 Conditions 中配置这些宏的默认值，可以让 si 把配置为不开启的宏视为无效代码，从而不进行符号检索。

  如果源代码中的开关宏太多，还可以使用 Condition Parsing 中的 Scan Files 来自动找出所有开关宏。

#### Symbol Lookups

没有特别的。

#### Display

- 显示配置和个人喜好和显示器的状态有关，偶用的 x60 小本，屏幕资源有限，所以在 Display Elements 里把 Project Window, Status Bar, Tool Bar，Clip Window 都关了，基本用快捷键可以代替它们。
- `Options->Horizontal scroll bars for each new window`。很多大师都教导我们说一行不要写太多代码。在这个指导思想下，我们不需要这个东东。
- `Show exact case of file names`。如果看不惯 si 把所有的文件名首字母都大写就勾上这个选项吧。
- `Tile source and destination windows for Source Link commands`。Source Link 很多时候用于外部命令输出结果的解析（如 Make, lint），这个功能会把解析结果与目标窗口自动 tile，很实用。
- `Trim long path names with ellipses`。这个建议不要选中。事实上这个主要影响标题栏，但一般来说标题栏上的空间是充裕的，选上之后往往会令我们不知道所编辑文件的具体位置。

#### Color

自己配置。

#### **Syntax Decorations**

- 可以把一些符号转换成特殊形式显示，如 -> 转换成→。如果要使用该功能，不能开启`Syntax Formatting->Basics->Use only color formatting`。
- `Auto annotations`下的三个功能都比较有用;
- `Show arrows at goto statements`可以在 goto 时显示一个向上或向下的箭头，表示是向上 goto 还是向下 goto，不过我们还是尽量不要用 goto 了。
- `Annotate closing braces with end-statement`。在”}” 后显示标识，表示该”}” 与哪个 if/switch 配对，而下面的`Annotate closing braces only for long blocks`则是一个补充选项，表示只在较长的语句块时才显示标识。

#### Syntax Formatting

如果让大家说出喜欢 si 的几个理由，我想语法着色一定会是其中之一。

- `Basics->Use only color formatting`。只启用 style 中关于颜色的设置。其它如粗体、斜体、阴影等都不启用。
- `Apply Styles for Lanugage Elements`。把分类启用 style，都选上吧。
- `Symbol Reference Lookups->Qualify references to members`。检测成员的有效性，如果不是类 / 结构体中的一部分，则不启用 style。虽然可能导致性能降低，但还是建议打开。同样`Qualify references to functions`也是。
- 这里有个按钮可以进入 Doc Types 页面（Options 菜单也可以进入），里面有很多重要选项：
  - `Editing Options`中， `Expand tabs, Visible tabs`可以帮助我们发现并转换 tab。
  - `Show right margint和Margin width`可以提醒我们是否把一行写得太长。
  - `Symbol Window`选项建议关闭（因为有快捷键）。
  - `Auto Indent`对话框中， 如果没有特别喜好，建议把`Smart Indent Options`的两个勾都去掉，同时`Auto Indent Type`选`Smart`。

其他的没有什么特别的了，最后简单说下 si 的配置文件。可以通过`Options->Load Configuration/Save Configuration`来导入 / 导出配置，可以导出全部，也可以导出某几个部分（如 style）。si 的配置有两级，一是全局配置，一是项目配置。出入方便考虑，统一一个配置就好了，在创建项目时选择用全局配置（默认值）。

### 参考文档

- [Windows、Unix、Mac 不同操作系统的换行问题 - 剖析回车符 \ r 和换行符 \ n - CSDN 博客](https://blog.csdn.net/TskyFree/article/details/8121951)
- [source insight 保存时删除多余空格，去除多余空格 space tab 键 - CSDN 博客](https://blog.csdn.net/lanmanck/article/details/8638391)
