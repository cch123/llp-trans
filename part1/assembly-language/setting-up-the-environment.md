2.1 环境配置

不写程序也就没有办法学会编程。所以我们现在就开始我们的汇编之旅。

本书中的例子使用的是下面的环境：

* 使用 Debian GNU/Linux 8.0 作为操作系统
* MASM 2.11.05 作为汇编语言的编译器
* GCC 4.9.2 作为 C 语言的编译器。指定这个版本用来生成 C 程序的汇编代码。也可以用 Clang
* GNU Make 4.0 作为构建系统
* GDB 7.7.1 作为调试器
* 任意文本编辑器\(最好能有语法高亮的那种\)。我们推荐使用 ViM

如果你希望搭建自己的系统，可以安装任意的 Linux 发行版并确保上面列表的程序都安装过一遍。以我们的经验而言，Windows 提供的 Subsystem for Linux 也可以适配这些工作。你可以安装上它，再用 apt-get 装好需要的工具。具体参考官方指引：[https://msdn.microsoft.com/en-us/commandline/wsl/install\_guide。](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide。)

在本书的 Apress 网站上 [http://www.apress.com/us/book/9781484224021](http://www.apress.com/us/book/9781484224021) 可以找到以下资源：

* 两个已经配置好了所有工具链的虚拟机。其中一个提供了桌面环境；另一个提供了最小的系统环境，可以用 SSH\(Secure Shell\) 直连进系统。安装指引和其它信息在压缩包内的 README 中都包含了。

* 一个 GitHub 的仓库链接，仓库里包含了书中的列表、问题的答案和一些解决方案。



